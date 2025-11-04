# 第5章：数据获取层 - 外部依赖管理

> **本章目标**：学会设计数据获取层，优雅地对接外部 API 和服务

---

## 🎯 核心概念

### 什么是数据获取层？

#### 在系统架构中的位置

```
┌─────────────┐
│  配置层     │ ← 读取配置
└─────────────┘
       ↓
┌─────────────┐
│ 数据获取层  │ ← 本章重点：获取外部数据
└─────────────┘
       ↓
┌─────────────┐
│ 业务逻辑层  │ ← 使用数据做决策
└─────────────┘
       ↓
┌─────────────┐
│  执行层     │ ← 执行操作
└─────────────┘
```

**数据获取层**位于配置层和业务逻辑层之间，专门负责：
- 从外部世界（API、数据库、文件）获取数据
- 把外部数据转换成内部统一格式
- 处理获取过程中的错误

#### 职责边界：只取数据，不做判断

```python
# ✅ 数据获取层应该做的事
class MarketDataFetcher:
    def get_price(self, symbol):
        """获取价格数据"""
        raw_data = self._call_api(symbol)  # 调用外部API
        return self._parse(raw_data)        # 转换格式

# ❌ 数据获取层不应该做的事
class MarketDataFetcher:
    def should_buy(self, symbol):
        """判断是否应该买入"""  # 这是业务逻辑！
        price = self.get_price(symbol)
        if price < 100:  # ❌ 不应该在这里做判断
            return True
```

**记住**：数据获取层是"翻译官"和"搬运工"，不是"决策者"。

#### 类比1：翻译官

```
外国厨师（外部API）说法语
         ↓
翻译官（数据获取层）翻译成中文
         ↓
你（业务逻辑）听中文做决定
```

你不需要学法语，只要翻译官统一翻译成中文就行。
换厨师了？没关系，换个翻译官就行，你还是听中文。

#### 类比2：电源适配器

```
美标插头（Binance API返回格式）
         ↓
适配器（数据获取层）
         ↓
国标插座（业务逻辑期待的格式）
```

你的电脑（业务逻辑）只认国标插座，不管电源是哪来的。
数据获取层就是这个适配器。

### 为什么需要数据获取层？

#### ❌ 不封装（直接调用外部 API）

```python
# decision.py
def make_decision():
    # 直接调用 Binance API
    url = "https://api.binance.com/api/v3/klines"
    params = {"symbol": "BTCUSDT", "interval": "3m", "limit": 100}
    response = requests.get(url, params=params)
    klines = response.json()

    # 直接调用 DeepSeek API
    ai_url = "https://api.deepseek.com/chat/completions"
    headers = {"Authorization": "Bearer sk-xxx"}
    ai_response = requests.post(ai_url, headers=headers, json={...})
```

**问题**：
- 🔴 换个交易所 → decision.py 要大改
- 🔴 API 返回格式变了 → 到处要改
- 🔴 需要重试 → 每个地方都要写重试逻辑
- 🔴 想加缓存 → 不知道改哪
- 🔴 测试困难 → 必须连接真实 API

#### ✅ 封装成数据获取层

```python
# market/data_fetcher.py（数据获取层）
class MarketDataFetcher:
    def fetch_klines(self, symbol, interval, limit):
        # 封装 API 调用
        # 处理错误
        # 自动重试
        # 格式转换
        pass

# decision.py（业务逻辑）
def make_decision():
    fetcher = MarketDataFetcher()
    klines = fetcher.fetch_klines("BTCUSDT", "3m", 100)  # 简单！
    # 不关心 API 细节
```

**优势**：
- ✅ business logic 简洁
- ✅ 换 API 只改一个地方
- ✅ 错误处理集中
- ✅ 容易测试（mock fetcher）

---

## 🧠 思维方法

设计数据获取层的核心思维方式：

### 方法一：隔离思维 - 画一道墙

**核心问题**：外部世界不可控，内部系统需要稳定

#### 识别"不可控"的外部

外部世界的问题：
- 🔴 API格式随时可能变化
- 🔴 网络随时可能中断
- 🔴 第三方服务可能限流、宕机
- 🔴 数据格式各不相同

#### 画一道保护墙

```
外部世界（不可控）              内部系统（可控）
    ↓                              ↑
    ↓                              ↑
    ↓    ┌──────────────────┐     ↑
    ↓    │  数据获取层      │     ↑
    ↓    │  （保护墙）      │     ↑
    ↓    └──────────────────┘     ↑
    ↓                              ↑
外部API变了？                  业务逻辑不受影响
网络断了？                      有重试机制
格式不同？                      统一转换
```

**思考问题**：
1. 哪些是外部的、不可控的？ → 放在墙外
2. 哪些是内部的、可控的？ → 放在墙内
3. 这道墙应该做什么？ → 隔离、转换、保护

#### 案例：NOFX 的隔离设计

```
Binance API（外部）              NOFX 决策引擎（内部）
Hyperliquid API（外部）    ←墙→  只关心统一的数据格式
DeepSeek API（外部）             不关心数据来源
```

如果 Binance 改了 API，只需要修改墙（数据获取层），内部系统完全不受影响。

---

### 方法二：抽象思维 - 找共性，定接口

**核心问题**：如何让业务逻辑不依赖具体的API？

#### 从具体到抽象

**❌ 具体思维（紧耦合）**：
```python
# 业务逻辑直接依赖 Binance
def make_decision():
    binance = BinanceAPI()
    price = binance.get_binance_ticker("BTCUSDT")  # 绑死了Binance
    # 如果要换交易所，这里全得改
```

**✅ 抽象思维（松耦合）**：
```python
# 业务逻辑依赖抽象
def make_decision(exchange):  # exchange 是抽象的
    price = exchange.get_price("BTCUSDT")  # 不关心具体是谁
    # 换交易所？传入不同的 exchange 对象即可
```

#### 找出共性

**思考问题**：
1. Binance、Hyperliquid、OKX 都能做什么？
   - ✅ 都能获取账户信息
   - ✅ 都能获取价格
   - ✅ 都能下单

2. 把共性提取成接口：
```python
class Exchange:  # 抽象接口
    def get_account(self): pass
    def get_price(self, symbol): pass
    def place_order(self, symbol, side, amount): pass
```

3. 具体实现去适配这个接口：
```python
class BinanceAPI(Exchange):  # 适配
class HyperliquidAPI(Exchange):  # 适配
```

**好处**：业务逻辑只依赖 `Exchange` 这个抽象，不依赖具体实现。

---

### 方法三：转换思维 - 外部千差万别，内部统一标准

**核心问题**：如何应对外部数据格式的差异？

#### 认识差异

不同API的账户信息格式完全不同：

```json
// Binance
{"totalWalletBalance": "1250.50", "availableBalance": "1000.00"}

// Hyperliquid
{"marginSummary": {"accountValue": "1250.50", "withdrawable": "1000.00"}}

// OKX
{"data": [{"totalEq": "1250.50", "availBal": "1000.00"}]}
```

如果业务逻辑直接用这些格式，每个交易所的代码都不一样，无法复用。

#### 定义内部统一模型

**思考问题**：
1. 业务逻辑真正需要什么信息？
   - 总权益
   - 可用余额
   - 已用保证金

2. 定义一个内部统一的 `Account` 类：
```python
class Account:
    total_equity: float
    available_balance: float
    margin_used: float
```

3. 数据获取层负责转换：
```
Binance格式 → 解析 → Account对象
Hyperliquid格式 → 解析 → Account对象
OKX格式 → 解析 → Account对象
```

#### 类比：标准化接口

```
110V 电源（美国）  ┐
220V 电源（中国）  ├→ 适配器 → 统一输出5V USB
240V 电源（英国）  ┘

你的手机（业务逻辑）只认 5V USB，不管输入是什么。
```

---

### 方法四：容错思维 - 失败是常态，不是例外

**核心问题**：外部API会失败，怎么办？

#### 改变心态

**❌ 错误心态**：
```python
data = api.get_data()  # 假设一定成功
process(data)  # 如果失败了，整个程序崩溃
```

**✅ 正确心态**：
```python
# 失败是常态，成功才是惊喜
data = api.get_data_with_retry()  # 重试
if data is None:  # 还是失败
    data = use_cached_data()  # 降级：使用缓存
    if data is None:  # 缓存也没有
        return default_value  # 兜底：返回默认值
```

#### 分层处理错误

1. **瞬时错误（可重试）**：
   - 网络超时 → 重试3次
   - 429限流 → 等待后重试

2. **持久错误（不可重试）**：
   - 401认证失败 → 直接抛出
   - 404资源不存在 → 直接抛出

3. **降级策略**：
   - 主API失败 → 用备用API
   - 都失败 → 用缓存数据
   - 缓存也没有 → 返回默认值或抛出异常

**思考问题**：
- 哪些错误是暂时的，可以重试？
- 哪些错误是永久的，不能重试？
- 如果都失败了，如何降级？
- 如何让业务逻辑不被外部错误影响？

---

### 总结：四大思维方式

| 思维方式 | 核心问题 | 解决方法 |
|---------|---------|---------|
| **隔离思维** | 外部不可控 | 画一道墙，保护内部 |
| **抽象思维** | 如何解耦 | 找共性，定接口 |
| **转换思维** | 格式不统一 | 定义内部标准，统一转换 |
| **容错思维** | 外部会失败 | 重试、降级、兜底 |

---

## 📐 实战设计

### 整体设计框架

在动手写代码之前，先理解数据获取层的**整体设计框架**。

#### 三层结构

数据获取层本身可以分为三层：

```
┌─────────────────────────────────────────┐
│         业务逻辑层（使用方）             │
│  decision.py, strategy.py...           │
└─────────────────────────────────────────┘
                    ↓ 调用
┌─────────────────────────────────────────┐
│    【第1层】抽象接口层（定义规范）        │
│    ExchangeAPI (ABC)                   │
│    - get_account()                     │  ← 统一的接口定义
│    - get_klines()                      │
│    - place_order()                     │
└─────────────────────────────────────────┘
                    ↑ 实现
        ┌───────────┼───────────┐
        ↓           ↓           ↓
┌──────────┐ ┌──────────┐ ┌──────────┐
│【第2层】  │ │ 具体实现  │ │ 具体实现  │
│ BinanceAPI│ │Hyperliquid│ │  OKX API │  ← 各自实现接口
│          │ │   API    │ │          │
└──────────┘ └──────────┘ └──────────┘
     ↓            ↓            ↓
     ↓            ↓            ↓
┌──────────┐ ┌──────────┐ ┌──────────┐
│【第3层】  │ │  数据转换 │ │  数据转换 │
│错误处理   │ │  + 缓存  │ │  + 重试  │  ← 容错和优化
│+ 重试     │ │          │ │          │
└──────────┘ └──────────┘ └──────────┘
     ↓            ↓            ↓
     ↓            ↓            ↓
┌─────────────────────────────────────────┐
│         外部API（不可控）                │
│  Binance、Hyperliquid、OKX...          │
└─────────────────────────────────────────┘
```

#### 为什么要分三层？

| 层次 | 作用 | 好处 |
|-----|------|------|
| **第1层：抽象接口** | 定义"做什么" | 业务逻辑不依赖具体实现，可随时切换 |
| **第2层：具体实现** | 定义"怎么做" | 每个API独立实现，互不影响 |
| **第3层：容错优化** | 处理"出错怎么办" | 集中处理错误、重试、缓存 |

#### 设计的核心逻辑

从外向内思考（从问题到解决方案）：

```
1. 外部问题：API各不相同
   ↓ 解决方案
   第1层：定义统一接口（抽象）

2. 实现问题：每个API格式不同
   ↓ 解决方案
   第2层：各自实现 + 数据转换（适配）

3. 运行问题：API会失败、会慢
   ↓ 解决方案
   第3层：错误处理 + 重试 + 缓存（容错）
```

#### 设计的五个步骤

按照从抽象到具体、从核心到外围的顺序：

```
步骤一：定义抽象接口        ← 定规范（最重要）
        ↓
步骤二：实现具体客户端      ← 适配器
        ↓
步骤三：定义统一数据模型    ← 内部标准
        ↓
步骤四：错误处理和重试      ← 容错机制
        ↓
步骤五：缓存策略            ← 性能优化（可选）
```

**为什么这个顺序？**
1. **先定接口**：明确"需要什么功能"
2. **再实现**：每个API去"实现这些功能"
3. **统一格式**：确保"返回的数据格式一致"
4. **处理错误**：保证"出错不崩溃"
5. **优化性能**：减少"不必要的请求"

#### 类比：开餐厅

```
第1层（抽象接口）   = 菜单（定义提供哪些菜）
第2层（具体实现）   = 厨房（不同厨师做不同菜系）
第3层（容错优化）   = 备用食材 + 库存管理（应对缺货）

顾客（业务逻辑）只看菜单点菜，不关心：
- 哪个厨师做的（具体实现）
- 食材从哪来的（外部API）
- 厨房如何应对缺货（错误处理）
```

---

### 步骤一：定义抽象接口

**目标**：找出不同外部API的共性，定义统一接口

#### 分析需求

以交易所为例，业务逻辑需要哪些功能？
1. 获取账户信息（总权益、可用余额）
2. 获取K线数据（用于分析）
3. 获取当前价格
4. 下单

#### 定义抽象接口

```python
from abc import ABC, abstractmethod

class ExchangeAPI(ABC):
    """交易所API的抽象接口"""

    @abstractmethod
    def get_account(self):
        """获取账户信息

        Returns:
            Account: 统一的账户对象
        """
        pass

    @abstractmethod
    def get_klines(self, symbol: str, interval: str, limit: int):
        """获取K线数据

        Args:
            symbol: 交易对，如 "BTCUSDT"
            interval: 时间间隔，如 "1m", "3m"
            limit: 获取数量

        Returns:
            list[Kline]: 统一的K线对象列表
        """
        pass

    @abstractmethod
    def get_price(self, symbol: str):
        """获取当前价格

        Args:
            symbol: 交易对

        Returns:
            float: 当前价格
        """
        pass

    @abstractmethod
    def place_order(self, symbol: str, side: str, quantity: float):
        """下单

        Args:
            symbol: 交易对
            side: "BUY" 或 "SELL"
            quantity: 数量

        Returns:
            Order: 统一的订单对象
        """
        pass
```

**关键点**：
- 使用 `ABC` (Abstract Base Class) 定义接口
- 只定义方法签名，不实现具体逻辑
- 返回统一的数据类型（Account、Kline、Order）

---

### 步骤二：实现具体客户端

**目标**：为每个外部API实现接口

#### 实现 Binance 客户端

```python
import requests

class BinanceAPI(ExchangeAPI):
    """Binance 交易所API实现"""

    def __init__(self, api_key, api_secret):
        self.api_key = api_key
        self.api_secret = api_secret
        self.base_url = "https://api.binance.com"

    def get_account(self):
        """获取 Binance 账户信息"""
        url = f"{self.base_url}/api/v3/account"
        headers = {"X-MBX-APIKEY": self.api_key}

        response = requests.get(url, headers=headers)
        raw_data = response.json()

        # 转换成统一格式
        return self._parse_account(raw_data)

    def get_klines(self, symbol, interval, limit):
        """获取 Binance K线数据"""
        url = f"{self.base_url}/api/v3/klines"
        params = {
            "symbol": symbol,
            "interval": interval,
            "limit": limit
        }

        response = requests.get(url, params=params)
        raw_data = response.json()

        # 转换成统一格式
        return [self._parse_kline(k) for k in raw_data]

    def _parse_account(self, raw_data):
        """将 Binance 格式转换为统一格式"""
        account = Account()
        account.total_equity = float(raw_data["totalWalletBalance"])
        account.available_balance = float(raw_data["availableBalance"])
        return account

    def _parse_kline(self, raw_kline):
        """将 Binance K线格式转换为统一格式"""
        return Kline(
            timestamp=raw_kline[0],
            open=float(raw_kline[1]),
            high=float(raw_kline[2]),
            low=float(raw_kline[3]),
            close=float(raw_kline[4]),
            volume=float(raw_kline[5])
        )
```

#### 实现 Hyperliquid 客户端

```python
class HyperliquidAPI(ExchangeAPI):
    """Hyperliquid 交易所API实现"""

    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.hyperliquid.xyz"

    def get_account(self):
        """获取 Hyperliquid 账户信息"""
        url = f"{self.base_url}/info"
        data = {"type": "clearinghouseState", "user": self.api_key}

        response = requests.post(url, json=data)
        raw_data = response.json()

        # 转换成统一格式（Hyperliquid 格式不同）
        return self._parse_account(raw_data)

    def _parse_account(self, raw_data):
        """将 Hyperliquid 格式转换为统一格式"""
        account = Account()
        summary = raw_data["marginSummary"]
        account.total_equity = float(summary["accountValue"])
        account.available_balance = float(summary["withdrawable"])
        return account
```

**关键点**：
- 继承抽象接口 `ExchangeAPI`
- 实现所有抽象方法
- 调用各自的API，返回统一格式

---

### 步骤三：定义统一数据模型

**目标**：定义内部使用的标准数据结构

```python
from dataclasses import dataclass

@dataclass
class Account:
    """统一的账户数据模型"""
    total_equity: float = 0.0        # 总权益
    available_balance: float = 0.0   # 可用余额
    margin_used: float = 0.0         # 已用保证金
    unrealized_pnl: float = 0.0      # 未实现盈亏

@dataclass
class Kline:
    """统一的K线数据模型"""
    timestamp: int      # 时间戳
    open: float         # 开盘价
    high: float         # 最高价
    low: float          # 最低价
    close: float        # 收盘价
    volume: float       # 成交量

@dataclass
class Order:
    """统一的订单数据模型"""
    order_id: str       # 订单ID
    symbol: str         # 交易对
    side: str           # BUY/SELL
    quantity: float     # 数量
    price: float        # 价格
    status: str         # 状态：PENDING/FILLED/CANCELLED
```

**好处**：
- 业务逻辑只用这些统一的类型
- 不关心数据来自哪个交易所
- 换交易所只需修改数据获取层的解析逻辑

---

### 步骤四：错误处理和重试

**目标**：优雅地处理外部API的各种错误

#### 基础错误处理

```python
def fetch_data(url):
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()  # 检查 HTTP 状态码
        return response.json()
    except requests.Timeout:
        print("请求超时")
        return None
    except requests.HTTPError as e:
        print(f"HTTP 错误: {e}")
        return None
    except Exception as e:
        print(f"未知错误: {e}")
        return None
```

#### 智能重试

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1, backoff=2):
    """
    重试装饰器
    max_attempts: 最多尝试次数
    delay: 初始延迟（秒）
    backoff: 延迟倍数（指数退避）
    """
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            attempt = 0
            current_delay = delay

            while attempt < max_attempts:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    attempt += 1
                    if attempt >= max_attempts:
                        raise  # 重试用尽，抛出异常

                    print(f"第{attempt}次尝试失败: {e}")
                    print(f"等待{current_delay}秒后重试...")
                    time.sleep(current_delay)
                    current_delay *= backoff  # 指数退避

        return wrapper
    return decorator

# 使用
@retry(max_attempts=3, delay=1, backoff=2)
def fetch_klines(symbol):
    response = requests.get(f"https://api.binance.com/api/v3/klines?symbol={symbol}")
    return response.json()

# 调用（自动重试）
data = fetch_klines("BTCUSDT")
# 失败会重试：等1秒、等2秒、等4秒
```

#### 分类错误处理

```python
def fetch_with_smart_retry(url):
    try:
        response = requests.get(url)

        # 429: 限流 → 等待更久
        if response.status_code == 429:
            retry_after = int(response.headers.get("Retry-After", 60))
            print(f"API 限流，等待 {retry_after} 秒")
            time.sleep(retry_after)
            return fetch_with_smart_retry(url)  # 重试

        # 5xx: 服务器错误 → 短暂重试
        elif 500 <= response.status_code < 600:
            print("服务器错误，3秒后重试")
            time.sleep(3)
            return fetch_with_smart_retry(url)

        # 4xx: 客户端错误 → 不重试（改了也没用）
        elif 400 <= response.status_code < 500:
            raise Exception(f"客户端错误: {response.status_code}")

        return response.json()

    except requests.Timeout:
        print("超时，重试")
        return fetch_with_smart_retry(url)
```

### 方法四：缓存策略

**问题**：重复请求浪费资源

```python
# ❌ 每次都请求
def get_klines():
    return requests.get("https://api.binance.com/...").json()

# 1分钟内调用10次 → 发送10次请求（浪费）
```

**解决**：缓存

```python
import time

class CachedDataFetcher:
    def __init__(self, cache_seconds=60):
        self.cache = {}  # {key: (data, timestamp)}
        self.cache_seconds = cache_seconds

    def fetch_klines(self, symbol, interval):
        # 生成缓存键
        cache_key = f"{symbol}_{interval}"

        # 检查缓存
        if cache_key in self.cache:
            data, timestamp = self.cache[cache_key]
            age = time.time() - timestamp

            if age < self.cache_seconds:
                print(f"使用缓存（{age:.1f}秒前）")
                return data

        # 缓存过期或不存在，重新获取
        print("获取新数据")
        data = self._fetch_from_api(symbol, interval)

        # 更新缓存
        self.cache[cache_key] = (data, time.time())

        return data
```

**缓存策略**：

| 数据类型 | 缓存时长 | 原因 |
|----------|----------|------|
| K线数据（1分钟） | 10秒 | 变化快 |
| K线数据（1小时） | 5分钟 | 变化慢 |
| 账户余额 | 1秒 | 需要实时 |
| 交易所配置 | 1小时 | 基本不变 |

---

## 📚 NOFX 案例分析

### 关于语言切换的说明

> **💡 重要提示**：
>
> 前面的示例我们用 **Python** 讲解概念，因为Python更容易理解。
>
> 但 **NOFX 项目是用 Go 语言**写的，所以这里会展示真实的 Go 代码。
>
> **不用担心**！即使你不懂Go，也能理解设计思路，因为：
> - ✅ 概念和Python是一样的
> - ✅ 我会用注释详细说明
> - ✅ 会提供Python和Go的对照
> - ✅ 重点是**设计思路**，不是语法细节

---

### Python vs Go 快速对照

#### 接口定义对照

<table>
<tr>
<td width="50%">

**Python 写法**
```python
from abc import ABC, abstractmethod

class Trader(ABC):
    @abstractmethod
    def get_account(self):
        """获取账户"""
        pass

    @abstractmethod
    def open_long(self, symbol, qty):
        """开多单"""
        pass
```

</td>
<td width="50%">

**Go 写法**
```go
// 接口定义
type Trader interface {
    // 获取账户
    GetAccount() (Account, error)

    // 开多单
    OpenLong(symbol string, qty float64) error
}
```

</td>
</tr>
</table>

#### 实现接口对照

<table>
<tr>
<td width="50%">

**Python 写法**
```python
class BinanceTrader(Trader):
    def get_account(self):
        # 调用API
        response = self.client.get_account()
        # 转换格式
        return self._parse(response)
```

</td>
<td width="50%">

**Go 写法**
```go
type BinanceTrader struct {
    client *BinanceClient
}

func (b *BinanceTrader) GetAccount() (Account, error) {
    // 调用API
    response := b.client.GetAccount()
    // 转换格式
    return b.parse(response)
}
```

</td>
</tr>
</table>

**关键点**：
- Python用 `class X(Interface)` 继承，Go自动实现接口（只要方法签名匹配）
- Python用 `self`，Go用结构体方法 `(b *BinanceTrader)`
- Go多返回值：`(Account, error)`，Python通常只返回一个或抛异常
- **但设计思路完全一样**：定义接口 → 各自实现 → 统一格式

---

### NOFX 的数据获取层设计

#### 1. Trader 接口（交易所抽象）

下面是NOFX真实代码，用Go实现的交易所接口：

```go
// trader/interface.go
package trader

// 【相当于Python的 dataclass】
// 统一的账户数据模型

type Account struct {
    TotalEquity      float64  // 总权益（Python: total_equity）
    AvailableBalance float64  // 可用余额（Python: available_balance）
    MarginUsed       float64  // 已用保证金（Python: margin_used）
    MarginUsedPct    float64  // 保证金使用率（Python: margin_used_pct）
}

// 【相当于Python的 dataclass】
type Position struct {
    Symbol           string   // 交易对，如 "BTCUSDT"
    Side             string   // "long" 或 "short"
    EntryPrice       float64  // 开仓价格
    MarkPrice        float64  // 标记价格
    Quantity         float64  // 持仓数量
    Leverage         int      // 杠杆倍数
    UnrealizedPnL    float64  // 未实现盈亏
    LiquidationPrice float64  // 强平价格
}

// 【相当于Python的抽象基类 ABC】
// Trader 接口：所有交易所必须实现这些方法
type Trader interface {
    // 账户信息（查询类）
    GetAccount() (Account, error)      // 获取账户信息
    GetPositions() ([]Position, error) // 获取持仓列表（[]相当于Python的list）

    // 交易操作（执行类）
    OpenLong(symbol string, quantity float64, leverage int) error   // 开多单
    OpenShort(symbol string, quantity float64, leverage int) error  // 开空单
    CloseLong(symbol string, quantity float64) error                // 平多单
    CloseShort(symbol string, quantity float64) error               // 平空单

    // 风控（风险管理）
    SetStopLoss(symbol string, side string, price float64) error    // 设置止损
    SetTakeProfit(symbol string, side string, price float64) error  // 设置止盈
}
```

#### 2. Binance 实现

**对应Python代码的结构**：
```python
class BinanceFutures(Trader):  # 相当于 Go 的 struct 实现 interface
    def __init__(self, api_key, secret_key):
        self.client = BinanceClient(api_key, secret_key)

    def get_account(self):
        # 1. 调用API
        # 2. 转换格式
        # 3. 返回统一的Account对象
```

**NOFX的Go实现**：

```go
// trader/binance_futures.go
package trader

import (
    "context"
    "fmt"
    "github.com/adshao/go-binance/v2/futures"  // Binance官方SDK
    "strconv"
)

// 【相当于Python的 class BinanceFutures(Trader)】
type BinanceFutures struct {
    client    *futures.Client  // Binance客户端（Python: self.client）
    apiKey    string           // API密钥
    secretKey string           // 密钥
}

// 【相当于Python的 __init__】
// 构造函数：创建BinanceFutures实例
func NewBinanceFutures(apiKey, secretKey string) *BinanceFutures {
    client := binance.NewFuturesClient(apiKey, secretKey)
    return &BinanceFutures{
        client:    client,
        apiKey:    apiKey,
        secretKey: secretKey,
    }
}

// 【相当于Python的 def get_account(self)】
// GetAccount 获取账户信息（实现Trader接口）
func (b *BinanceFutures) GetAccount() (Account, error) {
    // 步骤1：调用 Binance API（外部调用）
    account, err := b.client.NewGetAccountService().Do(context.Background())
    if err != nil {
        // Go的错误处理（Python会用 try-except）
        return Account{}, fmt.Errorf("获取账户失败: %w", err)
    }

    // 步骤2：数据转换（Binance格式 → 统一格式）
    // Binance返回的是字符串，需要转成浮点数
    totalEquity, _ := strconv.ParseFloat(account.TotalWalletBalance, 64)
    availableBalance, _ := strconv.ParseFloat(account.AvailableBalance, 64)
    marginUsed, _ := strconv.ParseFloat(account.TotalInitialMargin, 64)

    // 步骤3：返回统一的Account对象
    return Account{
        TotalEquity:      totalEquity,              // Binance字段映射
        AvailableBalance: availableBalance,         // 到统一格式
        MarginUsed:       marginUsed,
        MarginUsedPct:    (marginUsed / totalEquity) * 100,  // 计算百分比
    }, nil  // Go返回两个值：结果 + 错误
}

// GetPositions 获取持仓
func (b *BinanceFutures) GetPositions() ([]Position, error) {
    // 1. 调用 API
    positions, err := b.client.NewGetPositionRiskService().Do(context.Background())
    if err != nil {
        return nil, fmt.Errorf("获取持仓失败: %w", err)
    }

    // 2. 过滤和转换
    var result []Position
    for _, pos := range positions {
        posAmt, _ := strconv.ParseFloat(pos.PositionAmt, 64)

        // 过滤掉空仓
        if posAmt == 0 {
            continue
        }

        entryPrice, _ := strconv.ParseFloat(pos.EntryPrice, 64)
        markPrice, _ := strconv.ParseFloat(pos.MarkPrice, 64)
        unrealizedPnL, _ := strconv.ParseFloat(pos.UnrealizedProfit, 64)
        liquidationPrice, _ := strconv.ParseFloat(pos.LiquidationPrice, 64)
        leverage, _ := strconv.Atoi(pos.Leverage)

        result = append(result, Position{
            Symbol:           pos.Symbol,
            Side:             getSide(posAmt),
            EntryPrice:       entryPrice,
            MarkPrice:        markPrice,
            Quantity:         abs(posAmt),
            Leverage:         leverage,
            UnrealizedPnL:    unrealizedPnL,
            LiquidationPrice: liquidationPrice,
        })
    }

    return result, nil
}

// OpenLong 开多
func (b *BinanceFutures) OpenLong(symbol string, quantity float64, leverage int) error {
    // 1. 设置杠杆
    _, err := b.client.NewChangeLeverageService().
        Symbol(symbol).
        Leverage(leverage).
        Do(context.Background())
    if err != nil {
        return fmt.Errorf("设置杠杆失败: %w", err)
    }

    // 2. 下市价买单
    _, err = b.client.NewCreateOrderService().
        Symbol(symbol).
        Side(futures.SideTypeBuy).
        Type(futures.OrderTypeMarket).
        Quantity(formatQuantity(quantity)).
        Do(context.Background())

    if err != nil {
        return fmt.Errorf("开多失败: %w", err)
    }

    return nil
}

// 辅助函数
func getSide(posAmt float64) string {
    if posAmt > 0 {
        return "long"
    }
    return "short"
}

func formatQuantity(q float64) string {
    return fmt.Sprintf("%.8f", q)
}
```

**🎯 关键点总结**：

无论Python还是Go，核心设计逻辑是一样的：

| 步骤 | 做什么 | Python | Go |
|-----|-------|--------|-----|
| 1 | 定义接口 | `class Trader(ABC)` | `type Trader interface` |
| 2 | 实现接口 | `class BinanceTrader(Trader)` | `type BinanceFutures struct` + 实现方法 |
| 3 | 调用API | `requests.get()` | `client.NewGetAccountService().Do()` |
| 4 | 转换格式 | `account.total_equity = float(raw["balance"])` | `totalEquity := strconv.ParseFloat(...)` |
| 5 | 返回统一对象 | `return Account(...)` | `return Account{...}, nil` |

**看懂Go代码的技巧**：
- 不要纠结语法细节（`:=`、`*`、`error` 等）
- 关注**三步走**：调用API → 转换格式 → 返回对象
- 注释里会标注对应的Python写法

---

#### 3. Hyperliquid 实现

**Python伪代码对照**：
```python
class HyperliquidTrader(Trader):  # 同样实现Trader接口
    def __init__(self, private_key, wallet_addr):
        self.private_key = private_key
        self.wallet_addr = wallet_addr

    def get_account(self):
        # 1. 调用 Hyperliquid API（与Binance不同）
        # 2. 转换格式（与Binance不同）
        # 3. 返回统一的 Account 对象（与Binance相同！）
```

**NOFX的Go实现**：

```go
// trader/hyperliquid_trader.go
package trader

// 【相当于Python的 class HyperliquidTrader(Trader)】
// 实现同样的 Trader 接口，但调用 Hyperliquid API
type HyperliquidTrader struct {
    privateKey string  // 私钥（Hyperliquid用私钥认证，不同于Binance）
    walletAddr string  // 钱包地址
    isTestnet  bool    // 是否测试网
}

func (h *HyperliquidTrader) GetAccount() (Account, error) {
    // Hyperliquid 的 API 调用
    // 返回统一的 Account 结构
}

func (h *HyperliquidTrader) OpenLong(symbol string, quantity float64, leverage int) error {
    // Hyperliquid 的下单逻辑
}

// ... 其他方法
```

**设计亮点**：
- ✅ 统一接口：`GetAccount()` 对所有交易所一样
- ✅ 内部实现不同：Binance 用 REST API，Hyperliquid 用签名消息
- ✅ 上层代码不关心：`AutoTrader` 只知道 `Trader` 接口

#### 4. Market Data 封装

```go
// market/data.go
package market

import (
    "github.com/markcheno/go-talib"
)

type Kline struct {
    OpenTime  int64
    Open      float64
    High      float64
    Low       float64
    Close     float64
    Volume    float64
    CloseTime int64
}

type MACD struct {
    MACD   float64
    Signal float64
    Hist   float64
}

type Data struct {
    Symbol    string
    Klines3m  []Kline  // 3分钟K线（100根）
    Klines4h  []Kline  // 4小时K线（100根）

    // 3分钟指标
    RSI3m     float64
    MACD3m    MACD
    EMA20_3m  float64

    // 4小时指标
    RSI4h     float64
    MACD4h    MACD
    EMA20_4h  float64
    EMA50_4h  float64
    ATR4h     float64
}

// FetchData 获取并计算所有数据
func FetchData(symbol string, exchange string) (*Data, error) {
    data := &Data{Symbol: symbol}

    // 1. 获取K线
    if err := fetchKlines(data, exchange); err != nil {
        return nil, err
    }

    // 2. 计算指标
    if err := calculateIndicators(data); err != nil {
        return nil, err
    }

    return data, nil
}

func fetchKlines(data *Data, exchange string) error {
    // 根据交易所选择 API
    switch exchange {
    case "binance":
        return fetchBinanceKlines(data)
    case "hyperliquid":
        return fetchHyperliquidKlines(data)
    default:
        return fmt.Errorf("不支持的交易所: %s", exchange)
    }
}

func fetchBinanceKlines(data *Data) error {
    client := binance.NewClient("", "")

    // 获取3分钟K线
    klines3m, err := client.NewKlinesService().
        Symbol(data.Symbol).
        Interval("3m").
        Limit(100).
        Do(context.Background())

    if err != nil {
        return err
    }

    // 转换格式
    for _, k := range klines3m {
        data.Klines3m = append(data.Klines3m, Kline{
            OpenTime:  k.OpenTime,
            Open:      parseFloat(k.Open),
            High:      parseFloat(k.High),
            Low:       parseFloat(k.Low),
            Close:     parseFloat(k.Close),
            Volume:    parseFloat(k.Volume),
            CloseTime: k.CloseTime,
        })
    }

    // 获取4小时K线（类似）
    // ...

    return nil
}

func calculateIndicators(data *Data) error {
    // 提取收盘价
    closes3m := extractCloses(data.Klines3m)
    closes4h := extractCloses(data.Klines4h)

    // 计算 RSI
    rsi3m := talib.Rsi(closes3m, 7)
    data.RSI3m = rsi3m[len(rsi3m)-1]

    rsi4h := talib.Rsi(closes4h, 14)
    data.RSI4h = rsi4h[len(rsi4h)-1]

    // 计算 MACD
    macd3m, signal3m, hist3m := talib.Macd(closes3m, 12, 26, 9)
    data.MACD3m = MACD{
        MACD:   macd3m[len(macd3m)-1],
        Signal: signal3m[len(signal3m)-1],
        Hist:   hist3m[len(hist3m)-1],
    }

    // 计算 EMA
    ema20_3m := talib.Ema(closes3m, 20)
    data.EMA20_3m = ema20_3m[len(ema20_3m)-1]

    // ... 其他指标

    return nil
}

func extractCloses(klines []Kline) []float64 {
    closes := make([]float64, len(klines))
    for i, k := range klines {
        closes[i] = k.Close
    }
    return closes
}
```

**设计亮点**：
- ✅ 一次调用获取所有数据
- ✅ 自动计算所有指标
- ✅ 返回统一的 `Data` 结构
- ✅ 上层不关心数据来源

#### 5. AI API 封装

```go
// mcp/client.go
package mcp

import (
    "bytes"
    "encoding/json"
    "net/http"
    "time"
)

type Client struct {
    apiKey  string
    model   string
    baseURL string
    timeout time.Duration
}

func NewClient(apiKey, model string) *Client {
    var baseURL string
    switch model {
    case "deepseek":
        baseURL = "https://api.deepseek.com/v1"
    case "qwen":
        baseURL = "https://dashscope.aliyuncs.com/api/v1"
    }

    return &Client{
        apiKey:  apiKey,
        model:   model,
        baseURL: baseURL,
        timeout: 120 * time.Second,
    }
}

// Chat 发送消息给 AI
func (c *Client) Chat(systemPrompt, userPrompt string) (string, error) {
    // 1. 构建请求
    reqBody := map[string]interface{}{
        "model": c.getModelName(),
        "messages": []map[string]string{
            {"role": "system", "content": systemPrompt},
            {"role": "user", "content": userPrompt},
        },
        "temperature": 0.7,
    }

    jsonData, _ := json.Marshal(reqBody)

    // 2. 发送请求
    req, _ := http.NewRequest("POST", c.baseURL+"/chat/completions", bytes.NewBuffer(jsonData))
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+c.apiKey)

    client := &http.Client{Timeout: c.timeout}
    resp, err := client.Do(req)
    if err != nil {
        return "", fmt.Errorf("AI API 调用失败: %w", err)
    }
    defer resp.Body.Close()

    // 3. 解析响应
    var result struct {
        Choices []struct {
            Message struct {
                Content string `json:"content"`
            } `json:"message"`
        } `json:"choices"`
    }

    json.NewDecoder(resp.Body).Decode(&result)

    if len(result.Choices) == 0 {
        return "", fmt.Errorf("AI 无响应")
    }

    return result.Choices[0].Message.Content, nil
}

func (c *Client) getModelName() string {
    switch c.model {
    case "deepseek":
        return "deepseek-chat"
    case "qwen":
        return "qwen-max"
    default:
        return c.model
    }
}
```

**设计亮点**：
- ✅ 封装不同 AI 的 API 差异
- ✅ 统一的调用接口
- ✅ 错误处理和超时
- ✅ 易于添加新 AI 模型

---

## 💪 实战练习

### 练习 1：设计数据获取接口（必做）

为你的项目设计数据获取接口：

```python
class DataSource:
    """数据源接口"""

    def fetch_xxx(self, params):
        """获取XXX数据"""
        pass

    def fetch_yyy(self, params):
        """获取YYY数据"""
        pass
```

**要求**：
- 定义 3-5 个核心方法
- 每个方法写清楚输入输出
- 考虑错误情况

---

### 练习 2：实现一个简单的 API 客户端（必做）

选择一个公开 API（例如：天气 API），实现：

```python
class WeatherAPI:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.weather.com"

    def get_weather(self, city):
        """获取天气"""
        try:
            url = f"{self.base_url}/current?city={city}&key={self.api_key}"
            response = requests.get(url, timeout=10)
            response.raise_for_status()
            return self._parse_response(response.json())
        except Exception as e:
            print(f"获取天气失败: {e}")
            return None

    def _parse_response(self, data):
        """解析响应，转换为统一格式"""
        return {
            "temperature": data["temp"],
            "humidity": data["humidity"],
            # ...
        }
```

---

### 练习 3：添加重试机制（必做）

为上面的 API 客户端添加重试：

```python
import time

def fetch_with_retry(self, url, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=10)
            return response.json()
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            print(f"第{attempt+1}次失败，重试...")
            time.sleep(2 ** attempt)  # 指数退避：2, 4, 8秒
```

---

### 练习 4：添加缓存（选做）

实现简单的缓存机制：

```python
class CachedAPI:
    def __init__(self):
        self.cache = {}

    def get_data(self, key):
        # 检查缓存
        if key in self.cache:
            data, timestamp = self.cache[key]
            if time.time() - timestamp < 60:  # 60秒内有效
                return data

        # 获取新数据
        data = self._fetch_from_api(key)
        self.cache[key] = (data, time.time())
        return data
```

---

### 练习 5：错误分类处理（选做）

根据不同错误类型采取不同策略：

```python
def smart_fetch(url):
    try:
        response = requests.get(url)

        if response.status_code == 429:
            # 限流：等待后重试
            time.sleep(60)
            return smart_fetch(url)

        elif response.status_code >= 500:
            # 服务器错误：短暂重试
            time.sleep(5)
            return smart_fetch(url)

        elif response.status_code >= 400:
            # 客户端错误：不重试
            raise Exception(f"请求错误: {response.status_code}")

        return response.json()

    except requests.Timeout:
        # 超时：重试
        return smart_fetch(url)
```

---

## 🤔 思考题

### 1. 为什么要定义统一的数据模型（如 Account、Position）？

<details>
<summary>点击查看答案</summary>

**不定义统一模型的问题**：

```python
# 业务逻辑要处理不同格式
def calculate_profit(exchange_type, account_data):
    if exchange_type == "binance":
        equity = float(account_data["totalWalletBalance"])
    elif exchange_type == "hyperliquid":
        equity = float(account_data["marginSummary"]["accountValue"])
    # 每个交易所都要判断！
```

**定义统一模型后**：

```python
def calculate_profit(account: Account):
    equity = account.total_equity  # 统一字段
    # 不关心数据来源
```

**好处**：
1. ✅ 业务逻辑简单
2. ✅ 易于添加新交易所（只需实现转换）
3. ✅ 类型安全（IDE 有提示）
4. ✅ 易于测试

</details>

---

### 2. 缓存什么时候会出问题？

<details>
<summary>点击查看答案</summary>

**问题场景**：

1. **数据过期**
```python
# 缓存账户余额 1 小时
balance = cache.get("balance")  # 1000 USDT

# 实际上 30 分钟前已经亏损到 500 USDT
# 但缓存还显示 1000，导致决策错误！
```

**解决**：根据数据特性设置合理的缓存时长
- 实时数据（余额）：1-5 秒
- 慢变数据（配置）：1 小时

2. **缓存一致性**
```python
# 进程A写缓存
cache["BTCUSDT"] = data1

# 进程B也写缓存
cache["BTCUSDT"] = data2  # 覆盖！
```

**解决**：使用分布式缓存（Redis）

3. **内存溢出**
```python
# 无限增长的缓存
for i in range(1000000):
    cache[f"key_{i}"] = large_data  # 内存爆了！
```

**解决**：设置缓存上限（LRU 淘汰）

</details>

---

### 3. 如何测试数据获取层？

<details>
<summary>点击查看答案</summary>

**方法1：Mock（模拟）**

```python
from unittest.mock import Mock

def test_get_account():
    # 创建 Mock API
    mock_api = Mock()
    mock_api.get_account.return_value = {
        "totalWalletBalance": "1000.0"
    }

    # 测试转换逻辑
    account = parse_binance_account(mock_api.get_account())

    assert account.total_equity == 1000.0
```

**方法2：测试服务器**

```python
# 启动本地测试服务器
@app.route("/api/account")
def mock_account():
    return {"totalWalletBalance": "1000.0"}

# 测试时连接测试服务器
client = BinanceAPI(base_url="http://localhost:5000")
```

**方法3：使用测试网**

```python
# 连接交易所的测试网
client = BinanceAPI(testnet=True)
account = client.get_account()  # 真实 API，但不是真钱
```

</details>

---

## 📖 本章总结

### 你学到了什么

✅ **核心概念**：
- 数据获取层的作用
- 为什么需要封装外部 API
- 统一数据模型的重要性

✅ **思维方法**：
- 接口抽象（依赖接口）
- 统一数据模型（内部格式）
- 错误处理和重试策略
- 缓存策略

✅ **实践技能**：
- 能设计数据获取接口
- 能实现 API 客户端
- 能添加重试和缓存
- 能处理各种错误

✅ **案例收获**：
- NOFX 的 Trader 接口设计
- 如何统一多个交易所
- 市场数据的获取和计算
- AI API 的封装

---

### 数据获取层检查清单

- [ ] **接口清晰**：定义明确的接口
- [ ] **统一格式**：内部使用统一数据模型
- [ ] **错误处理**：捕获并处理各种错误
- [ ] **重试机制**：网络错误自动重试
- [ ] **超时设置**：避免长时间等待
- [ ] **缓存策略**：减少重复请求
- [ ] **日志记录**：记录 API 调用
- [ ] **易于测试**：支持 Mock 测试

---

### 下一步

完成练习后，你应该有：
- ✅ 数据获取接口设计
- ✅ 一个简单的 API 客户端实现
- ✅ 重试和缓存机制
- ✅ 错误处理策略

准备好后，进入 **第 6 章：业务逻辑层**，学习核心算法设计！

---

## 📚 延伸阅读

- Python requests 库文档
- Go http 包文档
- 《重试策略：指数退避算法》
- 《API 设计最佳实践》

---

## ❓ FAQ

**Q1：所有 API 调用都要封装吗？**
A：
- 重复使用的 → 必须封装
- 只用一次的 → 可以不封装
- 外部依赖的 → 建议封装（方便替换）

**Q2：重试次数设置多少合适？**
A：
- 一般 3 次
- 关键操作 5 次
- 幂等操作可以多次
- 非幂等操作谨慎重试

**Q3：什么时候用缓存，什么时候不用？**
A：
- 用缓存：数据变化慢、请求频繁
- 不用缓存：数据实时性要求高

**Q4：如何处理 API 限流？**
A：
- 读取 Retry-After 头部
- 实现令牌桶限流
- 降低请求频率

---

**🎉 恭喜完成第 5 章！**

你已经掌握了数据获取层设计！

记住：**好的封装让系统灵活，坏的封装让系统脆弱。**

准备好了吗？进入第 6 章！💪
