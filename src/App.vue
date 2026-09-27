<script setup lang="ts">
import { onBeforeUnmount, provide, ref, watch } from 'vue'
import { RouterView } from 'vue-router'

// `.to-dark` 挂在 <html> 上而不是应用根节点：PrimeVue 会把 Dialog 等浮层传送到 body，
// 只有放在 html 上，这些浮层才能同时拿到 PrimeVue 暗色色板和下方的 --app-* 变量。
// 初始值由 index.html 中的内联脚本在首帧前写入，这里直接读取即可。
const root = document.documentElement
const isDark = ref(root.classList.contains('to-dark'))
watch(isDark, value => root.classList.toggle('to-dark', value))

// 只有手动切换才写入 localStorage；未写入时持续跟随系统主题变化。
const systemDark = window.matchMedia('(prefers-color-scheme: dark)')
function followSystem(event: MediaQueryListEvent) {
  let saved = null
  try { saved = localStorage.getItem('jh-theme') } catch {}
  if (!saved) isDark.value = event.matches
}
systemDark.addEventListener('change', followSystem)
onBeforeUnmount(() => systemDark.removeEventListener('change', followSystem))

provide('isDark', isDark)
provide('toggleTheme', () => {
  isDark.value = !isDark.value
  try { localStorage.setItem('jh-theme', isDark.value ? 'dark' : 'light') } catch {}
})
</script>

<template>
  <div class="app-shell">
    <RouterView v-slot="{ Component }">
    <Transition name="page" mode="out-in">
      <component :is="Component" />
    </Transition>
    </RouterView>
  </div>
</template>

<style>
/* 全站颜色统一走这组变量，页面样式不再写死黑白色值，暗色只需在这里改一处。
 * --app-field：输入框、卡片、气泡等"纸面"；--app-muted / --app-faint：次要 / 更弱的文字；
 * --app-line：页面内分隔线；--app-hover：悬停与选中底色；--app-map-*：无地图 Key 时的校区示意图。
 * 需要反色强调（选中按钮、我方气泡）时直接用 --app-text 作底、--app-bg 作字。 */
:root {
  color-scheme: light;
  --app-bg: #f7f5ef; --app-surface: #fcfbf8; --app-field: #fff; --app-text: #24231f;
  --app-muted: #777; --app-faint: #999; --app-border: #dedbd3; --app-line: #ddd; --app-hover: #ebebe7;
  --app-map-bg: #eeefea; --app-map-road: #dfdfdb; --app-map-river: #d4deda; --app-map-label: #92968e;
  --grok-plate: #f7f5ef;
}
:root.to-dark {
  color-scheme: dark;
  --app-bg: #1f1e1b; --app-surface: #292722; --app-field: #2f2d28; --app-text: #f2eee5;
  --app-muted: #aaa598; --app-faint: #8c877c; --app-border: #514d45; --app-line: #3e3b35; --app-hover: #3a3731;
  --app-map-bg: #262521; --app-map-road: #36342e; --app-map-river: #2e3835; --app-map-label: #7f7b71;
  --grok-plate: #292722;
}
/* PrimeVue 的输入框与次要按钮默认是冷色 zinc/slate 色板，改为引用上面的暖色变量。
 * PrimeVue 运行时注入的变量选择器是 `:root,:host` 且位置在后，这里用 html:root 提高优先级，避免被覆盖。
 * 另：PrimeVue 5 的色值使用 light-dark()，依赖上面 color-scheme 的切换。 */
html:root {
  --p-form-field-background: var(--app-field); --p-form-field-color: var(--app-text);
  --p-form-field-border-color: var(--app-line); --p-form-field-hover-border-color: var(--app-faint);
  --p-form-field-placeholder-color: var(--app-faint);
  --p-button-secondary-background: var(--app-hover); --p-button-secondary-border-color: var(--app-hover); --p-button-secondary-color: var(--app-text);
  --p-button-secondary-hover-background: var(--app-line); --p-button-secondary-hover-border-color: var(--app-line); --p-button-secondary-hover-color: var(--app-text);
}
html, body, #app { min-height: 100%; }
body { background: var(--app-bg); color: var(--app-text); }
.app-shell { min-height: 100vh; background: var(--app-bg); color: var(--app-text); }
/* PrimeVue Sidebar 的 DOM 由组件内部生成，使用全局选择器覆盖其默认纯白背景。
 * 只匹配 sidebar 系列组件；[data-pc-section="root"] 会命中所有 PrimeVue 按钮，把它们的底色也盖掉。 */
.app-shell :where([data-pc-name^="sidebar"], .p-sidebar) { background: var(--app-surface) !important; color: var(--app-text); }
.app-shell :where([data-pc-name="sidebar"] *) { border-color: var(--app-border); }
/* Dialog 被 PrimeVue 传送到 body、位于 .app-shell 之外，所以这里不能加 .app-shell 前缀。 */
:where(.p-dialog, .p-dialog-content, .p-dialog-header) { background: var(--app-surface) !important; color: var(--app-text) !important; border-color: var(--app-border) !important; }
:where(.p-dialog-content input, .p-dialog-content textarea, .p-dialog-content select) { background: var(--app-bg) !important; color: var(--app-text) !important; border-color: var(--app-border) !important; }
:where(.p-dialog-content input::placeholder, .p-dialog-content textarea::placeholder) { color: color-mix(in srgb, var(--app-text) 55%, transparent) !important; }
:where(.post-form, .login-form) { color: var(--app-text); }
:where(.post-form > p, .post-preview small, .login-intro p) { color: color-mix(in srgb, var(--app-text) 60%, transparent) !important; }
/* 顶部快捷按钮显式指定前景/背景，避免 PrimeVue contrast 色板在浅色主题下把文字和按钮都变成白色。 */
.app-shell :where(.header-action, .theme-action, .ink-button) { appearance: none; background: var(--app-text) !important; color: var(--app-bg) !important; border: 1px solid var(--app-text) !important; opacity: 1 !important; }
.app-shell :where(.header-action:hover, .theme-action:hover, .ink-button:hover) { filter: brightness(.9); }
.app-shell :where(.theme-action) { background: transparent !important; color: var(--app-text) !important; border-color: var(--app-border) !important; }
.page-enter-active,
.page-leave-active {
  transition: opacity 0.25s ease;
}

.page-enter-from,
.page-leave-to {
  opacity: 0;
}

@media (prefers-reduced-motion: reduce) {
  .page-enter-active,
  .page-leave-active {
    transition: none;
  }
}
</style>
