# 第15章：测试和调试 - 质量保证

> **本章目标**：学会测试和调试技巧，提高代码质量

---

## 📋 本章大纲

1. [为什么需要测试](#1-为什么需要测试)
2. [测试类型](#2-测试类型)
3. [单元测试](#3-单元测试)
4. [调试技巧](#4-调试技巧)
5. [常见Bug模式](#5-常见bug模式)
6. [NOFX 的测试实践](#6-nofx-的测试实践)
7. [实战练习](#7-实战练习)

**预计学习时间**：3-4 小时

---

## 1. 为什么需要测试

### 1.1 测试的价值

```python
# 没有测试的代码
def calculate_discount(price, discount_rate):
    return price * discount_rate

# 看起来很简单，但有bug！
# 折扣后价格应该是 price * (1 - discount_rate)

# 如果没有测试，用户使用时才会发现：
result = calculate_discount(100, 0.1)  # 期望90，实际得到10 ❌
```

**测试的作用**：
- ✅ 发现 bug
- ✅ 防止回归（改代码时不破坏原功能）
- ✅ 文档作用（测试展示如何使用）
- ✅ 提高信心（敢于重构）

---

## 2. 测试类型

### 2.1 测试金字塔

```
             ┌─────────┐
             │ UI测试  │  ← 少量（慢、脆弱）
             └─────────┘
         ┌───────────────┐
         │  集成测试      │  ← 适量（测试模块协作）
         └───────────────┘
    ┌──────────────────────┐
    │     单元测试          │  ← 大量（快、稳定）
    └──────────────────────┘
```

### 2.2 测试类型对比

| 类型 | 测试范围 | 速度 | 成本 | 比例 |
|------|---------|------|------|------|
| **单元测试** | 单个函数/类 | 快 | 低 | 70% |
| **集成测试** | 多个模块 | 中 | 中 | 20% |
| **端到端测试** | 完整流程 | 慢 | 高 | 10% |

---

## 3. 单元测试

### 3.1 第一个测试

```python
# calculator.py
def add(a, b):
    return a + b

# test_calculator.py
import pytest

def test_add():
    # 测试正常情况
    assert add(1, 2) == 3
    assert add(0, 0) == 0
    assert add(-1, 1) == 0

def test_add_floats():
    # 测试浮点数
    result = add(0.1, 0.2)
    assert abs(result - 0.3) < 0.0001  # 浮点数比较

def test_add_strings():
    # 测试边界情况
    with pytest.raises(TypeError):
        add("hello", "world")
```

**运行测试**：
```bash
pytest test_calculator.py

# 输出：
# test_calculator.py::test_add PASSED
# test_calculator.py::test_add_floats PASSED
# test_calculator.py::test_add_strings PASSED
# ======================== 3 passed in 0.01s =========================
```

### 3.2 测试用例设计

**AAA 模式**：Arrange（准备）- Act（执行）- Assert（断言）

```python
def test_create_order():
    # Arrange：准备测试数据
    user = User(id=1, balance=1000)
    product = Product(id=1, price=500)

    # Act：执行操作
    order = Order.create(user, product, quantity=1)

    # Assert：验证结果
    assert order.total == 500
    assert user.balance == 500
```

### 3.3 使用 Mock

**场景**：测试时不想真的调用外部API

```python
from unittest.mock import Mock, patch

# 原函数
def get_weather(city):
    response = requests.get(f"https://api.weather.com/{city}")
    return response.json()["temperature"]

# 测试
def test_get_weather():
    # 使用 patch 替换 requests.get
    with patch('requests.get') as mock_get:
        # 设置 mock 返回值
        mock_response = Mock()
        mock_response.json.return_value = {"temperature": 25}
        mock_get.return_value = mock_response

        # 执行测试
        temperature = get_weather("Beijing")

        # 验证
        assert temperature == 25
        mock_get.assert_called_once_with("https://api.weather.com/Beijing")
```

### 3.4 测试覆盖率

```bash
# 安装
pip install pytest-cov

# 运行并查看覆盖率
pytest --cov=myapp tests/

# 输出HTML报告
pytest --cov=myapp --cov-report=html tests/
```

**覆盖率目标**：
- 核心业务逻辑：80-90%
- 工具函数：90%以上
- UI层：可以较低（50-60%）

---

## 4. 调试技巧

### 4.1 print 调试

```python
def calculate_total(items):
    total = 0
    for item in items:
        print(f"处理 {item}")  # 调试输出
        print(f"当前 total: {total}")
        total += item["price"] * item["quantity"]
        print(f"累加后 total: {total}")
    return total
```

### 4.2 使用 pdb 调试器

```python
import pdb

def buggy_function(data):
    result = []
    for item in data:
        pdb.set_trace()  # 在这里暂停，进入调试模式
        result.append(item * 2)
    return result

# 调试命令：
# n (next): 执行下一行
# s (step): 进入函数
# c (continue): 继续执行
# p variable: 打印变量
# l (list): 显示代码
# q (quit): 退出
```

### 4.3 日志调试

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)

def process_order(order):
    logger.debug(f"开始处理订单: {order.id}")

    try:
        logger.info(f"订单金额: {order.total}")
        payment_result = process_payment(order)
        logger.info(f"支付结果: {payment_result}")
    except Exception as e:
        logger.error(f"处理订单失败: {e}", exc_info=True)
        raise

    logger.debug("订单处理完成")
```

### 4.4 断言调试

```python
def divide(a, b):
    assert b != 0, "除数不能为零"
    assert isinstance(a, (int, float)), "a 必须是数字"
    assert isinstance(b, (int, float)), "b 必须是数字"

    return a / b

# 使用
result = divide(10, 0)  # AssertionError: 除数不能为零
```

---

## 5. 常见Bug模式

### 5.1 Off-by-One 错误

```python
# ❌ 错误
for i in range(len(items)):
    if i < len(items) - 1:  # 最后一个元素没处理
        process(items[i])

# ✅ 正确
for item in items:
    process(item)
```

### 5.2 浮点数比较

```python
# ❌ 错误
assert 0.1 + 0.2 == 0.3  # False!

# ✅ 正确
assert abs((0.1 + 0.2) - 0.3) < 1e-9
```

### 5.3 可变默认参数

```python
# ❌ 错误
def add_item(item, items=[]):
    items.append(item)
    return items

list1 = add_item(1)  # [1]
list2 = add_item(2)  # [1, 2]  ← bug!

# ✅ 正确
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

---

## 6. NOFX 的测试实践

### 6.1 手动测试

NOFX 主要使用手动测试：
- 在测试网运行
- 观察日志输出
- 验证交易结果

### 6.2 可以添加的测试

```go
// trader/binance_futures_test.go
func TestCalculatePnL(t *testing.T) {
    // 测试盈亏计算
    tests := []struct {
        name        string
        entryPrice  float64
        currentPrice float64
        quantity    float64
        leverage    int
        expected    float64
    }{
        {
            name: "做多盈利",
            entryPrice: 50000,
            currentPrice: 51000,
            quantity: 0.1,
            leverage: 10,
            expected: 100,  // (51000-50000) * 0.1 * 10
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := calculatePnL(tt.entryPrice, tt.currentPrice, tt.quantity, tt.leverage)
            if result != tt.expected {
                t.Errorf("期望 %v, 实际 %v", tt.expected, result)
            }
        })
    }
}
```

---

## 7. 实战练习

### 练习 1：为交易系统编写测试

为第21章的交易系统添加测试。

**要求**：
- 测试移动平均计算
- 测试买卖信号判断
- 使用 mock 测试 API 调用

### 练习 2：调试训练

找出以下代码的bug：

```python
def find_max(numbers):
    max_num = 0
    for num in numbers:
        if num > max_num:
            max_num = num
    return max_num

# 测试
print(find_max([5, 3, 8, 1]))  # 正确：8
print(find_max([-5, -3, -8, -1]))  # 错误：0（应该是-1）
```

---

## 本章总结

### 测试原则

1. **早测试，常测试**
2. **单元测试为主**
3. **测试金字塔**
4. **不要测试实现细节**

### 调试技巧

1. **复现问题**
2. **隔离问题**
3. **使用日志**
4. **使用调试器**
5. **橡皮鸭调试法**（向别人解释问题）

---

**💡 记住**：测试不是浪费时间，是节省时间！
