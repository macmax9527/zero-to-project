# 第19章：性能优化 - 瓶颈分析

> **本章目标**：学会识别和解决性能瓶颈，提升系统性能

---

## 📋 本章大纲

1. [性能优化的误区](#1-性能优化的误区)
2. [性能分析工具](#2-性能分析工具)
3. [数据库优化](#3-数据库优化)
4. [缓存策略](#4-缓存策略)
5. [前端性能优化](#5-前端性能优化)
6. [后端性能优化](#6-后端性能优化)
7. [NOFX 性能分析](#7-nofx-性能分析)
8. [实战练习](#8-实战练习)

**预计学习时间**：4-5 小时

---

## 1. 性能优化的误区

### 1.1 过早优化

> "过早优化是万恶之源" —— Donald Knuth

```python
# ❌ 过早优化的例子
# 还没写完基本功能就开始优化

def calculate_total(items):
    # 使用复杂的位运算"优化"
    total = 0
    for item in items:
        total += (item['price'] << 1) >> 1  # 这毫无意义
    return total

# ✅ 先让代码正确、清晰
def calculate_total(items):
    return sum(item['price'] for item in items)
```

### 1.2 没有测量就优化

```python
# ❌ 错误：凭感觉优化
# "我觉得这里慢，优化一下"

# ✅ 正确：先测量，找到真正的瓶颈
import time

start = time.time()
result = slow_function()
end = time.time()
print(f"耗时: {end - start}秒")

# 使用 profiler 找到瓶颈
import cProfile
cProfile.run('slow_function()')
```

### 1.3 优化原则

1. **先让代码正确**
2. **然后让代码清晰**
3. **测量找到瓶颈**
4. **优化瓶颈部分**
5. **再次测量验证**

---

## 2. 性能分析工具

### 2.1 Python性能分析

**time模块（简单计时）**：

```python
import time

def benchmark(func, *args, **kwargs):
    start = time.time()
    result = func(*args, **kwargs)
    end = time.time()
    print(f"{func.__name__} 耗时: {end - start:.4f}秒")
    return result

# 使用
result = benchmark(slow_function, arg1, arg2)
```

**timeit模块（精确测试）**：

```python
import timeit

# 测试代码片段
code = """
result = sum(range(1000))
"""

# 运行1000次，取平均
time_taken = timeit.timeit(code, number=1000)
print(f"平均耗时: {time_taken / 1000 * 1000:.4f}毫秒")

# 比较不同实现
list_comp = timeit.timeit('[i for i in range(1000)]', number=10000)
generator = timeit.timeit('list(i for i in range(1000))', number=10000)
print(f"列表推导式: {list_comp:.4f}秒")
print(f"生成器: {generator:.4f}秒")
```

**cProfile（函数级分析）**：

```python
import cProfile
import pstats

def slow_function():
    total = 0
    for i in range(1000000):
        total += i
    return total

# 分析性能
cProfile.run('slow_function()', 'profile_stats')

# 查看统计
stats = pstats.Stats('profile_stats')
stats.strip_dirs()
stats.sort_stats('cumulative')
stats.print_stats(10)  # 显示前10个最慢的函数
```

**输出示例**：
```
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.052    0.052    0.052    0.052 <stdin>:1(slow_function)
  1000000    0.045    0.000    0.045    0.000 {built-in method builtins.range}
```

**line_profiler（行级分析）**：

```python
# 安装
# pip install line_profiler

# 使用装饰器
from line_profiler import LineProfiler

def slow_function():
    total = 0
    for i in range(1000):
        total += i * 2
        total -= i
    return total

lp = LineProfiler()
lp.add_function(slow_function)
lp_wrapper = lp(slow_function)
lp_wrapper()
lp.print_stats()
```

### 2.2 Go性能分析

**pprof（CPU分析）**：

```go
package main

import (
    "os"
    "runtime/pprof"
)

func main() {
    // 开启 CPU profiling
    f, _ := os.Create("cpu.prof")
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()

    // 运行要分析的代码
    slowFunction()
}

// 分析结果
// go tool pprof cpu.prof
// (pprof) top 10
// (pprof) list slowFunction
```

**基准测试（Benchmark）**：

```go
// main_test.go
package main

import "testing"

func BenchmarkSlowFunction(b *testing.B) {
    for i := 0; i < b.N; i++ {
        slowFunction()
    }
}

func BenchmarkFastFunction(b *testing.B) {
    for i := 0; i < b.N; i++ {
        fastFunction()
    }
}

// 运行基准测试
// go test -bench=. -benchmem
```

**输出示例**：
```
BenchmarkSlowFunction-8    1000000    1234 ns/op    512 B/op    10 allocs/op
BenchmarkFastFunction-8   10000000     123 ns/op     64 B/op     2 allocs/op
```

### 2.3 前端性能分析

**Chrome DevTools**：
- **Performance** 标签：记录页面加载和运行性能
- **Network** 标签：查看资源加载时间
- **Lighthouse**：自动化性能审计

**关键指标**：
- **FCP**（First Contentful Paint）：首次内容绘制
- **LCP**（Largest Contentful Paint）：最大内容绘制
- **TTI**（Time to Interactive）：可交互时间

---

## 3. 数据库优化

### 3.1 慢查询识别

**MySQL慢查询日志**：

```sql
-- 启用慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- 超过1秒的查询

-- 查看慢查询
SHOW VARIABLES LIKE 'slow_query%';
```

**使用EXPLAIN分析查询**：

```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 输出：
-- id | select_type | table | type | possible_keys | key  | rows | Extra
--  1 | SIMPLE      | users | ALL  | NULL          | NULL | 1000 | Using where
```

**关键字段**：
- **type**：访问类型（ALL 最慢，const 最快）
- **possible_keys**：可能使用的索引
- **key**：实际使用的索引
- **rows**：扫描的行数

### 3.2 索引优化

**创建索引**：

```sql
-- ❌ 没有索引：全表扫描
SELECT * FROM users WHERE email = 'test@example.com';
-- 扫描 100万行

-- ✅ 创建索引
CREATE INDEX idx_users_email ON users(email);
-- 只扫描 1行

-- 复合索引
CREATE INDEX idx_users_name_age ON users(name, age);

-- 使用复合索引
SELECT * FROM users WHERE name = '张三' AND age = 30;  -- ✅ 使用索引
SELECT * FROM users WHERE age = 30;  -- ❌ 不使用索引（没有name）
```

**索引原则**：
1. 在 WHERE、ORDER BY、GROUP BY 列上建索引
2. 选择性高的列建索引（值差异大）
3. 不要过度索引（影响写入性能）
4. 复合索引遵循最左前缀原则

### 3.3 查询优化

**避免 SELECT ***：

```sql
-- ❌ 慢：查询所有字段
SELECT * FROM users;

-- ✅ 快：只查询需要的字段
SELECT id, name, email FROM users;
```

**避免 N+1 查询**：

```python
# ❌ N+1 查询问题
users = User.query.all()  # 1次查询
for user in users:
    orders = Order.query.filter_by(user_id=user.id).all()  # N次查询

# ✅ 使用 JOIN 或预加载
users = User.query.options(joinedload(User.orders)).all()  # 1次查询
for user in users:
    orders = user.orders  # 不再查询数据库
```

**使用分页**：

```python
# ❌ 查询所有数据
users = User.query.all()  # 可能返回100万条记录

# ✅ 分页查询
users = User.query.paginate(page=1, per_page=20)
```

### 3.4 连接池优化

```python
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

# 配置连接池
engine = create_engine(
    'postgresql://user:pass@localhost/db',
    poolclass=QueuePool,
    pool_size=10,        # 连接池大小
    max_overflow=20,     # 最大溢出连接数
    pool_timeout=30,     # 获取连接超时
    pool_recycle=3600,   # 连接回收时间（秒）
)
```

---

## 4. 缓存策略

### 4.1 缓存层次

```
请求 → 浏览器缓存 → CDN → 应用缓存 → 数据库查询缓存 → 数据库
       ↑ 最快                                              ↑ 最慢
```

### 4.2 应用级缓存

**简单缓存（内存字典）**：

```python
from functools import lru_cache
from datetime import datetime, timedelta

# 使用 functools.lru_cache
@lru_cache(maxsize=128)
def expensive_computation(n):
    # 复杂计算
    return sum(i * i for i in range(n))

# 第一次调用：计算
result1 = expensive_computation(1000)  # 慢

# 第二次调用：从缓存返回
result2 = expensive_computation(1000)  # 快
```

**带过期时间的缓存**：

```python
class Cache:
    def __init__(self):
        self._cache = {}

    def get(self, key):
        if key in self._cache:
            value, expiry = self._cache[key]
            if datetime.now() < expiry:
                return value
            else:
                del self._cache[key]
        return None

    def set(self, key, value, ttl=60):
        """TTL: 过期时间（秒）"""
        expiry = datetime.now() + timedelta(seconds=ttl)
        self._cache[key] = (value, expiry)

    def delete(self, key):
        if key in self._cache:
            del self._cache[key]

# 使用
cache = Cache()
cache.set('user:1', user_data, ttl=300)  # 缓存5分钟

# 获取
data = cache.get('user:1')
if data is None:
    data = database.query(...)
    cache.set('user:1', data, ttl=300)
```

### 4.3 Redis缓存

```python
import redis
import json

# 连接 Redis
r = redis.Redis(host='localhost', port=6379, decode_responses=True)

def get_user(user_id):
    # 先从缓存获取
    cache_key = f"user:{user_id}"
    cached_data = r.get(cache_key)

    if cached_data:
        return json.loads(cached_data)

    # 缓存未命中，查询数据库
    user = database.query(User).get(user_id)

    # 写入缓存（过期时间5分钟）
    r.setex(cache_key, 300, json.dumps(user.to_dict()))

    return user

# 删除缓存（数据更新时）
def update_user(user_id, data):
    user = database.query(User).get(user_id)
    user.update(data)
    database.commit()

    # 删除缓存
    r.delete(f"user:{user_id}")
```

### 4.4 缓存策略

**Cache-Aside（旁路缓存）**：

```python
def get_data(key):
    # 1. 从缓存读取
    data = cache.get(key)
    if data:
        return data

    # 2. 缓存未命中，从数据库读取
    data = database.query(key)

    # 3. 写入缓存
    cache.set(key, data)

    return data
```

**Write-Through（写穿）**：

```python
def update_data(key, value):
    # 1. 更新数据库
    database.update(key, value)

    # 2. 同步更新缓存
    cache.set(key, value)
```

**Write-Behind（写回）**：

```python
def update_data(key, value):
    # 1. 先更新缓存
    cache.set(key, value)

    # 2. 异步更新数据库
    async_queue.add_task(lambda: database.update(key, value))
```

---

## 5. 前端性能优化

### 5.1 减少HTTP请求

**合并文件**：
```html
<!-- ❌ 多个请求 -->
<script src="jquery.js"></script>
<script src="utils.js"></script>
<script src="app.js"></script>

<!-- ✅ 合并为一个 -->
<script src="bundle.js"></script>
```

**使用CSS Sprites**：
```css
/* 将多个小图标合并为一张图 */
.icon-home { background: url(sprites.png) 0 0; }
.icon-user { background: url(sprites.png) -20px 0; }
```

### 5.2 资源压缩

**代码压缩（Minify）**：
```javascript
// 原始代码
function calculateTotal(items) {
    let total = 0;
    for (let item of items) {
        total += item.price;
    }
    return total;
}

// 压缩后
function calculateTotal(t){let e=0;for(let l of t)e+=l.price;return e}
```

**Gzip压缩**：
```nginx
# Nginx配置
gzip on;
gzip_types text/plain text/css application/json application/javascript;
gzip_min_length 1000;
```

### 5.3 懒加载

**图片懒加载**：
```html
<img data-src="image.jpg" class="lazy" alt="描述">

<script>
document.addEventListener('DOMContentLoaded', function() {
    let lazyImages = document.querySelectorAll('.lazy');

    let imageObserver = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                let img = entry.target;
                img.src = img.dataset.src;
                img.classList.remove('lazy');
                imageObserver.unobserve(img);
            }
        });
    });

    lazyImages.forEach(img => imageObserver.observe(img));
});
</script>
```

**代码分割（React）**：
```javascript
import React, { lazy, Suspense } from 'react';

// 懒加载组件
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
    return (
        <Suspense fallback={<div>加载中...</div>}>
            <HeavyComponent />
        </Suspense>
    );
}
```

### 5.4 CDN加速

```html
<!-- 使用CDN加载库 -->
<script src="https://cdn.jsdelivr.net/npm/react@18/umd/react.production.min.js"></script>

<!-- 字体CDN -->
<link href="https://fonts.googleapis.com/css2?family=Roboto" rel="stylesheet">
```

---

## 6. 后端性能优化

### 6.1 避免阻塞操作

```python
# ❌ 同步阻塞
def process_request():
    data1 = fetch_from_api1()  # 等待1秒
    data2 = fetch_from_api2()  # 等待1秒
    return combine(data1, data2)  # 总共2秒

# ✅ 并发执行
import asyncio

async def process_request():
    data1, data2 = await asyncio.gather(
        fetch_from_api1(),  # 并发
        fetch_from_api2()   # 并发
    )
    return combine(data1, data2)  # 总共1秒
```

### 6.2 批量操作

```python
# ❌ 逐条插入
for item in items:
    db.insert(item)  # 1000次数据库操作

# ✅ 批量插入
db.bulk_insert(items)  # 1次数据库操作
```

### 6.3 对象池

```python
from queue import Queue

class ObjectPool:
    def __init__(self, factory, size=10):
        self.factory = factory
        self.pool = Queue(maxsize=size)

        # 预创建对象
        for _ in range(size):
            self.pool.put(factory())

    def acquire(self):
        """获取对象"""
        return self.pool.get()

    def release(self, obj):
        """归还对象"""
        self.pool.put(obj)

# 使用
pool = ObjectPool(lambda: DatabaseConnection(), size=10)

conn = pool.acquire()
try:
    result = conn.query(...)
finally:
    pool.release(conn)
```

---

## 7. NOFX 性能分析

### 7.1 API调用优化

**问题**：频繁调用交易所API

```go
// ❌ 每次都调用API
func (t *Trader) GetCurrentPrice() float64 {
    return t.client.GetPrice()  // 网络请求
}

// ✅ 缓存价格
type Trader struct {
    priceCache      float64
    priceCacheTime  time.Time
    cacheDuration   time.Duration
}

func (t *Trader) GetCurrentPrice() float64 {
    now := time.Now()
    if now.Sub(t.priceCacheTime) < t.cacheDuration {
        return t.priceCache
    }

    price := t.client.GetPrice()
    t.priceCache = price
    t.priceCacheTime = now

    return price
}
```

### 7.2 并发处理

```go
// ✅ 并发查询多个交易员的持仓
func (s *Server) GetAllPositions() []Position {
    var positions []Position
    var mu sync.Mutex
    var wg sync.WaitGroup

    for _, trader := range s.traders {
        wg.Add(1)
        go func(t Trader) {
            defer wg.Done()

            pos, _ := t.GetPositions()

            mu.Lock()
            positions = append(positions, pos...)
            mu.Unlock()
        }(trader)
    }

    wg.Wait()
    return positions
}
```

### 7.3 数据结构优化

```go
// ❌ 使用切片查找（O(n)）
type PositionManager struct {
    positions []Position
}

func (pm *PositionManager) Find(symbol string) *Position {
    for i := range pm.positions {
        if pm.positions[i].Symbol == symbol {
            return &pm.positions[i]
        }
    }
    return nil
}

// ✅ 使用map查找（O(1)）
type PositionManager struct {
    positions map[string]*Position
}

func (pm *PositionManager) Find(symbol string) *Position {
    return pm.positions[symbol]
}
```

---

## 8. 实战练习

### 练习 1：优化慢查询

给定以下查询，使用索引优化：

```sql
SELECT orders.*, users.name
FROM orders
JOIN users ON orders.user_id = users.id
WHERE orders.status = 'pending'
AND orders.created_at > '2024-01-01'
ORDER BY orders.created_at DESC
LIMIT 20;
```

**要求**：
- 使用 EXPLAIN 分析
- 创建合适的索引
- 测量优化前后性能

### 练习 2：实现带过期的缓存装饰器

```python
def cached(ttl=60):
    """缓存装饰器"""
    def decorator(func):
        # 实现缓存逻辑
        pass
    return decorator

@cached(ttl=300)
def expensive_function(n):
    # 复杂计算
    return sum(i * i for i in range(n))
```

### 练习 3：性能分析

使用 cProfile 分析以下代码，找出瓶颈并优化：

```python
def process_data(data):
    results = []
    for item in data:
        result = expensive_operation(item)
        results.append(result)
    return results

# 测试数据
data = list(range(10000))
result = process_data(data)
```

---

## 本章总结

### 性能优化流程

1. **测量**：使用profiler找到瓶颈
2. **分析**：理解为什么慢
3. **优化**：针对性优化
4. **验证**：测量优化效果

### 常见优化技巧

**数据库**：
- 创建索引
- 避免N+1查询
- 使用连接池
- 查询结果分页

**缓存**：
- 应用级缓存（内存）
- 分布式缓存（Redis）
- 浏览器缓存
- CDN缓存

**前端**：
- 减少HTTP请求
- 资源压缩
- 懒加载
- 代码分割

**后端**：
- 异步非阻塞
- 批量操作
- 对象池
- 并发处理

### 性能优化原则

1. **不要过早优化**
2. **先测量再优化**
3. **优化20%的代码解决80%的问题**
4. **保持代码清晰优先于性能**

---

**💡 记住**：性能优化是一个持续的过程。先让代码正确，再让代码快速！
