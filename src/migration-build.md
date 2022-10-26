# 用于迁移的构建版本

## 概述

`@vue/compat` (即“迁移构建版本”) 是一个 Vue 3 的构建版本，提供了可配置的兼容 Vue 2 的行为。

该构建版本默认运行在 Vue 2 的模式下——大部分公有 API 的行为和 Vue 2 一致，仅有一小部分例外。使用在 Vue 3 中发生改变或被废弃的特性时会抛出运行时警告。一个特性的兼容性也可以基于单个组件进行开启或禁用。

### 预期用例


- 将一个 Vue 2 应用升级为 Vue 3 (存在[限制](#known-limitations))
- 迁移一个库以支持 Vue 3
- 对于尚未尝试 Vue 3 的资深 Vue 2 开发者来说，迁移构建版本可以用来代替 Vue 3 以更好地学习版本之间的差异。

### 已知的限制 {#known-limitations}

虽然我们已经努力使迁移构建版本尽可能地模拟 Vue 2 的行为，但仍有一些限制可能会阻止应用的顺利升级：

- 基于 Vue 2 内部 API 或文档中未记载行为的依赖。最常见的情况就是使用 `VNodes` 上的私有 property。如果你的项目依赖诸如 [Vuetify](https://vuetifyjs.com/zh-Hans/)、[Quasar](https://quasar.dev/) 或 [Element UI](https://element.eleme.io/#/zh-CN) 等组件库，那么最好等待一下它们的 Vue 3 兼容版本。

- 对 IE11 的支持：[Vue 3 已经官方放弃对 IE11 的支持](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0038-vue3-ie11-support.md)。如果仍然需要支持 IE11 或更低版本，那你仍需继续使用 Vue 2。

- 服务端渲染：该迁移构建版本可以被用于服务端渲染，但是迁移一个自定义的服务端渲染配置还有很多工作要做。大致的思路是将 `vue-server-renderer` 替换为 [`@vue/server-renderer`](https://github.com/vuejs/vue-next/tree/master/packages/server-renderer)。Vue 3 不再提供一个包渲染器，且我们推荐使用 [Vite](https://cn.vitejs.dev/guide/ssr.html) 以支持 Vue 3 服务端渲染。如果你正在使用 [Nuxt.js](https://zh.nuxtjs.org/)，可能最好还是等待一下 Nuxt 3。

### 预期 {#expectations}

请注意迁移构建版本旨在覆盖在文档中公开记载的 Vue 2 API 和行为。如果应用依赖了未记载的行为导致在迁移构建下运行失败，我们可能不太会调整迁移构建版本以迎合这种特殊情况。请考虑重构或移除导致这些问题行为的依赖。

多留意一下：如果你的应用较大且复杂，即便有了迁移构建版本，整个迁移过程也会是一个挑战。如果你的应用不幸无法顺利升级，请留意我们正在计划将组合式 API 等其它 Vue 3 特性迁移回 Vue 2.7 (预计在 2021 年第三季度末)。

如果应用在迁移构建版本中顺利运行，你**可以**在迁移完成之前将其发布到生产环境。尽管存在一些小的性能或包大小的问题，但应该不会显著地影响到生产环境的用户体验。当你有基于 Vue 2 行为的依赖且无法升级/替换时，可能不得不这样做。

该迁移构建版本会从 3.1 开始提供，且会随着 3.2 的发布计划进行持续发布。我们计划在将来某个小版本号起最终停止发布迁移构建版本 (在 2021 年底前至少不会)，因此你仍需要在此之前将其迁移到标准构建版本。

## 升级流程 {#upgrade-workflow}

下面的工作流程讲述了将一个实际的 Vue 2 应用 (Vue HackerNews 2.0) 迁移到 Vue 3 的过程。完整的提交记录在[这里](https://github.com/vuejs/vue-hackernews-2.0/compare/migration)。请注意，你的项目所需要的实际步骤可能有所不同。下面的步骤仅应被视为一般性的指南，而非严格的教程。

### 准备工作 {#preparations}

- 如果你仍然使用[废弃的具名/作用域插槽语法](https://cn.vuejs.org/v2/guide/components-slots.html#废弃了的语法)，请先将其更新至 (2.6 已经支持的) 最新的语法。

### 安装

1. Upgrade tooling if applicable.

   - If using custom webpack setup: Upgrade `vue-loader` to `^16.0.0`.
   - If using `vue-cli`: upgrade to the latest `@vue/cli-service` with `vue upgrade`
   - (Alternative) migrate to [Vite](https://vitejs.dev/) + [vite-plugin-vue2](https://github.com/underfin/vite-plugin-vue2). [[Example commit](https://github.com/vuejs/vue-hackernews-2.0/commit/565b948919eb58f22a32afca7e321b490cb3b074)]

2. In `package.json`, update `vue` to 3.1, install `@vue/compat` of the same version, and replace `vue-template-compiler` (if present) with `@vue/compiler-sfc`:

   ```diff
   "dependencies": {
   -  "vue": "^2.6.12",
   +  "vue": "^3.1.0",
   +  "@vue/compat": "^3.1.0"
      ...
   },
   "devDependencies": {
   -  "vue-template-compiler": "^2.6.12"
   +  "@vue/compiler-sfc": "^3.1.0"
   }
   ```

   [Example commit](https://github.com/vuejs/vue-hackernews-2.0/commit/14f6f1879b43f8610add60342661bf915f5c4b20)

3. In the build setup, alias `vue` to `@vue/compat` and enable compat mode via Vue compiler options.

   **Example Configs**

   <details>
     <summary><b>vue-cli</b></summary>

   ```js
   // vue.config.js
   module.exports = {
     chainWebpack: (config) => {
       config.resolve.alias.set('vue', '@vue/compat')

       config.module
         .rule('vue')
         .use('vue-loader')
         .tap((options) => {
           return {
             ...options,
             compilerOptions: {
               compatConfig: {
                 MODE: 2
               }
             }
           }
         })
     }
   }
   ```

   </details>

   <details>
     <summary><b>Plain webpack</b></summary>

   ```js
   // webpack.config.js
   module.exports = {
     resolve: {
       alias: {
         vue: '@vue/compat'
       }
     },
     module: {
       rules: [
         {
           test: /\.vue$/,
           loader: 'vue-loader',
           options: {
             compilerOptions: {
               compatConfig: {
                 MODE: 2
               }
             }
           }
         }
       ]
     }
   }
   ```

   </details>

   <details>
     <summary><b>Vite</b></summary>

   ```js
   // vite.config.js
   export default {
     resolve: {
       alias: {
         vue: '@vue/compat'
       }
     },
     plugins: [
       vue({
         template: {
           compilerOptions: {
             compatConfig: {
               MODE: 2
             }
           }
         }
       })
     ]
   }
   ```

   </details>

4. If you are using TypeScript, you will also need to modify `vue`'s typing to expose the default export (which is no longer present in Vue 3) by adding a `*.d.ts` file with the following:

   ```ts
   declare module 'vue' {
     import { CompatVue } from '@vue/runtime-dom'
     const Vue: CompatVue
     export default Vue
     export * from '@vue/runtime-dom'
     const { configureCompat } = Vue
     export { configureCompat }
   }
   ```

5. At this point, your application may encounter some compile-time errors / warnings (e.g. use of filters). Fix them first. If all compiler warnings are gone, you can also set the compiler to Vue 3 mode.

   [Example commit](https://github.com/vuejs/vue-hackernews-2.0/commit/b05d9555f6e115dea7016d7e5a1a80e8f825be52)

6. After fixing the errors, the app should be able to run if it is not subject to the [limitations](#known-limitations) mentioned above.

   You will likely see a LOT of warnings from both the command line and the browser console. Here are some general tips:

   - You can filter for specific warnings in the browser console. It's a good idea to use the filter and focus on fixing one item at a time. You can also use negated filters like `-GLOBAL_MOUNT`.

   - You can suppress specific deprecations via [compat configuration](#compat-configuration).

   - Some warnings may be caused by a dependency that you use (e.g. `vue-router`). You can check this from the warning's component trace or stack trace (expanded on click). Focus on fixing the warnings that originate from your own source code first.

   - If you are using `vue-router`, note `<transition>` and `<keep-alive>` will not work with `<router-view>` until you upgrade to `vue-router` v4.

7. Update [`<transition>` class names](/breaking-changes/transition.html). This is the only feature that does not have a runtime warning. You can do a project-wide search for `.*-enter` and `.*-leave` CSS class names.

   [Example commit](https://github.com/vuejs/vue-hackernews-2.0/commit/d300103ba622ae26ac26a82cd688e0f70b6c1d8f)

8. Update app entry to use [new global mounting API](/breaking-changes/global-api.html#a-new-global-api-createapp).

   [Example commit](https://github.com/vuejs/vue-hackernews-2.0/commit/a6e0c9ac7b1f4131908a4b1e43641f608593f714)

9. [Upgrade `vuex` to v4](https://next.vuex.vuejs.org/guide/migrating-to-4-0-from-3-x.html).

   [Example commit](https://github.com/vuejs/vue-hackernews-2.0/commit/5bfd4c61ee50f358cd5daebaa584f2c3f91e0205)

10. [Upgrade `vue-router` to v4](https://next.router.vuejs.org/index.html). If you also use `vuex-router-sync`, you can replace it with a store getter.

    After the upgrade, to use `<transition>` and `<keep-alive>` with `<router-view>` requires using the new [scoped-slot based syntax](https://router.vuejs.org/guide/migration/#router-view-keep-alive-and-transition).

    [Example commit](https://github.com/vuejs/vue-hackernews-2.0/commit/758961e73ac4089890079d4ce14996741cf9344b)

11. Pick off individual warnings. Note some features have conflicting behavior between Vue 2 and Vue 3 - for example, the render function API, or the functional component vs. async component change. To migrate to Vue 3 API without affecting the rest of the application, you can opt-in to Vue 3 behavior on a per-component basis with the [`compatConfig` option](#per-component-config).

    [Example commit](https://github.com/vuejs/vue-hackernews-2.0/commit/d0c7d3ae789be71b8fd56ce79cb4cb1f921f893b)

12. When all warnings are fixed, you can remove the migration build and switch to Vue 3 proper. Note you may not be able to do so if you still have dependencies that rely on Vue 2 behavior.

    [Example commit](https://github.com/vuejs/vue-hackernews-2.0/commit/9beb45490bc5f938c9e87b4ac1357cfb799565bd)

## Compat Configuration

### Global Config

Compat features can be disabled individually:

```js
import { configureCompat } from 'vue'

// disable compat for certain features
configureCompat({
  FEATURE_ID_A: false,
  FEATURE_ID_B: false
})
```

Alternatively, the entire application can default to Vue 3 behavior, with only certain compat features enabled:

```js
import { configureCompat } from 'vue'

// default everything to Vue 3 behavior, and only enable compat
// for certain features
configureCompat({
  MODE: 3,
  FEATURE_ID_A: true,
  FEATURE_ID_B: true
})
```

### Per-Component Config

A component can use the `compatConfig` option, which expects the same options as the global `configureCompat` method:

```js
export default {
  compatConfig: {
    MODE: 3, // opt-in to Vue 3 behavior for this component only
    FEATURE_ID_A: true // features can also be toggled at component level
  }
  // ...
}
```

### Compiler-specific Config

Features that start with `COMPILER_` are compiler-specific: if you are using the full build (with in-browser compiler), they can be configured at runtime. However if using a build setup, they must be configured via the `compilerOptions` in the build config instead (see example configs above).

## Feature Reference

### Compatibility Types

- ✔ Fully compatible
- ◐ Partially Compatible with caveats
- ⨂ Incompatible (warning only)
- ⭘ Compat only (no warning)

### Incompatible

> Should be fixed upfront or will likely lead to errors

| ID                                    | Type | Description                                                             | Docs                                                                                       |
| ------------------------------------- | ---- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| GLOBAL_MOUNT_CONTAINER                | ⨂    | Mounted application does not replace the element it's mounted to        | [link](/breaking-changes/mount-changes.html)                                               |
| CONFIG_DEVTOOLS                       | ⨂    | production devtools is now a build-time flag                            | [link](https://github.com/vuejs/core/tree/master/packages/vue#bundler-build-feature-flags) |
| COMPILER_V_IF_V_FOR_PRECEDENCE        | ⨂    | `v-if` and `v-for` precedence when used on the same element has changed | [link](/breaking-changes/v-if-v-for.html)                                                  |
| COMPILER_V_IF_SAME_KEY                | ⨂    | `v-if` branches can no longer have the same key                         | [link](/breaking-changes/key-attribute.html#on-conditional-branches)                       |
| COMPILER_V_FOR_TEMPLATE_KEY_PLACEMENT | ⨂    | `<template v-for>` key should now be placed on `<template>`             | [link](/breaking-changes/key-attribute.html#with-template-v-for)                           |
| COMPILER_SFC_FUNCTIONAL               | ⨂    | `<template functional>` is no longer supported in SFCs                  | [link](/breaking-changes/functional-components.html#single-file-components-sfcs)           |

### Partially Compatible with Caveats

| ID                       | Type | Description                                                                                                                                                                                | Docs                                                                                                           |
| ------------------------ | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| CONFIG_IGNORED_ELEMENTS  | ◐    | `config.ignoredElements` is now `config.compilerOptions.isCustomElement` (only in browser compiler build). If using build setup, `isCustomElement` must be passed via build configuration. | [link](/breaking-changes/global-api.html#config-ignoredelements-is-now-config-compileroptions-iscustomelement) |
| COMPILER_INLINE_TEMPLATE | ◐    | `inline-template` removed (compat only supported in browser compiler build)                                                                                                                | [link](/breaking-changes/inline-template-attribute.html)                                                       |
| PROPS_DEFAULT_THIS       | ◐    | props default factory no longer have access to `this` (in compat mode, `this` is not a real instance - it only exposes props, `$options` and injections)                                   | [link](/breaking-changes/props-default-this.html)                                                              |
| INSTANCE_DESTROY         | ◐    | `$destroy` instance method removed (in compat mode, only supported on root instance)                                                                                                       |                                                                                                                |
| GLOBAL_PRIVATE_UTIL      | ◐    | `Vue.util` is private and no longer available                                                                                                                                              |                                                                                                                |
| CONFIG_PRODUCTION_TIP    | ◐    | `config.productionTip` no longer necessary                                                                                                                                                 | [link](/breaking-changes/global-api.html#config-productiontip-removed)                                         |
| CONFIG_SILENT            | ◐    | `config.silent` removed                                                                                                                                                                    |                                                                                                                |

### Compat only (no warning)

| ID                 | Type | Description                            | Docs                                      |
| ------------------ | ---- | -------------------------------------- | ----------------------------------------- |
| TRANSITION_CLASSES | ⭘    | Transition enter/leave classes changed | [link](/breaking-changes/transition.html) |

### Fully Compatible

| ID                           | Type | Description                                                           | Docs                                                                                        |
| ---------------------------- | ---- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| GLOBAL_MOUNT                 | ✔    | new Vue() -> createApp                                                | [link](/breaking-changes/global-api.html#mounting-app-instance)                             |
| GLOBAL_EXTEND                | ✔    | Vue.extend removed (use `defineComponent` or `extends` option)        | [link](/breaking-changes/global-api.html#vue-extend-removed)                                |
| GLOBAL_PROTOTYPE             | ✔    | `Vue.prototype` -> `app.config.globalProperties`                      | [link](/breaking-changes/global-api.html#vue-prototype-replaced-by-config-globalproperties) |
| GLOBAL_SET                   | ✔    | `Vue.set` removed (no longer needed)                                  |                                                                                             |
| GLOBAL_DELETE                | ✔    | `Vue.delete` removed (no longer needed)                               |                                                                                             |
| GLOBAL_OBSERVABLE            | ✔    | `Vue.observable` removed (use `reactive`)                             | [link](https://vuejs.org/api/reactivity-core.html#reactive)                                 |
| CONFIG_KEY_CODES             | ✔    | config.keyCodes removed                                               | [link](/breaking-changes/keycode-modifiers.html)                                            |
| CONFIG_WHITESPACE            | ✔    | In Vue 3 whitespace defaults to `"condense"`                          |                                                                                             |
| INSTANCE_SET                 | ✔    | `vm.$set` removed (no longer needed)                                  |                                                                                             |
| INSTANCE_DELETE              | ✔    | `vm.$delete` removed (no longer needed)                               |                                                                                             |
| INSTANCE_EVENT_EMITTER       | ✔    | `vm.$on`, `vm.$off`, `vm.$once` removed                               | [link](/breaking-changes/events-api.html)                                                   |
| INSTANCE_EVENT_HOOKS         | ✔    | Instance no longer emits `hook:x` events                              | [link](/breaking-changes/vnode-lifecycle-events.html)                                       |
| INSTANCE_CHILDREN            | ✔    | `vm.$children` removed                                                | [link](/breaking-changes/children.html)                                                     |
| INSTANCE_LISTENERS           | ✔    | `vm.$listeners` removed                                               | [link](/breaking-changes/listeners-removed.html)                                            |
| INSTANCE_SCOPED_SLOTS        | ✔    | `vm.$scopedSlots` removed; `vm.$slots` now exposes functions          | [link](/breaking-changes/slots-unification.html)                                            |
| INSTANCE_ATTRS_CLASS_STYLE   | ✔    | `$attrs` now includes `class` and `style`                             | [link](/breaking-changes/attrs-includes-class-style.html)                                   |
| OPTIONS_DATA_FN              | ✔    | `data` must be a function in all cases                                | [link](/breaking-changes/data-option.html)                                                  |
| OPTIONS_DATA_MERGE           | ✔    | `data` from mixin or extension is now shallow merged                  | [link](/breaking-changes/data-option.html)                                                  |
| OPTIONS_BEFORE_DESTROY       | ✔    | `beforeDestroy` -> `beforeUnmount`                                    |                                                                                             |
| OPTIONS_DESTROYED            | ✔    | `destroyed` -> `unmounted`                                            |                                                                                             |
| WATCH_ARRAY                  | ✔    | watching an array no longer triggers on mutation unless deep          | [link](/breaking-changes/watch.html)                                                        |
| V_ON_KEYCODE_MODIFIER        | ✔    | `v-on` no longer supports keyCode modifiers                           | [link](/breaking-changes/keycode-modifiers.html)                                            |
| CUSTOM_DIR                   | ✔    | Custom directive hook names changed                                   | [link](/breaking-changes/custom-directives.html)                                            |
| ATTR_FALSE_VALUE             | ✔    | No longer removes attribute if binding value is boolean `false`       | [link](/breaking-changes/attribute-coercion.html)                                           |
| ATTR_ENUMERATED_COERCION     | ✔    | No longer special case enumerated attributes                          | [link](/breaking-changes/attribute-coercion.html)                                           |
| TRANSITION_GROUP_ROOT        | ✔    | `<transition-group>` no longer renders a root element by default      | [link](/breaking-changes/transition-group.html)                                             |
| COMPONENT_ASYNC              | ✔    | Async component API changed (now requires `defineAsyncComponent`)     | [link](/breaking-changes/async-components.html)                                             |
| COMPONENT_FUNCTIONAL         | ✔    | Functional component API changed (now must be plain functions)        | [link](/breaking-changes/functional-components.html)                                        |
| COMPONENT_V_MODEL            | ✔    | Component v-model reworked                                            | [link](/breaking-changes/v-model.html)                                                      |
| RENDER_FUNCTION              | ✔    | Render function API changed                                           | [link](/breaking-changes/render-function-api.html)                                          |
| FILTERS                      | ✔    | Filters removed (this option affects only runtime filter APIs)        | [link](/breaking-changes/filters.html)                                                      |
| COMPILER_IS_ON_ELEMENT       | ✔    | `is` usage is now restricted to `<component>` only                    | [link](/breaking-changes/custom-elements-interop.html)                                      |
| COMPILER_V_BIND_SYNC         | ✔    | `v-bind.sync` replaced by `v-model` with arguments                    | [link](/breaking-changes/v-model.html)                                                      |
| COMPILER_V_BIND_PROP         | ✔    | `v-bind.prop` modifier removed                                        |                                                                                             |
| COMPILER_V_BIND_OBJECT_ORDER | ✔    | `v-bind="object"` is now order sensitive                              | [link](/breaking-changes/v-bind.html)                                                       |
| COMPILER_V_ON_NATIVE         | ✔    | `v-on.native` modifier removed                                        | [link](/breaking-changes/v-on-native-modifier-removed.html)                                 |
| COMPILER_V_FOR_REF           | ✔    | `ref` in `v-for` (compiler support)                                   |                                                                                             |
| COMPILER_NATIVE_TEMPLATE     | ✔    | `<template>` with no special directives now renders as native element |                                                                                             |
| COMPILER_FILTERS             | ✔    | filters (compiler support)                                            |                                                                                             |
