# 第11章：并发和异步 - 性能优化

> **本章目标**：理解并发编程，学会使用多线程、协程提升性能

---

## 📋 本章大纲

1. [并发 vs 并行](#1-并发-vs-并行)
2. [Python 并发编程](#2-python-并发编程)
3. [Go 并发编程](#3-go-并发编程)
4. [常见并发问题](#4-常见并发问题)
5. [异步编程](#5-异步编程)
6. [NOFX 的并发实践](#6-nofx-的并发实践)
7. [实战练习](#7-实战练习)

**预计学习时间**：4-5 小时

---

## 1. 并发 vs 并行

### 1.1 概念区分

```
并发（Concurrency）：多个任务交替执行
┌─────┐ ┌─────┐ ┌─────┐
│任务A│ │任务B│ │任务A│  ← 单核CPU快速切换
└─────┘ └─────┘ └─────┘
    时间 ────────→

并行（Parallelism）：多个任务同时执行
┌─────┐
│任务A│  ← CPU核心1
└─────┘
┌─────┐
│任务B│  ← CPU核心2
└─────┘
    时间 ────────→
```

### 1.2 生活化类比

**并发**：一个厨师同时做多道菜
- 炒菜A → 等待煮沸 → 切菜B → 回去翻炒A → 处理菜C
- 看起来在"同时"做，实际是快速切换

**并行**：多个厨师各自做菜
- 厨师1做菜A
- 厨师2做菜B
- 厨师3做菜C
- 真正的同时进行

### 1.3 什么时候需要并发

**适合并发的场景**：
- ✅ **I/O 密集型**：网络请求、文件读写、数据库查询
- ✅ **等待时间长**：等待API响应、等待用户输入
- ❌ **CPU 密集型**：复杂计算、图像处理（需要并行）

**例子**：
```python
# 串行执行（慢）
def download_files(urls):
    results = []
    for url in urls:
        data = download(url)  # 每个下载需要2秒
        results.append(data)
    return results

# 下载100个文件需要 100 * 2 = 200秒

# 并发执行（快）
def download_files_concurrent(urls):
    with ThreadPoolExecutor(max_workers=10) as executor:
        results = executor.map(download, urls)
    return list(results)

# 下载100个文件只需 100 / 10 * 2 = 20秒
```

---

## 2. Python 并发编程

### 2.1 多线程（Threading）

**适用场景**：I/O 密集型任务

```python
import threading
import time
import requests

# 示例：并发下载多个网页
def download_page(url):
    print(f"[{threading.current_thread().name}] 开始下载 {url}")
    response = requests.get(url)
    print(f"[{threading.current_thread().name}] 下载完成 {url}，大小 {len(response.content)} 字节")
    return response.content

# 方法1：手动创建线程
urls = [
    "https://example.com/page1",
    "https://example.com/page2",
    "https://example.com/page3",
]

threads = []
for url in urls:
    thread = threading.Thread(target=download_page, args=(url,))
    threads.append(thread)
    thread.start()

# 等待所有线程完成
for thread in threads:
    thread.join()

print("所有下载完成")
```

**方法2：使用线程池**（推荐）

```python
from concurrent.futures import ThreadPoolExecutor

urls = ["https://example.com/page1", "https://example.com/page2", "https://example.com/page3"]

# 创建线程池，最多5个并发线程
with ThreadPoolExecutor(max_workers=5) as executor:
    # 提交任务
    futures = [executor.submit(download_page, url) for url in urls]

    # 获取结果
    for future in futures:
        result = future.result()  # 阻塞等待结果
        print(f"获得结果，大小 {len(result)} 字节")
```

**使用 map（更简洁）**：

```python
with ThreadPoolExecutor(max_workers=5) as executor:
    results = executor.map(download_page, urls)
    for result in results:
        print(f"下载完成，大小 {len(result)} 字节")
```

### 2.2 多进程（Multiprocessing）

**适用场景**：CPU 密集型任务

```python
from multiprocessing import Pool
import time

# CPU密集型任务：计算大数的平方
def calculate_square(n):
    print(f"计算 {n} 的平方")
    result = 0
    for i in range(n):
        result += i * i
    return result

if __name__ == "__main__":
    numbers = [10000000, 20000000, 30000000, 40000000]

    # 串行执行
    start = time.time()
    results = [calculate_square(n) for n in numbers]
    print(f"串行耗时: {time.time() - start:.2f}秒")

    # 并行执行
    start = time.time()
    with Pool(processes=4) as pool:
        results = pool.map(calculate_square, numbers)
    print(f"并行耗时: {time.time() - start:.2f}秒")
```

**多线程 vs 多进程**：

| | 多线程 | 多进程 |
|---|---|---|
| **适用场景** | I/O 密集型 | CPU 密集型 |
| **内存** | 共享内存 | 独立内存 |
| **开销** | 小 | 大 |
| **GIL限制** | 受限（Python） | 不受限 |
| **通信** | 简单 | 复杂 |

### 2.3 asyncio（异步协程）

**适用场景**：高并发 I/O（如 Web 服务器）

```python
import asyncio
import aiohttp
import time

# 异步下载
async def download_page_async(session, url):
    print(f"开始下载 {url}")
    async with session.get(url) as response:
        content = await response.read()
        print(f"下载完成 {url}，大小 {len(content)} 字节")
        return content

async def main():
    urls = [
        "https://example.com/page1",
        "https://example.com/page2",
        "https://example.com/page3",
    ]

    async with aiohttp.ClientSession() as session:
        # 创建所有任务
        tasks = [download_page_async(session, url) for url in urls]
        # 并发执行
        results = await asyncio.gather(*tasks)

    print(f"下载 {len(results)} 个页面")

# 运行
if __name__ == "__main__":
    start = time.time()
    asyncio.run(main())
    print(f"总耗时: {time.time() - start:.2f}秒")
```

**对比三种方式**：

```python
# 1. 串行（最慢）
def download_serial(urls):
    return [download(url) for url in urls]

# 2. 多线程（适合I/O，中等速度）
def download_threading(urls):
    with ThreadPoolExecutor(max_workers=10) as executor:
        return list(executor.map(download, urls))

# 3. asyncio（适合高并发I/O，最快）
async def download_async(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [download_page_async(session, url) for url in urls]
        return await asyncio.gather(*tasks)
```

---

## 3. Go 并发编程

### 3.1 Goroutine

**Go 的并发模型**：轻量级协程

```go
package main

import (
    "fmt"
    "time"
)

func downloadPage(url string) {
    fmt.Printf("开始下载 %s\n", url)
    time.Sleep(2 * time.Second) // 模拟下载
    fmt.Printf("下载完成 %s\n", url)
}

func main() {
    urls := []string{
        "https://example.com/page1",
        "https://example.com/page2",
        "https://example.com/page3",
    }

    // 启动并发 goroutine
    for _, url := range urls {
        go downloadPage(url)  // go 关键字启动协程
    }

    // 等待所有 goroutine 完成
    time.Sleep(3 * time.Second)
    fmt.Println("所有下载完成")
}
```

### 3.2 Channel（通道）

**用于 goroutine 间通信**：

```go
package main

import "fmt"

func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        fmt.Printf("Worker %d 处理任务 %d\n", id, job)
        results <- job * 2  // 发送结果到 channel
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)

    // 启动3个 worker
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }

    // 发送9个任务
    for j := 1; j <= 9; j++ {
        jobs <- j
    }
    close(jobs)

    // 收集结果
    for a := 1; a <= 9; a++ {
        result := <-results
        fmt.Printf("结果: %d\n", result)
    }
}
```

**输出示例**：
```
Worker 1 处理任务 1
Worker 2 处理任务 2
Worker 3 处理任务 3
Worker 1 处理任务 4
...
结果: 2
结果: 4
结果: 6
...
```

### 3.3 WaitGroup

**等待多个 goroutine 完成**：

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func task(id int, wg *sync.WaitGroup) {
    defer wg.Done()  // 完成时通知

    fmt.Printf("任务 %d 开始\n", id)
    time.Sleep(time.Second)
    fmt.Printf("任务 %d 完成\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)  // 增加等待计数
        go task(i, &wg)
    }

    wg.Wait()  // 阻塞等待所有任务完成
    fmt.Println("所有任务完成")
}
```

### 3.4 实战：并发抓取 API 数据

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "sync"
)

type Result struct {
    URL    string
    Status int
    Error  error
}

func fetchURL(url string, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()

    resp, err := http.Get(url)
    if err != nil {
        results <- Result{URL: url, Error: err}
        return
    }
    defer resp.Body.Close()

    results <- Result{URL: url, Status: resp.StatusCode}
}

func main() {
    urls := []string{
        "https://api.binance.com/api/v3/ticker/price?symbol=BTCUSDT",
        "https://api.binance.com/api/v3/ticker/price?symbol=ETHUSDT",
        "https://api.binance.com/api/v3/ticker/price?symbol=BNBUSDT",
    }

    results := make(chan Result, len(urls))
    var wg sync.WaitGroup

    // 启动并发请求
    for _, url := range urls {
        wg.Add(1)
        go fetchURL(url, results, &wg)
    }

    // 关闭结果通道（在所有任务完成后）
    go func() {
        wg.Wait()
        close(results)
    }()

    // 收集结果
    for result := range results {
        if result.Error != nil {
            fmt.Printf("❌ %s 失败: %v\n", result.URL, result.Error)
        } else {
            fmt.Printf("✅ %s 成功 (状态码: %d)\n", result.URL, result.Status)
        }
    }
}
```

---

## 4. 常见并发问题

### 4.1 竞态条件（Race Condition）

**问题**：多个线程同时修改同一变量

```python
import threading

counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1  # 不是原子操作！

# 启动两个线程
t1 = threading.Thread(target=increment)
t2 = threading.Thread(target=increment)
t1.start()
t2.start()
t1.join()
t2.join()

print(f"Counter: {counter}")  # 期望 200000，实际可能小于
```

**原因**：`counter += 1` 分为3步
1. 读取 counter
2. 加 1
3. 写回 counter

两个线程可能同时执行，导致丢失更新。

### 4.2 使用锁（Lock）

**Python 解决方案**：

```python
import threading

counter = 0
lock = threading.Lock()

def increment_safe():
    global counter
    for _ in range(100000):
        with lock:  # 获取锁
            counter += 1  # 临界区，同一时间只有一个线程执行

t1 = threading.Thread(target=increment_safe)
t2 = threading.Thread(target=increment_safe)
t1.start()
t2.start()
t1.join()
t2.join()

print(f"Counter: {counter}")  # 正确输出 200000
```

**Go 解决方案**：

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var counter int
    var mu sync.Mutex
    var wg sync.WaitGroup

    increment := func() {
        defer wg.Done()
        for i := 0; i < 100000; i++ {
            mu.Lock()
            counter++
            mu.Unlock()
        }
    }

    wg.Add(2)
    go increment()
    go increment()
    wg.Wait()

    fmt.Printf("Counter: %d\n", counter)  // 正确输出 200000
}
```

### 4.3 死锁（Deadlock）

**问题**：两个线程互相等待对方释放锁

```python
import threading

lock1 = threading.Lock()
lock2 = threading.Lock()

def task1():
    with lock1:
        print("任务1获得锁1")
        time.sleep(0.1)
        with lock2:  # 等待锁2
            print("任务1获得锁2")

def task2():
    with lock2:
        print("任务2获得锁2")
        time.sleep(0.1)
        with lock1:  # 等待锁1
            print("任务2获得锁1")

# 可能死锁！
```

**解决方案**：
1. **固定获取锁的顺序**（总是先获取 lock1，再获取 lock2）
2. **使用超时**（`lock.acquire(timeout=1)`）
3. **避免嵌套锁**

---

## 5. 异步编程

### 5.1 什么是异步

**同步 vs 异步**：

```python
# 同步（阻塞）
def get_data():
    data = database.query()  # 等待数据库返回（阻塞）
    return data

# 异步（非阻塞）
async def get_data_async():
    data = await database.query()  # 等待时可以执行其他任务
    return data
```

**类比**：
- **同步**：去餐厅点餐，站在柜台等餐（阻塞）
- **异步**：去餐厅点餐，拿号牌回座位等叫号（非阻塞）

### 5.2 Python asyncio 详解

**基础示例**：

```python
import asyncio

async def fetch_data(delay, id):
    print(f"[{id}] 开始获取数据")
    await asyncio.sleep(delay)  # 模拟异步I/O
    print(f"[{id}] 数据获取完成")
    return f"数据{id}"

async def main():
    # 并发执行3个任务
    results = await asyncio.gather(
        fetch_data(2, 1),
        fetch_data(1, 2),
        fetch_data(3, 3),
    )
    print(f"所有结果: {results}")

asyncio.run(main())
```

**输出**：
```
[1] 开始获取数据
[2] 开始获取数据
[3] 开始获取数据
[2] 数据获取完成  ← 1秒后
[1] 数据获取完成  ← 2秒后
[3] 数据获取完成  ← 3秒后
所有结果: ['数据1', '数据2', '数据3']
```

**实战：异步API调用**：

```python
import asyncio
import aiohttp

async def fetch_price(session, symbol):
    url = f"https://api.binance.com/api/v3/ticker/price?symbol={symbol}"
    async with session.get(url) as response:
        data = await response.json()
        return {symbol: data['price']}

async def get_all_prices():
    symbols = ['BTCUSDT', 'ETHUSDT', 'BNBUSDT', 'ADAUSDT', 'DOGEUSDT']

    async with aiohttp.ClientSession() as session:
        tasks = [fetch_price(session, symbol) for symbol in symbols]
        results = await asyncio.gather(*tasks)

    # 合并结果
    prices = {}
    for result in results:
        prices.update(result)

    return prices

# 运行
prices = asyncio.run(get_all_prices())
print(prices)
# {'BTCUSDT': '45000.00', 'ETHUSDT': '3000.00', ...}
```

---

## 6. NOFX 的并发实践

### 6.1 Goroutine 启动 Trader

**文件**：`cmd/main.go:85`

```go
// 为每个交易员启动 goroutine
for _, trader := range traders {
    wg.Add(1)
    go func(t interfaces.Trader) {
        defer wg.Done()

        if err := t.Start(); err != nil {
            log.Printf("启动交易员失败: %v", err)
        }
    }(trader)
}

// 等待所有交易员完成
wg.Wait()
```

**作用**：多个交易员可以并发运行，互不阻塞。

### 6.2 定时器并发

**文件**：`trader/binance_futures.go:172`

```go
func (t *BinanceFuturesTrader) Start() error {
    ticker := time.NewTicker(t.config.UpdateInterval)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            // 定时执行策略
            if err := t.executeStrategy(); err != nil {
                log.Printf("执行策略失败: %v", err)
            }
        case <-t.stopChan:
            return nil
        }
    }
}
```

**说明**：
- `time.NewTicker`：创建定时器
- `select`：等待多个 channel 事件
- 每隔一段时间执行一次策略判断

### 6.3 API 并发查询

**文件**：`api/server.go:150`

```go
func (s *Server) handleAllPositions(c *gin.Context) {
    var allPositions []interface{}
    var mu sync.Mutex
    var wg sync.WaitGroup

    // 并发获取所有交易员的持仓
    for _, trader := range s.traderManager.GetAllTraders() {
        wg.Add(1)
        go func(t interfaces.Trader) {
            defer wg.Done()

            positions, _ := t.GetPositions()

            mu.Lock()
            allPositions = append(allPositions, positions...)
            mu.Unlock()
        }(trader)
    }

    wg.Wait()
    c.JSON(200, allPositions)
}
```

**优势**：多个交易员的持仓查询并发执行，响应更快。

---

## 7. 实战练习

### 练习 1：并发下载图片

编写程序，并发下载10张图片。

**要求**：
- 使用 `ThreadPoolExecutor`
- 显示下载进度
- 保存到本地

**提示**：
```python
from concurrent.futures import ThreadPoolExecutor
import requests

def download_image(url, filename):
    # 实现下载逻辑
    pass

urls = [
    "https://example.com/image1.jpg",
    "https://example.com/image2.jpg",
    # ...
]

with ThreadPoolExecutor(max_workers=5) as executor:
    # 提交任务
    pass
```

### 练习 2：异步爬虫

使用 `asyncio` 和 `aiohttp` 抓取多个新闻网站的标题。

**要求**：
- 至少5个URL
- 提取标题（使用 BeautifulSoup）
- 测量总耗时

### 练习 3：Go Worker Pool

实现一个 Worker Pool，处理100个任务。

**要求**：
- 5个 worker goroutine
- 使用 channel 传递任务
- 统计每个 worker 处理的任务数

---

## 本章总结

### 核心概念

1. **并发 ≠ 并行**
   - 并发：交替执行
   - 并行：同时执行

2. **选择合适的工具**
   - I/O密集型 → 多线程 / asyncio
   - CPU密集型 → 多进程
   - 高并发I/O → asyncio / goroutine

3. **注意并发问题**
   - 竞态条件 → 使用锁
   - 死锁 → 固定锁顺序
   - 资源泄漏 → 使用 `with` / `defer`

### 最佳实践

1. **不要过度并发**：并发不是越多越好
2. **合理设置并发数**：`ThreadPoolExecutor(max_workers=10)`
3. **使用高级抽象**：优先使用线程池、asyncio，而非手动管理
4. **测试并发代码**：容易出现难以复现的bug

---

**💡 记住**：并发是为了提高效率，而不是增加复杂度。先让代码正确运行，再考虑并发优化！
