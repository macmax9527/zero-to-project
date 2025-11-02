# 第12章：错误处理和容错 - 健壮性设计

> **本章目标**：学会设计健壮的错误处理机制，让程序优雅地应对异常情况

---

## 📋 本章大纲

1. [为什么需要错误处理](#1-为什么需要错误处理)
2. [错误类型和分类](#2-错误类型和分类)
3. [错误处理策略](#3-错误处理策略)
4. [重试机制](#4-重试机制)
5. [熔断器模式](#5-熔断器模式)
6. [降级和限流](#6-降级和限流)
7. [NOFX 的错误处理](#7-nofx-的错误处理)
8. [实战练习](#8-实战练习)

**预计学习时间**：3-4 小时

---

## 1. 为什么需要错误处理

### 1.1 现实世界的不完美

**比喻**：程序就像**开车**

```
理想情况（教科书）：
1. 上车
2. 启动
3. 开车到目的地
4. 停车

现实情况：
1. 上车
2. 启动 ← ❌ 车没油了
3. 加油
4. 启动 ← ❌ 电瓶没电了
5. 充电
6. 启动成功
7. 开车 ← ❌ 遇到堵车
8. 绕路
9. 开车 ← ❌ 下雨路滑
10. 减速慢行
11. 到达目的地 ✅

错误处理 = 应对各种意外情况
```

### 1.2 常见的错误场景

| 场景 | 错误类型 | 示例 |
|------|---------|------|
| **网络请求** | 超时、断网、服务器错误 | API 调用失败 |
| **数据库** | 连接失败、查询超时 | 数据库宕机 |
| **文件操作** | 文件不存在、权限不足 | 读取配置文件失败 |
| **用户输入** | 格式错误、非法数据 | 输入字母而不是数字 |
| **资源限制** | 内存不足、磁盘满了 | 无法写入日志 |
| **第三方服务** | 服务不可用、限流 | 支付接口报错 |

### 1.3 不处理错误的后果

```python
# ❌ 不处理错误的代码
def transfer_money(from_account, to_account, amount):
    from_account.balance -= amount  # 如果余额不足会怎样？
    to_account.balance += amount    # 如果这一步失败了怎么办？
    # 没有错误处理，可能导致：
    # - 钱扣了，但没有加到对方账户（钱消失了！）
    # - 余额变成负数
    # - 程序崩溃
```

```python
# ✅ 正确的错误处理
def transfer_money(from_account, to_account, amount):
    # 1. 验证
    if from_account.balance < amount:
        raise ValueError("余额不足")

    # 2. 开始事务
    try:
        from_account.balance -= amount
        to_account.balance += amount
        # 提交事务
    except Exception as e:
        # 回滚
        # 记录日志
        raise

    return True
```

---

## 2. 错误类型和分类

### 2.1 可恢复 vs 不可恢复

```python
# 可恢复错误：可以重试或有替代方案
try:
    response = requests.get("https://api.example.com/data")
except requests.Timeout:
    # 超时，可以重试 ✅
    response = requests.get("https://api.example.com/data", timeout=10)

# 不可恢复错误：程序无法继续
try:
    result = 10 / 0
except ZeroDivisionError:
    # 除以零，无法恢复 ❌
    # 只能记录日志并退出
    logging.error("除以零错误")
    raise
```

### 2.2 错误分类

**按严重程度**：

| 级别 | 说明 | 处理方式 | 示例 |
|------|------|---------|------|
| **Fatal（致命）** | 程序必须终止 | 记录日志，退出 | 配置文件损坏 |
| **Error（错误）** | 功能失败 | 记录日志，通知用户 | API 调用失败 |
| **Warning（警告）** | 潜在问题 | 记录日志，继续运行 | 响应时间过长 |
| **Info（信息）** | 正常事件 | 可选记录 | 用户登录 |

### 2.3 Python 中的异常层次

```python
BaseException
├── SystemExit         # 系统退出
├── KeyboardInterrupt  # Ctrl+C
└── Exception          # 常规异常
    ├── ValueError     # 值错误
    ├── TypeError      # 类型错误
    ├── KeyError       # 键不存在
    ├── IndexError     # 索引越界
    ├── FileNotFoundError  # 文件不存在
    ├── ConnectionError    # 连接错误
    └── ...

# 自定义异常
class InsufficientBalanceError(Exception):
    """余额不足异常"""
    pass

class APIError(Exception):
    """API 错误基类"""
    pass

class APITimeoutError(APIError):
    """API 超时"""
    pass

class APIRateLimitError(APIError):
    """API 限流"""
    pass
```

---

## 3. 错误处理策略

### 3.1 策略1：捕获并处理

```python
def read_config(filename):
    """读取配置文件"""
    try:
        with open(filename, 'r') as f:
            config = json.load(f)
        return config
    except FileNotFoundError:
        print(f"配置文件 {filename} 不存在，使用默认配置")
        return get_default_config()
    except json.JSONDecodeError as e:
        print(f"配置文件格式错误: {e}")
        return get_default_config()
    except Exception as e:
        print(f"读取配置文件时发生未知错误: {e}")
        raise  # 重新抛出，让上层处理
```

### 3.2 策略2：转换异常

```python
class DatabaseError(Exception):
    """数据库错误"""
    pass

def get_user(user_id):
    """获取用户"""
    try:
        result = db.query(f"SELECT * FROM users WHERE id = {user_id}")
        return result
    except psycopg2.OperationalError as e:
        # 转换为自定义异常
        raise DatabaseError(f"数据库连接失败: {e}") from e
    except psycopg2.ProgrammingError as e:
        raise DatabaseError(f"SQL 语句错误: {e}") from e

# 调用者只需处理 DatabaseError
try:
    user = get_user(123)
except DatabaseError as e:
    print(f"获取用户失败: {e}")
```

### 3.3 策略3：提前验证

```python
def divide(a, b):
    """除法运算"""
    # ❌ 不好：等到出错才处理
    try:
        return a / b
    except ZeroDivisionError:
        return None

    # ✅ 好：提前检查
    if b == 0:
        raise ValueError("除数不能为零")
    return a / b

def process_order(order):
    """处理订单"""
    # 提前验证所有条件
    if not order.items:
        raise ValueError("订单不能为空")

    if order.total <= 0:
        raise ValueError("订单金额必须大于0")

    if not user.has_sufficient_balance(order.total):
        raise InsufficientBalanceError("余额不足")

    # 所有检查通过，开始处理
    ...
```

### 3.4 策略4：优雅降级

```python
def get_user_avatar(user_id):
    """获取用户头像"""
    try:
        # 尝试从 CDN 获取
        return cdn.get_avatar(user_id)
    except CDNError:
        try:
            # CDN 失败，从主服务器获取
            return main_server.get_avatar(user_id)
        except ServerError:
            # 主服务器也失败，返回默认头像
            return get_default_avatar()

def show_recommendations(user_id):
    """显示推荐内容"""
    try:
        # 尝试获取个性化推荐
        recommendations = ai_service.get_recommendations(user_id)
    except AIServiceError:
        # AI 服务失败，显示热门内容
        recommendations = get_popular_items()

    return recommendations
```

---

## 4. 重试机制

### 4.1 为什么需要重试？

**瞬时故障**：短暂的网络波动、服务器临时过载

```
时间轴：
0s   请求 → ❌ 超时
1s   重试 → ❌ 超时
2s   重试 → ✅ 成功

如果不重试，用户看到的就是失败 ❌
有了重试，用户感觉只是慢了一点 ✅
```

### 4.2 简单重试

```python
import time

def fetch_data_with_retry(url, max_retries=3):
    """带重试的数据获取"""
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=5)
            response.raise_for_status()
            return response.json()
        except requests.RequestException as e:
            if attempt < max_retries - 1:
                wait_time = 2 ** attempt  # 指数退避：2s, 4s, 8s
                print(f"第 {attempt + 1} 次尝试失败，{wait_time}秒后重试...")
                time.sleep(wait_time)
            else:
                print(f"重试 {max_retries} 次后仍然失败")
                raise
```

### 4.3 装饰器实现重试

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1, backoff=2, exceptions=(Exception,)):
    """
    重试装饰器

    Args:
        max_attempts: 最大尝试次数
        delay: 初始延迟（秒）
        backoff: 退避倍数
        exceptions: 需要重试的异常类型
    """
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            attempt = 0
            current_delay = delay

            while attempt < max_attempts:
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    attempt += 1
                    if attempt >= max_attempts:
                        print(f"❌ 重试 {max_attempts} 次后仍然失败")
                        raise

                    print(f"⚠️  第 {attempt} 次尝试失败: {e}")
                    print(f"   {current_delay}秒后重试...")
                    time.sleep(current_delay)
                    current_delay *= backoff  # 指数退避

        return wrapper
    return decorator

# 使用装饰器
@retry(max_attempts=3, delay=1, backoff=2, exceptions=(requests.RequestException,))
def fetch_user_data(user_id):
    """获取用户数据"""
    response = requests.get(f"https://api.example.com/users/{user_id}")
    response.raise_for_status()
    return response.json()

# 调用
try:
    user = fetch_user_data(123)
except Exception as e:
    print(f"获取用户数据失败: {e}")
```

### 4.4 重试策略

**1. 固定延迟**

```python
重试间隔：1s, 1s, 1s
适用：快速恢复的故障
```

**2. 指数退避**

```python
重试间隔：1s, 2s, 4s, 8s
适用：服务器过载、限流
```

**3. 随机延迟**

```python
import random

delay = random.uniform(1, 3)  # 1-3秒随机
# 避免大量客户端同时重试（雷鸣群效应）
```

### 4.5 何时不应该重试？

```python
# ❌ 不应该重试的情况：

# 1. 客户端错误（4xx）
if response.status_code == 400:
    # 请求参数错误，重试也没用
    raise ValueError("请求参数错误")

# 2. 身份验证失败
if response.status_code == 401:
    # Token 过期或无效，需要重新登录
    raise AuthenticationError("身份验证失败")

# 3. 不幂等的操作
def create_order():
    # 创建订单不能重试，否则会重复下单
    pass

# 4. 致命错误
try:
    config = load_config()
except ConfigError:
    # 配置文件损坏，重试也不会成功
    raise
```

---

## 5. 熔断器模式

### 5.1 什么是熔断器？

**比喻**：家里的**保险丝**

```
正常情况：
电流 → 保险丝 → 家用电器 ✅

电流过大：
电流 → 保险丝断开 → 保护电器 ✅
       （熔断）

程序中的熔断器：
请求 → 第三方服务 ✅

服务故障：
请求 → 熔断器打开 → 直接返回错误
       （停止请求）     （保护系统）
```

### 5.2 熔断器三种状态

```
┌──────────┐
│   关闭    │ 正常工作，请求通过
│ (Closed) │
└─────┬────┘
      │ 失败次数达到阈值
      ↓
┌──────────┐
│   打开    │ 拒绝所有请求，直接返回错误
│  (Open)  │
└─────┬────┘
      │ 等待一段时间（如30秒）
      ↓
┌──────────┐
│  半开    │ 允许少量请求通过
│(Half-Open)│
└─────┬────┘
      │
      ├─ 成功 → 关闭
      └─ 失败 → 打开
```

### 5.3 实现熔断器

```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"       # 关闭（正常）
    OPEN = "open"           # 打开（熔断）
    HALF_OPEN = "half_open" # 半开（尝试恢复）

class CircuitBreaker:
    """熔断器"""
    def __init__(self, failure_threshold=5, recovery_timeout=30, expected_exception=Exception):
        """
        Args:
            failure_threshold: 失败次数阈值
            recovery_timeout: 恢复超时（秒）
            expected_exception: 预期的异常类型
        """
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.expected_exception = expected_exception

        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED

    def call(self, func, *args, **kwargs):
        """调用函数（带熔断保护）"""
        if self.state == CircuitState.OPEN:
            # 检查是否可以尝试恢复
            if time.time() - self.last_failure_time > self.recovery_timeout:
                self.state = CircuitState.HALF_OPEN
                print("🔄 熔断器进入半开状态，尝试恢复...")
            else:
                raise Exception("❌ 熔断器已打开，拒绝请求")

        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except self.expected_exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        """成功时的处理"""
        if self.state == CircuitState.HALF_OPEN:
            print("✅ 半开状态测试成功，关闭熔断器")
            self.state = CircuitState.CLOSED

        self.failure_count = 0

    def _on_failure(self):
        """失败时的处理"""
        self.failure_count += 1
        self.last_failure_time = time.time()

        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
            print(f"⚠️  失败次数达到阈值({self.failure_threshold})，打开熔断器")

# 使用示例
def unreliable_api_call():
    """不稳定的 API 调用"""
    response = requests.get("https://unreliable-api.com/data", timeout=5)
    response.raise_for_status()
    return response.json()

# 创建熔断器
breaker = CircuitBreaker(
    failure_threshold=3,
    recovery_timeout=30,
    expected_exception=requests.RequestException
)

# 使用熔断器调用
for i in range(10):
    try:
        result = breaker.call(unreliable_api_call)
        print(f"✅ 第 {i+1} 次调用成功")
    except Exception as e:
        print(f"❌ 第 {i+1} 次调用失败: {e}")

    time.sleep(2)
```

### 5.4 熔断器 + 降级

```python
def get_product_recommendations(user_id):
    """获取商品推荐（带熔断和降级）"""
    try:
        # 尝试调用 AI 推荐服务
        return breaker.call(ai_service.get_recommendations, user_id)
    except Exception:
        # 熔断器打开或服务失败，使用降级方案
        print("⚠️  推荐服务不可用，返回热门商品")
        return get_popular_products()
```

---

## 6. 降级和限流

### 6.1 降级（Degradation）

**定义**：当服务不可用时，提供有限的功能

```python
def show_user_profile(user_id):
    """显示用户资料"""
    try:
        # 完整版：从数据库获取详细信息
        profile = database.get_full_profile(user_id)
        return render_full_profile(profile)
    except DatabaseError:
        # 降级：只显示基本信息（从缓存）
        basic_info = cache.get_basic_info(user_id)
        return render_basic_profile(basic_info)
```

**降级策略**：

| 场景 | 降级方案 |
|------|---------|
| 推荐系统故障 | 显示热门内容 |
| 搜索服务故障 | 显示历史搜索 |
| 支付服务故障 | 线下支付说明 |
| 图片服务故障 | 显示占位图 |

### 6.2 限流（Rate Limiting）

**目的**：保护系统不被过载

**1. 固定窗口限流**

```python
import time
from collections import defaultdict

class RateLimiter:
    """固定窗口限流器"""
    def __init__(self, max_requests, window_size):
        """
        Args:
            max_requests: 窗口内最大请求数
            window_size: 窗口大小（秒）
        """
        self.max_requests = max_requests
        self.window_size = window_size
        self.requests = defaultdict(list)

    def allow_request(self, key):
        """检查是否允许请求"""
        now = time.time()
        window_start = now - self.window_size

        # 清理过期请求
        self.requests[key] = [
            req_time for req_time in self.requests[key]
            if req_time > window_start
        ]

        # 检查是否超过限制
        if len(self.requests[key]) >= self.max_requests:
            return False

        # 记录请求
        self.requests[key].append(now)
        return True

# 使用
limiter = RateLimiter(max_requests=10, window_size=60)  # 每分钟10个请求

def api_endpoint(user_id):
    if not limiter.allow_request(user_id):
        raise Exception("请求过于频繁，请稍后再试")

    # 处理请求
    ...
```

**2. 令牌桶限流**

```python
import time

class TokenBucket:
    """令牌桶限流器"""
    def __init__(self, capacity, refill_rate):
        """
        Args:
            capacity: 桶容量
            refill_rate: 每秒补充的令牌数
        """
        self.capacity = capacity
        self.refill_rate = refill_rate
        self.tokens = capacity
        self.last_refill = time.time()

    def allow_request(self):
        """检查是否允许请求"""
        # 补充令牌
        now = time.time()
        time_passed = now - self.last_refill
        self.tokens = min(
            self.capacity,
            self.tokens + time_passed * self.refill_rate
        )
        self.last_refill = now

        # 尝试消耗令牌
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False

# 使用
bucket = TokenBucket(capacity=10, refill_rate=1)  # 桶容量10，每秒补充1个

def api_call():
    if not bucket.allow_request():
        raise Exception("请求过于频繁")
    # 处理请求
```

---

## 7. NOFX 的错误处理

### 7.1 API 调用的错误处理

```go
// trader/binance_futures.go
func (b *BinanceFutures) GetAccount() (Account, error) {
    account, err := b.client.NewGetAccountService().Do(context.Background())
    if err != nil {
        // 记录错误
        log.Printf("❌ 获取币安账户信息失败: %v", err)

        // 返回错误，让上层处理
        return Account{}, fmt.Errorf("binance get account: %w", err)
    }

    // 转换数据
    return convertAccount(account), nil
}
```

### 7.2 重试机制

```go
// trader/auto_trader.go
func (at *AutoTrader) Run() error {
    for {
        // 错误处理：捕获 panic
        func() {
            defer func() {
                if r := recover(); r != nil {
                    log.Printf("❌ Panic recovered: %v", r)
                }
            }()

            // 获取账户信息（带重试）
            account, err := at.getAccountWithRetry(3)
            if err != nil {
                log.Printf("⚠️  获取账户信息失败，跳过本次循环: %v", err)
                return
            }

            // 执行交易逻辑
            at.executeTrading(account)
        }()

        // 等待下一个周期
        time.Sleep(at.scanInterval)
    }
}

func (at *AutoTrader) getAccountWithRetry(maxRetries int) (Account, error) {
    var lastErr error

    for i := 0; i < maxRetries; i++ {
        account, err := at.trader.GetAccount()
        if err == nil {
            return account, nil
        }

        lastErr = err
        if i < maxRetries - 1 {
            waitTime := time.Duration(i+1) * time.Second
            log.Printf("⚠️  第 %d 次尝试失败，%v 后重试", i+1, waitTime)
            time.Sleep(waitTime)
        }
    }

    return Account{}, lastErr
}
```

### 7.3 错误分类处理

```go
func (at *AutoTrader) handleAPIError(err error) {
    // 根据错误类型采取不同策略
    switch {
    case isRateLimitError(err):
        // 限流错误：等待更长时间
        log.Printf("⚠️  触发限流，等待60秒")
        time.Sleep(60 * time.Second)

    case isAuthError(err):
        // 认证错误：停止交易
        log.Printf("❌ 认证失败，停止交易")
        at.Stop()

    case isNetworkError(err):
        // 网络错误：重试
        log.Printf("⚠️  网络错误，将在下次循环重试")

    default:
        // 未知错误：记录并继续
        log.Printf("⚠️  未知错误: %v", err)
    }
}
```

---

## 8. 实战练习

### 练习 1：实现带重试的文件下载

```python
def download_file(url, filename, max_retries=3):
    """
    下载文件（带重试）

    要求：
    1. 支持断点续传
    2. 网络错误时重试
    3. 使用指数退避
    4. 显示下载进度
    """
    pass
```

### 练习 2：为 API 添加熔断器

为练习题中的 API 客户端添加熔断器保护。

**要求**：
- 5次失败后熔断
- 30秒后尝试恢复
- 熔断时返回缓存数据

### 练习 3：实现错误监控

创建一个错误监控系统，记录和统计错误。

**要求**：
- 记录错误类型、次数、时间
- 按小时/天统计
- 错误率超过阈值时告警

---

## 本章总结

### 错误处理原则

1. **预期错误**：提前思考可能的错误
2. **分类处理**：不同错误采用不同策略
3. **优雅降级**：服务不可用时提供有限功能
4. **快速失败**：无法恢复的错误立即报告
5. **记录日志**：所有错误都要记录

### 容错模式

| 模式 | 用途 | 适用场景 |
|------|------|---------|
| **重试** | 处理瞬时故障 | 网络请求、数据库查询 |
| **熔断器** | 防止雪崩 | 第三方服务调用 |
| **降级** | 保证核心功能 | 非关键服务故障 |
| **限流** | 保护系统 | 防止过载 |
| **超时** | 避免无限等待 | 所有外部调用 |

### NOFX 的启示

- 简单的重试机制就很有效
- 记录详细的错误日志
- 用 panic recovery 防止程序崩溃
- 根据错误类型采取不同策略

---

**💡 记住**：完美的程序不存在，但我们可以让程序优雅地处理错误。好的错误处理不是让程序永不出错，而是让程序在出错时仍能正常运转！
