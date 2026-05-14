<template>
  <div class="slidev-layout end">
    <div v-if="$frontmatter.eyebrow" class="end-eyebrow">{{ $frontmatter.eyebrow }}</div>
    <div class="end-rule" />
    <h1 class="end-title">{{ $frontmatter.title || 'Questions?' }}</h1>
    <div class="end-meta">
      <div v-if="$frontmatter.speaker" class="end-speaker">
        <strong>{{ $frontmatter.speaker }}</strong><span v-if="$frontmatter.organisation"> · {{ $frontmatter.organisation }}</span>
      </div>
      <div v-if="$frontmatter.email" class="end-email">
        <a :href="'mailto:' + $frontmatter.email">{{ $frontmatter.email }}</a>
      </div>
      <div v-if="$frontmatter.footer" class="end-footer">{{ $frontmatter.footer }}</div>
      <div v-if="$slots.default" class="end-extra"><slot /></div>
    </div>
  </div>
</template>

<style scoped>
.end {
  background: var(--deck-navy-deep);
  color: white;
  padding: 5rem 5rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 100%;
}
.end-eyebrow {
  font-size: 0.75rem;
  letter-spacing: 0.25em;
  color: var(--deck-blue-pale);
  font-weight: 600;
}
.end-rule {
  width: 80px;
  height: 3px;
  background: var(--deck-blue-accent);
  margin: 1.25rem 0 1.5rem;
}
.end-title {
  font-size: 4rem;
  color: white;
  font-weight: 700;
  margin: 0;
  letter-spacing: -0.02em;
}
.end-meta {
  margin-top: 2rem;
  font-size: 1.1rem;
  color: var(--deck-blue-pale);
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}
.end-meta a {
  color: var(--deck-blue-accent);
  text-decoration: none;
}
.end-meta a:hover {
  text-decoration: underline;
}
.end-footer {
  margin-top: 0.6rem;
  font-size: 0.9rem;
  opacity: 0.8;
}
.end-extra {
  margin-top: 0.6rem;
}
</style>
