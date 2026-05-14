<template>
  <div class="before-after">
    <div class="ba-col before">
      <div class="ba-label">Before</div>
      <div class="ba-content"><slot name="before" /></div>
    </div>
    <div class="ba-col after">
      <div class="ba-label after-label">After</div>
      <div class="ba-content"><slot name="after" /></div>
    </div>
  </div>
</template>

<style scoped>
.before-after {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin: 0.5rem 0;
}
.ba-col {
  min-width: 0;
}
.ba-label {
  font-size: 0.7rem;
  letter-spacing: 0.25em;
  font-weight: 700;
  color: var(--deck-muted);
  margin-bottom: 0.5rem;
}
.after-label {
  color: var(--deck-blue-accent);
}
.ba-content :deep(pre) {
  margin: 0 !important;
  font-size: 0.78rem;
}
</style>
