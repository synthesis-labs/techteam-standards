# Presentation Examples — Fence+OCC Reference Implementation

Source material for the presentation. Each section has a before/after pair and notes on
what the refactor achieved. These were all real changes made to this codebase.

---

## 1. Making impossible states unrepresentable — `FencedCommand`

The most important tool a type system gives you is the ability to make illegal states
_unrepresentable at compile time_, so you never need to write the runtime code that
handles them.

### Before

```kotlin
data class FencedCommand<out C : Any>(
    val entity: C,
    val deps: Deps,
    val operationId: OperationId,
    val expectedVersion: UInt?,   // null = "this is a create", UInt = "this is a mutate"
)
```

**The problem:** `expectedVersion: UInt?` is a sentinel. The same class carries two
completely different intents depending on whether one field is null. The framework code
that consumes this has to branch on null and hope callers set it correctly:

```kotlin
// In CommandProcessor — the null check is load-bearing but invisible at the call site
if (fencedCmd.expectedVersion == null) {
    // create path
} else {
    // mutate path — but what does expectedVersion=0u mean? "not found" or a real version?
}
```

And when the entity was missing and a conflict was reported, the error became:

```kotlin
data class VersionConflict(val expected: UInt, val actual: UInt)
// actual=0u was the sentinel for "entity not found" — indistinguishable from version 0
```

### After

```kotlin
@Serializable
sealed class FencedCommand<out C : Any> {
    abstract val deps: Deps
    abstract val operationId: OperationId

    @Serializable @SerialName("create")
    data class Create<out C : Any>(
        val entity: C,
        override val deps: Deps,
        override val operationId: OperationId,
    ) : FencedCommand<C>()

    @Serializable @SerialName("mutate")
    data class Mutate<out C : Any>(
        val entity: C,
        override val deps: Deps,
        override val operationId: OperationId,
        val expectedVersion: UInt,   // non-nullable — Mutate always has a version
    ) : FencedCommand<C>()
}
```

And the error hierarchy becomes self-documenting:

```kotlin
sealed class AggregateError {
    sealed class VersionConflict : AggregateError() {
        data class AlreadyExists(val currentVersion: UInt) : VersionConflict()  // create on existing
        data class NotFound(val expected: UInt)            : VersionConflict()  // mutate on missing
        data class Mismatch(val expected: UInt, val actual: UInt) : VersionConflict()  // stale write
    }
    data class DomainError(val message: String) : AggregateError()
    data class DepCheckFailed(val required: Deps, val observed: Deps) : AggregateError()
}
```

**What this achieves:**
- You cannot construct a `Mutate` without a version — the compiler enforces it.
- You cannot construct a `Create` with a version — it doesn't have that field.
- The three conflict cases are distinct types, not a `(0u, actual)` sentinel.
- `when` expressions over `FencedCommand` are exhaustive with no `else` branch.
- The wire format gains a `"type": "create"/"mutate"` discriminator as a side effect.

---

## 2. Splitting the evolve contract — no more silent deletion bug

### Before

```kotlin
interface Aggregate<Cmd, Evt, Entity, DepView> {
    fun evolve(entity: Versioned<Entity>?, event: Evt): Versioned<Entity>?
}
```

The single `evolve` method was used for both the create path and all mutate paths.
`null` input meant "no entity yet (create)"; `null` output meant "entity was deleted".

This produced a silent runtime bug in the framework:

```kotlin
// In CommandProcessor — for a create event:
val next = aggregate.evolve(null, event)  // passes null in for both null=create AND null=after-delete
if (next == null) store.delete(key) else store.put(key, next)
// If evolve() returned null, the store silently discarded the entity.
// A badly implemented aggregate returning null from a create would delete instead of create.
```

And the implementation was forced to handle the impossible:

```kotlin
override fun evolve(entity: Versioned<ExistingOrder>?, event: OrderEvent): Versioned<ExistingOrder>? {
    return when (event) {
        is OrderEvent.OrderCreated  -> Versioned(1u, event.order)  // entity param is always null here
        is OrderEvent.OrderUpdated  -> entity?.let { Versioned(it.version + 1u, event.succ) }  // ?.let hides the bug
        is OrderEvent.OrderDeleted  -> null
        is OrderEvent.OrderConfirmed -> entity?.let { Versioned(it.version + 1u, it.entity.copy(status = CONFIRMED)) }
    }
}
```

The `?.let` on the mutate paths silently returns `null` if `entity` is unexpectedly missing —
which causes the framework to delete the entity. The type signature made the bug invisible.

### After

```kotlin
interface Aggregate<CreateCmd, MutateCmd, Evt, Entity, DepView> {
    fun evolveCreate(event: Evt): Versioned<Entity>          // always non-null: creates always produce an entity
    fun evolveMutate(entity: Versioned<Entity>, event: Evt): Versioned<Entity>?  // null = deletion
}
```

```kotlin
// In CommandProcessor — paths are now separate:
val next = when (fencedCmd) {
    is FencedCommand.Create -> aggregate.evolveCreate(event)   // return type is non-nullable
    is FencedCommand.Mutate -> aggregate.evolveMutate(currentEntity!!, event)
}
if (next == null) store.delete(key) else store.put(key, next)
```

And the implementation is now honest about what each path can receive:

```kotlin
override fun evolveCreate(event: OrderEvent): Versioned<ExistingOrder> = when (event) {
    is OrderEvent.OrderCreated -> Versioned(1u, event.order)
    else -> error("evolveCreate received non-create event: ${event::class.simpleName}")
}

override fun evolveMutate(entity: Versioned<ExistingOrder>, event: OrderEvent): Versioned<ExistingOrder>? = when (event) {
    is OrderEvent.OrderUpdated   -> Versioned(entity.version + 1u, event.succ)
    is OrderEvent.OrderDeleted   -> null
    is OrderEvent.OrderConfirmed -> Versioned(entity.version + 1u, entity.entity.copy(status = CONFIRMED))
    else -> error("evolveMutate received non-mutate event: ${event::class.simpleName}")
}
```

**What this achieves:**
- `evolveCreate` has a non-nullable return type — the compiler prevents returning null on create.
- `evolveMutate` receives a non-nullable `entity` — the `?.let` silent-delete bug is gone.
- Each `when` expression covers only the events that can actually reach it.
- The two concerns (bootstrapping an entity vs. applying a delta) are cleanly separated.

---

## 3. The type system as documentation — Aggregate interface with 5 type params

### Before

```kotlin
interface Aggregate<Cmd, Evt, Entity, DepView> {
    val entityId: (Cmd) -> String    // extract the entity key from any command
    val entityKey: (Evt) -> String   // extract the entity key from any event

    fun processCreate(cmd: Cmd, deps: DepView): Evt
    fun processMutate(cmd: Cmd, entity: Versioned<Entity>, deps: DepView): Evt
    fun evolve(entity: Versioned<Entity>?, event: Evt): Versioned<Entity>?
}
```

There are several problems here:
1. `entityId` and `entityKey` are lambdas on the interface but they don't help — if the producer
   keys the command incorrectly, having a way to re-extract a key from the command doesn't fix the bug.
   It's false safety that adds noise.
2. Both `processCreate` and `processMutate` accept the same `Cmd` type. The framework can't
   distinguish "this command is only valid for creates" from "this command is only valid for mutates"
   at the type level.
3. Every aggregate implementation had a dead `else -> error("unexpected command")` branch:

```kotlin
override fun processCreate(cmd: OrderCommand, deps: Unit): OrderEvent = when (cmd) {
    is NewOrder -> OrderEvent.OrderCreated(...)
    else -> error("processCreate received mutate command")  // unreachable, but required by the type
}
```

### After

```kotlin
interface Aggregate<CreateCmd : Any, MutateCmd : Any, Evt : Any, Entity : Any, DepView : Any> {
    context(raise: Raise<AggregateError>)
    fun processCreate(cmd: CreateCmd, deps: DepView): Evt

    context(raise: Raise<AggregateError>)
    fun processMutate(cmd: MutateCmd, entity: Versioned<Entity>, deps: DepView): Evt

    fun evolveCreate(event: Evt): Versioned<Entity>
    fun evolveMutate(entity: Versioned<Entity>, event: Evt): Versioned<Entity>?
}
```

The sealed command hierarchy mirrors this:

```kotlin
@Serializable sealed class OrderCommand {
    @Serializable sealed class Create : OrderCommand()
    @Serializable sealed class Mutate : OrderCommand()
}
data class NewOrder(...)    : OrderCommand.Create()
data class UpdateOrder(...) : OrderCommand.Mutate()
data class DeleteOrder(...) : OrderCommand.Mutate()
data class ConfirmOrder(...) : OrderCommand.Mutate()
```

And the aggregate implementation is now exhaustive without dead branches:

```kotlin
object OrderAggregate : Aggregate<OrderCommand.Create, OrderCommand.Mutate, OrderEvent, ExistingOrder, Unit> {

    override fun processCreate(cmd: OrderCommand.Create, deps: Unit): OrderEvent = when (cmd) {
        is NewOrder -> OrderEvent.OrderCreated(...)
        // exhaustive — no else needed, compiler guarantees it
    }

    override fun processMutate(cmd: OrderCommand.Mutate, entity: Versioned<ExistingOrder>, deps: Unit): OrderEvent = when (cmd) {
        is UpdateOrder  -> ...
        is DeleteOrder  -> ...
        is ConfirmOrder -> ...
        // exhaustive — no else needed
    }
}
```

**What this achieves:**
- `entityId`/`entityKey` removed — dead code that added a false sense of safety.
- The compiler now guarantees `processCreate` only receives create commands, and `processMutate` only receives mutate commands.
- Adding a new command (e.g. `ConfirmOrder`) requires only: add to the sealed hierarchy, add a branch to `processMutate`. The compiler will flag any aggregate that doesn't handle it. Zero chance of forgetting.

---

## 4. Removing a resource leak — `submitAndAwait`

### Before

Every HTTP route handler repeated the same pattern manually:

```kotlin
post("/orders") {
    val operationId = OperationId.generate()
    val deferred = replyBus.register(operationId)
    producer.send(ProducerRecord(topics.commands, orderId.value, payload))
    when (val reply = withTimeout(5_000L) { deferred.await() }) {
        is CommandReply.Success  -> call.respond(HttpStatusCode.Created, ...)
        is CommandReply.Conflict -> call.respond(HttpStatusCode.Conflict, ...)
        is CommandReply.Rejected -> call.respond(HttpStatusCode.UnprocessableEntity, ...)
    }
    // ← replyBus.unregister() was missing entirely
    // If withTimeout() threw, the entry lived in the ConcurrentHashMap forever
}
```

This had two problems:
1. `replyBus.unregister()` was never called. Every timed-out request leaked an entry in the `ConcurrentHashMap`.
2. All three reply cases + the timeout path were duplicated across every route in every service.

### After

```kotlin
suspend fun ApplicationCall.submitAndAwait(
    producer: KafkaProducer<String, ByteArray>,
    replyBus: ReplyBus,
    topic: String,
    key: String,
    payload: ByteArray,
    operationId: OperationId,
    onSuccess: suspend ApplicationCall.() -> Unit,
) {
    val deferred = replyBus.register(operationId)
    try {
        producer.send(ProducerRecord(topic, key, payload))
        when (val reply = withTimeout(COMMAND_TIMEOUT_MS) { deferred.await() }) {
            is CommandReply.Success  -> onSuccess()
            is CommandReply.Conflict -> respond(HttpStatusCode.Conflict, ConflictResponse(...))
            is CommandReply.Rejected -> respond(HttpStatusCode.UnprocessableEntity, RejectedResponse(...))
        }
    } finally {
        replyBus.unregister(operationId)   // always runs, even on timeout
    }
}
```

Each route becomes:

```kotlin
post("/orders") {
    // ...build fencedCmd...
    call.submitAndAwait(producer, replyBus, topics.commands, orderId.value, payload, operationId) {
        respond(HttpStatusCode.Created, CreateResponse(operationId.value, orderId.value))
    }
}
```

**What this achieves:**
- The leak is structurally impossible — `finally` always unregisters.
- The three reply-case branches exist in exactly one place.
- Routes express only their unique concerns (building the command, the success response body).
- New routes cannot forget to unregister or handle the conflict/rejected cases.

---

## 5. DRY at the service level — `runFenceService`

### Before

Both `order-service/Application.kt` and `invoice-service/Application.kt` were ~80 lines each
of nearly identical Kafka + Ktor bootstrap code: create producer, configure streams, start
reply consumer loop, start Ktor server. Any change (e.g. enabling EOS, adding idempotence,
changing timeout) had to be applied in two places.

### After

```kotlin
// fence-occ-service/FenceService.kt — written once
fun <Cmd : Any, Evt : Any, Entity : Any> runFenceService(
    applicationId: String,
    defaultPort: Int = 8080,
    topics: TopicNames,
    codec: ServiceCodec<Cmd, Evt, Entity>,
    buildTopology: (TopicNames) -> Topology,
    configure: Route.(KafkaProducer<String, ByteArray>, ReplyBus, KafkaStreams, TopicNames) -> Unit,
): Unit = runBlocking { /* ~50 lines of bootstrap */ }
```

```kotlin
// order-service/Application.kt — the entire file
fun main() = runFenceService(
    applicationId = "order-service",
    defaultPort = 8080,
    topics = TopicNames.forEntity("orders"),
    codec = OrderCodec,
    buildTopology = ::buildOrderTopology,
    configure = Route::orderRoutes,
)
```

```kotlin
// invoice-service/Application.kt — the entire file
fun main() = runFenceService(
    applicationId = "invoice-service",
    defaultPort = 8081,
    topics = TopicNames.forEntity("invoices"),
    codec = InvoiceCodec,
    buildTopology = { invoiceTopics -> buildInvoiceTopology(invoiceTopics, TopicNames.forEntity("orders").events) },
    configure = Route::invoiceRoutes,
)
```

**What this achieves:**
- EOS (`EXACTLY_ONCE_V2`) and producer idempotence are in one place. They can't be forgotten or
  misconfigured on a new service.
- `TopicNames.forEntity("orders")` generates all four topic names from one string — no magic string
  drift between Streams config and the HTTP routes.
- Adding a third service is a 10-line file.

---

## 6. Codec ownership — `OrderEventCodec` lives in `order-types`

### Before

`invoice-service/InvoiceSerdes.kt` contained:

```kotlin
// In the invoice-service module — decoding a type it doesn't own
private val fencedOrderEvtS = Fenced.serializer(serializer<OrderEvent>())

fun decodeFencedOrderEvent(bytes: ByteArray): Fenced<OrderEvent> =
    json.decodeFromString(fencedOrderEvtS, bytes.toString(Charsets.UTF_8))
```

The wire format for `OrderEvent` was defined by whoever consumed it, not whoever produced it.
Any consumer of `OrderEvent` from another service would have to write the same decoder independently.

### After

```kotlin
// order-types/OrderEventCodec.kt — owned by the module that defines OrderEvent
object OrderEventCodec {
    val json = Json { ignoreUnknownKeys = true; encodeDefaults = true }
    private val fencedOrderEvtS = Fenced.serializer(serializer<OrderEvent>())

    fun decodeFencedEvent(b: ByteArray): Fenced<OrderEvent> =
        json.decodeFromString(fencedOrderEvtS, b.toString(Charsets.UTF_8))
}
```

**What this achieves:**
- The codec travels with the type. Any service that takes an `order-types` dependency gets the
  decoder for free — no re-implementation.
- If the `OrderEvent` schema changes, there is one place to update the codec.
- The `ignoreUnknownKeys = true` policy (forward compatibility for consumers) is set once by
  the type owner, not independently by each consumer.

---

## 7. Error handling at the boundary — `parseFenceHeader` returns `Result`

### Before

```kotlin
fun String.parseFenceHeader(): Deps {
    // threw IllegalArgumentException on malformed input
    // callers in HTTP handlers didn't catch it — became an unhandled 500
}
```

### After

```kotlin
fun String.parseFenceHeader(): Result<Deps> = runCatching {
    if (isBlank()) return@runCatching emptyDeps()
    trim().split(",").associate { coord ->
        val parts = coord.trim().split(":")
        require(parts.size == 3) { "Invalid fence coordinate '$coord': expected 'topic:partition:offset'" }
        StreamId(topic = parts[0], partition = parts[1].toInt()) to parts[2].toLong()
    }
}
```

At the HTTP boundary:

```kotlin
val deps = call.request.headers["If-Fence"]
    ?.let { it.parseFenceHeader().getOrElse { e ->
        return@post call.respond(HttpStatusCode.BadRequest, mapOf("error" to (e.message ?: "invalid If-Fence header")))
    }}
    ?: emptyDeps()
```

**What this achieves:**
- A malformed `If-Fence` header now returns 400 instead of 500.
- The return type forces callers to handle the error case — it cannot be silently swallowed.
- The parse logic and the HTTP response concern stay separated.

---

## 8. What "good foundations" enables — adding a new command in one diff

When the `ConfirmOrder` workflow was added, the entire change was:

**`order-types/OrderCommand.kt`** — add one line:
```kotlin
data class ConfirmOrder(val orderId: OrderId) : OrderCommand.Mutate()
```

**`order-types/OrderEvent.kt`** — add one variant:
```kotlin
data class OrderConfirmed(val orderId: OrderId) : OrderEvent()
```

**`order-service/OrderAggregate.kt`** — add one branch (compiler required it):
```kotlin
is ConfirmOrder -> {
    raise.ensure(entity.entity.status == OrderStatus.PENDING) {
        AggregateError.DomainError("only pending orders can be confirmed")
    }
    OrderEvent.OrderConfirmed(orderId = entity.entity.id)
}
```

And in `evolveMutate`:
```kotlin
is OrderEvent.OrderConfirmed -> Versioned(entity.version + 1u, entity.entity.copy(status = OrderStatus.CONFIRMED))
```

**`order-service/OrderRoutes.kt`** — add one route:
```kotlin
post("/orders/{id}/confirm") {
    // ...
    call.submitAndAwait(...) { respond(MutationResponse(operationId.value)) }
}
```

No changes needed to: the framework, the Kafka topology, `CommandProcessor`, `processCommand`,
serialization config, the test harness, or any other service.

**The compiler caught every missing piece.** The sealed hierarchy made the `when` in
`processMutate` non-exhaustive the moment `ConfirmOrder` was added to the hierarchy —
the build failed and pointed to exactly the line that needed updating.

This is the payoff: the cost of adding a new command is proportional to the complexity
of that command, not to the size of the codebase.

---

## 9. Separating layers — `Versioned<T>` and `Fenced<T>`

One of the subtler but most consequential refactors: pulling protocol-level concerns
_out_ of domain types. It looks like a minor tidying exercise. The payoff is that the
domain layer becomes genuinely ignorant of the infrastructure around it.

### Before — version and causal metadata embedded in domain types

```kotlin
// The domain entity carried its own version
data class Order(
    val id: OrderId,
    val version: UInt,          // OCC counter — a protocol concern living in the domain
    val status: OrderStatus,
    val description: String,
    val deps: Map<StreamId, Long>,  // causal fence position — definitely not domain
)

// Events also carried their own version + deps
data class OrderCreated(
    val order: Order,           // already includes version and deps
    val producedAtOffset: Long, // more infrastructure noise
) : OrderEvent()
```

Because the domain types owned version, the aggregate had to manage it:

```kotlin
override fun processCreate(cmd: NewOrder): OrderEvent {
    return OrderCreated(
        order = Order(
            id = cmd.orderId,
            version = 1u,       // aggregate sets the initial version — wrong layer
            status = PENDING,
            description = cmd.description,
            deps = cmd.deps,    // aggregate threads deps through — wrong layer
        ),
        producedAtOffset = 0,
    )
}

override fun processMutate(cmd: UpdateOrder, entity: Order): OrderEvent {
    // Domain logic has to read and increment the version counter
    val nextVersion = entity.version + 1u
    val updated = cmd.patches.fold(entity) { acc, patch -> /* apply patch */ }
    return OrderUpdated(succ = updated.copy(version = nextVersion))
}
```

And the state store compared versions by reading `entity.version`:

```kotlin
// In CommandProcessor — domain object queried for protocol information
if (cmd.expectedVersion != entity.version) {
    // conflict
}
store.put(key, updatedEntity)  // updatedEntity.version already incremented by the aggregate
```

### After — protocol concerns lifted into wrapper types

```kotlin
// Domain entity: pure domain state only
data class Order<ID>(
    val id: ID,
    val status: OrderStatus,
    val description: String,
    // no version, no deps, no offsets
)

typealias ExistingOrder = Order<OrderId>
```

```kotlin
// Framework type: pairs any entity with its OCC counter
data class Versioned<T>(val version: UInt, val entity: T)

// Framework type: pairs any payload with its causal fence position
data class Fenced<out T>(val entity: T, val deps: Deps)
```

The aggregate now expresses only domain intent:

```kotlin
override fun processCreate(cmd: OrderCommand.Create, deps: Unit): OrderEvent = when (cmd) {
    is NewOrder -> OrderEvent.OrderCreated(
        order = Order(id = cmd.orderId, status = PENDING, description = cmd.description),
        // no version, no deps — not the aggregate's concern
    )
}

override fun processMutate(cmd: OrderCommand.Mutate, entity: Versioned<ExistingOrder>, deps: Unit): OrderEvent = when (cmd) {
    is UpdateOrder -> {
        val updated = cmd.patches.fold(entity.entity) { acc, patch -> /* apply patch */ }
        OrderEvent.OrderUpdated(succ = updated)
        // version not touched — not the aggregate's concern
    }
    is DeleteOrder  -> OrderEvent.OrderDeleted(orderId = entity.entity.id)
    is ConfirmOrder -> OrderEvent.OrderConfirmed(orderId = entity.entity.id)
}
```

Version management lives entirely in the framework:

```kotlin
// In AggregateProcessor — protocol checks happen before domain code is called
raise.ensure(currentEntity.version == fencedCmd.expectedVersion) {
    AggregateError.VersionConflict.Mismatch(fencedCmd.expectedVersion, currentEntity.version)
}

// In CommandProcessor — version increment happens after domain code returns
val next = when (fencedCmd) {
    is FencedCommand.Create -> aggregate.evolveCreate(event)         // returns Versioned(1u, entity)
    is FencedCommand.Mutate -> aggregate.evolveMutate(currentEntity!!, event)  // returns Versioned(n+1, entity)
}

// Causal deps stamped on the event envelope — aggregate never sees this
ctx.forward(Record(key, codec.encodeFencedEvent(Fenced(event, observed)), ...), EVENTS_SINK)
```

**What this achieves:**

- Domain types are _portable_. `Order` is just data — no Kafka, no OCC, no vector clocks.
  You can use it in unit tests without constructing any protocol machinery.
- Domain logic is _testable in isolation_. `processCreate` and `processMutate` receive
  pure domain types and return pure domain events. No protocol state to set up or verify.
- **The version can never drift.** When version was inside `Order`, an aggregate could
  increment it incorrectly, forget to increment it, or increment it twice. Now it's
  impossible — the aggregate doesn't have write access to the version counter.
- **The causal fence can never be wrong.** When deps were threaded through commands and
  events by hand, any path through the code could corrupt or drop them. Now `Fenced<T>`
  is stamped at the framework level, once, using the actual observed offsets at the
  moment of processing.
- **Adding a new field to `Order` doesn't touch the protocol.** Versioning, fencing, and
  event envelopes are completely orthogonal to what fields the domain entity carries.

This is the "leaky abstraction" lesson in concrete form: when two different layers share
the same type, changes to either layer force changes in the other. The wrapper type is the
seam — each layer only knows about its own concern.

---

## Summary table

| Refactor | Problem | Mechanism | Outcome |
|---|---|---|---|
| `FencedCommand` sealed class | Null sentinel for create/mutate | Sealed subclasses | Illegal states unrepresentable |
| `evolveCreate`/`evolveMutate` split | Silent null-delete bug | Non-nullable return on create path | Bug structurally impossible |
| Aggregate `CreateCmd`/`MutateCmd` split | Dead `else -> error()` branches | Sealed type params | Exhaustive `when` with no dead code |
| `submitAndAwait` extension | ReplyBus memory leak + 3× duplication | `try/finally` + extension function | Leak impossible, one place to update |
| `runFenceService` | Duplicated 80-line bootstrap per service | Generic bootstrapper | New service = 10-line `Application.kt` |
| `OrderEventCodec` in `order-types` | Decoder owned by the consumer | Codec lives with the type | Any consumer gets it free |
| `parseFenceHeader` returns `Result` | Malformed header → unhandled 500 | `Result<T>` return type | Forced error handling, 400 not 500 |
| `Versioned<T>` / `Fenced<T>` wrappers | Protocol concerns embedded in domain types | Generic wrapper types as layer seams | Domain logic portable, version drift impossible |
| `ConfirmOrder` addition | — | All of the above | 4-file change, compiler-guided |
