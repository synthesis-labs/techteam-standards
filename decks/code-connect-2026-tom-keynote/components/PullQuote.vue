<template>
  <div class="pull-quote">
    <slot />
  </div>
</template>

<style scoped>
.pull-quote {
  border-left: 4px solid var(--deck-blue-accent);
  padding: 0.5rem 1.5rem;
  font-size: 1.7rem;
  font-style: italic;
  color: var(--deck-navy);
  font-weight: 400;
  line-height: 1.4;
  max-width: 24ch;
}
.pull-quote :deep(em) {
  color: var(--deck-blue-accent);
  font-style: italic;
}
.pull-quote :deep(p) {
  margin: 0;
}
</style>
