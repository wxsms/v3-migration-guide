# Vue 3 迁移指南

本指南主要是为有 Vue 2 经验的、希望了解 Vue 3 的新功能和更改的用户而提供的。**在试用 Vue 3 之前，你不必完整阅读这些内容**。推荐通过阅读[新的文档](https://cn.vuejs.org)来学习 Vue 3。

<!-- VueMastery Start -->
<style>
.vue-mastery-link {
  background-color: #f9f9f9;
  border-radius: 8px;
  padding: 8px 16px 8px 8px;
}

.vue-mastery-link a {
  display: flex;
  align-items: center;
  text-decoration: none;
}

.vue-mastery-link .banner {
  background-color: #f9f9f9;
  border-radius: 4px;
  width:96px;
  height:56px;
  object-fit: cover;
}

.vue-mastery-link .description {
  flex: 1;
  font-weight: 500;
  font-size: 14px;
  line-height: 20px;
  color: #213547;
  margin: 0 0 0 16px;
}

.vue-mastery-link .description span {
  color: #42b883;
}

.vue-mastery-link .logo-wrapper {
  position: relative;
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background-color: #ffffff;
  display: flex;
  justify-content: center;
  align-items: center;
}

.vue-mastery-link .logo-wrapper img {
  width: 25px;
  object-fit: contain;
}

@media (max-width: 576px) {
  .vue-mastery-link .banner {
    width:56px;
  }

  .vue-mastery-link .description {
    font-size: 12px;
    line-height: 18px;
  }
  .vue-mastery-link .logo-wrapper {
    position: relative;
    width: 32px;
    height: 32px;
  }
}
</style>

<div class="vue-mastery-link">
  <a href="https://www.vuemastery.com/migration-guide-cheat-sheet/" target="_blank">
    <div class="banner-wrapper">
      <img class="banner" alt="Vue Mastery banner" width="96px" height="56px" src="https://storage.googleapis.com/vue-mastery.appspot.com/flamelink/media/vuemastery-graphical-link-96x56.png" />
    </div>
    <p class="description">Get the free Migration Guide Cheat Sheet at <span>VueMastery.com</span></p>
    <div class="logo-wrapper">
        <img alt="Vue Mastery Logo" width="25px" src="https://storage.googleapis.com/vue-mastery.appspot.com/flamelink/media/vue-mastery-logo.png" />
    </div>
  </a>
</div>
<!-- VueMastery End -->

<style>
.note {
  color: #476582;
}
</style>

## 值得注意的新功能

这里有某些需要注意的 Vue 3 新功能，包括：

- [组合式 API](https://cn.vuejs.org/guide/extras/composition-api-faq.html)<span class="note">\*</span>
- [SFC 组合式 API 语法糖 (`<script setup>`)](https://cn.vuejs.org/api/sfc-script-setup.html)<span class="note">\*</span>
- [Teleport](https://cn.vuejs.org/guide/built-ins/teleport.html)
- [Fragments](/new/fragments.html)
- [Emits 组件选项](https://cn.vuejs.org/api/options-state.html#emits)<span class="note">\*\*</span>
- [来自 `@vue/runtime-core` 的 `createRenderer` API](https://cn.vuejs.org/api/custom-renderer.html) to create custom renderers
- [SFC 状态驱动的 CSS 变量 (`<style>` 中的 `v-bind`)](https://cn.vuejs.org/api/sfc-css-features.html#v-bind-in-css)<span class="note">\*</span>
- [SFC `<style scoped>` 现在可以包含全局规则或针对插槽内容的规则](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0023-scoped-styles-changes.md)
- [Suspense](https://vuejs.org/guide/built-ins/suspense.html) <sup class="warning">experimental</sup>

<sub class="note"><b>\*</b> 现在也在 <a href="https://blog.vuejs.org/posts/vue-2-7-naruto.html" target="_blank">Vue 2.7</a> 中可用</sub> <br>
<sub class="note"><b>\*\*</b> Vue 2.7 中支持，但仅限于类型推断</sub>

## 不兼容的改动

Vue 2 和 Vue 3 之间的不兼容改动[在此](/breaking-changes/)列出。

## 新的推荐框架

新的推荐框架[在此](/recommendations)列出。

## 用于迁移的构建版本

如果你想要将已存在的 Vue 2 项目或库希望升级到 Vue 3，我们提供了一个与 Vue 2 API 兼容的 Vue 3 构建版本。详见[用于迁移的构建版本](./migration-build.html)。