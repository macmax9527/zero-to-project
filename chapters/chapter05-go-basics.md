# Go 快速入门（面向Python开发者）

> 10分钟快速了解Go语法，专为会Python的读者设计

---

## 🎯 阅读指南

**你不需要**：
- ❌ 深入学习Go
- ❌ 记住所有语法
- ❌ 能写Go代码

**你只需要**：
- ✅ 能看懂NOFX文档中的Go代码
- ✅ 理解基本语法对应关系
- ✅ 关注设计思路而非语法细节

---

## 📊 Python vs Go 核心对照表

### 1. 变量定义

<table>
<tr>
<td width="50%">

**Python**
```python
# 直接赋值
name = "Tom"
age = 25
price = 99.9

# 类型注解（可选）
name: str = "Tom"
age: int = 25
```

</td>
<td width="50%">

**Go**
```go
// 方式1：自动推断类型
name := "Tom"
age := 25
price := 99.9

// 方式2：显式声明类型
var name string = "Tom"
var age int = 25
```

</td>
</tr>
</table>

**对应关系**：
- Python的 `=` ≈ Go的 `:=` （自动推断）
- Python的 `: type =` ≈ Go的 `var name type =` （显式声明）

---

### 2. 函数定义

<table>
<tr>
<td width="50%">

**Python**
```python
def add(a, b):
    return a + b

# 有类型注解
def add(a: int, b: int) -> int:
    return a + b

# 多返回值（用tuple）
def divide(a, b):
    if b == 0:
        return None, "除数不能为0"
    return a / b, None
```

</td>
<td width="50%">

**Go**
```go
func add(a int, b int) int {
    return a + b
}

// 简写（类型相同）
func add(a, b int) int {
    return a + b
}

// 多返回值（原生支持）
func divide(a, b int) (float64, error) {
    if b == 0 {
        return 0, errors.New("除数不能为0")
    }
    return float64(a) / float64(b), nil
}
```

</td>
</tr>
</table>

**关键差异**：
- Go的类型写在后面：`name string` vs Python的 `name: str`
- Go原生支持多返回值，Python用tuple
- Go的 `nil` = Python的 `None`

---

### 3. 类和方法

<table>
<tr>
<td width="50%">

**Python**
```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hello, {self.name}"

# 使用
user = User("Tom", 25)
print(user.greet())
```

</td>
<td width="50%">

**Go**
```go
// struct（结构体）= Python的class
type User struct {
    Name string  // 大写=public
    Age  int
}

// 方法（关联到struct）
func (u *User) Greet() string {
    return fmt.Sprintf("Hello, %s", u.Name)
}

// 使用
user := User{Name: "Tom", Age: 25}
fmt.Println(user.Greet())
```

</td>
</tr>
</table>

**对应关系**：
- Python的 `class` ≈ Go的 `struct`
- Python的 `def method(self)` ≈ Go的 `func (u *User) Method()`
- Python的 `self` ≈ Go的 `u` （接收者）

---

### 4. 接口

<table>
<tr>
<td width="50%">

**Python**
```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof"

class Cat(Animal):
    def speak(self):
        return "Meow"
```

</td>
<td width="50%">

**Go**
```go
// 接口定义
type Animal interface {
    Speak() string
}

// Dog实现（无需显式声明）
type Dog struct{}
func (d Dog) Speak() string {
    return "Woof"
}

// Cat实现
type Cat struct{}
func (c Cat) Speak() string {
    return "Meow"
}
```

</td>
</tr>
</table>

**关键差异**：
- Python需要 `class Dog(Animal)` 显式继承
- Go **自动实现**：只要有 `Speak()` 方法，就实现了 `Animal` 接口
- Go的接口更灵活（鸭子类型）

---

### 5. 错误处理

<table>
<tr>
<td width="50%">

**Python**
```python
def read_file(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        print("文件不存在")
        return None
    except Exception as e:
        print(f"错误: {e}")
        return None
```

</td>
<td width="50%">

**Go**
```go
func readFile(path string) (string, error) {
    data, err := os.ReadFile(path)

    // 检查错误
    if err != nil {
        return "", fmt.Errorf("读取失败: %w", err)
    }

    return string(data), nil
}

// 调用时检查错误
content, err := readFile("test.txt")
if err != nil {
    fmt.Println("错误:", err)
    return
}
```

</td>
</tr>
</table>

**关键差异**：
- Python用 `try-except` 捕获异常
- Go返回 `error` 类型，调用者检查 `if err != nil`
- Go的错误处理更显式

---

### 6. 列表/数组

<table>
<tr>
<td width="50%">

**Python**
```python
# 列表（动态）
numbers = [1, 2, 3]
numbers.append(4)

# 遍历
for num in numbers:
    print(num)

# 切片
first_two = numbers[:2]
```

</td>
<td width="50%">

**Go**
```go
// 切片（动态）
numbers := []int{1, 2, 3}
numbers = append(numbers, 4)

// 遍历
for _, num := range numbers {
    fmt.Println(num)
}

// 切片
firstTwo := numbers[:2]
```

</td>
</tr>
</table>

**对应关系**：
- Python的 `list` ≈ Go的 `slice`
- Python的 `for x in list` ≈ Go的 `for _, x := range slice`
- Go的 `_` 表示忽略索引

---

### 7. 字典/Map

<table>
<tr>
<td width="50%">

**Python**
```python
# 字典
user = {
    "name": "Tom",
    "age": 25
}

# 访问
print(user["name"])

# 遍历
for key, value in user.items():
    print(f"{key}: {value}")
```

</td>
<td width="50%">

**Go**
```go
// Map
user := map[string]interface{}{
    "name": "Tom",
    "age":  25,
}

// 访问
fmt.Println(user["name"])

// 遍历
for key, value := range user {
    fmt.Printf("%s: %v\n", key, value)
}
```

</td>
</tr>
</table>

---

### 8. 包和导入

<table>
<tr>
<td width="50%">

**Python**
```python
# 导入
import os
from typing import List
import json

# 使用
data = json.loads('{"name": "Tom"}')
```

</td>
<td width="50%">

**Go**
```go
// 导入（写在开头）
import (
    "os"
    "encoding/json"
    "fmt"
)

// 使用
var data map[string]string
json.Unmarshal([]byte(`{"name":"Tom"}`), &data)
```

</td>
</tr>
</table>

---

## 🔍 看懂NOFX代码的关键

### 重点关注这些

| Go语法 | 含义 | Python对应 |
|--------|------|-----------|
| `:=` | 定义并赋值 | `=` |
| `func (b *Binance)` | 方法 | `def method(self)` |
| `type X interface` | 接口 | `class X(ABC)` |
| `type X struct` | 结构体/类 | `class X` |
| `if err != nil` | 错误检查 | `except` |
| `[]int` | 整数切片 | `list[int]` |
| `map[string]int` | 字典 | `dict[str, int]` |

### 不用关心的细节

| Go语法 | 含义 | 建议 |
|--------|------|------|
| `*` | 指针 | 暂时不用理解 |
| `&` | 取地址 | 看成传递引用即可 |
| `:=` vs `var` | 两种定义方式 | 都是定义变量 |
| `context.Background()` | 上下文参数 | Go特有，跳过即可 |

---

## 📖 实战示例：对照阅读

### Python版本

```python
from abc import ABC, abstractmethod

class Exchange(ABC):
    """交易所接口"""

    @abstractmethod
    def get_account(self):
        """获取账户信息"""
        pass

class Binance(Exchange):
    """Binance实现"""

    def __init__(self, api_key, secret):
        self.api_key = api_key
        self.secret = secret

    def get_account(self):
        # 1. 调用API
        response = self._call_api("/api/v3/account")

        # 2. 转换格式
        return {
            "total": float(response["totalBalance"]),
            "available": float(response["availableBalance"])
        }
```

### Go版本（NOFX）

```go
// 【对应Python的 ABC】
type Exchange interface {
    // 获取账户信息
    GetAccount() (Account, error)
}

// 【对应Python的 dataclass】
type Account struct {
    Total     float64
    Available float64
}

// 【对应Python的 class Binance(Exchange)】
type Binance struct {
    apiKey string
    secret string
}

// 【对应Python的 def get_account(self)】
func (b *Binance) GetAccount() (Account, error) {
    // 1. 调用API
    response, err := b.callAPI("/api/v3/account")
    if err != nil {
        return Account{}, err
    }

    // 2. 转换格式
    return Account{
        Total:     parseFloat(response["totalBalance"]),
        Available: parseFloat(response["availableBalance"]),
    }, nil
}
```

**阅读技巧**：
1. 先看Python版本，理解逻辑
2. 再看Go版本，找对应关系
3. 忽略语法细节，关注三步走：
   - 定义接口
   - 实现接口
   - 转换数据

---

## 🎯 10个最常见的Go代码模式

看懂这10个模式，就能理解NOFX文档中90%的Go代码：

### 1. 定义接口
```go
type Animal interface {
    Speak() string  // 方法签名
}
```

### 2. 定义结构体
```go
type Dog struct {
    Name string
    Age  int
}
```

### 3. 实现方法
```go
func (d *Dog) Speak() string {
    return "Woof"
}
```

### 4. 创建实例
```go
dog := Dog{Name: "Max", Age: 3}
// 或
dog := &Dog{Name: "Max", Age: 3}  // 指针
```

### 5. 错误检查
```go
result, err := someFunction()
if err != nil {
    // 处理错误
    return err
}
// 使用result
```

### 6. 遍历切片
```go
for _, item := range items {
    fmt.Println(item)
}
```

### 7. 字符串格式化
```go
msg := fmt.Sprintf("Hello, %s", name)
```

### 8. 导入包
```go
import (
    "fmt"
    "encoding/json"
)
```

### 9. 返回多个值
```go
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("除数不能为0")
    }
    return a / b, nil
}
```

### 10. 类型转换
```go
floatNum := float64(intNum)
str := strconv.Itoa(num)  // int转string
```

---

## 💡 学习建议

### 现在该做的

1. ✅ **快速浏览本文档** - 了解基本对应关系
2. ✅ **遇到Go代码时** - 找对应的Python写法
3. ✅ **关注逻辑而非语法** - 理解做了什么，不纠结怎么写
4. ✅ **使用对照表** - 不懂的语法查表

### 现在不要做的

1. ❌ **深入学习Go** - 浪费时间
2. ❌ **记忆Go语法** - 没有必要
3. ❌ **用Go写代码** - 除非你需要
4. ❌ **纠结指针、goroutine等概念** - 暂时不影响理解

### 如果将来需要学Go

**那时候会很快**，因为：
- ✅ 你已经理解了编程思维
- ✅ 你已经理解了项目设计
- ✅ 只需要学新语法（1-2周）

---

## 📚 延伸阅读

**想深入学习Go？**（可选）
- [Go官方教程](https://go.dev/tour/)
- [Go by Example](https://gobyexample.com/)

**但记住**：现阶段不需要！专注Python和项目思维。

---

## 🎯 总结

**核心要点**：
1. Go语法和Python很相似，只是写法不同
2. 看NOFX的Go代码时，关注逻辑而非语法
3. 用本文档作为速查表，不懂就查
4. 不要花时间深入学Go，除非你真的需要

**记住**：设计思维 > 编程语言

Python能实现的，Go也能实现，反之亦然。重点是**设计**，不是**语言**。

---

**返回** → [第5章：数据获取层](chapter05-data-layer.md) | [完整学习指南](../COMPLETE_GUIDE.md)
