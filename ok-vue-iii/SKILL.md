---
name: ok-vue-iii
description: 在编写、修改、重构或审查 Vue 3 组件与页面时使用，遵循统一的编码与组织约定，保持代码风格一致、组件职责清晰。
---

## 总则

- 使用 TypeScript 与 `<script setup lang="ts">`。
- 默认使用单文件组件（`.vue`），按 `<template>`、`<script setup lang="ts">`、`<style scoped>` 的顺序组织；仅在用户明确要求时使用 JSX/TSX。
- 组件目录和文件名使用 kebab-case；一个组件文件只包含一个主要组件，页面需要多个组件时拆分到各自的文件。
- 页面中的删除操作应提供确认交互，用户确认后再执行删除。
- 推荐使用 `@` 作为指向 `src` 的路径别名，也可按需采用其他前缀；保持构建工具与 TypeScript 的路径映射一致。

## 组件 Props、Emits 与双向绑定

- 使用 `interface` 定义 Props，并通过基于类型的 `defineProps<Props>()` 声明；需要默认值时使用 `withDefaults`，数组与对象默认值使用工厂函数返回。项目版本支持 Vue 3.5+ 时，可直接使用响应式 Props 解构赋默认值。
- 使用基于类型的 `defineEmits` 声明事件；组件需要双向绑定时，使用 `defineModel()`。

示例：

```ts
// types.ts
export interface AuthCardProps {
  title?: string;
  tags?: readonly string[];
}

export interface AuthCardEmits {
  submit: [value: string];
}
```

```vue
<!-- auth-card.vue -->
<template>
  <section>
    <h2>{{ props.title }}</h2>
    <input v-model="keyword" />
    <button @click="handleSubmit">提交</button>
  </section>
</template>

<script setup lang="ts">
import type { AuthCardEmits, AuthCardProps } from "./types";

const props = withDefaults(defineProps<AuthCardProps>(), {
  title: "无标题",
  tags: () => [],
});

const emit = defineEmits<AuthCardEmits>();
const keyword = defineModel<string>({ default: "" });

const handleSubmit = () => {
  emit("submit", keyword.value);
};
</script>

<style scoped></style>
```

## Vue JSX（TSX）

- 有状态或需要生命周期的 TSX 组件，使用 `defineComponent` 的 setup 写法；纯展示的轻量渲染片段使用箭头函数组件。
- Props 与 Emits 同样使用 `interface` 定义，公共组件拆分到 `types.ts` 并通过 `index.ts` 统一导出。
- 具名插槽使用 `v-slots` 对象写法；样式使用 CSS Modules（如 `*.module.css`）或项目约定的样式方案。

## 响应式状态与侦听器

- 优先使用 `ref` 定义响应式状态，并将组件内的 `ref` 声明集中在脚本中的同一区域编写。
- 能从当前 Props 或响应式状态计算的值，使用 `computed` 派生；提交、删除等由用户操作触发的副作用，放在对应事件处理函数中。
- 不同同步目标、不同依赖的 `watch` 分开编写；只需监听对象中的部分字段时，使用 getter（如 `() => query.roomId`）作为侦听源。
- `watch`、`watchEffect` 中产生的异步请求、定时器或订阅，通过 `onCleanup` 回调清理（项目版本支持 Vue 3.5+ 时，也可在同步阶段调用 `onWatcherCleanup`）。

## 错误处理

- 按 Props 契约和实际业务状态组织渲染，区分空数据与操作失败。
- 复用项目已有的错误处理机制；组件需要提供恢复操作或错误反馈时，再添加局部处理。

## 公共组件组织

- 封装公共组件时，将组件实现与类型定义拆分到独立文件，并通过 `index.ts` 统一导出。例如：

```text
common-input/
├── index.ts
├── common-input.vue
└── types.ts
```

```ts
export { default as CommonInput } from "./common-input.vue";
export type { CommonInputProps } from "./types";
```
