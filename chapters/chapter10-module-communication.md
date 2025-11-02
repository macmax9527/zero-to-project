# 第10章：模块间通信 - 协作设计

> **本章目标**：学会设计模块间的通信机制，让各个模块高效协作

---

## 📋 本章大纲

1. [为什么需要模块间通信](#1-为什么需要模块间通信)
2. [通信方式对比](#2-通信方式对比)
3. [直接调用](#3-直接调用)
4. [事件驱动](#4-事件驱动)
5. [消息队列](#5-消息队列)
6. [共享状态](#6-共享状态)
7. [NOFX 的通信设计](#7-nofx-的通信设计)
8. [实战练习](#8-实战练习)

**预计学习时间**：3-4 小时

---

## 1. 为什么需要模块间通信

### 1.1 模块不是孤岛

**比喻**：模块就像**公司的部门**

```
公司（项目）
├── 销售部（用户模块）
├── 财务部（支付模块）
├── 仓库（库存模块）
└── 客服（通知模块）

场景：用户下单
1. 销售部接到订单 → 需要告诉仓库
2. 仓库确认库存 → 需要告诉财务部
3. 财务部扣款成功 → 需要告诉客服
4. 客服发送通知 → 告诉用户

如果部门之间不沟通，订单无法完成！
```

### 1.2 通信的目标

| 目标 | 说明 | 举例 |
|------|------|------|
| **数据传递** | 模块间传递信息 | 订单模块把订单ID传给支付模块 |
| **触发动作** | 一个模块通知另一个模块做事 | 支付成功后，触发发货 |
| **状态同步** | 多个模块共享状态 | 多个模块都能看到当前订单状态 |
| **解耦** | 降低模块间的依赖 | 订单模块不知道有哪些通知方式 |

---

## 2. 通信方式对比

### 2.1 四种主要方式

```
1. 直接调用（Function Call）
   ┌────────┐  调用方法  ┌────────┐
   │ 模块 A  │ ─────────→ │ 模块 B  │
   └────────┘  ←───────  └────────┘
                返回结果
   特点：简单、同步、紧耦合

2. 事件驱动（Event Driven）
   ┌────────┐  发布事件  ┌────────┐
   │ 模块 A  │ ─────────→ │ 事件总线 │
   └────────┘             └────────┘
                             ↓
                    ┌────────┴────────┐
                    ↓                 ↓
                ┌────────┐        ┌────────┐
                │ 模块 B  │        │ 模块 C  │
                └────────┘        └────────┘
   特点：解耦、异步、灵活

3. 消息队列（Message Queue）
   ┌────────┐  发送消息  ┌────────┐  接收消息  ┌────────┐
   │ 模块 A  │ ─────────→ │  队列   │ ─────────→ │ 模块 B  │
   └────────┘             └────────┘             └────────┘
   特点：可靠、异步、解耦

4. 共享状态（Shared State）
   ┌────────┐
   │ 模块 A  │ ←┐
   └────────┘  │
                ├─→ ┌────────────┐
   ┌────────┐  │    │ 共享内存/DB │
   │ 模块 B  │ ←┘    └────────────┘
   └────────┘
   特点：简单、可能冲突、需要同步
```

### 2.2 选择标准

| 场景 | 推荐方式 | 原因 |
|------|---------|------|
| 简单调用，需要返回值 | 直接调用 | 最简单直接 |
| 多个模块对同一事件感兴趣 | 事件驱动 | 解耦，易扩展 |
| 需要可靠传递，允许异步 | 消息队列 | 可靠性高 |
| 多个模块共享数据 | 共享状态 | 数据一致性 |
| 不知道谁会处理 | 事件驱动 | 动态订阅 |

---

## 3. 直接调用

### 3.1 最简单的方式

**特点**：模块 A 直接调用模块 B 的方法

```python
# order.py
class Order:
    def __init__(self):
        self.items = []
        self.total = 0

    def add_item(self, item, price):
        self.items.append(item)
        self.total += price

# payment.py
class Payment:
    def process(self, order, amount):
        if amount >= order.total:
            print(f"支付成功！总额：{order.total}")
            return True
        return False

# main.py
from order import Order
from payment import Payment

order = Order()
order.add_item("商品A", 100)
order.add_item("商品B", 200)

payment = Payment()
result = payment.process(order, 300)  # 直接调用
```

### 3.2 优缺点

**✅ 优点**：
- 简单直接
- 容易理解和调试
- 可以立即得到返回值

**❌ 缺点**：
- 模块间紧耦合（Payment 必须知道 Order 的结构）
- 不灵活（添加新功能需要修改代码）
- 同步阻塞（调用完成前无法做其他事）

### 3.3 适用场景

```python
# ✅ 适合：简单的、必需的调用
def calculate_tax(amount):
    return amount * 0.1

total = 100
tax = calculate_tax(total)  # 直接调用，合理

# ✅ 适合：需要立即得到结果
def validate_user(username, password):
    # 验证用户
    return True

if validate_user("zhang", "123456"):
    print("登录成功")

# ❌ 不适合：多个模块都关心的事件
def process_order(order):
    # 需要通知多个模块
    notify_warehouse(order)      # 紧耦合
    notify_accounting(order)     # 紧耦合
    notify_customer(order)       # 紧耦合
    # 添加新通知时，需要修改这里 ❌
```

---

## 4. 事件驱动

### 4.1 发布-订阅模式

**核心思想**：发布者发布事件，订阅者接收事件

```python
# event_bus.py
class EventBus:
    """事件总线"""
    def __init__(self):
        self.listeners = {}  # {事件名: [监听器列表]}

    def subscribe(self, event_name, listener):
        """订阅事件"""
        if event_name not in self.listeners:
            self.listeners[event_name] = []
        self.listeners[event_name].append(listener)
        print(f"✅ 订阅事件：{event_name}")

    def publish(self, event_name, data):
        """发布事件"""
        print(f"\n📢 发布事件：{event_name}")
        if event_name in self.listeners:
            for listener in self.listeners[event_name]:
                listener(data)

# 创建全局事件总线
event_bus = EventBus()
```

### 4.2 完整示例：订单系统

```python
# order.py
from event_bus import event_bus

class Order:
    def __init__(self, order_id, items, total):
        self.order_id = order_id
        self.items = items
        self.total = total
        self.status = "待支付"

    def create(self):
        """创建订单"""
        print(f"创建订单：{self.order_id}")
        # 发布事件：订单已创建
        event_bus.publish("order_created", {
            "order_id": self.order_id,
            "total": self.total
        })

    def pay(self):
        """支付订单"""
        print(f"支付订单：{self.order_id}")
        self.status = "已支付"
        # 发布事件：支付成功
        event_bus.publish("payment_success", {
            "order_id": self.order_id,
            "amount": self.total
        })

# warehouse.py
from event_bus import event_bus

class Warehouse:
    def __init__(self):
        # 订阅事件：支付成功
        event_bus.subscribe("payment_success", self.prepare_shipment)

    def prepare_shipment(self, data):
        """准备发货"""
        print(f"  [仓库] 收到通知，准备发货 订单#{data['order_id']}")

# notification.py
from event_bus import event_bus

class Notification:
    def __init__(self):
        # 订阅多个事件
        event_bus.subscribe("order_created", self.send_order_confirm)
        event_bus.subscribe("payment_success", self.send_payment_confirm)

    def send_order_confirm(self, data):
        """发送订单确认"""
        print(f"  [通知] 发送短信：订单#{data['order_id']}已创建")

    def send_payment_confirm(self, data):
        """发送支付确认"""
        print(f"  [通知] 发送邮件：支付{data['amount']}元成功")

# accounting.py
from event_bus import event_bus

class Accounting:
    def __init__(self):
        event_bus.subscribe("payment_success", self.record_income)

    def record_income(self, data):
        """记录收入"""
        print(f"  [财务] 记账：收入{data['amount']}元")

# main.py
from order import Order
from warehouse import Warehouse
from notification import Notification
from accounting import Accounting

def main():
    print("="*50)
    print("初始化系统...")
    print("="*50)

    # 1. 初始化各个模块（自动订阅事件）
    warehouse = Warehouse()
    notification = Notification()
    accounting = Accounting()

    print("\n" + "="*50)
    print("开始处理订单...")
    print("="*50)

    # 2. 创建订单
    order = Order(order_id="001", items=["商品A", "商品B"], total=500)
    order.create()

    # 3. 支付订单
    print("\n用户支付...")
    order.pay()

    print("\n" + "="*50)
    print("订单处理完成！")
    print("="*50)

if __name__ == "__main__":
    main()
```

**运行结果**：

```
==================================================
初始化系统...
==================================================
✅ 订阅事件：payment_success
✅ 订阅事件：order_created
✅ 订阅事件：payment_success
✅ 订阅事件：payment_success

==================================================
开始处理订单...
==================================================
创建订单：001

📢 发布事件：order_created
  [通知] 发送短信：订单#001已创建

用户支付...
支付订单：001

📢 发布事件：payment_success
  [仓库] 收到通知，准备发货 订单#001
  [通知] 发送邮件：支付500元成功
  [财务] 记账：收入500元

==================================================
订单处理完成！
==================================================
```

### 4.3 优缺点

**✅ 优点**：
- 解耦：发布者不知道谁会处理事件
- 灵活：添加新监听器无需修改现有代码
- 扩展性强：随时可以添加新功能

**❌ 缺点**：
- 调试困难：不知道谁处理了事件
- 执行顺序不确定：监听器的执行顺序可能变化
- 无返回值：发布者无法得到处理结果

### 4.4 改进：带优先级的事件系统

```python
# advanced_event_bus.py
class AdvancedEventBus:
    def __init__(self):
        self.listeners = {}

    def subscribe(self, event_name, listener, priority=0):
        """订阅事件（带优先级）"""
        if event_name not in self.listeners:
            self.listeners[event_name] = []
        self.listeners[event_name].append({
            "listener": listener,
            "priority": priority
        })
        # 按优先级排序（数字越大优先级越高）
        self.listeners[event_name].sort(key=lambda x: x["priority"], reverse=True)

    def publish(self, event_name, data):
        """发布事件"""
        if event_name in self.listeners:
            for item in self.listeners[event_name]:
                item["listener"](data)

# 使用
bus = AdvancedEventBus()
bus.subscribe("payment_success", warehouse.prepare, priority=10)  # 先执行
bus.subscribe("payment_success", notification.send, priority=5)   # 后执行
```

---

## 5. 消息队列

### 5.1 为什么需要消息队列？

**问题场景**：事件驱动的缺点

```python
# 事件驱动问题：
event_bus.publish("send_email", {"to": "user@example.com"})
# 如果此时邮件服务宕机了，消息就丢失了 ❌

# 解决方案：消息队列
queue.put("send_email", {"to": "user@example.com"})
# 消息会保存在队列中，即使服务宕机也不会丢失 ✅
# 恢复后会继续处理
```

### 5.2 简单的消息队列实现

```python
# message_queue.py
import queue
import threading
import time

class MessageQueue:
    """简单的消息队列"""
    def __init__(self):
        self.queue = queue.Queue()
        self.handlers = {}
        self.running = False

    def register_handler(self, message_type, handler):
        """注册消息处理器"""
        self.handlers[message_type] = handler
        print(f"✅ 注册处理器：{message_type}")

    def send(self, message_type, data):
        """发送消息"""
        message = {
            "type": message_type,
            "data": data,
            "timestamp": time.time()
        }
        self.queue.put(message)
        print(f"📤 发送消息：{message_type}")

    def start(self):
        """启动消息处理"""
        self.running = True
        thread = threading.Thread(target=self._process_messages)
        thread.daemon = True
        thread.start()
        print("🚀 消息队列已启动")

    def _process_messages(self):
        """处理消息（在后台线程运行）"""
        while self.running:
            try:
                # 获取消息（等待最多1秒）
                message = self.queue.get(timeout=1)

                # 查找处理器
                message_type = message["type"]
                if message_type in self.handlers:
                    handler = self.handlers[message_type]
                    print(f"📥 处理消息：{message_type}")
                    handler(message["data"])
                else:
                    print(f"⚠️  未找到处理器：{message_type}")

                self.queue.task_done()
            except queue.Empty:
                continue

    def stop(self):
        """停止消息处理"""
        self.running = False
```

### 5.3 使用消息队列

```python
# email_service.py
import time

class EmailService:
    def __init__(self, mq):
        # 注册处理器
        mq.register_handler("send_email", self.send_email)

    def send_email(self, data):
        """发送邮件"""
        print(f"  [邮件服务] 发送邮件到 {data['to']}")
        time.sleep(2)  # 模拟发送耗时
        print(f"  [邮件服务] 邮件发送成功")

# sms_service.py
class SMSService:
    def __init__(self, mq):
        mq.register_handler("send_sms", self.send_sms)

    def send_sms(self, data):
        """发送短信"""
        print(f"  [短信服务] 发送短信到 {data['phone']}")
        time.sleep(1)
        print(f"  [短信服务] 短信发送成功")

# main.py
from message_queue import MessageQueue
from email_service import EmailService
from sms_service import SMSService
import time

def main():
    # 1. 创建消息队列
    mq = MessageQueue()

    # 2. 初始化服务（注册处理器）
    email_service = EmailService(mq)
    sms_service = SMSService(mq)

    # 3. 启动消息队列
    mq.start()

    # 4. 发送消息
    print("\n开始发送消息...")
    mq.send("send_email", {"to": "user@example.com", "subject": "欢迎"})
    mq.send("send_sms", {"phone": "13812345678", "content": "验证码123456"})
    mq.send("send_email", {"to": "admin@example.com", "subject": "通知"})

    print("\n消息已发送到队列，后台处理中...")

    # 5. 等待处理完成
    time.sleep(10)

    print("\n所有消息处理完成！")

if __name__ == "__main__":
    main()
```

**运行结果**：

```
✅ 注册处理器：send_email
✅ 注册处理器：send_sms
🚀 消息队列已启动

开始发送消息...
📤 发送消息：send_email
📤 发送消息：send_sms
📤 发送消息：send_email

消息已发送到队列，后台处理中...
📥 处理消息：send_email
  [邮件服务] 发送邮件到 user@example.com
  [邮件服务] 邮件发送成功
📥 处理消息：send_sms
  [短信服务] 发送短信到 13812345678
  [短信服务] 短信发送成功
📥 处理消息：send_email
  [邮件服务] 发送邮件到 admin@example.com
  [邮件服务] 邮件发送成功

所有消息处理完成！
```

### 5.4 真实的消息队列

在生产环境中，通常使用成熟的消息队列：

| 消息队列 | 特点 | 适用场景 |
|---------|------|---------|
| **Redis** | 简单、快速 | 小型项目、缓存队列 |
| **RabbitMQ** | 功能丰富、可靠 | 企业应用 |
| **Kafka** | 高吞吐量 | 大数据、日志处理 |
| **Python queue** | 内置、简单 | 单机、线程间通信 |

**使用 Redis 作为消息队列**：

```python
import redis
import json

class RedisQueue:
    def __init__(self):
        self.redis = redis.Redis(host='localhost', port=6379)

    def send(self, queue_name, message):
        """发送消息"""
        self.redis.rpush(queue_name, json.dumps(message))

    def receive(self, queue_name, timeout=1):
        """接收消息"""
        result = self.redis.blpop(queue_name, timeout=timeout)
        if result:
            return json.loads(result[1])
        return None

# 使用
queue = RedisQueue()
queue.send("emails", {"to": "user@example.com", "subject": "Hello"})

# 另一个进程
message = queue.receive("emails")
if message:
    send_email(message)
```

---

## 6. 共享状态

### 6.1 使用共享对象

```python
# shared_state.py
class AppState:
    """应用全局状态"""
    def __init__(self):
        self.users_online = 0
        self.orders_today = 0
        self.total_revenue = 0

    def add_user(self):
        self.users_online += 1

    def remove_user(self):
        self.users_online -= 1

    def add_order(self, amount):
        self.orders_today += 1
        self.total_revenue += amount

    def get_stats(self):
        return {
            "users_online": self.users_online,
            "orders_today": self.orders_today,
            "total_revenue": self.total_revenue
        }

# 创建全局状态
app_state = AppState()
```

**使用**：

```python
# user_module.py
from shared_state import app_state

class UserModule:
    def user_login(self, user_id):
        app_state.add_user()
        print(f"用户 {user_id} 登录，在线人数：{app_state.users_online}")

    def user_logout(self, user_id):
        app_state.remove_user()
        print(f"用户 {user_id} 登出，在线人数：{app_state.users_online}")

# order_module.py
from shared_state import app_state

class OrderModule:
    def create_order(self, order_id, amount):
        app_state.add_order(amount)
        print(f"订单 {order_id} 已创建，今日订单数：{app_state.orders_today}")

# dashboard.py
from shared_state import app_state

class Dashboard:
    def show_stats(self):
        stats = app_state.get_stats()
        print("\n" + "="*40)
        print("实时统计：")
        print(f"  在线用户：{stats['users_online']}")
        print(f"  今日订单：{stats['orders_today']}")
        print(f"  今日营收：{stats['total_revenue']} 元")
        print("="*40)
```

### 6.2 并发问题

**问题**：多个线程同时修改共享状态

```python
import threading

# ❌ 不安全的代码
counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1  # 不是原子操作

# 启动多个线程
threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(counter)  # 期望 1000000，实际可能小于这个值 ❌
```

**解决方案：加锁**

```python
import threading

# ✅ 安全的代码
counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        with lock:  # 加锁
            counter += 1

# 启动多个线程
threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(counter)  # 正确：1000000 ✅
```

**改进的共享状态**：

```python
import threading

class SafeAppState:
    """线程安全的应用状态"""
    def __init__(self):
        self._lock = threading.Lock()
        self._users_online = 0
        self._orders_today = 0
        self._total_revenue = 0

    def add_user(self):
        with self._lock:
            self._users_online += 1

    def remove_user(self):
        with self._lock:
            self._users_online -= 1

    def add_order(self, amount):
        with self._lock:
            self._orders_today += 1
            self._total_revenue += amount

    def get_stats(self):
        with self._lock:
            return {
                "users_online": self._users_online,
                "orders_today": self._orders_today,
                "total_revenue": self._total_revenue
            }
```

---

## 7. NOFX 的通信设计

### 7.1 NOFX 使用的通信方式

**1. 直接调用（主要方式）**

```go
// manager/trader_manager.go
type TraderManager struct {
    traders map[string]*trader.AutoTrader
}

func (tm *TraderManager) GetTrader(id string) (*trader.AutoTrader, error) {
    return tm.traders[id], nil
}

// api/server.go
func (s *Server) handleAccount(c *gin.Context) {
    traderID := c.Query("trader_id")
    trader, _ := s.traderManager.GetTrader(traderID)  // 直接调用
    account, _ := trader.GetAccountInfo()              // 直接调用
    c.JSON(200, account)
}
```

**2. 接口抽象（解耦不同交易所）**

```go
// trader/interface.go
type Trader interface {
    GetAccount() (Account, error)
    GetPositions() ([]Position, error)
}

// trader/auto_trader.go
type AutoTrader struct {
    trader Trader  // 接口类型
}

func (at *AutoTrader) Run() {
    // 统一调用，不关心具体实现
    account, _ := at.trader.GetAccount()
    positions, _ := at.trader.GetPositions()
}
```

**3. 共享状态（内存存储）**

```go
// manager/trader_manager.go
type TraderManager struct {
    traders map[string]*trader.AutoTrader  // 共享的 trader 列表
    mu      sync.RWMutex                   // 读写锁保护
}

func (tm *TraderManager) GetAllTraders() map[string]*trader.AutoTrader {
    tm.mu.RLock()         // 读锁
    defer tm.mu.RUnlock()

    result := make(map[string]*trader.AutoTrader)
    for id, t := range tm.traders {
        result[id] = t
    }
    return result
}
```

### 7.2 NOFX 通信流程

```
前端请求
  ↓
API Server (api/server.go)
  ↓ 直接调用
TraderManager (manager/trader_manager.go)
  ↓ 从 map 中获取
AutoTrader (trader/auto_trader.go)
  ↓ 通过接口调用
Trader Interface (trader/interface.go)
  ↓ 多态
  ├─ BinanceFutures (trader/binance_futures.go)
  └─ HyperliquidTrader (trader/hyperliquid_trader.go)
  ↓ HTTP 请求
交易所 API
  ↓ 返回数据
  ↓
原路返回给前端
```

### 7.3 为什么 NOFX 选择直接调用？

| 原因 | 说明 |
|------|------|
| **简单** | 项目规模小，不需要复杂的消息系统 |
| **同步需求** | 前端需要立即得到结果 |
| **单机部署** | 无需分布式通信 |
| **性能** | 直接调用最快 |

**如果 NOFX 需要扩展**，可以考虑：

```go
// 添加事件系统
type Event struct {
    Type string
    Data interface{}
}

type EventBus struct {
    listeners map[string][]func(Event)
}

// 使用
eventBus.Subscribe("trade_executed", func(e Event) {
    // 记录到数据库
    // 发送通知
    // 更新统计
})

// 交易执行后
eventBus.Publish(Event{
    Type: "trade_executed",
    Data: trade,
})
```

---

## 8. 实战练习

### 练习 1：实现观察者模式

创建一个股票价格监控系统，当价格变化时通知多个观察者。

**要求**：
```python
# 价格源
class StockPrice:
    def __init__(self, symbol):
        self.symbol = symbol
        self.price = 0
        self.observers = []

    def attach(self, observer):
        """添加观察者"""
        pass

    def set_price(self, price):
        """设置价格并通知"""
        pass

# 观察者1：价格显示
class PriceDisplay:
    def update(self, symbol, price):
        print(f"[显示] {symbol} 价格：{price}")

# 观察者2：价格警报
class PriceAlert:
    def __init__(self, threshold):
        self.threshold = threshold

    def update(self, symbol, price):
        if price > self.threshold:
            print(f"[警报] {symbol} 价格超过 {self.threshold}！")
```

### 练习 2：消息队列应用

为练习 1 添加消息队列，实现异步处理。

**要求**：
- 价格变化时发送到队列
- 后台线程处理通知
- 支持多个处理器

### 练习 3：为 NOFX 添加事件系统

**任务**：为 NOFX 添加事件系统，当以下事件发生时通知：
- 交易执行（trade_executed）
- 持仓变化（position_changed）
- 账户余额变化（balance_changed）

**要求**：
- 实现事件总线
- 添加日志记录监听器
- 添加统计监听器

---

## 本章总结

### 通信方式选择

```
项目规模小、简单 → 直接调用
需要解耦、多个模块关心同一事件 → 事件驱动
需要可靠性、异步处理 → 消息队列
需要共享数据 → 共享状态（加锁保护）
```

### 最佳实践

1. **优先简单**：能用直接调用就不用事件
2. **适时解耦**：当模块间依赖变复杂时，引入事件
3. **注意并发**：共享状态要加锁
4. **考虑可靠性**：重要消息用消息队列
5. **便于测试**：通过接口隔离依赖

### NOFX 的启示

- 小项目用直接调用就够了
- 通过接口实现不同交易所的解耦
- 用 sync.RWMutex 保护共享状态
- 简单就是最好的设计

---

## 📚 扩展阅读

- [设计模式：观察者模式](https://refactoring.guru/design-patterns/observer)
- [消息队列选择指南](https://www.rabbitmq.com/getstarted.html)
- [Python 多线程和锁](https://docs.python.org/3/library/threading.html)

---

**💡 记住**：通信方式没有绝对的好坏，关键是选择适合项目规模和需求的方案。NOFX 告诉我们：简单直接的通信方式，对小型项目来说就是最好的！
