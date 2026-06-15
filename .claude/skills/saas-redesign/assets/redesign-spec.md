# SaaS Redesign — edit recipe

This is the exact transformation recipe for converting the inventory-management client from a
top-nav layout to a left-sidebar SaaS layout with a design-token system. The `saas-redesign` skill
hands this file to the `vue-expert` agent for all `.vue` edits. Preserve **all** behavior: routes,
filters, i18n, modals, the backend contract. **No emojis.** Consume tokens — never reintroduce
hardcoded hex/px for things a token covers.

---

## 1. `client/src/main.js` (non-.vue — done directly by the skill, not vue-expert)

Add a single import of the token stylesheet **before** the app is mounted / before other style
imports, so the `:root` variables are available globally:

```js
import './styles/tokens.css'
```

## 2. `client/src/styles/tokens.css` (new)

Install verbatim from the skill's `assets/tokens.css`. If the skill was invoked with an accent
argument, override only the `--accent` / `--accent-hover` / `--accent-soft` triplet (see presets at
the bottom of that file).

## 3. `client/src/components/Sidebar.vue` (new)

Install from the skill's `assets/Sidebar.vue`. Before finalizing, verify against the real app:

- **i18n keys exist**: `nav.companyName`, `nav.subtitle`, `nav.overview`, `nav.inventory`,
  `nav.orders`, `nav.finance`, `nav.demandForecast`. Check `client/src/locales/en.js`. If any key is
  missing, fall back to the literal label the old top-nav used. `/reports` stays the literal
  `'Reports'` (matching the original).
- **`useI18n` signature**: confirm it exports `t`. Import path `../composables/useI18n`.
- **Child components**: `LanguageSwitcher.vue` and `ProfileMenu.vue` must accept being placed in the
  bottom `.account` row. `ProfileMenu` emits `show-profile-details` and `show-tasks` — re-emit them
  from Sidebar (already wired in the asset) so App.vue keeps controlling the modals.

## 4. `client/src/App.vue`

**Template** — replace the `<header class="top-nav">…</header>` block (and its inner
`.nav-container`, `.logo`, `.nav-tabs`, `LanguageSwitcher`, `ProfileMenu`) with `<Sidebar/>`, and
wrap the filter bar + main in a content column:

```vue
<template>
  <div class="app">
    <Sidebar
      @show-profile-details="showProfileDetails = true"
      @show-tasks="showTasks = true"
    />
    <div class="content">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <!-- modals unchanged: ProfileDetailsModal, TasksModal, etc. -->
  </div>
</template>
```

**Script** — register `Sidebar` (import `./components/Sidebar.vue`). Remove the now-unused
`LanguageSwitcher` / `ProfileMenu` imports/registrations from App.vue **only if** they are no longer
referenced there (they now live inside Sidebar). Keep all modal state (`showProfileDetails`,
`showTasks`, `tasks`, `addTask`, etc.) exactly as-is.

**Styles** — rewrite the layout + refactor shared globals to tokens:

```css
.app { display: flex; flex-direction: row; min-height: 100vh; }
.content { flex: 1; min-width: 0; display: flex; flex-direction: column; }
.main-content {
  flex: 1;
  width: 100%;
  max-width: var(--content-max);
  margin: 0 auto;
  padding: var(--space-6) var(--space-7);
}
body { background: var(--color-bg); color: var(--text-primary); font-family: var(--font-sans); }
```

Delete the old `.top-nav`, `.nav-container`, `.logo`, `.nav-tabs` rules (now in Sidebar).

Refactor the **shared global classes** that views depend on so spacing/colors become consistent —
map current hardcoded values to tokens (same visual result, now centralized):

| Class | Use tokens |
|-------|-----------|
| `.card` | `background: var(--color-surface)`, `border: 1px solid var(--color-border)`, `border-radius: var(--radius-lg)`, `padding: var(--space-6)`, `box-shadow: var(--shadow-sm)` |
| `.page-header` | margins via `--space-6`/`--space-4`; title `font-size: var(--text-2xl)` |
| `.stats-grid` | `gap: var(--space-5)` (keep `repeat(auto-fit, minmax(280px, 1fr))`) |
| `.table-container` | `border: 1px solid var(--color-border)`, `border-radius: var(--radius-lg)`, `background: var(--color-surface)` |
| badges (`.success/.warning/.danger/.info`) | use the semantic `--*` + `--*-soft` pairs |

Do **not** restyle individual view internals beyond what these shared classes propagate.

## 5. `client/src/components/FilterBar.vue`

- Change sticky offset: `top: 70px` → `top: 0` (it now sits at the top of the content column, not
  under a global header).
- The container no longer needs to reserve the old nav width; keep `max-width: var(--content-max)`
  and center, but inside `.content` it spans the area beside the sidebar.
- Restyle with tokens: `background: var(--color-surface)`, `border-bottom: 1px solid
  var(--color-border)`, `padding: var(--space-3) var(--space-7)`, `z-index: var(--z-filterbar)`.
- `<select>` controls: `border-radius: var(--radius-sm)`, `border-color: var(--color-border)`,
  hover `--color-border-hover`. **Keep all filter logic and `useFilters` bindings unchanged.**

## 6. `client/src/components/ProfileMenu.vue` and `LanguageSwitcher.vue`

Only adjust dropdown **positioning** so menus open correctly from the sidebar's bottom-left (e.g.
open upward / to the right) instead of down-from-top-right. Do not change their logic, auth, or i18n.

## 7. Responsive

The icon-rail collapse at `max-width: 1024px` is already in `Sidebar.vue`. Confirm `.main-content`
padding still reads well at the narrower width; no other breakpoints required.

## 8. Acceptance (manual — skill does not auto-verify)

- All six routes render; the active route is highlighted in the sidebar (accent bar + soft bg).
- FilterBar still filters every view; `useFilters` state shared as before.
- Language switch + profile menu + all modals still work; no console errors.
- Below 1024px the sidebar collapses to an icon rail and content reflows.
- `git diff` shows changes confined to `client/src/` (no `server/` edits).
