<template>
  <div class="slidev-layout cover">
    <img v-if="$frontmatter.logo" :src="import.meta.env.BASE_URL + $frontmatter.logo.replace(/^\//, '')" class="cover-logo" :alt="$frontmatter.organisation || 'logo'" />
    <div v-if="$frontmatter.eyebrow" class="cover-eyebrow">{{ $frontmatter.eyebrow }}</div>
    <h1 class="cover-title" v-html="$frontmatter.title"></h1>
    <div class="cover-rule" />
    <div v-if="$frontmatter.subtitle" class="cover-subtitle">{{ $frontmatter.subtitle }}</div>
    <div class="cover-footer">
      <span>{{ $frontmatter.author }}</span>
      <span>{{ $frontmatter.date }}</span>
    </div>
    <div v-if="$slots.default" class="cover-extra">
      <slot />
    </div>
  </div>
</template>

<style scoped>
.cover {
  background: var(--deck-navy-deep);
  color: white;
  padding: 4rem 5rem 3rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 100%;
  position: relative;
}
.cover-logo {
  position: absolute;
  top: 2.5rem;
  right: 5rem;
  height: 36px;
  width: auto;
  object-fit: contain;
  opacity: 0.9;
}
.cover-eyebrow {
  font-size: 0.75rem;
  letter-spacing: 0.25em;
  color: var(--deck-blue-pale);
  font-weight: 600;
}
.cover-title {
  font-size: 4.25rem;
  font-weight: 700;
  color: white;
  margin: 1.25rem 0 0;
  line-height: 1.05;
  letter-spacing: -0.02em;
}
.cover-rule {
  width: 80px;
  height: 3px;
  background: var(--deck-blue-accent);
  margin: 1.5rem 0;
}
.cover-subtitle {
  font-size: 1.5rem;
  color: var(--deck-blue-pale);
  font-weight: 400;
}
.cover-footer {
  position: absolute;
  bottom: 3rem;
  left: 5rem;
  right: 5rem;
  display: flex;
  justify-content: space-between;
  font-size: 0.85rem;
  color: var(--deck-blue-pale);
  border-top: 1px solid #1E2A40;
  padding-top: 1rem;
}
.cover-extra {
  margin-top: 1.5rem;
  font-size: 0.95rem;
  color: var(--deck-blue-pale);
}
</style>
