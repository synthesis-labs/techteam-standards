<template>
  <div class="caveat-card">
    <div class="caveat-label">{{ label || 'Caveats' }}</div>
    <div class="caveat-body"><slot /></div>
  </div>
</template>

<script setup lang="ts">
defineProps<{ label?: string }>()
</script>

<style scoped>
.caveat-card {
  background: var(--deck-warn-bg);
  border-left: 4px solid var(--deck-warn-border);
  border-radius: 4px;
  padding: 1rem 1.25rem;
  margin: 0.75rem 0;
}
.caveat-label {
  color: var(--deck-warn-border);
  font-weight: 600;
  margin-bottom: 0.4rem;
}
.caveat-body :deep(p) {
  margin: 0;
}
</style>
