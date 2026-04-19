---
name: prototype-ui
description: Rapidly prototype a UI feature with N radically different designs (default 3), toggleable via an in-page picker. Dev-only route, auth-bypassed. Favors @nuxt/ui v4 components. Use when user wants to prototype, mock up, explore UI layouts, or compare variants.
---

# Prototype UI

Scaffold one Nuxt page with N radically different variants and a picker to switch between them live. Dev-only, auth bypassed. Use @nuxt/ui v4 components first — no custom CSS unless unavoidable.

## Workflow

1. **Ask one line**: what are we prototyping, target route, how many variants (default 3), any existing data shape?
2. **Pick N radically different axes** (not cosmetic tweaks). Examples:
   - Card grid vs data table vs timeline
   - Sidebar-first vs top-tabs vs single-column wizard
   - Inline edit vs `UModal` vs `USlideover`
3. **Scaffold** under `gestime-web/app/pages/prototypes/<feature>/`:
   - `index.vue` — host page with picker + dev gate
   - `VariantA.vue`, `VariantB.vue`, … — self-contained, one file per variant
4. **Run** `pnpm dev`, share `/prototypes/<feature>`.

## Rules

- @nuxt/ui first: `UCard`, `UTable`, `UForm`, `UButton`, `UBadge`, `UModal`, `USlideover`, `UTabs`, `UDashboardPanel`, etc. Check `nuxt-ui` skill if unsure.
- Design tokens only — no hardcoded colors, no raw hex.
- Mock data inline (`const rows = [...]`). No API calls, no stores.
- Each variant self-contained — deleting one must not break the others.
- Variants must look obviously different at a glance.

## Picker template (`index.vue`)

```vue
<script setup lang="ts">
import VariantA from './VariantA.vue'
import VariantB from './VariantB.vue'
import VariantC from './VariantC.vue'

definePageMeta({
  layout: false,
  auth: false,
  middleware: [(to) => {
    if (!import.meta.dev) return abortNavigation({ statusCode: 404 })
  }]
})

const variants = [
  { label: 'A — <name>', value: 'a', component: VariantA },
  { label: 'B — <name>', value: 'b', component: VariantB },
  { label: 'C — <name>', value: 'c', component: VariantC }
]
const active = ref('a')
const current = computed(() => variants.find(v => v.value === active.value)!.component)
</script>

<template>
  <UContainer class="py-6 space-y-6">
    <UTabs v-model="active" :items="variants" color="primary" variant="link" />
    <component :is="current" />
  </UContainer>
</template>
```

Swap `UTabs` for `USelectMenu` if variants > 4. `definePageMeta` block is mandatory — it 404s the route in prod and skips auth in dev.
