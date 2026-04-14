# 编程语言最佳实践参考

> 本文件按语言分类，每种语言包含：命名惯例、代码风格、错误处理、常用模式、性能注意点。
> 根据用户使用的语言，自动应用对应规范。

---

## 目录

- [一、系统级语言](#一系统级语言)
  - [C](#c) | [C++](#c-1) | [Rust](#rust) | [Go](#go) | [Zig](#zig) | [Nim](#nim) | [D](#d) | [Ada](#ada) | [Fortran](#fortran)
- [二、应用级语言](#二应用级语言)
  - [Java](#java) | [Kotlin](#kotlin) | [C#](#c-1) | [Swift](#swift) | [Objective-C](#objective-c) | [Dart](#dart) | [Scala](#scala) | [Groovy](#groovy)
- [三、脚本/动态语言](#三脚本动态语言)
  - [Python](#python) | [JavaScript](#javascript) | [TypeScript](#typescript) | [Ruby](#ruby) | [PHP](#php) | [Perl](#perl) | [Lua](#lua) | [R](#r) | [Julia](#julia) | [MATLAB](#matlab)
- [四、函数式语言](#四函数式语言)
  - [Haskell](#haskell) | [Clojure](#clojure) | [Elixir](#elixir) | [Erlang](#erlang) | [F#](#f) | [OCaml](#ocaml) | [Scheme](#scheme) | [Racket](#racket)
- [五、数据/查询语言](#五数据查询语言)
  - [SQL](#sql) | [GraphQL](#graphql) | [Cypher](#cypher)
- [六、标记/配置语言](#六标记配置语言)
  - [HTML/CSS](#htmlcss) | [YAML/TOML/JSON](#yamltomljson) | [XML](#xml) | [Markdown/LaTeX](#markdownlatex)
- [七、Shell/运维](#七shell运维)
  - [Bash/Zsh](#bashzsh) | [PowerShell](#powershell) | [Makefile](#makefile) | [Dockerfile](#dockerfile)
- [八、移动/跨平台](#八移动跨平台)
  - [Flutter](#flutter) | [React Native](#react-native) | [.NET MAUI](#net-maui)
- [九、领域特定语言](#九领域特定语言)
  - [Solidity](#solidity) | [VHDL/Verilog](#vhdlverilog) | [GLSL/WASM](#glslwasm)

---

## 一、系统级语言

### C

**命名惯例：** `snake_case`（函数/变量），`UPPER_SNAKE_CASE`（宏/常量），`PascalCase` 或 `Module_prefix_`（类型/结构体）
**代码风格：**
- 遵循 Linux kernel coding style 或项目既有风格
- 头文件使用 include guard（`#ifndef FOO_H`）或 `#pragma once`
- 函数长度控制在 24 行以内（Linux kernel 建议）

**错误处理：**
- 检查所有返回值，尤其是 `malloc`、`open`、`read`、`write`
- 使用 `errno` 和 `perror`/`strerror` 报告错误
- 资源获取后必须对应释放（free、close、fclose）
- 使用 `goto cleanup` 模式处理多资源清理

**内存安全：**
- `malloc` 后检查是否为 `NULL`
- `free` 后将指针置为 `NULL`（避免悬垂指针）
- 使用 `sizeof(*ptr)` 而非 `sizeof(Type)` 避免类型不一致
- `memcpy` 注意目标缓冲区大小
- 字符串操作使用 `strncpy`/`snprintf` 而非 `strcpy`/`sprintf`

**常用模式：**
- RAII 不可用，手动管理资源
- 回调函数 + `void*` userdata 实现泛型
- 位域和联合体用于紧凑数据表示

---

### C++

**命名惯例：** `PascalCase`（类/结构体/命名空间），`camelCase`（函数/变量），`UPPER_SNAKE_CASE`（常量/宏），`m_` 或 `_` 前缀（成员变量，视项目而定）
**代码风格：**
- 遵循 Google C++ Style Guide、LLVM Coding Standards 或项目既有风格
- 头文件使用 `#pragma once`
- 优先使用 `const`、`constexpr`、`auto`

**现代 C++（C++17/20/23）特性优先：**
- 智能指针（`unique_ptr`/`shared_ptr`）替代裸 `new`/`delete`
- `std::optional` 替代特殊返回值表示"无结果"
- `std::variant` 替代联合体
- `std::string_view` 替代 `const std::string&`
- 范围 `for`、结构化绑定、`if constexpr`
- Concepts 替代 SFINAE（C++20+）
- Ranges 库（C++20+）

**错误处理：**
- 构造函数失败抛异常，其他情况视场景选择异常或 `std::expected`（C++23）
- 析构函数不抛异常
- 使用 `[[nodiscard]]` 标注返回值必须检查的函数

**性能注意：**
- 移动语义（`std::move`）避免不必要的拷贝
- 谨慎使用 `std::shared_ptr`（有引用计数开销）
- `reserve`/`resize` 预分配容器大小
- 热路径避免动态内存分配

---

### Rust

**命名惯例：** `snake_case`（函数/变量/模块），`PascalCase`（类型/traits），`UPPER_SNAKE_CASE`（常量/静态变量）
**代码风格：**
- `rustfmt` 格式化，`clippy` 检查
- 优先使用 `impl Trait` 而非具体类型（泛型）
- 错误类型使用 `thiserror` 定义，错误处理用 `anyhow`

**所有权与借用：**
- 理解所有权规则：每个值只有一个 owner
- 借用检查器是你的朋友，不要用 `unsafe` 绕过（除非绝对必要且有充分注释）
- 生命周期标注：函数签名需要时标注，结构体字段需要时标注
- `Clone` vs `Copy`：小类型实现 `Copy`，大类型用 `Clone`

**错误处理：**
- 使用 `Result<T, E>` 而非 `unwrap()`（生产代码）
- 使用 `?` 操作符传播错误
- 自定义错误类型实现 `std::error::Error`
- `unwrap()`/`expect()` 仅在测试或确定不会失败的代码中使用

**常用模式：**
- `Iterator` + `Option`/`Result` 组合器链式处理
- `enum` + `match` 替代继承
- `trait` 替代接口
- `Arc<Mutex<T>>` 或 `Arc<RwLock<T>>` 共享可变状态
- `serde` 进行序列化/反序列化

**性能注意：**
- 零成本抽象：高阶抽象不应有运行时开销
- `Cow<str>` 延迟克隆
- `Box<T>` 堆分配大对象或递归类型
- `#[inline]` 谨慎使用，编译器通常比人更懂内联

---

### Go

**命名惯例：** `PascalCase`（导出），`camelCase`（未导出），接口名用 `-er` 后缀（`Reader`、`Writer`）
**代码风格：**
- `gofmt`/`goimports` 格式化
- 一个目录一个包，避免循环依赖
- 接口小而精（1-3 个方法）
- 接受接口、返回结构体

**错误处理：**
- 错误是值，不使用异常
- `if err != nil` 是常态，不要忽略
- 错误信息要有意义，包含上下文（`fmt.Errorf("doX: %w", err)`）
- 使用 `errors.Is`/`errors.As` 检查错误
- `panic` 仅用于不可恢复的错误（程序初始化失败）

**并发：**
- `goroutine` 轻量并发，`channel` 通信
- `sync.WaitGroup` 等待一组 goroutine
- `sync.RWMutex` 保护共享状态（读多写少场景）
- `context.Context` 传递超时、取消信号
- `select` 处理多 channel 操作

**常用模式：**
- 表驱动测试（table-driven tests）
- 函数多返回值（value, error）
- `defer` 确保资源释放
- 接口隐式实现

---

### Zig

**命名惯例：** `camelCase`（函数/变量），`PascalCase`（类型）
**代码风格：**
- 无隐式行为，所有控制流显式
- `comptime` 编译期计算
- 使用 `defer`/`errdefer` 管理资源
- `alloc` 必须显式传入 allocator

**错误处理：**
- 错误是值，使用 `!` 操作符 unwrap 或 `try` 传播
- 错误集（error set）类似枚举
- `errdefer` 确保错误时资源释放

---

### Nim

**命名惯例：** `snake_case`（过程/变量），`PascalCase`（类型），`UPPER_SNAKE_CASE`（常量）
**代码风格：**
- 缩进敏感（类似 Python）
- `result` 变量自动返回
- `defer` 管理资源
- `template`/`macro` 元编程

**错误处理：**
- `try`/`except` 异常处理
- `Option[T]` 和 `Result[T, E]` 可选
- `raises` 标注可能抛出的异常

---

### D

**命名惯例：** `camelCase`（函数/变量），`PascalCase`（类/结构体），`UPPER_SNAKE_CASE`（常量）
**代码风格：**
- 遵循 Phobos 标准库风格
- `@safe` 标注内存安全代码
- `immutable`/`const` 广泛使用
- `scope`/`ref` 管理资源

**错误处理：**
- `Exception` 类异常
- `assert`/`enforce` 前置条件检查
- `std.exception.assumeWontThrow` 标注不抛异常的代码

---

### Ada

**命名惯例：** `PascalCase`（类型/包），`snake_case`（过程/变量）
**代码风格：**
- 强类型系统，充分利用 range 和 subtype
- `with`/`use` 引入包
- `pragma` 指导编译器优化

**错误处理：**
- 异常机制（`raise`/`exception when`）
- 前置/后置条件用 `pragma Assert`
- `SPARK` 子集用于形式化验证

---

### Fortran

**命名惯例：** `snake_case` 或 `UPPERCASE`（传统风格）
**代码风格：**
- 优先使用 Fortran 2003+ 特性（面向对象、协程）
- `implicit none` 必须使用
- 模块化组织代码

**性能注意：**
- 数组操作向量化
- `contiguous` 属性帮助优化
- 避免在循环中分配/释放内存

---

## 二、应用级语言

### Java

**命名惯例：** `PascalCase`（类/接口），`camelCase`（方法/变量），`UPPER_SNAKE_CASE`（常量），`package.snake_case`（包名）
**代码风格：**
- 遵循 Google Java Style Guide 或项目既有风格
- 使用 `final` 标注不可变变量
- 优先使用 record（Java 16+）、sealed class（Java 17+）
- Stream API 处理集合操作

**错误处理：**
- 检查异常（checked）vs 非检查异常（unchecked）合理使用
- 不要吞异常（空 catch 块）
- 使用 try-with-resources 管理资源
- 自定义异常类继承 `RuntimeException` 或 `Exception`

**性能注意：**
- 字符串拼接用 `StringBuilder`（非循环中 `+` 也可以，编译器优化）
- 对象池谨慎使用（现代 GC 下通常不需要）
- `ArrayList` 优于 `LinkedList`（大多数场景）
- `HashMap` 初始容量设置避免扩容

---

### Kotlin

**命名惯例：** 与 Java 一致（`PascalCase`/`camelCase`/`UPPER_SNAKE_CASE`）
**代码风格：**
- 遵循 Kotlin Coding Conventions
- 优先使用不可变（`val`/`immutable`）
- data class 替代 Java POJO
- sealed class/interface 替代枚举+继承
- 扩展函数谨慎使用，避免滥用
- 协程替代线程（`kotlinx.coroutines`）

**错误处理：**
- `Result<T>` 封装可能失败的操作
- `?.`、`?:`、`let` 处理空值
- `require`/`check`/`error` 前置条件
- `runCatching` 捕获异常并转为 Result

---

### C#

**命名惯例：** `PascalCase`（类/方法/属性/命名空间），`camelCase`（局部变量/参数），`_camelCase`（私有字段），`IPascalCase`（接口）
**代码风格：**
- 遵循 .NET Coding Guidelines
- 使用 `var`（类型可推断时）、`nameof()`、`$""` 字符串插值
- record 类型（C# 9+）替代简单类
- pattern matching（C# 7+）简化条件逻辑
- nullable reference types（C# 8+）标注空值

**错误处理：**
- 异常处理，自定义异常继承 `Exception`
- `try`/`catch`/`finally`，`using`/`using var` 管理资源
- `ArgumentNullException` 等标准异常
- `Result<T>` 模式可选

**性能注意：**
- `Span<T>`/`Memory<T>` 减少分配
- `ValueTask` 替代 `Task`（无 await 的热路径）
- `struct` 用于小值类型
- `StringBuilder`/`string.Create` 处理大量字符串

---

### Swift

**命名惯例：** `camelCase`（函数/变量/枚举case），`PascalCase`（类/结构体/协议/枚举）
**代码风格：**
- 遵循 Swift API Design Guidelines
- 值类型（struct）优先于引用类型（class）
- `guard let` 提前退出
- `protocol` + extension 实现默认行为
- `@escaping`/`@Sendable` 标注闭包

**错误处理：**
- `do`/`try`/`catch` 异常处理
- `Result<Success, Failure>` 可选
- `throws` 标注可能失败的方法
- `try?` 忽略错误（谨慎使用）

**性能注意：**
- `@inlinable` 标注性能关键代码
- `withUnsafeBytes` 处理大数据
- Copy-on-Write 优化（Array、String 等标准库类型已内置）

---

### Objective-C

**命名惯例：** `camelCase`（方法/变量），`PascalCase`（类名带前缀如 `NS`、`UI`）
**代码风格：**
- 遵循 Apple Coding Guidelines
- ARC 管理内存，避免 retain cycle（`__weak`/`__strong`）
- 使用 `nil` 检查，消息发送给 `nil` 不崩溃
- `@property` 属性修饰符（`strong`/`weak`/`copy`/`assign`）

**错误处理：**
- `NSError` 指针参数模式
- `@try`/`@catch`/`@finally` 异常（仅用于严重错误）
- `NSAssert` 调试断言

---

### Dart

**命名惯例：** `camelCase`（函数/变量），`PascalCase`（类/枚举/类型），`lowercase_with_underscores`（文件名/库前缀）
**代码风格：**
- 遵循 Effective Dart
- `final` 广泛使用
- `late` 延迟初始化
- 命名构造函数（`Person.fromJson`）
- extension methods

**错误处理：**
- `try`/`catch`/`finally`
- `Future<T>` + `async`/`await` 异步
- `Error`/`Exception` 区分

---

### Scala

**命名惯例：** `PascalCase`（类/对象/trait），`camelCase`（方法/变量），`UPPER_SNAKE_CASE`（常量）
**代码风格：**
- 函数式风格优先（不可变、纯函数）
- `case class`/`sealed trait` 模式匹配
- `Option`/`Either`/`Try` 替代 null 和异常
- `implicit`/`given`（Scala 3）谨慎使用

**错误处理：**
- `Either[Error, T]` 或 `Try[T]` 封装可能失败的操作
- `Option[T]` 替代 null
- `match` 全面匹配

---

### Groovy

**命名惯例：** 与 Java 一致
**代码风格：**
- 利用 Groovy 动态特性简化代码
- GString（`""`）替代字符串拼接
- `?.` 安全导航，`*.` 展开操作符
- 闭包（`{ -> }`）广泛使用
- `@Grab` 依赖管理（脚本场景）

---

## 三、脚本/动态语言

### Python

**命名惯例：** `snake_case`（函数/变量/模块），`PascalCase`（类），`UPPER_SNAKE_CASE`（常量）
**代码风格：**
- 遵循 PEP 8，使用 `ruff` 或 `black` 格式化
- type hints（`int`、`str`、`list[int]`、`Optional[T]`）
- f-string 替代 `.format()` 和 `%`
- `pathlib.Path` 替代 `os.path`
- `dataclass`/`pydantic` 替代手写 `__init__`
- `asyncio` 替代 `threading`（I/O 密集型）

**错误处理：**
- 具体异常类型，避免裸 `except:`
- `raise ... from e` 异常链
- `contextlib` 管理资源
- `logging` 替代 `print`

**性能注意：**
- 列表推导式优于 for 循环 + append
- `set`/`dict` 查找 O(1)，`list` 查找 O(n)
- `__slots__` 减少内存
- `mmap` 处理大文件
- `functools.lru_cache` 缓存

**包管理：** `pyproject.toml` + `uv`/`pip`，虚拟环境隔离

---

### JavaScript

**命名惯例：** `camelCase`（变量/函数），`PascalCase`（类/构造函数），`UPPER_SNAKE_CASE`（常量）
**代码风格：**
- 使用现代语法（ES2022+）
- `const`/`let`，不用 `var`
- 箭头函数（注意 `this` 绑定差异）
- 模板字符串（`` ` ``）
- 解构赋值、展开运算符
- `Array.isArray()` 检查数组

**异步：**
- `async`/`await` 替代回调/Promise 链
- `Promise.allSettled` 替代 `Promise.all`（需要全部结果时）
- `AbortController` 取消请求

**错误处理：**
- `try`/`catch`/`finally`
- 可选链 `?.` 和空值合并 `??`
- `Error` 子类自定义错误

**Node.js 特定：**
- `fs/promises` 异步文件操作
- `stream` 处理大文件
- `worker_threads` CPU 密集任务

---

### TypeScript

**命名惯例：** 与 JavaScript 一致
**代码风格：**
- 优先 TypeScript，类型定义完整
- `interface` 定义对象形状，`type` 定义联合/交叉类型
- 泛型使用合理，不过度泛化
- `as const` 满足类型常量
- `satisfies` 操作符（TS 4.9+）
- `strict: true` 启用所有严格检查

**类型技巧：**
- `unknown` 替代 `any`
- `never` 标注不可达分支
- `Readonly<T>`/`Partial<T>`/`Required<T>` 工具类型
- 条件类型和映射类型处理复杂场景
- `zod`/`valibot` 运行时校验

---

### Ruby

**命名惯例：** `snake_case`（方法/变量），`PascalCase`（类/模块），`UPPER_SNAKE_CASE`（常量），`?` 后缀（布尔方法），`!` 后缀（危险方法）
**代码风格：**
- 遵循 Ruby Style Guide
- `do`/`end` 多行块，`{}` 单行块
- 符号（`:name`）替代字符串作为 key
- `freeze` 冻结不可变对象

**错误处理：**
- `begin`/`rescue`/`ensure`/`end`
- 具体异常类，避免裸 `rescue`
- 自定义异常继承 `StandardError`

**常用模式：**
- Duck typing
- `module` mixin
- `block`/`yield` 回调
- `Enumerable` 组合操作

---

### PHP

**命名惯例：** `camelCase`（方法/变量），`PascalCase`（类/接口/特性）
**代码风格：**
- 遵循 PSR-12 编码标准
- 严格类型模式（`declare(strict_types=1)`）
- 命名空间（`namespace`）
- Composer 依赖管理

**错误处理：**
- `try`/`catch`/`finally`
- 自定义异常类
- `Throwable` 接口捕获所有错误

**安全注意：**
- 参数化查询防止 SQL 注入
- `htmlspecialchars` 防止 XSS
- CSRF token 验证
- 密码使用 `password_hash`/`password_verify`

---

### Perl

**命名惯例：** `snake_case`（变量/函数），`PascalCase`（包/类）
**代码风格：**
- `use strict; use warnings;` 必须启用
- `use v5.30+` 启用现代特性
- `my` 声明词法变量

**错误处理：**
- `eval { ... }; if ($@) { ... }`
- `Try::Tiny` 模块更安全
- `Carp` 模块报告错误（`croak`/`confess`）

---

### Lua

**命名惯例：** `snake_case`（变量/函数），`PascalCase`（模块/类）
**代码风格：**
- `local` 声明所有变量
- 表（table）是核心数据结构
- `pcall`/`xpcall` 错误处理

**性能注意：**
- 预分配表大小（`{n=100}`）
- 避免频繁字符串拼接
- `local` 缓存全局变量/函数引用

---

### R

**命名惯例：** `snake_case`（函数/变量），`PascalCase`（S3/S4 类）
**代码风格：**
- 遵循 tidyverse 风格指南
- `magrittr` 管道 `%>%` 或原生 `|>`（R 4.1+）
- 向量化操作替代循环
- `tibble` 替代 `data.frame`

**性能注意：**
- `data.table` 处理大数据
- `Rcpp` 集成 C++ 热路径
- 预分配向量大小

---

### Julia

**命名惯例：** `snake_case`（函数/变量），`PascalCase`（类型/模块）
**代码风格：**
- 多分派（multiple dispatch）是核心
- 类型注解帮助编译器优化
- `!` 后缀表示原地修改
- `@time`/`@benchmarked` 性能测量

**性能注意：**
- 类型稳定（type stable）代码更快
- 避免全局变量
- `@inbounds`/`@simd` 微优化
- 预分配数组

---

### MATLAB

**命名惯例：** `camelCase`（函数/变量），`UPPERCASE`（常量）
**代码风格：**
- 向量化操作替代循环
- 预分配数组大小
- 使用 `~` 忽略不需要的输出

---

## 四、函数式语言

### Haskell

**命名惯例：** `camelCase`（函数/变量），`PascalCase`（类型/类/模块）
**代码风格：**
- 类型签名必须标注
- 纯函数优先，`IO` monad 隔离副作用
- 模式匹配替代嵌套 `if`
- `where`/`let` 组织局部定义
- `newtype` 替代 `data`（无运行时开销）

**错误处理：**
- `Maybe`/`Either`/`ExceptT` 替代异常
- `error`/`undefined` 仅用于不可达分支

---

### Clojure

**命名惯例：** `kebab-case`（函数/变量），`PascalCase`（协议/记录/类型）
**代码风格：**
- 不可变数据结构
- `->>`/`->` 线程宏
- `spec` 数据验证
- `core.async` 并发

**错误处理：**
- `try`/`catch`/`finally`
- `ex-info`/`ex-data` 结构化异常

---

### Elixir

**命名惯例：** `snake_case`（函数/变量/原子），`PascalCase`（模块）
**代码风格：**
- 模式匹配无处不在
- `pipe` 操作符 `|>` 链式调用
- `GenServer`/`Supervisor` OTP 行为
- `ExUnit` 测试

**错误处理：**
- `{:ok, result}` / `{:error, reason}` 元组模式
- `try`/`rescue`/`after`
- `raise`/`throw`/`exit` 三种异常级别

---

### Erlang

**命名惯例：** `snake_case`（函数/变量），`UPPER_SNAKE_CASE`（宏/常量）
**代码风格：**
- 模式匹配
- `receive` 处理消息
- OTP 行为（`gen_server`/`supervisor`）
- 热代码升级

**错误处理：**
- `{'EXIT', ...}` 链接进程退出信号
- `try`/`catch`
- `let it crash` 哲学

---

### F#

**命名惯例：** `camelCase`（函数/值/参数），`PascalCase`（类型/模块/命名空间）
**代码风格：**
- 函数式优先，避免 `mutable`
- 管道 `|>` 和组合 `>>`
- 计算表达式（computation expressions）
- 类型提供者（type providers）

**错误处理：**
- `Option<'T>` / `Result<'T, 'TError>`
- `Choice<'T1, 'T2, ...>`
- `failwithf` 严重错误

---

### OCaml

**命名惯例：** `snake_case`（值/函数），`PascalCase`（类型/模块）
**代码风格：**
- 模式匹配
- `Option` 类型
- 函数柯里化
- `Jane Street Core` 或标准库

---

### Scheme

**命名惯例：** `kebab-case`（函数/变量）
**代码风格：**
- S 表达式，括号匹配
- `define`/`lambda`
- 尾递归优化
- `cond`/`case`/`match`

---

### Racket

**命名惯例：** `kebab-case`（函数/变量）
**代码风格：**
- `#lang racket`
- `match` 模式匹配
- `contract` 系统契约式编程
- `require` 模块化

---

## 五、数据/查询语言

### SQL

**代码风格：**
- 关键字大写（`SELECT`、`FROM`、`WHERE`），表名/列名小写
- 缩进对齐提高可读性
- 参数化查询防止 SQL 注入
- CTE（`WITH` 子句）组织复杂查询

**性能注意：**
- `EXPLAIN ANALYZE` 分析执行计划
- 合适的索引策略
- 避免 `SELECT *`
- 分页查询（`LIMIT`/`OFFSET` 或 keyset pagination）
- 批量操作替代循环单条

**安全：**
- 永远使用参数化查询
- 最小权限原则
- 存储过程/视图限制直接表访问

---

### GraphQL

**代码风格：**
- PascalCase（类型/字段），camelCase（输入/参数）
- Schema-first 设计
- 描述（description）标注每个类型和字段

**性能注意：**
- N+1 问题使用 DataLoader
- 查询复杂度分析
- 分页（cursor-based 优于 offset-based）

---

### Cypher

**代码风格：**
- 节点标签 PascalCase，关系类型 UPPER_SNAKE_CASE
- 参数化查询（`$param`）
- `PROFILE`/`EXPLAIN` 分析性能

---

## 六、标记/配置语言

### HTML/CSS

**HTML：**
- 语义化标签（`<header>`/`<nav>`/`<main>`/`<article>`/`<footer>`）
- 无障碍（ARIA 属性、alt 文本）
- SEO 友好（meta 标签、结构化数据）

**CSS：**
- BEM 或 Utility-First（Tailwind）命名
- CSS 变量（`--var`）管理主题
- 响应式设计（media query / container query）
- `rem`/`em` 替代 `px`
- `will-change`/`transform` 优化动画

---

### YAML/TOML/JSON

**YAML：**
- 一致缩进（2空格）
- 列表用 `- `，映射用 `key: value`
- 复杂值用引号包裹
- 锚点（`&`）和别名（`*`）复用

**TOML：**
- 适合配置文件
- `[section]`/`[[array]]` 语法

**JSON：**
- 标准 JSON 格式
- 注释用 JSONC 或 JSON5 扩展

---

### XML

- UTF-8 编码声明
- 合理使用属性 vs 子元素
- XSD/DTD 验证
- XPath 查询

---

### Markdown/LaTeX

**Markdown：**
- 标题层级清晰
- 代码块标注语言
- 链接和图片使用相对路径

**LaTeX：**
- 使用 `\begin{...}`/`\end{...}` 环境
- 数学模式用 `$...$` 或 `$$...$$`
- `\usepackage` 按需引入

---

## 七、Shell/运维

### Bash/Zsh

**代码风格：**
- `set -euo pipefail`（Bash）
- 变量引用 `"${var}"` 防止 word splitting
- `[[ ]]` 替代 `[ ]`
- `printf` 替代 `echo`（复杂输出）
- `shellcheck` 静态检查

**错误处理：**
- 检查命令返回值
- `trap cleanup EXIT` 确保清理
- `set -e` 遇错即停

**可移植性：**
- POSIX 兼容 vs Bash 特性
- `#!/usr/bin/env bash` shebang
- 避免平台特定命令

---

### PowerShell

**命名惯例：** `Verb-Noun`（cmdlet），`camelCase`（变量），`PascalCase`（函数）
**代码风格：**
- `Set-StrictMode -Version Latest`
- `try`/`catch`/`finally`
- 管道（`|`）操作
- `ShouldProcess` 确认提示

---

### Makefile

- 使用 `.PHONY` 标注伪目标
- Tab 缩进（不是空格）
- 变量管理（`$(VAR)`）
- 并行构建（`-j`）

---

### Dockerfile

- 多阶段构建减小镜像
- 合理利用缓存（少变层在前）
- 非 root 用户运行
- `.dockerignore` 排除不必要文件
- 精简基础镜像（alpine/slim）

---

## 八、移动/跨平台

### Flutter (Dart)

- Widget 组合模式
- `const` 构造函数优化重建
- `Provider`/`Riverpod` 状态管理
- `flutter_lints` 代码规范

---

### React Native (JS/TS)

- 组件化设计
- `StyleSheet.create` 样式
- `FlatList`/`SectionList` 列表性能
- 原生模块桥接谨慎使用

---

### .NET MAUI (C#)

- MVVM 模式
- `CommunityToolkit.Mvvm` 简化
- 平台特定代码用 `#if`
- XAML 热重载

---

## 九、领域特定语言

### Solidity

**命名惯例：** `camelCase`（函数/变量），`PascalCase`（合约/事件）
**安全注意：**
- 重入攻击防护（`ReentrancyGuard`）
- 整数溢出（Solidity 0.8+ 内置检查）
- 访问控制（`onlyOwner`）
- 外部调用检查（`check-effects-interactions` 模式）

---

### VHDL/Verilog

- 信号（signal）vs 变量（variable）区分
- 时钟域交叉注意
- 同步复位 vs 异步复位
- 仿真测试平台（testbench）

---

### GLSL/WASM

**GLSL：**
- 精度限定符（`highp`/`mediump`/`lowp`）
- 向量化操作
- 纹理采样注意 mipmap

**WASM：**
- 线性内存模型
- 导入/导出接口
- `wasm-bindgen` 与 JS 互操作
