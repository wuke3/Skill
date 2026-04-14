# 调试方法论参考

> 系统化的调试流程和常见错误模式速查。
> 当用户报告 bug 或错误时，按照本文件的方法论进行分析和修复。

---

## 目录

- [一、调试黄金流程](#一调试黄金流程)
- [二、错误分类与定位](#二错误分类与定位)
- [三、常见错误模式速查](#三常见错误模式速查)
- [四、调试工具与技巧](#四调试工具与技巧)
- [五、预防性编程](#五预防性编程)

---

## 一、调试黄金流程

```
┌─────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ 1.复现问题   │ →  │ 2.收集信息    │ →  │ 3.定位根因    │ →  │ 4.修复验证    │ →  │ 5.预防复发    │
│             │    │              │    │              │    │              │    │              │
│ 什么条件触发？│    │ 错误信息/日志 │    │ 缩小范围     │    │ 最小修改     │    │ 加测试/检查   │
│ 稳定复现？   │    │ 堆栈跟踪     │    │ 二分法排除   │    │ 验证修复     │    │ 文档记录     │
│ 环境信息？   │    │ 输入/输出    │    │ 假设→验证    │    │ 回归测试     │    │ Code Review  │
└─────────────┘    └──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

### 1.1 复现问题
- 最小化复现步骤（去掉无关步骤）
- 确定是必现还是偶现
- 记录环境信息（OS、版本、依赖、配置）
- 检查是否是环境问题（权限、网络、磁盘空间）

### 1.2 收集信息
- **错误信息**：完整复制，不要省略
- **堆栈跟踪**：从上往下读，找到自己代码的第一行
- **日志**：查看相关时间段的日志
- **输入/输出**：记录触发问题的输入和实际输出

### 1.3 定位根因
- **二分法**：注释掉一半代码，看问题是否还在
- **打印法**：在关键位置打印变量值
- **假设→验证**：提出假设，用证据验证或推翻
- ** rubber ducking**：向别人解释代码，经常自己发现 bug

### 1.4 修复验证
- 最小化修改（不要"顺便"重构）
- 修复后验证原问题已解决
- 检查是否引入新问题（回归测试）
- 确认边界情况也修复了

### 1.5 预防复发
- 添加测试用例覆盖此 bug
- Code Review 让同事检查
- 如果是系统性问题，考虑改进流程

---

## 二、错误分类与定位

### 2.1 按错误类型分类

| 类型 | 常见表现 | 定位方法 |
|---|---|---|
| **语法错误** | 编译/解析失败 | 查看错误行号和提示 |
| **类型错误** | 类型不匹配、undefined 属性 | 类型检查器、IDE 提示 |
| **空值错误** | NullPointerException / undefined | 检查变量是否可能为空 |
| **逻辑错误** | 结果不符合预期 | 断点调试、日志、单元测试 |
| **并发错误** | 竞态条件、死锁 | 压力测试、竞态检测工具 |
| **资源错误** | 内存泄漏、连接泄漏、文件未关闭 | 资源监控、linter |
| **性能问题** | 响应慢、内存占用高 | Profiler、基准测试 |
| **安全漏洞** | 注入、XSS、越权 | 安全扫描、渗透测试 |

### 2.2 按错误信息关键词快速定位

| 关键词 | 可能原因 | 排查方向 |
|---|---|---|
| `Cannot read properties of undefined` | 访问 undefined/null 的属性 | 检查对象是否存在、API 返回值结构 |
| `TypeError: X is not a function` | 变量不是函数 | 检查变量类型、import 是否正确 |
| `null pointer` / `NullPointerException` | 空指针访问 | 检查所有可能为 null 的变量 |
| `ECONNREFUSED` | 连接被拒绝 | 检查服务是否启动、端口是否正确 |
| `ETIMEDOUT` | 连接超时 | 检查网络、防火墙、服务响应时间 |
| `permission denied` | 权限不足 | 检查文件权限、用户权限 |
| `out of memory` | 内存不足 | 检查内存泄漏、增加内存限制 |
| `stack overflow` | 栈溢出 | 检查无限递归、过深的调用链 |
| `module not found` | 模块未安装/路径错误 | 检查 import 路径、安装依赖 |
| `syntax error` | 语法错误 | 查看错误行号、检查括号/引号匹配 |
| `segmentation fault` | 非法内存访问 | 检查指针、数组越界、use-after-free |
| `deadlock` | 死锁 | 检查锁的获取顺序、超时设置 |

---

## 三、常见错误模式速查

### 3.1 空值/未定义相关

**模式：** 对可能为 null/undefined 的值直接调用方法

```javascript
// ❌ 错误
const name = user.name;  // user 可能是 undefined

// ✅ 修复
const name = user?.name ?? "default";
```

```python
# ❌ 错误
name = data["name"]  # data 可能没有 "name" key

# ✅ 修复
name = data.get("name", "default")
```

**模式：** API 返回值结构与预期不符

```javascript
// ❌ 错误 —— 直接使用返回值
const users = await db.query('SELECT * FROM users');

// ✅ 修复 —— 根据驱动正确解构
const [users] = await db.query('SELECT * FROM users');  // mysql2
const { rows: users } = await db.query('SELECT * FROM users');  // pg
```

### 3.2 异步相关

**模式：** 忘记 await

```javascript
// ❌ 错误 —— result 是 Promise 而非实际值
const result = fetchData();

// ✅ 修复
const result = await fetchData();
```

**模式：** 竞态条件

```javascript
// ❌ 错误 —— 多次调用可能返回旧数据
let cache;
async function getData() {
    if (!cache) cache = await fetch();
    return cache;
}

// ✅ 修复 —— 用 Promise 去重
let pending;
async function getData() {
    if (cache) return cache;
    if (!pending) pending = fetch().then(d => { cache = d; return d; });
    return pending;
}
```

**模式：** 资源未释放

```python
# ❌ 错误 —— 文件可能不会关闭（异常时）
f = open("data.txt")
data = f.read()

# ✅ 修复
with open("data.txt") as f:
    data = f.read()
```

```go
// ❌ 错误 —— 响应体未关闭
resp, _ := http.Get(url)
body, _ := io.ReadAll(resp.Body)

// ✅ 修复
resp, err := http.Get(url)
if err != nil { return err }
defer resp.Body.Close()
body, err := io.ReadAll(resp.Body)
```

### 3.3 并发相关

**模式：** 数据竞争

```go
// ❌ 错误 —— 并发读写 map
m := make(map[string]int)
go func() { m["a"] = 1 }()
go func() { _ = m["a"] }()  // data race!

// ✅ 修复
var mu sync.RWMutex
m := make(map[string]int)
go func() {
    mu.Lock()
    m["a"] = 1
    mu.Unlock()
}()
go func() {
    mu.RLock()
    _ = m["a"]
    mu.RUnlock()
}()
```

**模式：** 死锁

```java
// ❌ 错误 —— 锁顺序不一致导致死锁
// Thread 1: lock(A) -> lock(B)
// Thread 2: lock(B) -> lock(A)

// ✅ 修复 —— 统一锁顺序
// 所有线程都先 lock(A) 再 lock(B)
```

### 3.4 类型/转换相关

**模式：** 隐式类型转换

```javascript
// ❌ 陷阱
"5" + 3     // "53"（字符串拼接）
"5" - 3     // 2（数字减法）
"5" == 5    // true（宽松比较）
"5" === 5   // false（严格比较）

// ✅ 修复 —— 显式转换
Number("5") + 3  // 8
```

**模式：** 整数溢出

```python
# ❌ 可能溢出
result = a * b * c  # 大数可能溢出

# ✅ 修复
import math
result = math.prod([a, b, c])
```

### 3.5 安全相关

**模式：** SQL 注入

```python
# ❌ 危险 —— 字符串拼接 SQL
cursor.execute(f"SELECT * FROM users WHERE name = '{name}'")

# ✅ 修复 —— 参数化查询
cursor.execute("SELECT * FROM users WHERE name = %s", (name,))
```

**模式：** XSS

```javascript
// ❌ 危险 —— 直接插入 HTML
element.innerHTML = userInput;

// ✅ 修复
element.textContent = userInput;
// 或使用 DOMPurify 清理
```

---

## 四、调试工具与技巧

### 4.1 通用技巧

| 技巧 | 说明 |
|---|---|
| **二分法** | 注释/删除一半代码，缩小问题范围 |
| **打印法** | 在关键位置打印变量值和状态 |
| **断点调试** | IDE 断点、条件断点、日志断点 |
| **最小复现** | 剥离无关代码，创建最小可复现示例 |
| **Rubber Ducking** | 向别人解释代码，经常自己发现 bug |
| **换视角** | 休息一下再看，或让同事帮忙看 |

### 4.2 语言特定调试工具

| 语言 | 调试器 | Profiler | 静态分析 |
|---|---|---|---|
| Python | `pdb`/`ipdb`、VS Code debugger | `cProfile`、`py-spy` | `ruff`、`mypy`、`pyright` |
| JavaScript/TS | Chrome DevTools、VS Code debugger | Chrome Performance、`clinic` | `ESLint`、`TypeScript` |
| Go | `dlv` | `pprof` | `go vet`、`staticcheck` |
| Rust | `rust-lldb`/`rust-gdb` | `perf`、`flamegraph` | `clippy`、`cargo check` |
| Java | IntelliJ debugger、jdb | VisualVM、JFR | `spotbugs`、`checkstyle` |
| C/C++ | GDB、LLDB | Valgrind、perf | `clang-tidy`、`cppcheck` |
| C# | Visual Studio debugger | dotnet-trace | Roslyn Analyzers |

### 4.3 日志最佳实践

```
[时间] [级别] [模块] 消息 {上下文数据}

# 好的日志
[2024-01-15 10:30:00] [ERROR] [AuthService] Login failed {"user_id": 1234, "reason": "invalid_password", "ip": "1.2.3.4"}

# 差的日志
error: login failed
```

**日志级别：**
- `TRACE`：最详细的调试信息
- `DEBUG`：开发调试信息
- `INFO`：关键业务流程节点
- `WARN`：潜在问题，不影响主流程
- `ERROR`：错误，需要关注
- `FATAL`：致命错误，服务不可用

---

## 五、预防性编程

### 5.1 编码时预防

- **输入验证**：对所有外部输入做校验（用户输入、API 响应、文件内容、环境变量）
- **类型安全**：利用类型系统，`unknown` > `any`，`Optional` > nullable
- **防御性拷贝**：返回不可变副本，避免外部修改内部状态
- **资源管理**：使用 RAII / `with` / `defer` / `try-with-resources`
- **前置条件检查**：函数入口检查参数合法性

### 5.2 测试预防

- **单元测试**：覆盖核心逻辑和边界情况
- **集成测试**：验证模块间交互
- **属性测试**（Property-based Testing）：自动生成大量测试用例
- **模糊测试**（Fuzz Testing）：自动生成随机输入发现崩溃
- **快照测试**：检测意外输出变化

### 5.3 流程预防

- **Code Review**：所有代码变更经过审查
- **CI/CD**：自动运行测试和静态分析
- **渐进式发布**：金丝雀发布、蓝绿部署
- **监控告警**：错误率、延迟、资源使用率监控
- **错误追踪**：Sentry、Bugsnag 等自动收集生产错误
