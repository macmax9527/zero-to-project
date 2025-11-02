# 第18章：扩展性设计 - 应对变化

> **本章目标**：学会设计可扩展的系统，优雅地应对需求变化

---

## 📋 本章大纲

1. [为什么需要扩展性](#1-为什么需要扩展性)
2. [设计原则](#2-设计原则)
3. [插件系统](#3-插件系统)
4. [策略模式](#4-策略模式)
5. [配置驱动](#5-配置驱动)
6. [依赖注入](#6-依赖注入)
7. [NOFX 的扩展性设计](#7-nofx-的扩展性设计)
8. [实战练习](#8-实战练习)

**预计学习时间**：4-5 小时

---

## 1. 为什么需要扩展性

### 1.1 需求变化是常态

**场景1：添加新功能**
```python
# 初始版本：只支持邮件通知
def send_notification(user, message):
    send_email(user.email, message)

# 需求变化：增加短信通知
# ❌ 糟糕的做法：直接修改原函数
def send_notification(user, message, type="email"):
    if type == "email":
        send_email(user.email, message)
    elif type == "sms":
        send_sms(user.phone, message)
    # 未来还会增加微信、钉钉...每次都要修改这个函数
```

**场景2：支持不同实现**
```python
# 初始版本：只支持 MySQL
class Database:
    def __init__(self):
        self.conn = mysql.connect(...)

# 需求变化：需要支持 PostgreSQL、MongoDB
# ❌ 糟糕的做法：到处 if-else
class Database:
    def __init__(self, db_type="mysql"):
        if db_type == "mysql":
            self.conn = mysql.connect(...)
        elif db_type == "postgresql":
            self.conn = psycopg2.connect(...)
        elif db_type == "mongodb":
            self.conn = pymongo.MongoClient(...)
```

### 1.2 扩展性的价值

**易于维护**：
- 新功能不影响旧代码
- 降低引入 bug 的风险

**快速响应**：
- 添加新功能只需几行代码
- 不需要大规模重构

**团队协作**：
- 不同人可以并行开发新功能
- 不会互相冲突

---

## 2. 设计原则

### 2.1 开闭原则（OCP）

**定义**：对扩展开放，对修改关闭

```python
# ✅ 良好的设计
class Notifier(ABC):
    @abstractmethod
    def send(self, user, message):
        pass

class EmailNotifier(Notifier):
    def send(self, user, message):
        send_email(user.email, message)

class SMSNotifier(Notifier):
    def send(self, user, message):
        send_sms(user.phone, message)

# 添加新的通知方式，不需要修改原有代码
class WeChatNotifier(Notifier):
    def send(self, user, message):
        send_wechat(user.wechat_id, message)

# 使用
def notify_user(user, message, notifier: Notifier):
    notifier.send(user, message)

# 灵活切换
notify_user(user, "Hello", EmailNotifier())
notify_user(user, "Hello", SMSNotifier())
notify_user(user, "Hello", WeChatNotifier())
```

### 2.2 依赖倒置原则（DIP）

**定义**：依赖抽象，不依赖具体实现

```python
# ❌ 糟糕：依赖具体实现
class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()  # 硬编码依赖 MySQL

    def create_order(self, order):
        self.db.save(order)

# ✅ 良好：依赖抽象
class OrderService:
    def __init__(self, db: Database):  # 依赖抽象接口
        self.db = db

    def create_order(self, order):
        self.db.save(order)

# 可以注入任何实现
mysql_service = OrderService(MySQLDatabase())
postgres_service = OrderService(PostgreSQLDatabase())
mongo_service = OrderService(MongoDatabase())
```

### 2.3 里氏替换原则（LSP）

**定义**：子类可以替换父类

```python
class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, amount) -> bool:
        """返回是否成功"""
        pass

class AlipayProcessor(PaymentProcessor):
    def process(self, amount) -> bool:
        # 支付宝支付逻辑
        return True

class WeChatPayProcessor(PaymentProcessor):
    def process(self, amount) -> bool:
        # 微信支付逻辑
        return True

# 任何 PaymentProcessor 的子类都可以互相替换
def checkout(processor: PaymentProcessor, amount):
    if processor.process(amount):
        print("支付成功")
    else:
        print("支付失败")

# 可以随意替换
checkout(AlipayProcessor(), 100)
checkout(WeChatPayProcessor(), 100)
```

---

## 3. 插件系统

### 3.1 简单插件系统

```python
# plugin_system.py
class PluginRegistry:
    def __init__(self):
        self.plugins = {}

    def register(self, name, plugin):
        """注册插件"""
        self.plugins[name] = plugin
        print(f"插件 '{name}' 已注册")

    def get(self, name):
        """获取插件"""
        return self.plugins.get(name)

    def list_plugins(self):
        """列出所有插件"""
        return list(self.plugins.keys())

# 全局注册表
registry = PluginRegistry()

# 插件基类
class Plugin(ABC):
    @abstractmethod
    def execute(self, data):
        pass

# 插件示例
class UpperCasePlugin(Plugin):
    def execute(self, data):
        return data.upper()

class ReversePlugin(Plugin):
    def execute(self, data):
        return data[::-1]

class LengthPlugin(Plugin):
    def execute(self, data):
        return len(data)

# 注册插件
registry.register("upper", UpperCasePlugin())
registry.register("reverse", ReversePlugin())
registry.register("length", LengthPlugin())

# 使用插件
def process_text(text, plugin_name):
    plugin = registry.get(plugin_name)
    if plugin:
        return plugin.execute(text)
    else:
        raise ValueError(f"插件 '{plugin_name}' 不存在")

# 测试
print(process_text("hello", "upper"))     # HELLO
print(process_text("hello", "reverse"))   # olleh
print(process_text("hello", "length"))    # 5
```

### 3.2 装饰器注册插件

```python
# 更优雅的注册方式
class PluginRegistry:
    def __init__(self):
        self.plugins = {}

    def register(self, name):
        """装饰器：注册插件"""
        def decorator(plugin_class):
            self.plugins[name] = plugin_class()
            return plugin_class
        return decorator

    def get(self, name):
        return self.plugins.get(name)

registry = PluginRegistry()

# 使用装饰器注册
@registry.register("markdown")
class MarkdownPlugin(Plugin):
    def execute(self, data):
        return f"**{data}**"

@registry.register("html")
class HTMLPlugin(Plugin):
    def execute(self, data):
        return f"<b>{data}</b>"

# 自动注册，无需手动调用 registry.register()
print(process_text("hello", "markdown"))  # **hello**
print(process_text("hello", "html"))      # <b>hello</b>
```

### 3.3 动态加载插件

```python
import importlib
import os

class PluginLoader:
    def __init__(self, plugin_dir="plugins"):
        self.plugin_dir = plugin_dir
        self.registry = PluginRegistry()

    def load_plugins(self):
        """动态加载插件目录中的所有插件"""
        for filename in os.listdir(self.plugin_dir):
            if filename.endswith(".py") and not filename.startswith("__"):
                module_name = filename[:-3]
                module_path = f"{self.plugin_dir}.{module_name}"

                # 动态导入模块
                module = importlib.import_module(module_path)

                # 查找插件类并注册
                for attr_name in dir(module):
                    attr = getattr(module, attr_name)
                    if isinstance(attr, type) and issubclass(attr, Plugin) and attr != Plugin:
                        plugin_name = attr_name.lower().replace("plugin", "")
                        self.registry.register(plugin_name, attr())

        print(f"已加载 {len(self.registry.list_plugins())} 个插件")

    def get_plugin(self, name):
        return self.registry.get(name)

# 使用
loader = PluginLoader("plugins")
loader.load_plugins()

# 插件目录结构：
# plugins/
#   ├── __init__.py
#   ├── csv_exporter.py    # CSVExporterPlugin
#   ├── json_exporter.py   # JSONExporterPlugin
#   └── xml_exporter.py    # XMLExporterPlugin
```

---

## 4. 策略模式

### 4.1 什么是策略模式

**定义**：定义一系列算法，将每个算法封装起来，使它们可以互相替换

```python
from abc import ABC, abstractmethod

# 策略接口
class PricingStrategy(ABC):
    @abstractmethod
    def calculate(self, price):
        pass

# 具体策略
class RegularPricing(PricingStrategy):
    """普通用户定价"""
    def calculate(self, price):
        return price

class VIPPricing(PricingStrategy):
    """VIP 用户：9折"""
    def calculate(self, price):
        return price * 0.9

class SVIPPricing(PricingStrategy):
    """超级VIP：8折"""
    def calculate(self, price):
        return price * 0.8

class PromotionPricing(PricingStrategy):
    """促销活动：7折"""
    def calculate(self, price):
        return price * 0.7

# 使用策略
class ShoppingCart:
    def __init__(self, pricing_strategy: PricingStrategy):
        self.pricing_strategy = pricing_strategy
        self.items = []

    def add_item(self, item):
        self.items.append(item)

    def calculate_total(self):
        subtotal = sum(item['price'] for item in self.items)
        return self.pricing_strategy.calculate(subtotal)

    def set_pricing_strategy(self, strategy: PricingStrategy):
        """动态切换策略"""
        self.pricing_strategy = strategy

# 使用示例
cart = ShoppingCart(RegularPricing())
cart.add_item({'name': '商品A', 'price': 100})
cart.add_item({'name': '商品B', 'price': 200})

print(f"普通用户价格: {cart.calculate_total()}")  # 300

# 切换为 VIP 定价策略
cart.set_pricing_strategy(VIPPricing())
print(f"VIP用户价格: {cart.calculate_total()}")   # 270

# 切换为促销定价
cart.set_pricing_strategy(PromotionPricing())
print(f"促销价格: {cart.calculate_total()}")      # 210
```

### 4.2 工厂模式创建策略

```python
class PricingStrategyFactory:
    """策略工厂"""
    _strategies = {
        'regular': RegularPricing,
        'vip': VIPPricing,
        'svip': SVIPPricing,
        'promotion': PromotionPricing,
    }

    @classmethod
    def create(cls, strategy_type):
        strategy_class = cls._strategies.get(strategy_type)
        if not strategy_class:
            raise ValueError(f"未知策略类型: {strategy_type}")
        return strategy_class()

    @classmethod
    def register(cls, name, strategy_class):
        """注册新策略"""
        cls._strategies[name] = strategy_class

# 使用工厂
strategy = PricingStrategyFactory.create('vip')
cart = ShoppingCart(strategy)

# 动态注册新策略
class FlashSalePricing(PricingStrategy):
    """限时抢购：5折"""
    def calculate(self, price):
        return price * 0.5

PricingStrategyFactory.register('flash_sale', FlashSalePricing)

# 使用新注册的策略
flash_strategy = PricingStrategyFactory.create('flash_sale')
cart.set_pricing_strategy(flash_strategy)
print(f"限时抢购价格: {cart.calculate_total()}")  # 150
```

---

## 5. 配置驱动

### 5.1 配置文件驱动功能

```python
# config.json
{
    "features": {
        "email_notification": true,
        "sms_notification": false,
        "wechat_notification": true,
        "discount": {
            "enabled": true,
            "rate": 0.1
        }
    },
    "plugins": [
        "markdown",
        "html",
        "csv_exporter"
    ]
}
```

```python
import json

class FeatureConfig:
    def __init__(self, config_file="config.json"):
        with open(config_file) as f:
            self.config = json.load(f)

    def is_enabled(self, feature_name):
        """检查功能是否启用"""
        features = self.config.get('features', {})
        return features.get(feature_name, False)

    def get_feature_config(self, feature_name):
        """获取功能配置"""
        features = self.config.get('features', {})
        return features.get(feature_name)

    def get_enabled_plugins(self):
        """获取启用的插件列表"""
        return self.config.get('plugins', [])

# 使用
config = FeatureConfig()

# 根据配置决定是否发送通知
if config.is_enabled('email_notification'):
    send_email(user, message)

if config.is_enabled('sms_notification'):
    send_sms(user, message)

if config.is_enabled('wechat_notification'):
    send_wechat(user, message)

# 获取折扣配置
discount_config = config.get_feature_config('discount')
if discount_config and discount_config['enabled']:
    discount_rate = discount_config['rate']
    price = price * (1 - discount_rate)
```

### 5.2 配置驱动的策略选择

```python
# config.yaml
pricing_strategy: "vip"  # regular, vip, svip, promotion
notification_channels:
  - email
  - wechat
database:
  type: "postgresql"
  connection:
    host: "localhost"
    port: 5432
```

```python
import yaml

class Application:
    def __init__(self, config_file="config.yaml"):
        with open(config_file) as f:
            self.config = yaml.safe_load(f)

        # 根据配置创建策略
        strategy_type = self.config['pricing_strategy']
        self.pricing_strategy = PricingStrategyFactory.create(strategy_type)

        # 根据配置创建通知器
        self.notifiers = []
        for channel in self.config['notification_channels']:
            if channel == 'email':
                self.notifiers.append(EmailNotifier())
            elif channel == 'wechat':
                self.notifiers.append(WeChatNotifier())

        # 根据配置创建数据库连接
        db_type = self.config['database']['type']
        db_config = self.config['database']['connection']
        self.db = DatabaseFactory.create(db_type, db_config)

    def calculate_price(self, price):
        return self.pricing_strategy.calculate(price)

    def notify(self, user, message):
        for notifier in self.notifiers:
            notifier.send(user, message)

# 更改配置文件即可改变应用行为，无需修改代码
app = Application("config.yaml")
```

---

## 6. 依赖注入

### 6.1 构造函数注入

```python
class UserService:
    def __init__(self, db: Database, cache: Cache, notifier: Notifier):
        self.db = db
        self.cache = cache
        self.notifier = notifier

    def create_user(self, user):
        # 使用注入的依赖
        self.db.save(user)
        self.cache.set(f"user:{user.id}", user)
        self.notifier.send(user, "欢迎注册")

# 注入依赖（可以轻松替换实现）
service = UserService(
    db=PostgreSQLDatabase(),
    cache=RedisCache(),
    notifier=EmailNotifier()
)

# 测试时注入 Mock 对象
test_service = UserService(
    db=MockDatabase(),
    cache=MockCache(),
    notifier=MockNotifier()
)
```

### 6.2 依赖注入容器

```python
class Container:
    """简单的依赖注入容器"""
    def __init__(self):
        self._services = {}
        self._singletons = {}

    def register(self, name, factory, singleton=False):
        """注册服务"""
        self._services[name] = {
            'factory': factory,
            'singleton': singleton
        }

    def get(self, name):
        """获取服务"""
        if name not in self._services:
            raise ValueError(f"服务 '{name}' 未注册")

        service_config = self._services[name]

        # 单例模式：只创建一次
        if service_config['singleton']:
            if name not in self._singletons:
                self._singletons[name] = service_config['factory']()
            return self._singletons[name]

        # 每次都创建新实例
        return service_config['factory']()

# 使用容器
container = Container()

# 注册服务
container.register('db', lambda: PostgreSQLDatabase(), singleton=True)
container.register('cache', lambda: RedisCache(), singleton=True)
container.register('notifier', lambda: EmailNotifier())

# 注册复合服务（依赖其他服务）
container.register('user_service', lambda: UserService(
    db=container.get('db'),
    cache=container.get('cache'),
    notifier=container.get('notifier')
))

# 获取服务
user_service = container.get('user_service')
user_service.create_user(user)
```

---

## 7. NOFX 的扩展性设计

### 7.1 策略接口

**文件**：`pkg/interfaces/strategy.go`

```go
type Strategy interface {
    Name() string
    ShouldOpenLong(data *MarketData) bool
    ShouldCloseLong(data *MarketData) bool
    ShouldOpenShort(data *MarketData) bool
    ShouldCloseShort(data *MarketData) bool
}
```

**扩展性**：
- 新增策略只需实现 `Strategy` 接口
- 不需要修改核心交易逻辑
- 遵循开闭原则

### 7.2 交易所接口

**文件**：`pkg/interfaces/trader.go`

```go
type Trader interface {
    Start() error
    Stop() error
    GetAccountInfo() (*Account, error)
    GetPositions() ([]Position, error)
    PlaceOrder(order *Order) error
}
```

**扩展性**：
- 支持币安、OKX、Hyperliquid 等多个交易所
- 添加新交易所只需实现接口
- 统一的调用方式

### 7.3 配置驱动

**文件**：`config/config.yaml`

```yaml
traders:
  - exchange: "binance"
    strategy: "ma_cross"
    parameters:
      short_period: 10
      long_period: 20

  - exchange: "okx"
    strategy: "rsi"
    parameters:
      period: 14
      overbought: 70
      oversold: 30
```

**扩展性**：
- 通过配置文件添加新的交易员
- 通过配置文件切换策略
- 无需修改代码

### 7.4 工厂模式创建对象

**文件**：`internal/trader/factory.go`

```go
func CreateTrader(config *Config) (interfaces.Trader, error) {
    switch config.Exchange {
    case "binance":
        return NewBinanceTrader(config)
    case "okx":
        return NewOKXTrader(config)
    case "hyperliquid":
        return NewHyperliquidTrader(config)
    default:
        return nil, fmt.Errorf("unsupported exchange: %s", config.Exchange)
    }
}

func CreateStrategy(name string, params map[string]interface{}) (interfaces.Strategy, error) {
    switch name {
    case "ma_cross":
        return NewMACrossStrategy(params)
    case "rsi":
        return NewRSIStrategy(params)
    default:
        return nil, fmt.Errorf("unsupported strategy: %s", name)
    }
}
```

---

## 8. 实战练习

### 练习 1：实现导出器插件系统

设计一个数据导出系统，支持多种导出格式。

**要求**：
- 定义 `Exporter` 接口
- 实现 CSV、JSON、XML 导出器
- 使用插件注册机制
- 支持动态添加新的导出格式

```python
# 提示
class Exporter(ABC):
    @abstractmethod
    def export(self, data, filename):
        pass

registry = PluginRegistry()

@registry.register("csv")
class CSVExporter(Exporter):
    def export(self, data, filename):
        # 实现 CSV 导出
        pass
```

### 练习 2：配置驱动的权限系统

实现一个权限系统，通过配置文件控制用户权限。

**要求**：
- 定义角色和权限配置（YAML/JSON）
- 实现权限检查器
- 支持动态修改权限（重新加载配置）

```yaml
# permissions.yaml
roles:
  admin:
    - create_user
    - delete_user
    - edit_user
    - view_user
  editor:
    - edit_user
    - view_user
  viewer:
    - view_user
```

### 练习 3：为 NOFX 添加新策略

为 NOFX 实现一个新的交易策略。

**要求**：
- 实现 `Strategy` 接口
- 添加配置支持
- 不修改现有代码

---

## 本章总结

### 扩展性设计原则

1. **开闭原则**：对扩展开放，对修改关闭
2. **依赖倒置**：依赖抽象，不依赖具体
3. **里氏替换**：子类可以替换父类

### 扩展性实现方式

1. **插件系统**：动态注册和加载
2. **策略模式**：算法可互相替换
3. **配置驱动**：通过配置改变行为
4. **依赖注入**：解耦依赖关系
5. **工厂模式**：统一创建对象

### 最佳实践

1. **接口优先**：先定义接口，再实现
2. **最小知识**：模块只知道必要的信息
3. **单一职责**：每个类只做一件事
4. **配置外部化**：行为由配置决定

---

**💡 记住**：好的扩展性设计不是一次完成的，而是在实践中不断重构和改进的。先让代码工作，再让代码优雅！
