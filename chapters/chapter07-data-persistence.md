# 第7章：数据持久化 - 存储设计

> **本章目标**：学会设计数据存储方案，理解数据库的选择和使用

---

## 📋 本章大纲

1. [什么是数据持久化](#1-什么是数据持久化)
2. [存储方案选择](#2-存储方案选择)
3. [数据模型设计](#3-数据模型设计)
4. [数据库操作](#4-数据库操作)
5. [数据迁移和版本管理](#5-数据迁移和版本管理)
6. [NOFX 的存储设计](#6-nofx-的存储设计)
7. [实战练习](#7-实战练习)

**预计学习时间**：3-4 小时

---

## 1. 什么是数据持久化

### 1.1 为什么需要持久化？

**比喻**：数据持久化就像是**把记忆写进日记**

```
不持久化（只在内存中）
┌─────────────────────┐
│ 你记住了一个电话号码  │
│ 💭 "138-1234-5678"  │
└─────────────────────┘
         ↓
    睡一觉醒来
         ↓
    ❌ 忘记了

持久化（存储到硬盘）
┌─────────────────────┐
│ 你把电话号码写进通讯录 │
│ 📓 "138-1234-5678"  │
└─────────────────────┘
         ↓
    睡一觉醒来
         ↓
    ✅ 翻通讯录就能找到
```

**程序也一样**：

```python
# ❌ 不持久化
user_data = {
    "name": "张三",
    "email": "zhang@example.com"
}
# 程序关闭后，数据消失

# ✅ 持久化
import json
with open("user.json", "w") as f:
    json.dump(user_data, f)
# 程序关闭后，数据还在文件里
```

### 1.2 持久化解决什么问题？

| 问题 | 说明 | 解决方案 |
|------|------|----------|
| **数据丢失** | 程序重启后数据消失 | 存储到硬盘 |
| **数据共享** | 多个程序需要访问同一数据 | 使用数据库 |
| **数据查询** | 快速查找特定数据 | 建立索引 |
| **数据备份** | 防止数据损坏 | 定期备份 |
| **数据分析** | 对历史数据进行统计 | 长期存储 |

---

## 2. 存储方案选择

### 2.1 常见存储方案对比

#### 2.1.1 文件存储

**适用场景**：小型项目、配置文件、日志

```python
# JSON 文件
{
    "users": [
        {"id": 1, "name": "张三"},
        {"id": 2, "name": "李四"}
    ]
}

# CSV 文件
id,name,email
1,张三,zhang@example.com
2,李四,li@example.com
```

**优点**：
- ✅ 简单直接，无需安装数据库
- ✅ 适合配置文件和日志
- ✅ 人类可读（JSON/CSV）

**缺点**：
- ❌ 大量数据时性能差
- ❌ 并发访问容易出问题
- ❌ 缺少查询功能
- ❌ 数据一致性难以保证

#### 2.1.2 SQLite（嵌入式数据库）

**适用场景**：桌面应用、小型 Web 应用、移动 App

```python
import sqlite3

# 创建数据库（自动保存到文件）
conn = sqlite3.connect('app.db')
cursor = conn.cursor()

# 创建表
cursor.execute('''
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT,
    email TEXT
)
''')

# 插入数据
cursor.execute("INSERT INTO users VALUES (1, '张三', 'zhang@example.com')")
conn.commit()

# 查询数据
cursor.execute("SELECT * FROM users WHERE name = ?", ('张三',))
print(cursor.fetchone())
```

**优点**：
- ✅ 无需安装服务器
- ✅ 支持 SQL 查询
- ✅ 支持事务
- ✅ 适合中小型数据

**缺点**：
- ❌ 不适合高并发
- ❌ 单个文件大小有限制
- ❌ 不支持多服务器

#### 2.1.3 MySQL/PostgreSQL（关系型数据库）

**适用场景**：中大型 Web 应用、企业系统

```sql
-- MySQL 示例
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (name, email) VALUES ('张三', 'zhang@example.com');

SELECT * FROM users WHERE email LIKE '%@example.com';
```

**优点**：
- ✅ 支持高并发
- ✅ 强大的查询功能
- ✅ 数据一致性保证（ACID）
- ✅ 成熟的生态系统

**缺点**：
- ❌ 需要安装和配置
- ❌ 学习成本较高
- ❌ 灵活性较低（需要预定义表结构）

#### 2.1.4 MongoDB（NoSQL 文档数据库）

**适用场景**：快速迭代、灵活数据结构、大数据

```javascript
// MongoDB 示例
db.users.insertOne({
    name: "张三",
    email: "zhang@example.com",
    tags: ["VIP", "活跃"],
    metadata: {
        lastLogin: new Date(),
        preferences: {...}
    }
})

// 灵活查询
db.users.find({ tags: "VIP" })
```

**优点**：
- ✅ 灵活的数据结构（无需预定义）
- ✅ 适合快速迭代
- ✅ 水平扩展方便
- ✅ 适合非结构化数据

**缺点**：
- ❌ 不支持复杂的关联查询
- ❌ 数据一致性较弱
- ❌ 学习成本高

#### 2.1.5 Redis（内存数据库）

**适用场景**：缓存、会话存储、实时数据

```python
import redis

r = redis.Redis()

# 存储字符串
r.set('user:1:name', '张三')

# 存储哈希
r.hset('user:1', mapping={
    'name': '张三',
    'email': 'zhang@example.com'
})

# 设置过期时间（10分钟后自动删除）
r.expire('user:1:session', 600)

# 列表操作
r.lpush('recent_visitors', 'user:1')
```

**优点**：
- ✅ 极快的读写速度
- ✅ 支持多种数据结构
- ✅ 自动过期机制
- ✅ 适合缓存和临时数据

**缺点**：
- ❌ 数据存储在内存中（成本高）
- ❌ 数据量受内存限制
- ❌ 持久化性能有限

### 2.2 如何选择存储方案？

**决策树**：

```
开始
 ├─ 数据量 < 10MB？
 │   ├─ 是 → 需要查询功能？
 │   │   ├─ 是 → SQLite
 │   │   └─ 否 → JSON/CSV 文件
 │   └─ 否 → 继续
 │
 ├─ 需要高速缓存？
 │   └─ 是 → Redis（配合持久化数据库）
 │
 ├─ 数据结构固定？
 │   ├─ 是 → MySQL/PostgreSQL
 │   └─ 否 → 数据关联复杂？
 │       ├─ 是 → PostgreSQL（支持 JSON）
 │       └─ 否 → MongoDB
 │
 └─ 并发量高？
     ├─ 是 → MySQL/PostgreSQL + Redis
     └─ 否 → SQLite
```

**实际项目示例**：

| 项目类型 | 推荐方案 | 原因 |
|---------|---------|------|
| 个人博客 | SQLite | 数据量小，部署简单 |
| 电商系统 | MySQL + Redis | 数据结构固定，需要事务，缓存提速 |
| 社交应用 | PostgreSQL + Redis | 复杂关联查询，会话缓存 |
| 实时消息 | MongoDB + Redis | 灵活数据结构，实时推送 |
| 数据分析 | PostgreSQL | 强大的分析函数 |

---

## 3. 数据模型设计

### 3.1 关系型数据库设计

#### 3.1.1 实体关系分析

**案例：交易系统**

```
实体识别：
┌─────────┐     ┌─────────┐     ┌─────────┐
│  用户    │────│  订单    │────│  持仓    │
└─────────┘     └─────────┘     └─────────┘
     │              │                │
     │              │                │
  ┌──┴──┐       ┌──┴──┐         ┌──┴──┐
  │ 账户 │       │ 交易 │         │ 资产 │
  └─────┘       └─────┘         └─────┘

关系：
- 用户 → 账户（1对多）
- 账户 → 订单（1对多）
- 订单 → 交易（1对多）
- 账户 → 持仓（1对多）
```

#### 3.1.2 表设计

```sql
-- 用户表
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_email (email),
    INDEX idx_username (username)
);

-- 账户表
CREATE TABLE accounts (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    exchange VARCHAR(50) NOT NULL,  -- 'hyperliquid', 'binance'
    api_key VARCHAR(255) NOT NULL,
    api_secret VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_exchange (user_id, exchange)
);

-- 持仓表
CREATE TABLE positions (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    account_id BIGINT NOT NULL,
    symbol VARCHAR(20) NOT NULL,        -- 'BTC-USDT'
    side VARCHAR(10) NOT NULL,          -- 'long', 'short'
    quantity DECIMAL(20, 8) NOT NULL,   -- 持仓数量
    entry_price DECIMAL(20, 8) NOT NULL,-- 开仓价格
    leverage INT NOT NULL,              -- 杠杆倍数
    unrealized_pnl DECIMAL(20, 8),      -- 未实现盈亏
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE,
    INDEX idx_account_symbol (account_id, symbol, side),
    UNIQUE KEY uk_position (account_id, symbol, side)  -- 同一账户同一币种同一方向只能有一个持仓
);

-- 交易记录表
CREATE TABLE trades (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    account_id BIGINT NOT NULL,
    symbol VARCHAR(20) NOT NULL,
    side VARCHAR(10) NOT NULL,          -- 'buy', 'sell'
    direction VARCHAR(10) NOT NULL,     -- 'open', 'close'
    quantity DECIMAL(20, 8) NOT NULL,
    price DECIMAL(20, 8) NOT NULL,
    fee DECIMAL(20, 8) DEFAULT 0,
    realized_pnl DECIMAL(20, 8),        -- 实现盈亏（平仓时）
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE,
    INDEX idx_account_symbol_time (account_id, symbol, created_at),
    INDEX idx_created_at (created_at)
);
```

### 3.2 设计原则

**范式化（Normalization）**

**第一范式（1NF）**：每个字段都是原子的

```sql
-- ❌ 违反 1NF
CREATE TABLE orders (
    id INT,
    items VARCHAR(500)  -- "苹果,香蕉,橙子"（不是原子的）
);

-- ✅ 符合 1NF
CREATE TABLE orders (
    id INT PRIMARY KEY
);
CREATE TABLE order_items (
    order_id INT,
    item_name VARCHAR(50),
    FOREIGN KEY (order_id) REFERENCES orders(id)
);
```

**第二范式（2NF）**：消除部分依赖

```sql
-- ❌ 违反 2NF
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(50),  -- product_name 只依赖 product_id
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- ✅ 符合 2NF
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

**第三范式（3NF）**：消除传递依赖

```sql
-- ❌ 违反 3NF
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT,
    department_name VARCHAR(50),  -- department_name 依赖 department_id
    department_location VARCHAR(100)
);

-- ✅ 符合 3NF
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    location VARCHAR(100)
);
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

---

## 4. NOFX 的存储设计

### 4.1 NOFX 的存储方案分析

通过分析 NOFX 的代码，我们发现它采用了**极简持久化策略**：

**存储方案**：
```
1. 配置数据 → JSON 文件（config.json）
2. 运行时数据 → 内存存储（Go maps）
3. 持仓/账户 → 交易所 API 实时查询
4. 日志 → 文本文件
```

**为什么 NOFX 不使用数据库？**

| 考虑因素 | NOFX 的选择 | 原因 |
|---------|------------|------|
| **数据量** | 小（几个trader） | ✅ 无需数据库 |
| **数据持久化** | 配置需要，运行数据不需要 | ✅ JSON足够 |
| **查询需求** | 简单（按trader ID查询） | ✅ map查找即可 |
| **数据恢复** | 重启时从交易所重新获取 | ✅ 无需本地存储 |
| **部署复杂度** | 越简单越好 | ✅ 避免依赖数据库 |

### 4.2 NOFX 的数据结构设计

#### 4.2.1 配置数据（JSON持久化）

文件：`config/config.go:54`

```go
type Config struct {
    Traders            []TraderConfig `json:"traders"`
    UseDefaultCoins    bool           `json:"use_default_coins"`
    DefaultCoins       []string       `json:"default_coins"`
    CoinPoolAPIURL     string         `json:"coin_pool_api_url"`
    APIServerPort      int            `json:"api_server_port"`
    MaxDailyLoss       float64        `json:"max_daily_loss"`
    Leverage           LeverageConfig `json:"leverage"`
}

// 加载配置
func LoadConfig(filename string) (*Config, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("读取配置文件失败: %w", err)
    }

    var config Config
    if err := json.Unmarshal(data, &config); err != nil {
        return nil, fmt.Errorf("解析配置文件失败: %w", err)
    }

    return &config, nil
}
```

**优点**：
- ✅ 人类可读可编辑
- ✅ 版本控制友好（Git）
- ✅ 无需数据库

#### 4.2.2 运行时数据（内存存储）

文件：`manager/trader_manager.go:13`

```go
type TraderManager struct {
    traders map[string]*trader.AutoTrader  // key: trader ID
    mu      sync.RWMutex                   // 读写锁，保证并发安全
}

func NewTraderManager() *TraderManager {
    return &TraderManager{
        traders: make(map[string]*trader.AutoTrader),
    }
}

// 获取trader（并发安全）
func (tm *TraderManager) GetTrader(id string) (*trader.AutoTrader, error) {
    tm.mu.RLock()         // 读锁
    defer tm.mu.RUnlock()

    t, exists := tm.traders[id]
    if !exists {
        return nil, fmt.Errorf("trader ID '%s' 不存在", id)
    }
    return t, nil
}
```

**为什么使用内存而不是数据库？**

1. **数据生命周期短**：Trader 状态在程序运行期间有效，重启后重新初始化
2. **性能要求高**：交易决策需要毫秒级响应
3. **数据来源是交易所**：真实持仓/账户数据从交易所 API 实时获取

### 4.3 NOFX 存储设计的优缺点

**优点**：

| 优点 | 说明 |
|------|------|
| **极简部署** | 不需要安装数据库，只需运行二进制文件 |
| **数据准确** | 持仓/账户数据实时从交易所获取，避免本地缓存不一致 |
| **高性能** | 内存访问速度快，适合高频交易决策 |
| **易于理解** | 代码简单，新手容易上手 |

**缺点**：

| 缺点 | 影响 | 解决方案 |
|------|------|----------|
| **无历史数据** | 重启后丢失交易历史 | 可添加日志文件或数据库 |
| **无统计分析** | 无法分析历史表现 | 可添加 SQLite 存储历史交易 |
| **数据不可恢复** | 程序崩溃可能丢失状态 | 对于交易系统可接受（数据在交易所） |

### 4.4 如果 NOFX 需要添加数据库

**场景**：用户希望保存历史交易记录用于分析

**推荐方案**：SQLite

**为什么选择 SQLite？**
- ✅ 无需独立服务器
- ✅ 单文件部署
- ✅ 与 NOFX 极简理念一致

**实现示例**：

```go
// repository/trade_history_repo.go
package repository

import (
    "database/sql"
    "time"
    _ "github.com/mattn/go-sqlite3"
)

type TradeHistory struct {
    ID         int64
    TraderID   string
    Symbol     string
    Side       string
    Quantity   float64
    Price      float64
    PnL        float64
    Timestamp  time.Time
}

type TradeHistoryRepo struct {
    db *sql.DB
}

func NewTradeHistoryRepo(dbPath string) (*TradeHistoryRepo, error) {
    db, err := sql.Open("sqlite3", dbPath)
    if err != nil {
        return nil, err
    }

    // 创建表
    _, err = db.Exec(`
        CREATE TABLE IF NOT EXISTS trade_history (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            trader_id TEXT NOT NULL,
            symbol TEXT NOT NULL,
            side TEXT NOT NULL,
            quantity REAL NOT NULL,
            price REAL NOT NULL,
            pnl REAL,
            timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
        );
        CREATE INDEX IF NOT EXISTS idx_trader_time
        ON trade_history(trader_id, timestamp);
    `)
    if err != nil {
        return nil, err
    }

    return &TradeHistoryRepo{db: db}, nil
}

func (r *TradeHistoryRepo) SaveTrade(trade *TradeHistory) error {
    query := `
        INSERT INTO trade_history (trader_id, symbol, side, quantity, price, pnl, timestamp)
        VALUES (?, ?, ?, ?, ?, ?, ?)
    `
    _, err := r.db.Exec(query,
        trade.TraderID, trade.Symbol, trade.Side,
        trade.Quantity, trade.Price, trade.PnL,
        trade.Timestamp,
    )
    return err
}

func (r *TradeHistoryRepo) GetTraderHistory(traderID string, days int) ([]*TradeHistory, error) {
    query := `
        SELECT * FROM trade_history
        WHERE trader_id = ? AND timestamp > datetime('now', '-' || ? || ' days')
        ORDER BY timestamp DESC
    `

    rows, err := r.db.Query(query, traderID, days)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var trades []*TradeHistory
    for rows.Next() {
        var t TradeHistory
        err := rows.Scan(&t.ID, &t.TraderID, &t.Symbol, &t.Side,
                         &t.Quantity, &t.Price, &t.PnL, &t.Timestamp)
        if err != nil {
            return nil, err
        }
        trades = append(trades, &t)
    }
    return trades, nil
}
```

---

## 5. 存储设计最佳实践

### 5.1 选择合适的存储方案

**决策标准**：

```python
def choose_storage(data_size, query_complexity, write_frequency):
    """
    根据项目需求选择存储方案

    data_size: 数据量（MB）
    query_complexity: 查询复杂度（1-5）
    write_frequency: 写入频率（次/秒）
    """
    if data_size < 10 and query_complexity <= 2:
        return "JSON/CSV 文件"

    if data_size < 1000 and query_complexity <= 3:
        return "SQLite"

    if query_complexity >= 4 or write_frequency > 100:
        return "MySQL/PostgreSQL"

    return "根据具体需求评估"

# 示例
print(choose_storage(5, 1, 0.1))     # "JSON/CSV 文件"
print(choose_storage(500, 3, 10))    # "SQLite"
print(choose_storage(50000, 5, 200)) # "MySQL/PostgreSQL"
```

### 5.2 数据备份策略

**自动备份脚本**：

```bash
#!/bin/bash
# backup_db.sh

DB_NAME="trading_db"
BACKUP_DIR="/var/backups/db"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${DATE}.sql"

# 创建备份目录
mkdir -p $BACKUP_DIR

# 备份数据库
mysqldump -u root -p$DB_PASSWORD $DB_NAME > $BACKUP_FILE

# 压缩备份文件
gzip $BACKUP_FILE

# 只保留最近 7 天的备份
find $BACKUP_DIR -name "${DB_NAME}_*.sql.gz" -mtime +7 -delete

echo "备份完成: ${BACKUP_FILE}.gz"
```

**定时备份（crontab）**：

```bash
# 每天凌晨 2 点自动备份
0 2 * * * /path/to/backup_db.sh
```

### 5.3 性能优化

**1. 使用索引**

```sql
-- ❌ 慢查询（无索引）
SELECT * FROM trades WHERE account_id = 123 AND symbol = 'BTCUSDT';
-- 执行时间：500ms（全表扫描）

-- ✅ 快查询（有索引）
CREATE INDEX idx_account_symbol ON trades(account_id, symbol);
SELECT * FROM trades WHERE account_id = 123 AND symbol = 'BTCUSDT';
-- 执行时间：5ms（索引查找）
```

**2. 避免 N+1 查询**

```python
# ❌ N+1 查询问题
orders = db.query("SELECT * FROM orders")
for order in orders:
    user = db.query(f"SELECT * FROM users WHERE id = {order.user_id}")
    print(f"{order.id}: {user.name}")
# 执行了 1 + N 次查询

# ✅ 使用 JOIN
orders = db.query("""
    SELECT o.*, u.name
    FROM orders o
    JOIN users u ON o.user_id = u.id
""")
for order in orders:
    print(f"{order.id}: {order.name}")
# 只执行 1 次查询
```

**3. 使用连接池**

```python
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    'mysql://user:pass@localhost/db',
    poolclass=QueuePool,
    pool_size=5,        # 连接池大小
    max_overflow=10,    # 超出 pool_size 后最多再创建的连接数
    pool_timeout=30,    # 获取连接的超时时间
    pool_recycle=3600   # 连接回收时间（秒）
)
```

---

## 6. 实战练习

### 练习 1：设计博客数据库

**需求**：
- 用户可以发布文章
- 文章可以有多个标签
- 用户可以评论文章

**任务**：
1. 画出实体关系图
2. 设计表结构（包括主键、外键、索引）
3. 写出 CREATE TABLE 语句

<details>
<summary>参考答案</summary>

```sql
-- 用户表
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 文章表
CREATE TABLE articles (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_user (user_id),
    INDEX idx_created (created_at)
);

-- 标签表
CREATE TABLE tags (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- 文章-标签关联表（多对多）
CREATE TABLE article_tags (
    article_id BIGINT,
    tag_id BIGINT,
    PRIMARY KEY (article_id, tag_id),
    FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);

-- 评论表
CREATE TABLE comments (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    article_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_article (article_id),
    INDEX idx_created (created_at)
);
```

</details>

### 练习 2：实现 Repository 模式

**任务**：为练习 1 的博客系统实现 `ArticleRepository`，包含以下方法：
- `Create(article)` - 创建文章
- `GetByID(id)` - 根据 ID 查询文章
- `GetByUser(userID)` - 查询用户的所有文章
- `Update(article)` - 更新文章
- `Delete(id)` - 删除文章

**要求**：使用 Go 或 Python 实现

### 练习 3：为 NOFX 添加交易历史存储

**任务**：
1. 设计 `trade_history` 表结构
2. 实现 `TradeHistoryRepository`
3. 在 `AutoTrader` 中集成，在每次平仓时保存记录
4. 实现查询功能：获取过去 N 天的交易记录

**提示**：参考 4.4 节的示例代码

### 练习 4：数据库性能优化

**场景**：以下查询很慢

```sql
SELECT * FROM trades
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31'
AND symbol = 'BTCUSDT'
ORDER BY created_at DESC;
```

**任务**：
1. 分析为什么慢
2. 添加合适的索引
3. 优化查询语句

<details>
<summary>参考答案</summary>

```sql
-- 1. 添加复合索引
CREATE INDEX idx_symbol_created ON trades(symbol, created_at);

-- 2. 优化后的查询（使用索引）
SELECT * FROM trades
WHERE symbol = 'BTCUSDT'
AND created_at BETWEEN '2024-01-01' AND '2024-12-31'
ORDER BY created_at DESC;

-- 3. 如果只需要部分字段，避免 SELECT *
SELECT id, symbol, side, quantity, price, created_at
FROM trades
WHERE symbol = 'BTCUSDT'
AND created_at BETWEEN '2024-01-01' AND '2024-12-31'
ORDER BY created_at DESC;
```

**原因**：
- 复合索引 `(symbol, created_at)` 可以同时满足 WHERE 和 ORDER BY
- 避免 `SELECT *` 减少数据传输量

</details>

---

## 7. 本章总结

### 核心概念

| 概念 | 说明 | 何时使用 |
|------|------|----------|
| **JSON/CSV** | 文件存储 | 小数据量（< 10MB），配置文件 |
| **SQLite** | 嵌入式数据库 | 中小型应用，单机部署 |
| **MySQL/PostgreSQL** | 关系型数据库 | 中大型应用，需要事务 |
| **MongoDB** | NoSQL数据库 | 灵活数据结构，快速迭代 |
| **Redis** | 内存数据库 | 缓存，实时数据 |
| **Repository 模式** | 数据访问层抽象 | 所有项目（解耦业务和存储） |

### 设计原则

1. **根据需求选择方案**：不要过度设计，也不要欠设计
2. **使用 Repository 模式**：解耦业务逻辑和数据存储
3. **考虑扩展性**：从简单开始，预留扩展空间
4. **重视数据安全**：备份、事务、并发控制

### NOFX 的启示

- **极简主义**：不需要数据库就不用，降低部署复杂度
- **数据源头**：交易数据在交易所，无需重复存储
- **明确需求**：区分哪些数据需要持久化，哪些不需要
- **渐进增强**：先运行起来，有需要再添加数据库

---

## 📚 扩展阅读

- [数据库设计规范](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)
- [SQLite 使用指南](https://www.sqlite.org/docs.html)
- [Repository Pattern 详解](https://martinfowler.com/eaaCatalog/repository.html)

---

## 📌 下一章预告

**第8章：API 服务层 - 前后端协作**
- RESTful API 设计
- HTTP 请求处理
- 认证和授权
- NOFX API 服务器分析

---

**💡 记住**：数据持久化不是必需的，要根据实际需求选择合适的方案。NOFX 的极简设计告诉我们：能不用数据库就不用，能用 JSON 就不用 SQLite，能用 SQLite 就不用 MySQL。**简单，就是最好的设计。**
