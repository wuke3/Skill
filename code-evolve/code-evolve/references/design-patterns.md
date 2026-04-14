# 设计模式参考

> 通用设计模式速查。根据用户场景自动推荐合适的设计模式。
> 按类别分组，每种模式包含：意图、适用场景、代码示例骨架、注意事项。

---

## 目录

- [创建型模式](#创建型模式)
- [结构型模式](#结构型模式)
- [行为型模式](#行为型模式)
- [并发模式](#并发模式)
- [架构模式](#架构模式)
- [函数式模式](#函数式模式)

---

## 创建型模式

### 单例模式 (Singleton)

**意图：** 确保一个类只有一个实例，并提供全局访问点。

**适用场景：** 配置管理、日志器、数据库连接池、线程池。

**注意事项：**
- 考虑是否真的需要单例 —— 全局状态使测试困难
- 多线程环境下需要双重检查锁或静态初始化
- 依赖注入通常比单例更好

**Python 示例：**
```python
class Database:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

**Go 示例（推荐 sync.Once）：**
```go
var instance *Database
var once sync.Once

func GetInstance() *Database {
    once.Do(func() {
        instance = &Database{}
    })
    return instance
}
```

---

### 工厂方法模式 (Factory Method)

**意图：** 定义创建对象的接口，让子类决定实例化哪个类。

**适用场景：** 框架中需要灵活创建对象、创建逻辑复杂需要封装。

**Python 示例：**
```python
from abc import ABC, abstractmethod

class Parser(ABC):
    @abstractmethod
    def parse(self, data: str): ...

class JSONParser(Parser):
    def parse(self, data: str): return json.loads(data)

class XMLParser(Parser):
    def parse(self, data: str): return xmltodict.parse(data)

def create_parser(format: str) -> Parser:
    parsers = {"json": JSONParser, "xml": XMLParser}
    return parsers[format]()
```

---

### 抽象工厂模式 (Abstract Factory)

**意图：** 提供创建一系列相关对象的接口，无需指定具体类。

**适用场景：** 跨平台 UI 组件、多数据库支持、多主题切换。

---

### 建造者模式 (Builder)

**意图：** 分步构建复杂对象，相同构建过程可创建不同表示。

**适用场景：** 复杂对象构造（参数多、有可选参数）、不可变对象、流畅 API。

**Python 示例：**
```python
@dataclass
class Query:
    table: str
    conditions: list[str]
    order_by: str = ""
    limit: int = 100

class QueryBuilder:
    def __init__(self, table: str):
        self._q = Query(table=table, conditions=[])

    def where(self, condition: str) -> "QueryBuilder":
        self._q.conditions.append(condition)
        return self

    def order(self, by: str) -> "QueryBuilder":
        self._q.order_by = by
        return self

    def build(self) -> Query:
        return self._q

# 使用
query = QueryBuilder("users").where("age > 18").order("name").build()
```

---

### 原型模式 (Prototype)

**意图：** 通过拷贝已有对象创建新对象，避免创建开销。

**适用场景：** 对象创建成本高、需要深拷贝。

---

## 结构型模式

### 适配器模式 (Adapter)

**意图：** 将不兼容的接口转换为客户端期望的接口。

**适用场景：** 集成第三方库、旧系统迁移、接口不兼容。

---

### 装饰器模式 (Decorator)

**意图：** 动态给对象添加职责，比继承更灵活。

**适用场景：** 日志、权限检查、缓存、压缩、加密等横切关注点。

**Python 示例（天然支持）：**
```python
def cache(func):
    _cache = {}
    def wrapper(*args):
        if args not in _cache:
            _cache[args] = func(*args)
        return _cache[args]
    return wrapper

@cache
def expensive_compute(n): ...
```

**TypeScript 示例（装饰器）：**
```typescript
function Log(target, key, descriptor) {
    const original = descriptor.value;
    descriptor.value = function(...args) {
        console.log(`Calling ${key} with`, args);
        return original.apply(this, args);
    };
}

class UserService {
    @Log
    getUser(id: number) { return db.find(id); }
}
```

---

### 代理模式 (Proxy)

**意图：** 为其他对象提供代理以控制访问。

**适用场景：** 延迟加载、访问控制、缓存代理、远程代理。

---

### 外观模式 (Facade)

**意图：** 为复杂子系统提供统一的高层接口。

**适用场景：** 简化复杂 API、封装第三方库、分层架构中的层间接口。

---

### 组合模式 (Composite)

**意图：** 将对象组合成树形结构，使单个对象和组合对象使用一致。

**适用场景：** 文件系统、UI 组件树、组织架构、表达式解析。

---

## 行为型模式

### 策略模式 (Strategy)

**意图：** 定义一系列算法，封装起来使它们可以互相替换。

**适用场景：** 多种算法可选、避免大量条件判断、排序/验证/格式化等。

**Python 示例：**
```python
class PricingStrategy(ABC):
    @abstractmethod
    def calculate(self, price: float) -> float: ...

class RegularPricing(PricingStrategy):
    def calculate(self, price): return price

class VIPPricing(PricingStrategy):
    def calculate(self, price): return price * 0.8

class Order:
    def __init__(self, strategy: PricingStrategy):
        self.strategy = strategy

    def get_total(self, price): return self.strategy.calculate(price)
```

---

### 观察者模式 (Observer)

**意图：** 定义对象间一对多的依赖，当一个对象改变状态时所有依赖者收到通知。

**适用场景：** 事件系统、消息队列、UI 数据绑定、状态管理。

---

### 命令模式 (Command)

**意图：** 将请求封装为对象，支持排队、日志、撤销操作。

**适用场景：** 撤销/重做、任务队列、宏命令、事务管理。

---

### 模板方法模式 (Template Method)

**意图：** 定义算法骨架，将某些步骤延迟到子类。

**适用场景：** 框架设计、算法不变部分复用、钩子方法。

---

### 迭代器模式 (Iterator)

**意图：** 提供一致方式遍历集合，不暴露集合内部结构。

**适用场景：** 统一遍历不同数据结构、惰性求值、分页遍历。

> 大多数现代语言已内置迭代器协议（Python `__iter__`、JS `Symbol.iterator`、Java `Iterable`、Rust `IntoIterator`）。

---

### 状态模式 (State)

**意图：** 允许对象在内部状态改变时改变行为。

**适用场景：** 状态机、订单状态流转、游戏角色状态、工作流引擎。

---

### 职责链模式 (Chain of Responsibility)

**意图：** 将请求沿处理者链传递，直到被处理。

**适用场景：** 中间件管道、日志处理、审批流程、异常处理链。

**Go 示例（中间件）：**
```go
type Handler func(http.Handler) http.Handler

func Logger(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("%s %s", r.Method, r.URL.Path)
        next.ServeHTTP(w, r)
    })
}

func Auth(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" {
            http.Error(w, "Unauthorized", 401)
            return
        }
        next.ServeHTTP(w, r)
    })
}

// 使用
mux := http.NewServeMux()
handler := Auth(Logger(mux))
```

---

## 并发模式

### 生产者-消费者 (Producer-Consumer)

**意图：** 通过缓冲队列解耦生产和消费。

**适用场景：** 任务队列、消息处理、日志系统。

**实现方式：**
- Go：`channel` + goroutine
- Python：`queue.Queue` + `threading`
- Java：`BlockingQueue`
- Rust：`crossbeam-channel` / `tokio::sync::mpsc`

---

### 读写锁模式 (Read-Write Lock)

**意图：** 允许多个读者并行，但写者独占。

**适用场景：** 读多写少的共享数据（配置、缓存）。

**实现方式：**
- Go：`sync.RWMutex`
- Java：`ReentrantReadWriteLock`
- Python：`threading.RLock`
- C++：`std::shared_mutex`

---

### Future/Promise 模式

**意图：** 表示异步操作的结果。

**适用场景：** 异步 I/O、并行计算、RPC 调用。

**实现方式：**
- JavaScript：`Promise` / `async-await`
- Python：`asyncio.Future`
- Rust：`tokio::sync::oneshot`
- Java：`CompletableFuture`
- C#：`Task<T>`

---

### Actor 模式

**意图：** 每个实体是独立 Actor，通过消息通信，无共享状态。

**适用场景：** 高并发系统、分布式系统、游戏服务器。

**实现方式：**
- Erlang/Elixir：原生 Actor
- Go：goroutine + channel
- Akka（Java/Scala）
- CAF（C++）

---

## 架构模式

### MVC (Model-View-Controller)

**适用：** Web 应用、桌面应用 UI 分层。

### MVVM (Model-View-ViewModel)

**适用：** 前端框架（Vue/React/Angular）、移动端（SwiftUI/Jetpack Compose）。

### 分层架构 (Layered Architecture)

**适用：** 大多数企业应用。表现层 → 业务层 → 数据层 → 基础设施层。

### 微服务 (Microservices)

**适用：** 大型分布式系统。服务拆分、API 网关、服务发现、配置中心。

### 事件驱动架构 (Event-Driven)

**适用：** 实时系统、物联网、金融交易。事件发布/订阅、事件溯源、CQRS。

### 六边形架构 (Hexagonal / Ports & Adapters)

**适用：** 需要高可测试性的系统。核心业务逻辑不依赖任何外部框架。

### CQRS (Command Query Responsibility Segregation)

**适用：** 读写比例差异大的系统。命令（写）和查询（读）分离，可独立优化。

---

## 函数式模式

### Maybe/Option 模式

**意图：** 用类型系统表示可能缺失的值，替代 null。

**实现：**
- Rust：`Option<T>`
- Haskell：`Maybe a`
- Swift：`Optional<T>`
- Kotlin：`T?`
- TypeScript：`T | null` + `ts-results` 库

### Either/Result 模式

**意图：** 用类型系统表示可能失败的操作，替代异常。

**实现：**
- Rust：`Result<T, E>`
- Haskell：`Either e a`
- Swift：`Result<T, E>`
- Go：`(T, error)` 元组
- Python：`typing.Result`（3.11+）或 `returns` 库

### 柯里化 (Currying)

**意图：** 将多参数函数转换为一系列单参数函数。

**适用：** 偏应用、函数组合、配置复用。

### 函数组合 (Function Composition)

**意图：** 将多个简单函数组合成复杂操作。

**实现：**
- Haskell：`.` 操作符
- F#：`>>` 操作符
- Elixir：`|>` 管道
- JavaScript：`_.flow` / `_.pipe`
- Python：函数嵌套或自定义 `pipe`
