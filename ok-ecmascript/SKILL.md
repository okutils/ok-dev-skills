---
name: ok-ecmascript
description: 在编写、修改、重构或审查 JavaScript、TypeScript 代码时使用，统一命名、模块组织、函数与类型定义、枚举及异步处理规范，适用于前端和 Node.js 项目。
---

## 命名

### 常规命名规范

- 变量、函数、参数和属性使用 camelCase；类、interface 和类型别名使用 PascalCase，基类以 `Base` 开头。
- 模块级固定配置、协议常量和枚举对象使用 UPPER_SNAKE_CASE，如 `REQUEST_TIMEOUT_MS`；对象实例、运行时计算结果和局部变量使用 camelCase，如 `apiClient`。
- 事件处理函数使用 `handle` 或 `on` 前缀。
- 异步函数以 `Async` 作为后缀，如 `fetchDataAsync`。
- 对于未使用的函数参数或解构变量，必须使用下划线 `_` 作为前缀。

### 布尔值

布尔值以 `is`、`has`、`can`、`should`、`will` 开头，如 `isReady`、`hasProcessed`、`canRetry`、`shouldRefresh`。类型守卫以 `is` 开头，断言函数以 `assert` 开头。

### 大小写原则

缩写词按普通单词处理大小写，如 `ApiClient`、`HttpRequest`、`userId`、`parseJson`。

### 文件、目录命名

- 目录名和文件名主体使用 kebab-case；文件用途及点分隔后缀的具体形式按照项目约束执行。
- `index.ts` 用于集中导出包或功能模块对外提供的内容。

## 模块与文件组织

- 一个 TypeScript 文件只负责一个主要职责；相互独立的职责应拆分到不同文件。
- 不同业务关注点应拆分到独立模块，通过明确的类型和接口协作。

## 函数

- 普通函数和回调使用箭头函数；类与对象的方法使用 `method() {}` 写法；生成器使用 `function*`；函数重载声明和需要由调用方决定 `this` 的回调使用 `function`。
- 包对外导出的函数、供其他模块调用的服务方法，明确写出参数和返回值类型；内部函数与回调优先使用类型推断。
- 回调函数避免使用单字名称，应当使用具备描述性的名称。
- 查询和转换函数使用 readonly 参数，表示输入只用于读取；直接修改传入数据的函数在名称中说明，如 `sortInPlace`。

- 当函数签名中包含函数类型时（无论是作为参数还是返回值），都应使用 `type` 显式定义该函数类型，而不是在签名中内联。例如：
  ```tsx
  type Callback = (data: string) => void;
  const process = (callback: Callback): void => {};
  ```

## 类型

- 固定结构的对象类型使用 `interface`；联合、元组、映射和条件类型使用 `type`；扩展已有对象类型使用 `extends`。
- 临时对象、JSON 数据和运行时内容会变化的对象，直接使用宽松的类型描述其内容；单独声明的专用类型用于结构稳定、需要复用的业务数据。
- 配置对象使用类型标注或 `satisfies` 检查结构。外部数据在接入代码中校验。
- 临时跳过类型检查时，使用 `@ts-expect-error` 并附上原因和相关任务编号；临时关闭 lint 检查时，注明具体规则和原因，并将范围限制在必要的代码行。
- 简单泛型使用 `T`, `U`, `K` 等单个大写字母。当泛型本身较为复杂，或在函数/类型中同时使用多个泛型参数时，应为所有泛型使用有意义的 PascalCase 名称。

### 未提供的值与空值

省略可选属性表示未提供该值，`null` 表示明确设为空值；查找不到结果时默认返回 `undefined`。

```typescript
interface UpdateUserInput {
  displayName?: string | null;
}

const unchanged: UpdateUserInput = {};
const cleared: UpdateUserInput = { displayName: null };
const renamed: UpdateUserInput = { displayName: "Luke" };
```

### 业务状态

同一时刻只能处于一种状态的业务数据，使用联合类型，并通过 `status` 等字段区分状态；处理状态的分支覆盖所有可能的状态。

```typescript
type LoadState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };
```

## 枚举

枚举统一使用 `as const` 对象，并从对象的值生成对应的联合类型。键使用 UPPER_SNAKE_CASE。枚举对应的文案等映射表使用 `satisfies Record<枚举类型, 值类型>`，确保每个枚举值都有对应项。

```typescript
export const USER_STATUS = {
  ACTIVE: "ACTIVE",
  INACTIVE: "INACTIVE",
} as const;
export type UserStatus = (typeof USER_STATUS)[keyof typeof USER_STATUS];

export const USER_STATUS_LABELS = {
  ACTIVE: "启用",
  INACTIVE: "停用",
} satisfies Record<UserStatus, string>;
```

## 循环

顺序执行、提前退出或包含多步操作时，优先使用 `for...of`。数组转换使用 `map`，筛选使用 `filter`，查找使用 `find`，条件判断使用 `some` 或 `every`。异步迭代器使用 `for await...of`。

## 异步

- 异步调用使用 `await` 等待结果，或返回 Promise 交给调用方处理。`void` 只表示忽略返回值，错误仍需处理。
- async 函数返回 Promise 时使用 `return await`。
- 依次执行任务使用 `for...of`；并行任务需要全部成功时使用 `Promise.all`；需要收集每项任务的成功或失败结果时使用 `Promise.allSettled`；需要限制同时执行的任务数时，限制并发数量。

## 错误处理

- 根据实际失败场景添加校验与兜底；已校验的数据和已成立的类型约束直接使用。
- 本层需要恢复、转换或报告错误时使用 `catch`，其余情况让错误向调用方传播；兜底结果应符合业务语义。

## 字符串

- 多行长字符串使用字符串数组按行组织，通过 `.join("\n")` 合并。
- 将 `unknown`、`object`、`Record`、动态 JSON 等宽泛类型的值转为字符串时，先收窄类型并提取所需字段；需要完整对象内容时，使用 `JSON.stringify` 显式序列化。
