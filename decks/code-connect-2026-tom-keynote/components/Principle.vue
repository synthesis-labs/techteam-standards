<template>
  <div class="principle-card">
    <div v-if="title" class="principle-title">{{ title }}</div>
    <div class="principle-body"><slot /></div>
  </div>
</template>

<script setup lang="ts">
defineProps<{ title?: string }>()
</script>

<style scoped>
.principle-card {
  background: var(--deck-tip-bg);
  border-left: 4px solid var(--deck-tip-border);
  border-radius: 4px;
  padding: 1rem 1.25rem;
  margin: 0.75rem 0;
}
.principle-title {
  color: var(--deck-tip-title);
  font-weight: 600;
  margin-bottom: 0.4rem;
}
.principle-body {
  color: var(--deck-body-ink);
}
.principle-body :deep(p) {
  margin: 0;
}
</style>
