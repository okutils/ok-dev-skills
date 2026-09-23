---
name: ok-golang
description: 在编写、修改、重构或审查 Go 代码时使用，遵循团队的命名、代码组织、枚举、错误处理、并发及测试约定。
---

## 命名

- 参数使用完整含义的名称，如 `value`、`index`。
- 常量统一导出。
- 包名使用简短、小写、单数单词；多单词导入别名使用 camelCase，如 `jsonUtil`。
- 未使用的命名参数和未导出的包级变量使用 `_` 前缀，如 `_ctx`、`_defaultPort`；包级错误变量使用 `err` 前缀即可。
- 布尔变量、参数、字段和判断函数使用正向的 `is`、`has`、`can`、`should`、`will` 前缀，如 `isReady`、`HasItems`、`_isReady`；查询结果沿用 `ok`，实现已有接口时沿用接口方法名。

## 代码组织

- 导入固定分为标准库、其他所有包两组，组间留空行。
- 应用入口放在 `cmd/<可执行文件名>`，业务及私有共享代码放在 `internal`，公共库放在 `pkg`；按需建目录。
- `go.mod` 中的直接依赖和间接依赖分组组织。
- 一个文件一般定义一个结构体，高度相关的结构体可放在一起；相关的 `import`、`const`、`var`、`type` 声明使用分组语法。
- 构造函数紧跟类型定义；同一类型的方法放在一起，导出函数和方法优先，其余按大致调用顺序排列，工具函数放在文件末尾。
- 含义不明显的调用参数添加行内参数名注释，如 `connect(host, /* isLocal */ true)`。

## 类型与初始化

- 函数类型的参数或返回值先用 `type` 命名，如 `type Callback func(value string) error`。
- 结构体优先使用具名字段；需要暴露内层类型的方法时才使用嵌入字段，并放在顶部，与普通字段空行分隔。
- 互斥锁使用具名值字段，如 `mutex sync.Mutex`。
- 序列化字段显式写出对应标签；接口实现添加编译期断言，如 `var _ http.Handler = (*Handler)(nil)`。
- 保存传入的切片、映射或返回内部集合时使用副本；受锁保护的数据在锁内复制，嵌套可变数据按隔离需求继续复制。
- 可变状态和可替换依赖通过构造函数或实例字段传入，如注入时钟函数。
- 结构体初始化写出字段名并省略零值字段；测试中为说明含义可保留零值。零值结构体使用 `var value T`，结构体指针使用 `&T{}`。
- 空切片默认使用 `var values []T`，预留容量使用 `make([]T, 0, size)`；接口要求空数组时使用非 nil 空切片。
- 在 Printf 风格调用之外声明的格式字符串使用 `const`。
- 外部协议使用数值时长时，字段名包含单位，如 `IntervalMillis`。

## 枚举

- 用 `type` 定义枚举类型，每个常量显式赋值，数字枚举也使用具体值替代 `iota`。
- 提供 `String()` 和导出的字符串解析函数；未知值转为 `"Unknown"`，无法识别的字符串解析为 `Unknown`。

```go
type Type int

const (
    Unknown   Type = -1
    Directory Type = 0
    Item      Type = 1
)

func (value Type) String() string {
	switch value {
	case Directory:
		return "Directory"
	case Item:
		return "Item"
	default:
		return "Unknown"
	}
}

func Parse(key string) Type {
	switch key {
	case "Directory":
		return Directory
	case "Item":
		return Item
	default:
		return Unknown
	}
}
```

## 控制流与错误处理

- 分支使用提前返回或 `switch`，不使用 `else`。
- 返回错误时用 `%w` 补充出错操作，如 `fmt.Errorf("read config: %w", err)`；日志由最终处理错误的代码记录一次。
- 对外承诺的错误变量、类型及包装关系写入 API 文档并测试。
- 初始化放在显式调用的构造或启动函数中；`init` 仅用于不依赖外部状态的复杂初始化或插件注册。
- 启动、运行及 `defer` 清理放在返回 `error` 的 `run` 函数中；`os.Exit` 或 `log.Fatal` 集中在 `main`，尽量只调用一次。

## 并发

- 等待多个 goroutine 使用 `sync.WaitGroup`；等待单个 goroutine 使用由其关闭的 `chan struct{}`。
- channel 容量默认使用 0 或 1；更大容量需说明大小依据及写满后的处理方式。
- 包中的后台 goroutine 由调用方显式启动，并提供 `Close`、`Stop` 或 `Shutdown` 方法；停止时先取消再等待退出，阻塞收发应响应取消。
- 原子操作统一使用 `go.uber.org/atomic`。
