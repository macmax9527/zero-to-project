# 第17章：代码组织和规范 - 可维护性

> **本章目标**：学会编写规范、易维护的代码

---

## 📋 本章大纲

1. [为什么需要规范](#1-为什么需要规范)
2. [命名规范](#2-命名规范)
3. [代码风格](#3-代码风格)
4. [项目结构](#4-项目结构)
5. [文档和注释](#5-文档和注释)
6. [代码审查](#6-代码审查)
7. [重构](#7-重构)
8. [NOFX 的代码组织](#8-nofx-的代码组织)
9. [实战练习](#9-实战练习)

**预计学习时间**：3-4 小时

---

## 1. 为什么需要规范

### 1.1 问题代码示例

```python
# ❌ 糟糕的代码
def f(x,y):
    z=x+y
    if z>100:
        return z*2
    else:
        return z

a=f(10,20)
b=f(50,60)
print(a,b)
```

**问题**：
- 变量名无意义（f, x, y, z, a, b）
- 缺少空格
- 没有注释
- 魔法数字（100, 2）

```python
# ✅ 良好的代码
def calculate_bonus(base_salary: float, performance_score: float) -> float:
    """
    计算员工奖金

    Args:
        base_salary: 基础工资
        performance_score: 绩效分数

    Returns:
        奖金金额
    """
    BONUS_THRESHOLD = 100.0
    BONUS_MULTIPLIER = 2.0

    total_score = base_salary + performance_score

    if total_score > BONUS_THRESHOLD:
        return total_score * BONUS_MULTIPLIER
    else:
        return total_score

# 使用
employee1_bonus = calculate_bonus(base_salary=10.0, performance_score=20.0)
employee2_bonus = calculate_bonus(base_salary=50.0, performance_score=60.0)
print(f"员工1奖金: {employee1_bonus}, 员工2奖金: {employee2_bonus}")
```

### 1.2 规范的价值

**6个月后重读代码**：
- ❌ 糟糕代码：完全看不懂
- ✅ 规范代码：一目了然

**团队协作**：
- ❌ 每个人风格不同 → 理解困难
- ✅ 统一规范 → 降低沟通成本

**维护成本**：
- ❌ 修改一个功能 → 花2小时理解代码
- ✅ 清晰的代码 → 5分钟定位问题

---

## 2. 命名规范

### 2.1 命名原则

**1. 见名知意**

```python
# ❌ 糟糕
d = 86400  # 什么是 d？

# ✅ 良好
SECONDS_PER_DAY = 86400
```

**2. 使用英文**

```python
# ❌ 拼音
yonghu_id = 123
jine = 100.0

# ✅ 英文
user_id = 123
amount = 100.0
```

**3. 避免缩写**（除非是通用缩写）

```python
# ❌ 不清楚
usr = get_usr_by_id(id)
msg = get_msg()

# ✅ 清晰
user = get_user_by_id(user_id)
message = get_message()

# ✅ 通用缩写可以接受
html = "<div>Hello</div>"
url = "https://example.com"
api = "/api/v1/users"
```

### 2.2 命名风格

**Python（PEP 8）**：

```python
# 变量/函数：snake_case
user_name = "张三"
total_amount = 100.0

def calculate_total(items):
    pass

# 类：PascalCase
class UserManager:
    pass

class OrderProcessor:
    pass

# 常量：UPPER_CASE
MAX_RETRY_COUNT = 3
DEFAULT_TIMEOUT = 30

# 私有变量/方法：前缀 _
class User:
    def __init__(self):
        self._internal_id = None  # 私有变量

    def _internal_method(self):  # 私有方法
        pass
```

**Go**：

```go
// 变量/函数：camelCase（包内）/ PascalCase（导出）
var userName string  // 包内
var TotalAmount float64  // 导出（其他包可访问）

func calculateTotal(items []Item) float64 {  // 包内
    // ...
}

func GetUser(id int) *User {  // 导出
    // ...
}

// 常量：PascalCase 或 camelCase
const MaxRetryCount = 3
const defaultTimeout = 30

// 接口：PascalCase，通常以 er 结尾
type Reader interface {
    Read(p []byte) (n int, err error)
}

type UserRepository interface {
    FindByID(id int) (*User, error)
}
```

**JavaScript/TypeScript**：

```javascript
// 变量/函数：camelCase
const userName = "张三";
let totalAmount = 100.0;

function calculateTotal(items) {
    // ...
}

// 类：PascalCase
class UserManager {
    // ...
}

// 常量：UPPER_CASE
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = "https://api.example.com";

// 私有字段（ES2022+）：前缀 #
class User {
    #internalId;  // 私有字段

    #internalMethod() {  // 私有方法
        // ...
    }
}
```

### 2.3 特殊命名

**布尔变量**：使用疑问词

```python
# ✅ 良好
is_active = True
has_permission = True
can_edit = False
should_retry = True

# 函数
def is_valid(user):
    return user.age >= 18

def has_access(user, resource):
    return user.role == "admin"
```

**集合变量**：使用复数

```python
# ✅ 良好
users = [user1, user2, user3]
items = get_all_items()
order_ids = [1, 2, 3]

# 遍历
for user in users:
    print(user.name)
```

**临时变量**：简短但有意义

```python
# ✅ 循环中可以使用简短变量
for i in range(10):
    print(i)

for user in users:
    print(user.name)

# ❌ 避免无意义的单字母
for x in y:  # x 和 y 是什么？
    z = x + 1
```

---

## 3. 代码风格

### 3.1 Python（PEP 8）

**缩进和空格**：

```python
# ✅ 4个空格缩进
def example():
    if condition:
        do_something()
    else:
        do_other_thing()

# ✅ 运算符前后加空格
result = a + b
is_valid = (x > 10) and (y < 20)

# ✅ 逗号后加空格
items = [1, 2, 3, 4]
user = User(name="张三", age=30)

# ❌ 不要这样
items=[1,2,3,4]
result=a+b
```

**行长度**：

```python
# ✅ 每行不超过 79-100 字符
def long_function_name(
    parameter1, parameter2, parameter3,
    parameter4, parameter5
):
    pass

# 字符串换行
message = (
    "这是一段很长的文本，"
    "需要分成多行来写，"
    "这样更易读"
)
```

**导入**：

```python
# ✅ 导入顺序
# 1. 标准库
import os
import sys
from datetime import datetime

# 2. 第三方库
import requests
from flask import Flask

# 3. 本地模块
from myapp.models import User
from myapp.utils import calculate_total
```

**空行**：

```python
# ✅ 两个空行分隔顶层函数和类
import os


def function1():
    pass


def function2():
    pass


class MyClass:
    # 一个空行分隔类中的方法
    def method1(self):
        pass

    def method2(self):
        pass
```

### 3.2 Go（Effective Go）

**格式化**：

```go
// 使用 gofmt 或 goimports 自动格式化
go fmt ./...
goimports -w .
```

**错误处理**：

```go
// ✅ 良好：及早返回
func DoSomething() error {
    data, err := fetchData()
    if err != nil {
        return fmt.Errorf("fetch data: %w", err)
    }

    result, err := processData(data)
    if err != nil {
        return fmt.Errorf("process data: %w", err)
    }

    return nil
}

// ❌ 避免深度嵌套
func DoSomethingBad() error {
    data, err := fetchData()
    if err == nil {
        result, err := processData(data)
        if err == nil {
            // 嵌套很深
        }
    }
    return err
}
```

**接收者命名**：

```go
// ✅ 使用简短一致的接收者名称
type User struct {
    Name string
}

func (u *User) GetName() string {
    return u.Name
}

func (u *User) SetName(name string) {
    u.Name = name
}

// ❌ 不要使用 this, self
func (this *User) GetName() string {  // 不好
    return this.Name
}
```

### 3.3 使用工具自动格式化

**Python**：

```bash
# Black（自动格式化）
pip install black
black myapp/

# Flake8（代码检查）
pip install flake8
flake8 myapp/

# isort（导入排序）
pip install isort
isort myapp/
```

**Go**：

```bash
# gofmt（官方格式化工具）
gofmt -w .

# goimports（自动添加/删除导入）
go install golang.org/x/tools/cmd/goimports@latest
goimports -w .

# golangci-lint（综合代码检查）
golangci-lint run
```

**JavaScript/TypeScript**：

```bash
# Prettier（格式化）
npm install --save-dev prettier
npx prettier --write .

# ESLint（代码检查）
npm install --save-dev eslint
npx eslint --fix .
```

---

## 4. 项目结构

### 4.1 Python 项目

**小项目**：

```
myapp/
├── app.py              # 入口文件
├── models.py           # 数据模型
├── views.py            # 视图函数
├── utils.py            # 工具函数
├── config.py           # 配置
├── requirements.txt    # 依赖
└── tests/
    └── test_app.py
```

**中型项目**：

```
myapp/
├── myapp/              # 主包
│   ├── __init__.py
│   ├── models/         # 模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── order.py
│   ├── views/          # 视图
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── order.py
│   ├── services/       # 业务逻辑
│   │   ├── __init__.py
│   │   └── order_service.py
│   └── utils/          # 工具
│       ├── __init__.py
│       └── validators.py
├── tests/              # 测试
│   ├── test_models.py
│   └── test_services.py
├── config/             # 配置文件
│   ├── development.py
│   └── production.py
├── app.py              # 入口
├── requirements.txt
└── README.md
```

**大型项目**：

```
myapp/
├── apps/               # 多个应用
│   ├── users/
│   │   ├── models.py
│   │   ├── views.py
│   │   └── services.py
│   ├── orders/
│   │   ├── models.py
│   │   ├── views.py
│   │   └── services.py
│   └── payments/
│       ├── models.py
│       ├── views.py
│       └── services.py
├── common/             # 共享代码
│   ├── utils/
│   ├── middlewares/
│   └── exceptions/
├── config/
├── tests/
├── docs/
├── scripts/            # 脚本
├── requirements/
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
└── manage.py           # 管理命令
```

### 4.2 Go 项目

**标准布局**：

```
myapp/
├── cmd/                # 入口点
│   └── main.go
├── internal/           # 私有代码（不可导入）
│   ├── config/
│   │   └── config.go
│   ├── models/
│   │   ├── user.go
│   │   └── order.go
│   ├── services/
│   │   └── order_service.go
│   └── handlers/
│       └── user_handler.go
├── pkg/                # 公共库（可被其他项目导入）
│   └── utils/
│       └── validator.go
├── api/                # API 定义（OpenAPI/gRPC）
├── web/                # 静态文件
├── scripts/            # 脚本
├── configs/            # 配置文件
├── docs/               # 文档
├── tests/              # 集成测试
├── go.mod
├── go.sum
└── README.md
```

### 4.3 前端项目（React）

```
my-react-app/
├── public/
│   └── index.html
├── src/
│   ├── components/     # 可复用组件
│   │   ├── Button/
│   │   │   ├── Button.jsx
│   │   │   ├── Button.css
│   │   │   └── Button.test.js
│   │   └── Header/
│   ├── pages/          # 页面组件
│   │   ├── Home/
│   │   └── Dashboard/
│   ├── services/       # API 调用
│   │   └── userService.js
│   ├── hooks/          # 自定义 Hooks
│   │   └── useAuth.js
│   ├── context/        # Context
│   │   └── AuthContext.js
│   ├── utils/          # 工具函数
│   ├── App.jsx
│   └── index.jsx
├── package.json
└── README.md
```

---

## 5. 文档和注释

### 5.1 注释原则

**注释"为什么"，而非"是什么"**：

```python
# ❌ 糟糕：注释显而易见的内容
# 设置 x 为 10
x = 10

# 循环遍历用户
for user in users:
    print(user.name)

# ✅ 良好：解释为什么
# 使用 10 秒超时，因为第三方 API 响应较慢
TIMEOUT = 10

# 只处理活跃用户，避免发送给已删除账户
for user in active_users:
    send_notification(user)
```

**复杂逻辑需要注释**：

```python
# ✅ 解释复杂算法
def calculate_shipping_cost(weight, distance):
    """
    计算运费

    运费算法：
    1. 基础运费 = 重量 * 2
    2. 距离附加费 = 距离 / 100 * 5
    3. 如果重量 > 10kg，折扣 10%
    """
    base_cost = weight * 2
    distance_fee = (distance / 100) * 5
    total = base_cost + distance_fee

    # 重物折扣
    if weight > 10:
        total *= 0.9

    return total
```

### 5.2 文档字符串（Docstring）

**Python**：

```python
def calculate_total(items: list[dict], tax_rate: float = 0.1) -> float:
    """
    计算订单总金额（含税）

    Args:
        items: 订单项列表，每项包含 'price' 和 'quantity'
        tax_rate: 税率（默认 10%）

    Returns:
        总金额（含税）

    Raises:
        ValueError: 如果 items 为空或税率为负数

    Examples:
        >>> items = [{'price': 10, 'quantity': 2}, {'price': 5, 'quantity': 3}]
        >>> calculate_total(items)
        38.5
    """
    if not items:
        raise ValueError("订单项不能为空")
    if tax_rate < 0:
        raise ValueError("税率不能为负数")

    subtotal = sum(item['price'] * item['quantity'] for item in items)
    return subtotal * (1 + tax_rate)
```

**Go**：

```go
// CalculateTotal 计算订单总金额（含税）
//
// items 是订单项列表，每项包含 Price 和 Quantity
// taxRate 是税率（例如 0.1 表示 10%）
//
// 返回总金额（含税）和可能的错误
//
// 示例：
//   items := []Item{{Price: 10, Quantity: 2}, {Price: 5, Quantity: 3}}
//   total, err := CalculateTotal(items, 0.1)
func CalculateTotal(items []Item, taxRate float64) (float64, error) {
    if len(items) == 0 {
        return 0, errors.New("订单项不能为空")
    }
    if taxRate < 0 {
        return 0, errors.New("税率不能为负数")
    }

    subtotal := 0.0
    for _, item := range items {
        subtotal += item.Price * float64(item.Quantity)
    }

    return subtotal * (1 + taxRate), nil
}
```

### 5.3 README 文档

```markdown
# 项目名称

> 简短描述项目是做什么的

## 功能特性

- ✅ 功能1
- ✅ 功能2
- 🚧 功能3（开发中）

## 快速开始

### 安装

```bash
git clone https://github.com/username/project.git
cd project
pip install -r requirements.txt
```

### 配置

```bash
cp config.example.yaml config.yaml
# 编辑 config.yaml
```

### 运行

```bash
python app.py
```

访问 http://localhost:8080

## 文档

详细文档请查看 [docs/](docs/)

## 贡献

欢迎提交 Issue 和 Pull Request

## 许可证

MIT License
```

---

## 6. 代码审查

### 6.1 审查清单

**功能**：
- [ ] 代码是否实现了需求？
- [ ] 边界情况是否处理？
- [ ] 错误处理是否完善？

**可读性**：
- [ ] 变量名是否清晰？
- [ ] 函数是否过长？（建议 < 50行）
- [ ] 是否有足够的注释？

**性能**：
- [ ] 是否有性能问题？
- [ ] 数据库查询是否优化？
- [ ] 是否有内存泄漏？

**安全**：
- [ ] 是否有SQL注入风险？
- [ ] 用户输入是否验证？
- [ ] 敏感信息是否加密？

**测试**：
- [ ] 是否有单元测试？
- [ ] 测试覆盖率是否足够？

### 6.2 提交代码前检查

```bash
# Python
# 1. 格式化
black .
isort .

# 2. 代码检查
flake8 .

# 3. 运行测试
pytest

# 4. 检查测试覆盖率
pytest --cov=myapp tests/

# Go
# 1. 格式化
gofmt -w .
goimports -w .

# 2. 代码检查
golangci-lint run

# 3. 运行测试
go test ./...

# 4. 检查测试覆盖率
go test -cover ./...
```

---

## 7. 重构

### 7.1 何时重构

**代码异味**：
- 函数过长（> 50行）
- 重复代码
- 过深的嵌套（> 3层）
- 过多的参数（> 5个）
- 复杂的条件判断

### 7.2 重构技巧

**提取函数**：

```python
# ❌ 重构前：函数过长
def process_order(order):
    # 验证订单
    if not order.items:
        raise ValueError("订单为空")
    if order.total <= 0:
        raise ValueError("订单金额无效")

    # 计算折扣
    discount = 0
    if order.total > 1000:
        discount = order.total * 0.1
    elif order.total > 500:
        discount = order.total * 0.05

    # 处理支付
    payment = Payment(
        amount=order.total - discount,
        method=order.payment_method
    )
    payment.process()

    # 发送通知
    send_email(order.user.email, "订单确认", f"您的订单已确认")
    send_sms(order.user.phone, "订单已确认")

# ✅ 重构后：提取函数
def process_order(order):
    validate_order(order)
    discount = calculate_discount(order.total)
    process_payment(order, discount)
    send_notifications(order.user)

def validate_order(order):
    if not order.items:
        raise ValueError("订单为空")
    if order.total <= 0:
        raise ValueError("订单金额无效")

def calculate_discount(total):
    if total > 1000:
        return total * 0.1
    elif total > 500:
        return total * 0.05
    return 0

def process_payment(order, discount):
    payment = Payment(
        amount=order.total - discount,
        method=order.payment_method
    )
    payment.process()

def send_notifications(user):
    send_email(user.email, "订单确认", "您的订单已确认")
    send_sms(user.phone, "订单已确认")
```

**减少嵌套**：

```python
# ❌ 重构前：深度嵌套
def get_user_discount(user):
    if user is not None:
        if user.is_vip:
            if user.order_count > 10:
                return 0.2
            else:
                return 0.1
        else:
            return 0
    else:
        return 0

# ✅ 重构后：及早返回
def get_user_discount(user):
    if user is None:
        return 0

    if not user.is_vip:
        return 0

    if user.order_count > 10:
        return 0.2

    return 0.1
```

---

## 8. NOFX 的代码组织

### 8.1 项目结构

```
nofx/
├── cmd/                    # 入口
│   └── main.go
├── internal/
│   ├── config/             # 配置
│   ├── trader/             # 交易员实现
│   ├── strategy/           # 策略
│   └── exchange/           # 交易所客户端
├── pkg/
│   └── interfaces/         # 接口定义
├── api/                    # Web API
├── web/                    # 前端
├── docs/                   # 文档
└── tests/                  # 测试
```

### 8.2 代码规范

**命名**：
```go
// ✅ 清晰的命名
type BinanceFuturesTrader struct { ... }
func (t *BinanceFuturesTrader) GetAccountInfo() (*AccountInfo, error) { ... }

// 接口以 er 结尾
type Trader interface { ... }
type StrategyExecutor interface { ... }
```

**错误处理**：
```go
// ✅ 包装错误，提供上下文
positions, err := t.client.GetPositions()
if err != nil {
    return fmt.Errorf("failed to get positions: %w", err)
}
```

---

## 9. 实战练习

### 练习 1：重构代码

重构以下代码：

```python
def f(x):
    r=[]
    for i in x:
        if i%2==0:
            r.append(i*2)
    return r
```

**要求**：
- 清晰的命名
- 添加文档字符串
- 符合 PEP 8

### 练习 2：项目结构

为一个博客系统设计项目结构。

**要求**：
- 包含用户、文章、评论模块
- 使用标准的项目布局
- 说明每个目录的作用

---

## 本章总结

### 代码规范要点

1. **命名**：见名知意、使用英文、符合规范
2. **风格**：使用自动格式化工具
3. **结构**：合理的项目布局
4. **文档**：注释"为什么"、完善的 docstring
5. **审查**：代码提交前自检
6. **重构**：保持代码清晰

### 最佳实践

1. **使用工具**：Black、ESLint、gofmt
2. **一致性**：团队统一规范
3. **持续改进**：定期重构
4. **自动化**：pre-commit hooks

---

**💡 记住**：代码是写给人看的，只是顺便让机器执行。好的代码应该像散文一样易读！
