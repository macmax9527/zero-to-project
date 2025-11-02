# 第3章扩展：Python 模块调用基础

> **本文档目标**：彻底搞懂函数、类、模块、接口，以及它们如何协作

---

## 📋 本文大纲

1. [函数 vs 类：最简单的理解](#1-函数-vs-类最简单的理解)
2. [模块和文件：如何组织代码](#2-模块和文件如何组织代码)
3. [import 导入：如何调用其他文件](#3-import-导入如何调用其他文件)
4. [程序运行流程：从 main.py 开始](#4-程序运行流程从-mainpy-开始)
5. [接口的真正含义](#5-接口的真正含义)
6. [完整项目示例：餐厅点餐系统](#6-完整项目示例餐厅点餐系统)
7. [NOFX 调用流程分析](#7-nofx-调用流程分析)

---

## 1. 函数 vs 类：最简单的理解

### 1.1 函数（def）：单一工具

**比喻**：函数就像是**一个独立的工具**

```python
# 函数：一个计算器
def add(a, b):
    """这是一个加法函数"""
    return a + b

# 使用函数
result = add(5, 3)
print(result)  # 输出：8
```

**函数的特点**：
- 做一件具体的事情
- 输入参数，输出结果
- 调用完就结束了，不保存状态

```
函数就像：洗衣机的"洗涤"按钮
┌─────────────┐
│   洗涤按钮   │ → 输入：脏衣服 → 输出：洗好的衣服
└─────────────┘
按一次就执行一次，执行完就结束
```

### 1.2 类（class）：工具箱 + 数据

**比喻**：类就像是**一个完整的工具箱，里面有多个工具和数据**

```python
# 类：一个计算器对象
class Calculator:
    """这是一个计算器类"""

    def __init__(self):
        """初始化：设置初始状态"""
        self.history = []  # 存储历史记录（数据）

    def add(self, a, b):
        """加法功能"""
        result = a + b
        self.history.append(f"{a} + {b} = {result}")
        return result

    def subtract(self, a, b):
        """减法功能"""
        result = a - b
        self.history.append(f"{a} - {b} = {result}")
        return result

    def show_history(self):
        """显示历史记录"""
        print("历史记录：")
        for record in self.history:
            print(f"  {record}")

# 创建一个计算器实例（对象）
calc = Calculator()

# 使用计算器的各种功能
calc.add(5, 3)
calc.subtract(10, 4)
calc.add(7, 2)

# 查看历史记录
calc.show_history()
```

**输出**：
```
历史记录：
  5 + 3 = 8
  10 - 4 = 6
  7 + 2 = 9
```

**类的特点**：
- 包含多个相关的功能（方法）
- 可以保存数据（属性）
- 创建对象后，对象会记住状态

```
类就像：一台完整的洗衣机
┌─────────────────────┐
│   洗衣机（类）       │
├─────────────────────┤
│ 数据（属性）：        │
│  - 当前水位          │
│  - 剩余时间          │
│  - 洗涤模式          │
├─────────────────────┤
│ 功能（方法）：        │
│  - 洗涤()            │
│  - 漂洗()            │
│  - 脱水()            │
│  - 查看状态()        │
└─────────────────────┘
创建实例后，洗衣机会记住自己的状态
```

### 1.3 什么时候用函数？什么时候用类？

| 场景 | 使用 | 原因 |
|------|------|------|
| 简单计算 | 函数 | 不需要保存状态 |
| 数据处理 | 函数 | 一次性操作 |
| 需要记住状态 | 类 | 对象会保存数据 |
| 多个相关功能 | 类 | 组织在一起更清晰 |
| 需要创建多个实例 | 类 | 每个实例独立 |

**示例对比**：

```python
# ✅ 用函数：简单计算，不需要状态
def calculate_area(width, height):
    return width * height

area = calculate_area(5, 3)

# ✅ 用类：需要保存用户信息
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
        self.login_count = 0

    def login(self):
        self.login_count += 1
        print(f"{self.name} 登录了，共登录 {self.login_count} 次")

user = User("张三", "zhang@example.com")
user.login()  # 张三 登录了，共登录 1 次
user.login()  # 张三 登录了，共登录 2 次
```

---

## 2. 模块和文件：如何组织代码

### 2.1 模块就是文件

**核心概念**：一个 `.py` 文件 = 一个模块

```
项目目录：
my_project/
├── main.py          ← 这是一个模块
├── calculator.py    ← 这是一个模块
└── utils.py         ← 这是一个模块

每个 .py 文件都是一个独立的模块
```

### 2.2 为什么要分成多个文件？

**比喻**：就像整理房间

```
❌ 所有东西堆在一个房间：
main.py (2000 行代码)
├── 用户管理代码
├── 订单管理代码
├── 支付代码
├── 通知代码
└── 工具函数
（找东西很困难，维护很麻烦）

✅ 分门别类：
project/
├── main.py          (主程序，100 行)
├── user.py          (用户管理，200 行)
├── order.py         (订单管理，300 行)
├── payment.py       (支付功能，200 行)
└── utils.py         (工具函数，100 行)
（清晰明了，容易维护）
```

---

## 3. import 导入：如何调用其他文件

### 3.1 基础导入方式

**创建两个文件来演示**：

**文件1：calculator.py**（被导入的模块）
```python
# calculator.py
def add(a, b):
    """加法函数"""
    return a + b

def multiply(a, b):
    """乘法函数"""
    return a * b

class Calculator:
    """计算器类"""
    def __init__(self):
        self.result = 0

    def add(self, a, b):
        self.result = a + b
        return self.result
```

**文件2：main.py**（主程序）
```python
# main.py

# 方式1：导入整个模块
import calculator

result1 = calculator.add(5, 3)
print(f"5 + 3 = {result1}")

calc = calculator.Calculator()
result2 = calc.add(10, 20)
print(f"10 + 20 = {result2}")

# 方式2：只导入需要的函数
from calculator import add, multiply

result3 = add(7, 8)  # 直接用函数名，不需要 calculator.add
print(f"7 + 8 = {result3}")

# 方式3：导入类
from calculator import Calculator

calc2 = Calculator()  # 直接用类名
result4 = calc2.add(1, 2)
print(f"1 + 2 = {result4}")

# 方式4：导入所有（不推荐）
from calculator import *

result5 = multiply(3, 4)
print(f"3 * 4 = {result5}")
```

### 3.2 import 的工作原理

```python
# 当你写 import calculator 时，Python 做了什么？

import calculator
# ↓
# 1. Python 在当前目录找 calculator.py
# 2. 执行 calculator.py 中的所有代码
# 3. 把 calculator.py 中的函数和类"装"进 calculator 这个名字里
# 4. 你可以通过 calculator.函数名 或 calculator.类名 来使用

# 所以：
calculator.add(5, 3)
# 等于说："去 calculator 模块里，找到 add 函数，执行它"
```

**可视化**：

```
main.py 文件内容：
┌─────────────────────────┐
│ import calculator       │ ← 导入 calculator 模块
│                         │
│ result = calculator.add(5, 3)  │ ← 调用 calculator 模块中的 add 函数
└─────────────────────────┘
         ↓ 导入
┌─────────────────────────┐
│ calculator.py           │
├─────────────────────────┤
│ def add(a, b):          │ ← 这个函数可以被调用
│     return a + b        │
│                         │
│ def multiply(a, b):     │ ← 这个函数也可以被调用
│     return a * b        │
└─────────────────────────┘
```

### 3.3 不同目录的导入

**项目结构**：
```
my_project/
├── main.py
└── utils/
    ├── __init__.py      ← 这个文件让 utils 成为一个"包"
    ├── math_utils.py
    └── string_utils.py
```

**utils/math_utils.py**：
```python
# utils/math_utils.py
def square(n):
    return n * n
```

**main.py**：
```python
# main.py

# 导入 utils 文件夹下的 math_utils 模块
from utils.math_utils import square

result = square(5)
print(f"5 的平方 = {result}")  # 输出：5 的平方 = 25
```

**理解 `__init__.py`**：
```python
# utils/__init__.py
# 这个文件可以为空，它的作用是告诉 Python：
# "utils 是一个包（package），里面有多个模块"

# 如果你在 __init__.py 中写：
from .math_utils import square

# 那么在 main.py 中就可以直接：
from utils import square
```

---

## 4. 程序运行流程：从 main.py 开始

### 4.1 完整示例：订单系统

让我们创建一个完整的项目，看看程序是如何运行的。

**项目结构**：
```
order_system/
├── main.py           # 主程序（程序入口）
├── user.py           # 用户模块
├── product.py        # 商品模块
└── order.py          # 订单模块
```

**user.py**（用户模块）：
```python
# user.py
class User:
    """用户类"""
    def __init__(self, name, balance):
        self.name = name
        self.balance = balance

    def show_info(self):
        print(f"用户：{self.name}，余额：{self.balance} 元")

    def pay(self, amount):
        if self.balance >= amount:
            self.balance -= amount
            print(f"支付成功！剩余余额：{self.balance} 元")
            return True
        else:
            print("余额不足！")
            return False
```

**product.py**（商品模块）：
```python
# product.py
class Product:
    """商品类"""
    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock

    def show_info(self):
        print(f"商品：{self.name}，价格：{self.price} 元，库存：{self.stock}")

    def check_stock(self, quantity):
        return self.stock >= quantity

    def reduce_stock(self, quantity):
        if self.check_stock(quantity):
            self.stock -= quantity
            return True
        return False
```

**order.py**（订单模块）：
```python
# order.py
from datetime import datetime

class Order:
    """订单类"""
    def __init__(self, user, product, quantity):
        self.user = user
        self.product = product
        self.quantity = quantity
        self.total = product.price * quantity
        self.created_at = datetime.now()
        self.status = "待支付"

    def show_info(self):
        print("\n" + "="*40)
        print("订单详情：")
        print(f"用户：{self.user.name}")
        print(f"商品：{self.product.name}")
        print(f"数量：{self.quantity}")
        print(f"总价：{self.total} 元")
        print(f"状态：{self.status}")
        print(f"下单时间：{self.created_at.strftime('%Y-%m-%d %H:%M:%S')}")
        print("="*40)

    def process(self):
        """处理订单"""
        print(f"\n开始处理订单...")

        # 1. 检查库存
        if not self.product.check_stock(self.quantity):
            print("❌ 库存不足，订单失败")
            self.status = "已取消"
            return False

        # 2. 扣款
        if not self.user.pay(self.total):
            print("❌ 支付失败，订单失败")
            self.status = "已取消"
            return False

        # 3. 减库存
        self.product.reduce_stock(self.quantity)

        # 4. 订单完成
        self.status = "已完成"
        print("✅ 订单处理成功！")
        return True
```

**main.py**（主程序 - 程序入口）：
```python
# main.py
"""
这是程序的入口文件
当你运行 python main.py 时，程序从这里开始执行
"""

# 导入我们创建的模块
from user import User
from product import Product
from order import Order

def main():
    """主函数：程序的核心流程"""
    print("欢迎使用订单系统！\n")

    # 1. 创建用户
    user = User(name="张三", balance=1000)
    user.show_info()

    # 2. 创建商品
    product = Product(name="iPhone", price=5999, stock=10)
    product.show_info()

    # 3. 创建订单
    print("\n张三想买 1 台 iPhone...")
    order = Order(user=user, product=product, quantity=1)
    order.show_info()

    # 4. 处理订单
    order.process()

    # 5. 查看订单结果
    order.show_info()

    # 6. 查看用户和商品状态
    print("\n订单完成后的状态：")
    user.show_info()
    product.show_info()

# 程序入口
if __name__ == "__main__":
    # 当直接运行 main.py 时，这里的代码会执行
    main()
```

### 4.2 程序运行流程详解

**当你运行 `python main.py` 时，发生了什么？**

```
第1步：Python 读取 main.py
┌────────────────────────────────┐
│ 执行 import 语句：              │
│ from user import User          │ → 读取 user.py，加载 User 类
│ from product import Product    │ → 读取 product.py，加载 Product 类
│ from order import Order        │ → 读取 order.py，加载 Order 类
└────────────────────────────────┘

第2步：定义 main() 函数
┌────────────────────────────────┐
│ def main():                    │
│     ... 函数体 ...              │ → 只是定义，还没执行
└────────────────────────────────┘

第3步：执行 if __name__ == "__main__":
┌────────────────────────────────┐
│ if __name__ == "__main__":     │ → 条件为真（直接运行 main.py）
│     main()                     │ → 调用 main() 函数
└────────────────────────────────┘

第4步：main() 函数开始执行
┌────────────────────────────────┐
│ user = User(...)               │ → 创建 User 对象
│ product = Product(...)         │ → 创建 Product 对象
│ order = Order(...)             │ → 创建 Order 对象
│ order.process()                │ → 调用 Order 的 process 方法
└────────────────────────────────┘

第5步：order.process() 内部调用其他方法
┌────────────────────────────────┐
│ self.product.check_stock()     │ → 调用 Product 的方法
│ self.user.pay()                │ → 调用 User 的方法
│ self.product.reduce_stock()    │ → 调用 Product 的方法
└────────────────────────────────┘
```

**完整的调用链**：

```
运行：python main.py
  ↓
main.py: if __name__ == "__main__": main()
  ↓
main.py: main() 函数执行
  ↓
main.py: user = User("张三", 1000)
  ↓ 调用 user.py
user.py: User 类的 __init__ 被调用
  ↓ 返回
main.py: product = Product("iPhone", 5999, 10)
  ↓ 调用 product.py
product.py: Product 类的 __init__ 被调用
  ↓ 返回
main.py: order = Order(user, product, 1)
  ↓ 调用 order.py
order.py: Order 类的 __init__ 被调用
  ↓ 返回
main.py: order.process()
  ↓ 调用 order.py
order.py: process() 方法执行
  ↓
order.py: self.product.check_stock(1)
  ↓ 调用 product.py
product.py: check_stock() 方法执行，返回 True
  ↓ 返回
order.py: self.user.pay(5999)
  ↓ 调用 user.py
user.py: pay() 方法执行，返回 True
  ↓ 返回
order.py: self.product.reduce_stock(1)
  ↓ 调用 product.py
product.py: reduce_stock() 方法执行
  ↓ 返回
order.py: process() 方法结束
  ↓ 返回
main.py: main() 函数结束
  ↓
程序退出
```

### 4.3 运行这个示例

将上面的代码保存为对应的文件，然后运行：

```bash
python main.py
```

**输出**：
```
欢迎使用订单系统！

用户：张三，余额：1000 元
商品：iPhone，价格：5999 元，库存：10

张三想买 1 台 iPhone...

========================================
订单详情：
用户：张三
商品：iPhone
数量：1
总价：5999 元
状态：待支付
下单时间：2024-11-01 10:30:45
========================================

开始处理订单...
余额不足！
❌ 支付失败，订单失败

========================================
订单详情：
用户：张三
商品：iPhone
数量：1
总价：5999 元
状态：已取消
下单时间：2024-11-01 10:30:45
========================================

订单完成后的状态：
用户：张三，余额：1000 元
商品：iPhone，价格：5999 元，库存：10
```

---

## 5. 接口的真正含义

> 💡 **如果你对接口和抽象基类还是不太理解，强烈推荐先阅读：**
>
> 📘 [**接口和抽象基类深度解析**](chapter03-interface-deep-dive.md)
>
> 这份补充文档用更详细、更通俗的方式讲解接口，包含：
> - 生活化比喻（厨师招聘、支付系统）
> - 一步步教你使用ABC
> - 常见错误和解决方案
> - 完整的实战案例（交易策略、数据库访问）
>
> **预计阅读时间：30-40分钟**

### 5.1 接口不是一个具体的东西！

**误区**：很多人以为"接口"是一个要创建的文件或函数
**真相**：接口是一个**约定**，一个**规范**

**比喻**：接口就像**电源插座的标准**

```
中国的电源插座（接口标准）：
┌─────────────┐
│  ⚫  ⚫  ⚫  │  ← 三个孔（接口定义）
└─────────────┘

任何符合这个标准的电器都能用：
- 台灯 ✅
- 手机充电器 ✅
- 电脑 ✅
- 冰箱 ✅

它们的内部结构完全不同，但都能插入同一个插座
因为它们都遵循了同一个"接口标准"
```

### 5.2 Python 中的接口：抽象基类

在 Python 中，接口通常用**抽象基类（Abstract Base Class）**来表示。

**示例：交易所接口**

```python
# trader_interface.py
from abc import ABC, abstractmethod

class TraderInterface(ABC):
    """
    交易所接口（抽象基类）
    这是一个"规范"，定义了所有交易所都必须实现的方法
    """

    @abstractmethod
    def get_account(self):
        """
        获取账户信息
        所有交易所都必须实现这个方法
        """
        pass

    @abstractmethod
    def get_price(self, symbol):
        """
        获取价格
        所有交易所都必须实现这个方法
        """
        pass

    @abstractmethod
    def buy(self, symbol, quantity):
        """
        买入
        所有交易所都必须实现这个方法
        """
        pass

    @abstractmethod
    def sell(self, symbol, quantity):
        """
        卖出
        所有交易所都必须实现这个方法
        """
        pass
```

**实现1：币安交易所**

```python
# binance_trader.py
from trader_interface import TraderInterface

class BinanceTrader(TraderInterface):
    """币安交易所实现"""

    def __init__(self, api_key):
        self.api_key = api_key
        self.exchange_name = "币安"

    def get_account(self):
        """币安的方式获取账户"""
        print(f"[{self.exchange_name}] 获取账户信息...")
        return {"balance": 1000, "exchange": "binance"}

    def get_price(self, symbol):
        """币安的方式获取价格"""
        print(f"[{self.exchange_name}] 获取 {symbol} 价格...")
        return 50000  # 模拟价格

    def buy(self, symbol, quantity):
        """币安的方式买入"""
        print(f"[{self.exchange_name}] 买入 {quantity} {symbol}")
        return {"status": "success", "order_id": "binance_123"}

    def sell(self, symbol, quantity):
        """币安的方式卖出"""
        print(f"[{self.exchange_name}] 卖出 {quantity} {symbol}")
        return {"status": "success", "order_id": "binance_456"}
```

**实现2：Hyperliquid 交易所**

```python
# hyperliquid_trader.py
from trader_interface import TraderInterface

class HyperliquidTrader(TraderInterface):
    """Hyperliquid 交易所实现"""

    def __init__(self, private_key):
        self.private_key = private_key
        self.exchange_name = "Hyperliquid"

    def get_account(self):
        """Hyperliquid 的方式获取账户（方式不同）"""
        print(f"[{self.exchange_name}] 通过钱包地址获取账户...")
        return {"balance": 2000, "exchange": "hyperliquid"}

    def get_price(self, symbol):
        """Hyperliquid 的方式获取价格"""
        print(f"[{self.exchange_name}] 获取 {symbol} 价格...")
        return 50100  # 模拟价格（可能略有不同）

    def buy(self, symbol, quantity):
        """Hyperliquid 的方式买入（实现不同）"""
        print(f"[{self.exchange_name}] 使用区块链签名买入 {quantity} {symbol}")
        return {"status": "success", "tx_hash": "0xabc123"}

    def sell(self, symbol, quantity):
        """Hyperliquid 的方式卖出"""
        print(f"[{self.exchange_name}] 使用区块链签名卖出 {quantity} {symbol}")
        return {"status": "success", "tx_hash": "0xdef456"}
```

**主程序：统一使用接口**

```python
# main.py
from binance_trader import BinanceTrader
from hyperliquid_trader import HyperliquidTrader

def execute_strategy(trader):
    """
    交易策略函数

    重点：这个函数不关心 trader 是币安还是 Hyperliquid
    只要 trader 实现了接口，就能正常工作
    """
    print("\n" + "="*50)
    print("开始执行交易策略...")
    print("="*50)

    # 1. 获取账户
    account = trader.get_account()
    print(f"账户余额：{account['balance']}")

    # 2. 获取价格
    price = trader.get_price("BTC")
    print(f"BTC 价格：{price}")

    # 3. 买入
    if price < 51000:
        result = trader.buy("BTC", 0.1)
        print(f"买入结果：{result}")

    # 4. 卖出
    if price > 49000:
        result = trader.sell("BTC", 0.05)
        print(f"卖出结果：{result}")

# 使用币安交易所
print("使用币安交易所：")
binance = BinanceTrader(api_key="binance_key_123")
execute_strategy(binance)

# 使用 Hyperliquid 交易所
print("\n\n使用 Hyperliquid 交易所：")
hyperliquid = HyperliquidTrader(private_key="0x123abc")
execute_strategy(hyperliquid)

# 关键点：
# execute_strategy() 函数不需要修改，就能支持不同的交易所
# 因为它们都实现了同一个接口（TraderInterface）
```

**运行结果**：

```
使用币安交易所：

==================================================
开始执行交易策略...
==================================================
[币安] 获取账户信息...
账户余额：1000
[币安] 获取 BTC 价格...
BTC 价格：50000
[币安] 买入 0.1 BTC
买入结果：{'status': 'success', 'order_id': 'binance_123'}
[币安] 卖出 0.05 BTC
卖出结果：{'status': 'success', 'order_id': 'binance_456'}


使用 Hyperliquid 交易所：

==================================================
开始执行交易策略...
==================================================
[Hyperliquid] 通过钱包地址获取账户...
账户余额：2000
[Hyperliquid] 获取 BTC 价格...
BTC 价格：50100
[Hyperliquid] 使用区块链签名买入 0.1 BTC
买入结果：{'status': 'success', 'tx_hash': '0xabc123'}
[Hyperliquid] 使用区块链签名卖出 0.05 BTC
卖出结果：{'status': 'success', 'tx_hash': '0xdef456'}
```

### 5.3 接口的核心价值

**问题**：是不是每个模块都要搭建一个接口？

**答案**：不是！只有在以下情况下才需要接口：

| 场景 | 是否需要接口 | 原因 |
|------|-------------|------|
| 多个不同实现 | ✅ 需要 | 币安、Hyperliquid 都是交易所，但实现不同 |
| 可能替换的功能 | ✅ 需要 | 数据库可能从 MySQL 换到 PostgreSQL |
| 需要统一调用 | ✅ 需要 | 不管哪个交易所，都用 `trader.buy()` |
| 只有一个实现 | ❌ 不需要 | 如果只用币安，不需要接口 |
| 不会变化的模块 | ❌ 不需要 | 配置模块通常不需要接口 |

**接口的好处**：

```python
# 没有接口：每个交易所的调用方式不同
if exchange == "binance":
    binance_api.place_order(symbol, quantity, "BUY")
elif exchange == "hyperliquid":
    hyperliquid_api.execute_trade(symbol, quantity, side="LONG")
# 添加新交易所时，需要修改所有代码 ❌

# 有接口：统一调用方式
trader.buy(symbol, quantity)
# 添加新交易所时，只需实现接口，不需要改其他代码 ✅
```

---

## 6. 完整项目示例：餐厅点餐系统

让我创建一个更完整的示例，展示模块如何协作。

**项目结构**：
```
restaurant/
├── main.py               # 主程序
├── menu.py               # 菜单模块
├── order.py              # 订单模块
├── kitchen.py            # 厨房模块
└── payment.py            # 支付模块
```

**menu.py**（菜单模块）：
```python
# menu.py
class Dish:
    """菜品类"""
    def __init__(self, name, price, cooking_time):
        self.name = name
        self.price = price
        self.cooking_time = cooking_time  # 烹饪时间（分钟）

class Menu:
    """菜单类"""
    def __init__(self):
        self.dishes = {
            "宫保鸡丁": Dish("宫保鸡丁", 38, 15),
            "麻婆豆腐": Dish("麻婆豆腐", 28, 10),
            "清蒸鲈鱼": Dish("清蒸鲈鱼", 68, 20),
            "米饭": Dish("米饭", 3, 1)
        }

    def show(self):
        """显示菜单"""
        print("\n" + "="*40)
        print("  餐厅菜单")
        print("="*40)
        for i, (name, dish) in enumerate(self.dishes.items(), 1):
            print(f"{i}. {name:10s} {dish.price:>5.0f}元  {dish.cooking_time}分钟")
        print("="*40)

    def get_dish(self, name):
        """获取菜品"""
        return self.dishes.get(name)
```

**order.py**（订单模块）：
```python
# order.py
from datetime import datetime

class OrderItem:
    """订单项"""
    def __init__(self, dish, quantity):
        self.dish = dish
        self.quantity = quantity
        self.subtotal = dish.price * quantity

class Order:
    """订单类"""
    def __init__(self, table_number):
        self.table_number = table_number
        self.items = []
        self.created_at = datetime.now()
        self.status = "待制作"

    def add_item(self, dish, quantity):
        """添加菜品"""
        item = OrderItem(dish, quantity)
        self.items.append(item)
        print(f"✅ 已添加：{dish.name} x {quantity}")

    def get_total(self):
        """计算总价"""
        return sum(item.subtotal for item in self.items)

    def get_total_time(self):
        """计算总烹饪时间"""
        return max(item.dish.cooking_time for item in self.items) if self.items else 0

    def show(self):
        """显示订单"""
        print("\n" + "="*40)
        print(f"  {self.table_number}号桌 订单")
        print("="*40)
        print("菜品                 数量    小计")
        print("-"*40)
        for item in self.items:
            print(f"{item.dish.name:15s}  {item.quantity:3d}   {item.subtotal:6.2f}元")
        print("-"*40)
        print(f"总计：{self.get_total():.2f}元")
        print(f"预计等待时间：{self.get_total_time()}分钟")
        print(f"状态：{self.status}")
        print("="*40)
```

**kitchen.py**（厨房模块）：
```python
# kitchen.py
import time

class Kitchen:
    """厨房类"""
    def __init__(self):
        self.queue = []  # 制作队列

    def receive_order(self, order):
        """接收订单"""
        self.queue.append(order)
        print(f"\n👨‍🍳 厨房收到 {order.table_number}号桌 的订单")
        order.status = "制作中"

    def cook(self, order):
        """制作菜品"""
        print(f"\n👨‍🍳 开始制作 {order.table_number}号桌 的菜品...")

        for item in order.items:
            print(f"   制作 {item.dish.name} x {item.quantity}...")
            time.sleep(1)  # 模拟制作过程（实际是 item.dish.cooking_time 分钟）

        order.status = "已完成"
        print(f"✅ {order.table_number}号桌 的菜品制作完成！")
```

**payment.py**（支付模块）：
```python
# payment.py
class Payment:
    """支付类"""
    def __init__(self):
        self.transactions = []

    def pay(self, order, amount, method="现金"):
        """处理支付"""
        total = order.get_total()

        print(f"\n💰 {order.table_number}号桌 结账")
        print(f"   总金额：{total:.2f}元")
        print(f"   支付方式：{method}")
        print(f"   收到：{amount:.2f}元")

        if amount >= total:
            change = amount - total
            if change > 0:
                print(f"   找零：{change:.2f}元")
            print("✅ 支付成功！")

            # 记录交易
            self.transactions.append({
                "table": order.table_number,
                "total": total,
                "method": method
            })
            return True
        else:
            print(f"❌ 金额不足！还差 {total - amount:.2f}元")
            return False

    def show_daily_report(self):
        """显示日报表"""
        print("\n" + "="*40)
        print("  今日营业报表")
        print("="*40)
        total_revenue = sum(t["total"] for t in self.transactions)
        print(f"订单数：{len(self.transactions)}")
        print(f"总营业额：{total_revenue:.2f}元")
        print("="*40)
```

**main.py**（主程序）：
```python
# main.py
"""
餐厅点餐系统
展示模块如何协作
"""

from menu import Menu
from order import Order
from kitchen import Kitchen
from payment import Payment

def main():
    """主流程"""
    print("\n🍽️  欢迎来到美食餐厅！")

    # 1. 创建各个模块
    menu = Menu()
    kitchen = Kitchen()
    payment_system = Payment()

    # 2. 顾客到达，分配桌号
    table_number = 5
    print(f"\n👥 顾客坐在 {table_number}号桌")

    # 3. 显示菜单
    menu.show()

    # 4. 顾客点餐
    print(f"\n{table_number}号桌 开始点餐...")
    order = Order(table_number)

    # 点菜
    order.add_item(menu.get_dish("宫保鸡丁"), 2)
    order.add_item(menu.get_dish("麻婆豆腐"), 1)
    order.add_item(menu.get_dish("米饭"), 3)

    # 显示订单
    order.show()

    # 5. 订单发送到厨房
    kitchen.receive_order(order)

    # 6. 厨房制作
    kitchen.cook(order)

    # 7. 上菜
    print(f"\n🍽️  {table_number}号桌 的菜已上齐，请慢用！")
    order.show()

    # 8. 顾客结账
    payment_system.pay(order, amount=200, method="微信支付")

    # 9. 模拟更多顾客...
    print("\n" + "-"*50)
    print("模拟更多顾客...")
    print("-"*50)

    # 第二桌
    order2 = Order(table_number=8)
    order2.add_item(menu.get_dish("清蒸鲈鱼"), 1)
    order2.add_item(menu.get_dish("米饭"), 2)
    kitchen.receive_order(order2)
    kitchen.cook(order2)
    payment_system.pay(order2, amount=100, method="支付宝")

    # 10. 显示日报表
    payment_system.show_daily_report()

# 程序入口
if __name__ == "__main__":
    main()
```

### 6.1 模块协作流程图

```
程序启动：python main.py
  ↓
main() 函数执行
  ↓
创建模块实例
  ├─ menu = Menu()          ← 菜单模块
  ├─ kitchen = Kitchen()    ← 厨房模块
  └─ payment = Payment()    ← 支付模块
  ↓
menu.show()
  ↓ 调用 menu.py
  显示菜单
  ↓
order = Order(5)
  ↓ 调用 order.py
  创建订单对象
  ↓
order.add_item(dish, quantity)
  ↓ 调用 order.py
  添加菜品到订单
  ↓
kitchen.receive_order(order)
  ↓ 调用 kitchen.py
  厨房接收订单
  ↓
kitchen.cook(order)
  ↓ 调用 kitchen.py
  厨房制作菜品（修改 order.status）
  ↓
payment.pay(order, 200)
  ↓ 调用 payment.py
  处理支付（读取 order.get_total()）
  ↓
程序结束
```

**关键点**：
1. **main.py** 是"总指挥"，协调各个模块
2. 各个模块各司其职：
   - `menu.py` 负责菜单
   - `order.py` 负责订单
   - `kitchen.py` 负责制作
   - `payment.py` 负责支付
3. 模块之间通过对象传递数据（如 `order` 对象）

---

## 7. NOFX 调用流程分析

现在让我们分析 NOFX 的真实调用流程。

### 7.1 NOFX 启动流程

**主程序**：`main.go`

```go
// main.go（简化版）
package main

import (
    "nofx/config"
    "nofx/manager"
    "nofx/api"
)

func main() {
    // 1. 加载配置
    cfg := config.LoadConfig("config.json")

    // 2. 创建 TraderManager
    tm := manager.NewTraderManager()

    // 3. 添加 Trader
    for _, traderCfg := range cfg.Traders {
        tm.AddTrader(traderCfg)
    }

    // 4. 启动所有 Trader
    tm.StartAll()

    // 5. 启动 API 服务器
    server := api.NewServer(tm, cfg.APIServerPort)
    server.Start()
}
```

### 7.2 调用流程图

```
启动：go run main.go
  ↓
main() 函数
  ↓
├─1. config.LoadConfig("config.json")
│   ↓ 调用 config/config.go
│   读取 JSON 配置文件
│   ↓ 返回
│   cfg 对象（包含所有配置）
│   ↓
├─2. manager.NewTraderManager()
│   ↓ 调用 manager/trader_manager.go
│   创建 TraderManager 对象
│   ↓ 返回
│   tm 对象（管理所有 trader）
│   ↓
├─3. tm.AddTrader(traderCfg)
│   ↓ 调用 manager/trader_manager.go
│   ├─ trader.NewAutoTrader(traderCfg)
│   │   ↓ 调用 trader/auto_trader.go
│   │   ├─ 根据 traderCfg.Exchange 创建交易所客户端
│   │   │   ↓ 如果是 "binance"
│   │   │   ├─ 调用 trader/binance_futures.go
│   │   │   ├─ BinanceFutures.NewClient()
│   │   │   └─ 返回 BinanceFutures 对象
│   │   │   ↓ 如果是 "hyperliquid"
│   │   │   ├─ 调用 trader/hyperliquid_trader.go
│   │   │   ├─ HyperliquidTrader.New()
│   │   │   └─ 返回 HyperliquidTrader 对象
│   │   └─ 返回 AutoTrader 对象
│   └─ 存储到 tm.traders map
│   ↓
├─4. tm.StartAll()
│   ↓ 调用 manager/trader_manager.go
│   for each trader:
│       go trader.Run()  ← 启动 goroutine（并发）
│       ↓ 调用 trader/auto_trader.go
│       ├─ 无限循环：
│       │   ├─ GetAccount() ← 调用交易所接口
│       │   ├─ GetPositions() ← 调用交易所接口
│       │   ├─ decision_engine.Decide() ← 调用 AI 决策
│       │   ├─ 执行交易操作
│       │   └─ sleep(ScanInterval)
│       └─ （持续运行）
│   ↓
└─5. api.NewServer(tm, port)
    ↓ 调用 api/server.go
    ├─ 创建 Gin 路由
    ├─ 设置路由：/api/account, /api/positions, etc.
    └─ server.Start() ← 启动 HTTP 服务器
        ↓
        监听端口 8080
        等待前端请求
        ↓
        当收到请求 GET /api/account?trader_id=qwen
        ↓
        handleAccount()
        ↓
        tm.GetTrader("qwen")
        ↓
        trader.GetAccountInfo()
        ↓
        trader.GetAccount() ← 调用交易所接口
        ↓
        返回 JSON 响应给前端
```

### 7.3 关键接口：Trader Interface

**文件**：`trader/interface.go`

```go
// trader/interface.go
package trader

// Trader 接口（所有交易所都要实现）
type Trader interface {
    GetAccount() (Account, error)           // 获取账户
    GetPositions() ([]Position, error)      // 获取持仓
    OpenLong(symbol string, quantity float64, leverage int) error
    OpenShort(symbol string, quantity float64, leverage int) error
    ClosePosition(symbol string) error
}

// BinanceFutures 实现 Trader 接口
// trader/binance_futures.go
type BinanceFutures struct {
    client *binance.Client
}

func (b *BinanceFutures) GetAccount() (Account, error) {
    // 币安的实现方式
    // ...
}

// HyperliquidTrader 实现 Trader 接口
// trader/hyperliquid_trader.go
type HyperliquidTrader struct {
    info *hyperliquid.Info
}

func (h *HyperliquidTrader) GetAccount() (Account, error) {
    // Hyperliquid 的实现方式
    // ...
}
```

**为什么需要接口？**

```go
// AutoTrader 不需要知道具体是哪个交易所
type AutoTrader struct {
    trader Trader  // 接口类型
    // ...
}

func (at *AutoTrader) Run() {
    // 统一调用方式，不管是币安还是 Hyperliquid
    account, _ := at.trader.GetAccount()
    positions, _ := at.trader.GetPositions()
    // ...
}

// 创建时传入具体实现
binanceTrader := &BinanceFutures{...}
autoTrader := NewAutoTrader(binanceTrader)  // 传入币安实现

hyperliquidTrader := &HyperliquidTrader{...}
autoTrader2 := NewAutoTrader(hyperliquidTrader)  // 传入 Hyperliquid 实现

// 两个 autoTrader 的代码完全一样，但连接不同的交易所
```

### 7.4 完整数据流

```
前端（浏览器）
  ↓ HTTP GET /api/account?trader_id=qwen
API 服务器（api/server.go）
  ↓ handleAccount()
TraderManager（manager/trader_manager.go）
  ↓ GetTrader("qwen")
  返回 AutoTrader 对象
  ↓
AutoTrader（trader/auto_trader.go）
  ↓ GetAccountInfo()
  ↓ at.trader.GetAccount()
  ↓
Trader 接口（trader/interface.go）
  ↓ （根据实际类型调用）
  ↓ 如果是 BinanceFutures
  ├─ BinanceFutures.GetAccount()
  │   ↓ 调用币安 API
  │   返回账户数据
  │   ↓
  └─ 返回给 AutoTrader
  ↓
  返回给 API 服务器
  ↓
  返回 JSON 给前端
  ↓
前端渲染显示
```

---

## 📝 总结

### 核心概念回顾

| 概念 | 定义 | 比喻 |
|------|------|------|
| **函数** | 做一件事的代码块 | 一个工具 |
| **类** | 包含数据和方法的对象 | 一台完整的机器 |
| **模块** | 一个 .py 文件 | 一个房间 |
| **导入** | 使用其他文件的代码 | 从另一个房间拿工具 |
| **接口** | 定义行为的规范 | 电源插座标准 |

### 关键要点

1. **模块 = 文件**：每个 `.py` 文件就是一个模块
2. **import 导入**：`import module` 就是加载另一个文件的代码
3. **类包含多个方法**：类是一个对象，可以保存数据和提供多个功能
4. **接口是规范**：定义了"必须实现哪些方法"，不是一个具体的文件
5. **main.py 是入口**：程序从 `if __name__ == "__main__":` 开始执行
6. **不是每个模块都需要接口**：只有需要多个不同实现时才用接口

### 学习建议

1. **动手敲代码**：把本文的示例都敲一遍，运行看效果
2. **修改参数**：改变一些值，看看会发生什么
3. **添加功能**：尝试给餐厅系统添加新功能（如折扣、会员）
4. **分析 NOFX**：用本文的知识，分析 NOFX 的每个文件

### 下一步

- 返回 [第3章：模块化拆分](chapter03-module-design.md)
- 继续学习 [第4章：配置系统](chapter04-configuration-system.md)
- 查看完整的 [餐厅点餐系统代码](../examples/restaurant/)（如果有）

---

**💡 记住**：编程就像搭积木，每个模块（文件）是一块积木，通过 import 把它们组合起来。接口就是积木的接口标准，保证不同的积木能拼在一起！
