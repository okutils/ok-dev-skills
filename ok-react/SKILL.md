---
name: ok-react
description: 在编写、修改、重构或审查 React 组件与页面时使用，遵循统一的编码与组织约定，保持代码风格一致、组件职责清晰。
---

## 总则

- 使用 TypeScript。
- 使用箭头函数定义 React 函数组件。
- 一个组件文件只应包含一个主要组件。若一个页面需要多个组件，应将它们拆分到各自的文件。
- 页面中的删除操作应提供确认交互，用户确认后再执行删除。
- 推荐使用 `@` 作为指向 `src` 的路径别名，也可按需采用其他前缀；保持构建工具与 TypeScript 的路径映射一致。

## 组件 Props 定义

- 使用 `interface` 定义组件 Props，并为属性标注明确类型；使用 `FC<Props>` 绑定 Props 类型，在组件内部解构所需的 Props 属性。

示例：

```tsx
// types.ts
import type { ReactNode } from "react";

export interface AuthCardProps {
  title?: string;
  content: ReactNode;
}
```

```tsx
// auth-card.tsx
import type { FC } from "react";
import type { AuthCardProps } from "./types";

const AuthCard: FC<AuthCardProps> = (props) => {
  const { title = "无标题", content } = props;
  // ...
};
```

## 状态与 Hooks

- 能从当前 Props、State 计算的值，在渲染时派生；提交、删除等由用户操作触发的副作用，放在对应事件处理函数中。
- 不同同步目标、不同依赖的 Effect 分开编写；Effect 只需要对象中的部分字段时，提取所需字段作为依赖，并在 Effect 中构造所需配置。如连接只依赖 `roomId` 时，在 Effect 中构造 `{ roomId }`，依赖数组使用 `[roomId]`。
- 数组、对象、函数的默认值若用于 Hook 依赖或传给 memo 子组件，使用模块级稳定常量；默认集合按只读数据使用。如在模块级定义 `const EMPTY_ITEMS: readonly Item[] = []`，组件内使用 `const { items = EMPTY_ITEMS } = props`。
- 项目版本支持时，Effect 创建的订阅回调可用 `useEffectEvent` 读取不应触发重新订阅的最新值；真正决定订阅的值继续作为依赖。Effect Event 仅用于 Effect 逻辑，不作为普通事件回调向子组件传递。如连接依赖 `roomId`，连接成功通知通过 Effect Event 读取最新 `theme`，连接 Effect 的依赖数组使用 `[roomId]`。

## 错误处理

- 按 Props 契约和实际业务状态组织渲染，区分空数据与操作失败。
- 复用项目已有的错误处理机制；组件需要提供恢复操作或错误反馈时，再添加局部处理。

## 公共组件组织

- 封装公共组件时，将组件实现与 Props 类型拆分到独立文件，并通过 `index.ts` 统一导出。例如：

```text
common-input/
├── index.ts
├── common-input.tsx
└── types.ts
```

```ts
export { CommonInput } from "./common-input";
export type { CommonInputProps } from "./types";
```
