<template>
  <div class="slidev-layout section">
    <div class="section-number">{{ $frontmatter.number || '01' }}</div>
    <div class="section-rule" />
    <h1 class="section-title">{{ $frontmatter.title }}</h1>
    <div v-if="$frontmatter.subtitle" class="section-subtitle">{{ $frontmatter.subtitle }}</div>
  </div>
</template>

<style scoped>
.section {
  background: var(--deck-navy);
  color: white;
  padding: 5rem 5rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 100%;
}
.section-number {
  font-size: 6rem;
  font-weight: 700;
  color: var(--deck-blue-accent);
  line-height: 1;
}
.section-rule {
  width: 64px;
  height: 3px;
  background: white;
  margin: 1.5rem 0;
}
.section-title {
  font-size: 3rem;
  color: white;
  font-weight: 600;
  margin: 0;
}
.section-subtitle {
  margin-top: 1rem;
  color: var(--deck-blue-pale);
  font-size: 1.25rem;
  max-width: 60ch;
}
</style>
