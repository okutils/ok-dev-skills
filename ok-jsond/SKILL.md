---
name: ok-jsond
description: 在 TypeScript 中使用 @okutils/json-d 为无需精确定义结构的临时 JSON 对象、动态接口数据、配置或通用 JSON 参数选择宽松类型。当任务提及 json-d、JSONObject、JSONValue、JSONScalar，或需要替换用于 JSON 数据的 any、object、宽泛 Record 类型时使用；固定业务模型和非 JSON 对象的类型设计不属于此技能范围。
---

## 使用时机

- 临时对象、动态字段、透传的接口数据或配置内容符合 JSON 值范围，且无需逐项定义字段时，使用 `@okutils/json-d`。
- 用于无需专用业务类型的 JSON 数据，不用于包含 `Date`、函数、`Map` 等非 JSON 内容的对象。

## 安装与类型选择

使用项目现有包管理器添加 `@okutils/json-d`，例如 `npm i -D @okutils/json-d`。该包仅提供类型，统一使用 `import type`：

```typescript
import type { JSONObject, JSONScalar, JSONValue } from "@okutils/json-d";
```

| 数据范围                             | 类型                                               |
| ------------------------------------ | -------------------------------------------------- |
| 顶层为对象，字段值可以是嵌套 JSON 值 | `JSONObject`                                       |
| 顶层可以是标量、对象或数组           | `JSONValue`                                        |
| 仅字符串、数字、布尔值或 `null`      | `JSONScalar`                                       |
| JSON 值数组                          | `JSONValue[]`；只读输入使用 `readonly JSONValue[]` |

`JSONValue` 接受只读数组，包括 `as const` 创建的数组。对象数组可使用 `JSONObject[]` 或 `readonly JSONObject[]`；包未导出 `JSONArray`。

## 接口返回值

接口返回 JSON 对象且无需逐项定义字段时，将请求函数的返回类型写为 `Promise<JSONObject>`：

```typescript
import type { JSONObject } from "@okutils/json-d";

const fetchUserAsync = async (userId: string): Promise<JSONObject> => {
  const response = await fetch(`/api/users/${encodeURIComponent(userId)}`);
  return await response.json();
};

const user = await fetchUserAsync("123");
```

返回对象数组时使用 `Promise<JSONObject[]>`；返回任意 JSON 值时使用 `Promise<JSONValue>`。

## 定义与读取

使用 `JSONObject` 定义对象；字段类型为 `JSONValue`，可通过 `typeof` 等方式收窄：

```typescript
const data: JSONObject = {
  name: "Alice",
  tags: ["TypeScript"],
  profile: { city: "Shanghai" },
  avatar: null,
};

const getDisplayName = (data: JSONObject): string => {
  const name = data.name;
  return typeof name === "string" ? name.toUpperCase() : "Anonymous";
};

const serializeJson = (value: JSONValue): string => JSON.stringify(value);

serializeJson(data);
serializeJson([121.47, 31.23] as const);
```

## 标量参数

使用 `JSONScalar` 接收字符串、数字、布尔值或 `null`：

```typescript
const formatCell = (value: JSONScalar): string => {
  return value === null ? "—" : String(value);
};
```
