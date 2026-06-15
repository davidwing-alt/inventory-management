<script setup>
/**
 * Sidebar.vue — left vertical navigation for the SaaS redesign.
 *
 * Installed by the `saas-redesign` skill at client/src/components/Sidebar.vue.
 * Replaces the old `.top-nav` in App.vue. Preserves the exact 6 routes, the
 * i18n labels, and the active-route logic of the original top nav.
 *
 * Behavior contract (do not change when adapting):
 *  - Same routes, same labels via t('nav.*'), same active detection.
 *  - Account section reuses the existing LanguageSwitcher + ProfileMenu.
 *  - No emojis (business UI). Icons are inline SVG, stroke = currentColor.
 */
import { useRoute } from 'vue-router'
import { useI18n } from '../composables/useI18n'
import LanguageSwitcher from './LanguageSwitcher.vue'
import ProfileMenu from './ProfileMenu.vue'

const emit = defineEmits(['show-profile-details', 'show-tasks'])

const route = useRoute()
const { t } = useI18n()

// labelKey -> resolved via t(); label -> literal (mirrors the original app,
// which hardcoded "Reports"). `icon` selects an inline SVG below.
const navItems = [
  { to: '/',          icon: 'dashboard', labelKey: 'nav.overview' },
  { to: '/inventory', icon: 'inventory', labelKey: 'nav.inventory' },
  { to: '/orders',    icon: 'orders',    labelKey: 'nav.orders' },
  { to: '/spending',  icon: 'spending',  labelKey: 'nav.finance' },
  { to: '/demand',    icon: 'demand',    labelKey: 'nav.demandForecast' },
  { to: '/reports',   icon: 'reports',   label: 'Reports' },
]

const isActive = (path) => route.path === path
const labelFor = (item) => (item.labelKey ? t(item.labelKey) : item.label)
</script>

<template>
  <aside class="sidebar">
    <div class="brand">
      <span class="brand-mark">{{ t('nav.companyName').charAt(0) }}</span>
      <div class="brand-text">
        <span class="brand-name">{{ t('nav.companyName') }}</span>
        <span class="brand-sub">{{ t('nav.subtitle') }}</span>
      </div>
    </div>

    <nav class="nav">
      <router-link
        v-for="item in navItems"
        :key="item.to"
        :to="item.to"
        class="nav-item"
        :class="{ active: isActive(item.to) }"
        :title="labelFor(item)"
      >
        <span class="nav-icon" aria-hidden="true">
          <!-- dashboard -->
          <svg v-if="item.icon === 'dashboard'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="3" width="7" height="9" rx="1" /><rect x="14" y="3" width="7" height="5" rx="1" /><rect x="14" y="12" width="7" height="9" rx="1" /><rect x="3" y="16" width="7" height="5" rx="1" />
          </svg>
          <!-- inventory -->
          <svg v-else-if="item.icon === 'inventory'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z" /><path d="M3.27 6.96 12 12.01l8.73-5.05" /><path d="M12 22.08V12" />
          </svg>
          <!-- orders -->
          <svg v-else-if="item.icon === 'orders'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M9 2h6a1 1 0 0 1 1 1v1h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2V3a1 1 0 0 1 1-1z" /><path d="M9 12h6" /><path d="M9 16h6" />
          </svg>
          <!-- spending -->
          <svg v-else-if="item.icon === 'spending'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="12" y1="1" x2="12" y2="23" /><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6" />
          </svg>
          <!-- demand -->
          <svg v-else-if="item.icon === 'demand'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <polyline points="23 6 13.5 15.5 8.5 10.5 1 18" /><polyline points="17 6 23 6 23 12" />
          </svg>
          <!-- reports -->
          <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="20" x2="18" y2="10" /><line x1="12" y1="20" x2="12" y2="4" /><line x1="6" y1="20" x2="6" y2="14" />
          </svg>
        </span>
        <span class="nav-label">{{ labelFor(item) }}</span>
      </router-link>
    </nav>

    <div class="account">
      <LanguageSwitcher />
      <ProfileMenu
        @show-profile-details="emit('show-profile-details')"
        @show-tasks="emit('show-tasks')"
      />
    </div>
  </aside>
</template>

<style scoped>
.sidebar {
  width: var(--sidebar-width);
  flex-shrink: 0;
  height: 100vh;
  position: sticky;
  top: 0;
  z-index: var(--z-sidebar);
  display: flex;
  flex-direction: column;
  background: var(--color-surface);
  border-right: 1px solid var(--color-border);
}

/* Brand */
.brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-5) var(--space-5);
  border-bottom: 1px solid var(--color-border);
  min-height: 70px;
}
.brand-mark {
  width: 36px;
  height: 36px;
  flex-shrink: 0;
  border-radius: var(--radius-md);
  background: var(--accent);
  color: var(--accent-contrast);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: var(--text-lg);
}
.brand-text { display: flex; flex-direction: column; overflow: hidden; }
.brand-name {
  font-size: var(--text-base);
  font-weight: 700;
  color: var(--text-primary);
  white-space: nowrap;
}
.brand-sub {
  font-size: var(--text-xs);
  color: var(--text-secondary);
  /* wrap instead of clip: subtitle is wider than the 240px sidebar text column */
  line-height: 1.3;
}

/* Nav */
.nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-4) var(--space-3);
  overflow-y: auto;
}
.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) var(--space-3);
  border-radius: var(--radius-md);
  color: var(--text-secondary);
  text-decoration: none;
  font-size: var(--text-base);
  font-weight: 500;
  position: relative;
  transition: background var(--transition), color var(--transition);
}
.nav-item:hover {
  background: var(--color-surface-alt);
  color: var(--text-primary);
}
.nav-item.active {
  background: var(--accent-soft);
  color: var(--accent);
  font-weight: 600;
}
.nav-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 3px;
  height: 60%;
  border-radius: var(--radius-full);
  background: var(--accent);
}
.nav-icon {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
  display: inline-flex;
}
.nav-icon svg { width: 100%; height: 100%; }
.nav-label { white-space: nowrap; }

/* Account section pinned to bottom */
.account {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  gap: var(--space-2);
  padding: var(--space-3);
  border-top: 1px solid var(--color-border);
}

/* Responsive: collapse to icon rail */
@media (max-width: 1024px) {
  .sidebar { width: var(--sidebar-rail); }
  .brand { justify-content: center; padding: var(--space-5) 0; }
  .brand-text { display: none; }
  .nav-item { justify-content: center; padding: var(--space-3) 0; }
  .nav-label { display: none; }
  .account { flex-direction: column; }
}
</style>
