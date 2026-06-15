---
name: saas-redesign
description: >
  Redesigns the inventory-management Vue 3 client into a modern SaaS-style UI: replaces the top
  navigation bar with a left vertical sidebar, installs a CSS design-token layer (color, spacing,
  radius, shadow) for consistent spacing and a polished professional look, and rewires App.vue and
  FilterBar accordingly. Preserves all behavior (routes, filters, i18n, modals, backend). Use this
  skill when the user types /saas-redesign, or asks to convert the UI to a sidebar layout, give the
  app a modern SaaS look, or apply a design system. Accepts an optional accent-color argument
  (e.g. /saas-redesign teal).
---

# SaaS Redesign Skill

## Purpose

Transforms the inventory-management Vue 3 client (`client/src/`) from its current sticky **top
navigation bar** into a modern **left vertical sidebar** SaaS layout, and introduces a CSS
**design-token system** so spacing, color, radius, and shadows are consistent across the app. The
result is a polished, professional interface with **no behavior changes** — routes, the filter
system, i18n, modals, and the backend contract all stay intact.

This skill works on **this app specifically** (it knows the 6 routes, `App.vue`, and `FilterBar`).

## Guardrails (read first)

- **Scope:** only edit files under `client/src/`. Never touch `server/`, `server/data/*.json`, API
  contracts, or build config.
- **Preserve behavior:** same 6 routes and labels, same `useFilters` filter system, same i18n,
  same modals, same active-route logic. This is a visual/layout refactor only.
- **No emojis** in the UI (business app). Icons are inline SVG.
- **Delegate `.vue` edits to the `vue-expert` agent.** The repo CLAUDE.md mandates that any
  significant `.vue` change goes through `vue-expert`. Non-`.vue` edits (tokens.css, the main.js
  import) you do directly.
- **Reversible:** before changing app code, make sure the user is on a throwaway branch (offer to
  create `saas-redesign` if they are on `main`).
- **Consume tokens** — do not reintroduce hardcoded hex/px for anything a token covers.

## Step 1: Parse the optional accent argument

If the user passed an argument (e.g. `/saas-redesign teal`), treat it as the **accent color**. Map
it to an `--accent` / `--accent-hover` / `--accent-soft` triplet (presets for teal, violet, emerald,
rose are listed at the bottom of `assets/tokens.css`; for any other named/hex color pick a sensible
darker hover and a light tint for soft). With no argument, keep the default blue (`#2563eb`).

## Step 2: Confirm the working tree is safe

Check `git status` / current branch in the repo. If the user is on `main` (or has the redesign not
yet isolated), offer to create and switch to a `saas-redesign` branch before editing. Proceed once
the tree is safe to modify.

## Step 3: Install the design-token layer (direct edit)

1. Copy `assets/tokens.css` (from this skill directory) to `client/src/styles/tokens.css`. Create
   the `styles/` folder if needed. If an accent was chosen in Step 1, apply the triplet override.
2. Add `import './styles/tokens.css'` near the top of `client/src/main.js`, before the app mounts.

## Step 4: Install the Sidebar component (via vue-expert)

Copy `assets/Sidebar.vue` to `client/src/components/Sidebar.vue`, then have **vue-expert** verify it
against the real app: confirm the `nav.*` i18n keys exist in `client/src/locales/en.js` (fall back
to literal labels if any are missing), confirm `useI18n` exports `t`, and confirm `LanguageSwitcher`
/ `ProfileMenu` render correctly inside the sidebar's bottom account row (re-emitting
`show-profile-details` and `show-tasks`).

## Step 5: Rewire App.vue, FilterBar, and dropdowns (via vue-expert)

Hand the **full recipe in `assets/redesign-spec.md`** to the `vue-expert` agent and have it apply:

- **App.vue** — switch `.app` to a row layout, replace the `<header class="top-nav">` block with
  `<Sidebar/>`, wrap `<FilterBar/>` + `<main>` in a `.content` column, and refactor the shared
  global classes (`.card`, `.page-header`, `.stats-grid`, `.table-container`, badges) to consume
  tokens. Keep all modal state and handlers.
- **FilterBar.vue** — sticky `top: 0`, token-based styling; filter logic untouched.
- **ProfileMenu.vue / LanguageSwitcher.vue** — reposition dropdowns to open well from the sidebar
  bottom; logic untouched.

Give vue-expert this exact instruction: *"Follow `.claude/skills/saas-redesign/assets/redesign-spec.md`
verbatim. Visual/layout only — preserve all routes, filters, i18n, modals, and the backend
contract. Use the CSS variables from tokens.css; no hardcoded colors or pixel spacing where a token
exists. No emojis."*

## Step 6: Sanity check (read-only)

After edits, grep `client/src/` to confirm: no remaining `.top-nav` / `.nav-tabs` references; no
`server/` files changed (`git diff --name-only` should be confined to `client/src/`). If the dev
server is running, note any Vite compile errors from `/tmp/inventory-frontend.log`.

## Step 7: Report (no auto-verification)

This skill makes **code changes only** — it does not run Playwright. Finish by reporting:

- the list of files created/modified,
- the accent used,
- a short **manual checklist** for the user: open `http://localhost:3000` and walk all six routes —
  confirm the sidebar highlights the active route, FilterBar still filters, the language switcher /
  profile menu / modals still work, there are no console errors, and the sidebar collapses to an
  icon rail below 1024px,
- the reminder that the change is on a branch and `git diff` / branch-switch reverts it cleanly.

## Files in this skill

- `assets/tokens.css` — canonical design-token layer (install to `client/src/styles/`).
- `assets/Sidebar.vue` — reference left-sidebar component (install to `client/src/components/`).
- `assets/redesign-spec.md` — exact App.vue / FilterBar / dropdown edit recipe for vue-expert.
