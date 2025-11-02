# 第3章：模块化拆分 - 分治思维

> **本章目标**：学会将大问题拆成小问题，掌握模块划分和接口设计的核心技能

---

## 🎯 核心概念

### 什么是模块化？

**用组装电脑类比**：

```
❌ 买一台不能拆的一体机
   - CPU 坏了 → 整台报废
   - 想升级内存 → 做不到

✅ 买组装机（模块化）
   - CPU、内存、硬盘、显卡都是独立模块
   - 坏了一个 → 只换那一个
   - 想升级 → 换对应模块
```

**软件模块化就是**：
- 将系统拆分成**独立的功能单元**
- 每个模块有**清晰的职责**
- 通过**接口**相互通信
- 可以**独立开发、测试、替换**

### 为什么需要模块化？

#### 场景：开发一个电商网站

**❌ 不模块化（所有代码混在一起）**：

```python
# app.py - 1000 行代码
def main():
    # 用户登录
    if request.path == "/login":
        username = request.form["username"]
        password = request.form["password"]
        user = db.query("SELECT * FROM users WHERE ...")
        if check_password(password, user.password_hash):
            session["user_id"] = user.id

    # 商品列表
    elif request.path == "/products":
        products = db.query("SELECT * FROM products ...")
        for p in products:
            p.discount_price = p.price * 0.8  # 打折逻辑

    # 下单
    elif request.path == "/order":
        cart = session["cart"]
        total = 0
        for item in cart:
            total += item.price * item.quantity
        # 计算运费...
        # 扣减库存...
        # 发送邮件...
        # ...
```

**问题**：
- 🔴 想改打折规则 → 要找到那段代码在哪
- 🔴 想换数据库 → 到处都在用 `db.query()`
- 🔴 想复用下单逻辑 → 复制粘贴代码
- 🔴 多人开发 → 都在改同一个文件，冲突不断
- 🔴 测试 → 必须启动整个系统

**✅ 模块化（职责清晰）**：

```
ecommerce/
├── auth/              # 认证模块
│   ├── login.py       # 登录
│   ├── register.py    # 注册
│   └── permissions.py # 权限检查
│
├── products/          # 商品模块
│   ├── catalog.py     # 商品列表
│   ├── detail.py      # 商品详情
│   └── pricing.py     # 价格计算（含打折）
│
├── orders/            # 订单模块
│   ├── cart.py        # 购物车
│   ├── checkout.py    # 结账
│   └── shipping.py    # 运费计算
│
├── inventory/         # 库存模块
│   └── stock.py       # 库存管理
│
├── notifications/     # 通知模块
│   ├── email.py       # 邮件
│   └── sms.py         # 短信
│
└── database/          # 数据访问模块
    └── db.py          # 统一数据库操作
```

**优势**：
- ✅ 修改打折规则 → 只改 `products/pricing.py`
- ✅ 换数据库 → 只改 `database/db.py`
- ✅ 测试下单流程 → 只测试 `orders/` 模块
- ✅ 多人协作 → A 做订单，B 做商品，互不干扰
- ✅ 复用代码 → 导入 `from products import pricing`

---

## 🧠 思维方法

### 方法一：按业务功能拆分（推荐）⭐

**原则**：一个模块 = 一个业务领域

#### NOFX 的业务功能拆分

```
交易系统的业务功能有哪些？

1. 配置管理 → config 模块
2. 交易执行 → trader 模块
3. 市场数据 → market 模块
4. 币种筛选 → pool 模块
5. AI 通信 → mcp 模块
6. 决策引擎 → decision 模块
7. 日志记录 → logger 模块
8. 多实例管理 → manager 模块
9. API 服务 → api 模块
10. 前端展示 → web 模块
```

**判断标准**：
- ✅ 这个模块能用一句话说清楚职责
- ✅ 删除这个模块，系统少一个核心功能
- ✅ 这个模块的代码改动，不影响其他模块

#### 反例：按技术拆分（不推荐）

```
❌ 按技术拆分（难以维护）
project/
├── models/       # 所有数据模型
├── views/        # 所有视图
├── controllers/  # 所有控制器
└── utils/        # 所有工具函数
```

**问题**：
- 开发一个功能要改 4 个目录
- 不知道某个功能的代码在哪
- 模块间依赖复杂

### 方法二：单一职责原则

**定义**：一个模块只做一件事，只有一个改变的理由

#### 好的例子

```go
// market/data.go - 只负责市场数据
package market

func FetchKlines(symbol string) ([]Kline, error) {
    // 获取 K 线数据
}

func CalculateRSI(prices []float64) float64 {
    // 计算 RSI 指标
}

func CalculateMACD(prices []float64) MACD {
    // 计算 MACD 指标
}
```

**改变的理由**：
- 交易所 API 变了 → 改 `FetchKlines`
- RSI 算法优化 → 改 `CalculateRSI`

**但不会因为**：
- 用户界面改了 → 不改这个模块
- 数据库换了 → 不改这个模块

#### 坏的例子

```go
// ❌ 一个模块做太多事
package everything

func FetchDataAndAnalyzeAndTrade(symbol string) {
    // 获取数据
    klines := fetch(symbol)

    // 分析数据
    rsi := calculateRSI(klines)

    // 做决策
    if rsi > 70 {
        // 执行交易
        placeSellOrder(symbol)
    }

    // 记录日志
    saveToDatabase(...)
}
```

**问题**：
- 改数据源 → 要改这个函数
- 改分析算法 → 要改这个函数
- 改交易逻辑 → 要改这个函数
- 改日志格式 → 还是要改这个函数

**一个函数有 4 个改变的理由！违反单一职责！**

### 方法三：高内聚、低耦合

#### 高内聚

**定义**：相关的东西放在一起

```
✅ 高内聚（好）
trader/
├── binance_futures.go      # Binance 交易
├── hyperliquid_trader.go   # Hyperliquid 交易
└── aster_trader.go         # Aster 交易

→ 所有交易相关的代码在一个模块

❌ 低内聚（坏）
project/
├── binance.go       # Binance 代码
├── config.go        # 配置
├── hyperliquid.go   # Hyperliquid 代码
└── logger.go        # 日志

→ 交易代码分散在各处
```

#### 低耦合

**定义**：模块间依赖尽量少

```
✅ 低耦合（通过接口）
decision 模块 → 调用 Trader 接口 → 不关心具体是哪个交易所

❌ 高耦合（直接依赖）
decision 模块 → 直接调用 BinanceFutures 类 → 换交易所要改代码
```

**耦合度对比**：

| 耦合类型 | 示例 | 耦合度 |
|----------|------|--------|
| 无耦合 | 两个模块互不知道 | 0 ⭐ |
| 数据耦合 | 通过参数传递数据 | 低 ⭐ |
| 接口耦合 | 调用定义好的接口 | 低 ⭐ |
| 控制耦合 | 传递控制标志 | 中 |
| 内容耦合 | 直接访问内部数据 | 高 ❌ |

### 方法四：依赖倒置（接口抽象）

**核心思想**：依赖抽象，不依赖具体实现

#### 传统方式（高层依赖低层）

```go
// ❌ 高层模块直接依赖低层模块
package decision

import "nofx/trader/binance_futures"

func MakeDecision() {
    trader := binance_futures.New()  // 直接依赖具体实现
    account := trader.GetAccount()
}
```

**问题**：
- 想换成 Hyperliquid → 要改 decision 代码
- 想同时支持多个交易所 → 困难

#### 依赖倒置（依赖接口）⭐

```go
// ✅ 高层模块依赖接口抽象
package decision

import "nofx/trader"

func MakeDecision(trader trader.Trader) {  // 依赖接口
    account := trader.GetAccount()
}

// trader/interface.go
type Trader interface {
    GetAccount() Account
    GetPositions() []Position
    OpenLong(...) error
    OpenShort(...) error
}

// 任何实现了这个接口的都可以用
// - BinanceFutures
// - HyperliquidTrader
// - AsterTrader
```

**优势**：
- ✅ `decision` 模块不知道具体交易所
- ✅ 添加新交易所不影响 `decision`
- ✅ 可以轻松替换实现

**依赖方向**：

```
传统方式：
高层模块 → 低层模块
(decision) → (BinanceFutures)

依赖倒置：
高层模块 → 接口 ← 低层模块
(decision) → (Trader Interface) ← (BinanceFutures)
                                ← (HyperliquidTrader)
```

---

## 📚 NOFX 案例分析

### NOFX 的模块划分全景

```
nofx/
├── main.go                  # 🎯 入口：初始化所有模块
│
├── config/                  # 📋 配置模块
│   └── config.go            # 加载 config.json
│
├── manager/                 # 👔 管理模块
│   └── trader_manager.go    # 管理多个 Trader 实例
│
├── trader/                  # 💱 交易执行模块
│   ├── interface.go         # Trader 接口定义
│   ├── binance_futures.go   # Binance 实现
│   ├── hyperliquid_trader.go# Hyperliquid 实现
│   ├── aster_trader.go      # Aster 实现
│   └── auto_trader.go       # 自动交易主控制器
│
├── market/                  # 📊 市场数据模块
│   └── data.go              # K线获取、技术指标计算
│
├── pool/                    # 🎲 币种池模块
│   └── coin_pool.go         # 候选币种筛选
│
├── mcp/                     # 🤖 AI 通信模块
│   └── client.go            # DeepSeek/Qwen API
│
├── decision/                # 🧠 决策引擎模块
│   └── engine.go            # 提示词构建、JSON 解析
│
├── logger/                  # 📝 日志模块
│   └── decision_logger.go   # 决策记录、性能追踪
│
├── api/                     # 🌐 API 服务模块
│   └── server.go            # Gin HTTP 服务器
│
└── web/                     # 🎨 前端模块
    ├── src/
    │   ├── App.tsx          # 主应用
    │   ├── components/      # 组件
    │   ├── lib/api.ts       # API 调用
    │   └── types/index.ts   # 类型定义
    └── package.json
```

### 模块详细分析

#### 模块 1：config（配置模块）

**职责**：
- 读取 `config.json` 文件
- 验证配置参数
- 提供配置给其他模块

**输入**：
- `config.json` 文件路径

**输出**：
- `Config` 结构体

**对外接口**：
```go
// config/config.go
type Config struct {
    Traders         []TraderConfig
    Leverage        LeverageConfig
    UseDefaultCoins bool
    APIServerPort   int
}

func LoadConfig(filepath string) (*Config, error)
```

**依赖**：
- 无（最底层模块）

**设计亮点**：
- ✅ 单一职责：只管配置
- ✅ 独立性强：不依赖其他模块
- ✅ 容易测试：传入不同 JSON 测试

---

#### 模块 2：trader（交易执行模块）

**职责**：
- 对接不同交易所 API
- 执行开仓、平仓操作
- 获取账户、持仓信息

**核心设计**：接口抽象

```go
// trader/interface.go
type Trader interface {
    // 账户相关
    GetAccount() (Account, error)
    GetPositions() ([]Position, error)

    // 交易相关
    OpenLong(symbol string, quantity float64, leverage int) error
    OpenShort(symbol string, quantity float64, leverage int) error
    CloseLong(symbol string, quantity float64) error
    CloseShort(symbol string, quantity float64) error

    // 风控相关
    SetStopLoss(symbol string, side string, price float64) error
    SetTakeProfit(symbol string, side string, price float64) error
}
```

**实现类**：
- `BinanceFutures` - 实现 Binance API 调用
- `HyperliquidTrader` - 实现 Hyperliquid API 调用
- `AsterTrader` - 实现 Aster API 调用

**依赖**：
- 第三方库：`github.com/adshao/go-binance/v2`

**设计亮点**：
- ✅ 接口统一：所有交易所用同一接口
- ✅ 易扩展：添加新交易所只需实现接口
- ✅ 可替换：切换交易所不影响上层代码

**代码示例**：

```go
// trader/binance_futures.go
type BinanceFutures struct {
    client    *futures.Client
    apiKey    string
    secretKey string
}

func (b *BinanceFutures) GetAccount() (Account, error) {
    account, err := b.client.NewGetAccountService().Do(context.Background())
    if err != nil {
        return Account{}, err
    }

    return Account{
        TotalEquity:      parseFloat(account.TotalWalletBalance),
        AvailableBalance: parseFloat(account.AvailableBalance),
    }, nil
}

func (b *BinanceFutures) OpenLong(symbol string, quantity float64, leverage int) error {
    // 1. 设置杠杆
    _, err := b.client.NewChangeLeverageService().
        Symbol(symbol).
        Leverage(leverage).
        Do(context.Background())

    // 2. 下市价单
    _, err = b.client.NewCreateOrderService().
        Symbol(symbol).
        Side(futures.SideTypeBuy).
        Type(futures.OrderTypeMarket).
        Quantity(fmt.Sprintf("%.8f", quantity)).
        Do(context.Background())

    return err
}
```

---

#### 模块 3：market（市场数据模块）

**职责**：
- 获取 K 线数据
- 计算技术指标（RSI、MACD、EMA）
- 获取持仓量数据

**对外接口**：
```go
// market/data.go
type Data struct {
    Symbol           string
    Klines3m         []Kline      // 3分钟 K 线
    Klines4h         []Kline      // 4小时 K 线
    RSI3m            float64      // 3分钟 RSI(7)
    RSI4h            float64      // 4小时 RSI(14)
    MACD3m           MACD         // 3分钟 MACD
    EMA20_3m         float64      // 3分钟 EMA20
    // ...
}

func FetchData(symbol string, exchange string) (*Data, error)
func CalculateIndicators(data *Data) error
```

**依赖**：
- TA-Lib：技术指标计算库
- 交易所 API：获取原始数据

**设计亮点**：
- ✅ 数据获取和计算分离
- ✅ 统一的数据结构
- ✅ 缓存机制（避免重复请求）

**代码示例**：

```go
func CalculateRSI(prices []float64, period int) float64 {
    rsi := talib.Rsi(prices, period)
    return rsi[len(rsi)-1]  // 返回最新值
}

func CalculateMACD(prices []float64) MACD {
    macd, signal, hist := talib.Macd(prices, 12, 26, 9)
    return MACD{
        MACD:   macd[len(macd)-1],
        Signal: signal[len(signal)-1],
        Hist:   hist[len(hist)-1],
    }
}
```

---

#### 模块 4：decision（决策引擎模块）

**职责**：
- 构建 AI 提示词
- 调用 AI API
- 解析 AI 返回的 JSON

**输入**：
- `Context`（账户、持仓、市场数据、历史表现）

**输出**：
- `FullDecision`（AI 的决策列表）

**对外接口**：
```go
// decision/engine.go
type Context struct {
    Account        AccountInfo
    Positions      []PositionInfo
    CandidateCoins []CandidateCoin
    MarketDataMap  map[string]*market.Data
    Performance    interface{}  // 历史表现
}

type Decision struct {
    Symbol          string
    Action          string  // "open_long", "close_long", ...
    Leverage        int
    PositionSizeUSD float64
    StopLoss        float64
    TakeProfit      float64
    Reasoning       string
}

func GetFullDecision(ctx *Context, mcpClient *mcp.Client) (*FullDecision, error)
```

**核心流程**：
```go
func GetFullDecision(ctx *Context, mcpClient *mcp.Client) (*FullDecision, error) {
    // 1. 获取市场数据
    fetchMarketDataForContext(ctx)

    // 2. 构建提示词
    systemPrompt := buildSystemPrompt(...)
    userPrompt := buildUserPrompt(ctx)

    // 3. 调用 AI
    response, err := mcpClient.Chat(systemPrompt, userPrompt)

    // 4. 解析 JSON
    var decisions []Decision
    json.Unmarshal([]byte(response), &decisions)

    return &FullDecision{
        UserPrompt: userPrompt,
        CoTTrace:   response,
        Decisions:  decisions,
    }, nil
}
```

**依赖**：
- `mcp` 模块：AI 通信
- `market` 模块：市场数据
- `logger` 模块：历史表现

**设计亮点**：
- ✅ 提示词构建逻辑集中
- ✅ 易于调整 AI 输入
- ✅ 完整记录输入输出（调试）

---

#### 模块 5：logger（日志模块）

**职责**：
- 记录每次决策
- 追踪交易历史
- 计算性能指标

**对外接口**：
```go
// logger/decision_logger.go
type DecisionLogger struct {
    traderID         string
    logDir           string
    openPositions    map[string]*OpenPosition  // 未平仓记录
    tradeHistory     []*CompletedTrade         // 已完成交易
    equitySnapshots  []EquitySnapshot          // 权益快照
}

func (l *DecisionLogger) LogDecision(decision *FullDecision, account Account) error
func (l *DecisionLogger) RecordOpenPosition(symbol, side string, ...) error
func (l *DecisionLogger) RecordClosePosition(symbol, side string, ...) error
func (l *DecisionLogger) GetPerformanceAnalysis() PerformanceAnalysis
```

**存储格式**：
- 决策日志：`decision_logs/{trader_id}/{timestamp}.json`
- 性能数据：内存数据库

**设计亮点**：
- ✅ 完整记录决策过程
- ✅ 自动配对开仓/平仓
- ✅ 实时计算性能指标

---

#### 模块 6：manager（管理模块）

**职责**：
- 创建多个 `AutoTrader` 实例
- 管理 Trader 生命周期
- 提供统一查询接口

**对外接口**：
```go
// manager/trader_manager.go
type TraderManager struct {
    traders map[string]*trader.AutoTrader
}

func NewTraderManager() *TraderManager
func (m *TraderManager) AddTrader(config TraderConfig) error
func (m *TraderManager) StartAll()
func (m *TraderManager) StopAll()
func (m *TraderManager) GetTrader(id string) *trader.AutoTrader
func (m *TraderManager) GetAllTraders() []*trader.AutoTrader
```

**设计亮点**：
- ✅ 统一管理多个实例
- ✅ 优雅启动/停止
- ✅ 提供查询 API

---

#### 模块 7：api（API 服务模块）

**职责**：
- 提供 HTTP API
- 处理前端请求
- 返回 JSON 数据

**对外接口**：
```go
// api/server.go
type Server struct {
    traderManager *manager.TraderManager
    router        *gin.Engine
    port          int
}

func NewServer(tm *manager.TraderManager, port int) *Server
func (s *Server) Start() error

// API 端点
GET /api/traders                   # 所有 Trader 列表
GET /api/account?trader_id=xxx     # 账户信息
GET /api/positions?trader_id=xxx   # 持仓列表
GET /api/decisions/latest?trader_id=xxx  # 最新决策
GET /api/competition               # 竞赛排行榜
```

**设计亮点**：
- ✅ RESTful 风格
- ✅ 统一错误处理
- ✅ CORS 支持

---

### 模块间依赖关系图

```
main.go (程序入口)
   │
   ├─→ config (加载配置)
   │
   ├─→ pool (设置币种池)
   │
   ├─→ manager (创建 TraderManager)
   │      │
   │      └─→ AutoTrader (每个 Trader 实例)
   │             │
   │             ├─→ trader (交易执行)
   │             │      └─→ Binance/Hyperliquid/Aster
   │             │
   │             ├─→ market (市场数据)
   │             │
   │             ├─→ pool (币种池)
   │             │
   │             ├─→ decision (决策引擎)
   │             │      ├─→ mcp (AI 通信)
   │             │      └─→ logger (历史表现)
   │             │
   │             └─→ logger (日志记录)
   │
   └─→ api (启动 API 服务)
          └─→ manager (查询 Trader 数据)
```

**依赖层级**：
```
Level 1（最底层）：config, pool
Level 2：trader, market, mcp
Level 3：decision, logger
Level 4：AutoTrader
Level 5：manager
Level 6：api, main
```

**原则**：
- ✅ 高层依赖低层
- ✅ 低层不依赖高层
- ✅ 同层之间尽量不依赖

---

## 💪 实战练习

### 练习 1：拆分你的项目模块（必做）

**步骤**：

1. **列出所有功能**（从第1章的需求清单）

2. **归类相似功能**（哪些功能相关？）

3. **定义模块**（每个模块一句话职责）

4. **画出目录结构**

**模板**：

```
my-project/
├── module_1/
│   职责：_____________________
│   包含功能：
│   - 功能A
│   - 功能B
│
├── module_2/
│   职责：_____________________
│   包含功能：
│   - 功能C
│   - 功能D
│
├── module_3/
│   职责：_____________________
│   包含功能：
│   - 功能E
```

---

### 练习 2：设计模块接口（必做）

选择你项目的 3 个核心模块，定义它们的接口：

**模板**：

```python
# module_1 接口
class Module1:
    """
    职责：__________
    """

    def function_1(self, param1, param2):
        """
        功能：__________
        输入：__________
        输出：__________
        """
        pass

    def function_2(self, param1):
        """
        功能：__________
        """
        pass
```

**要求**：
- 函数名清晰表达功能
- 写清楚输入输出
- 只暴露必要的接口

---

### 练习 3：分析模块依赖（必做）

画出你的模块依赖关系图：

```
模块A
  ↓ 依赖
模块B
  ↓ 依赖
模块C
```

**检查**：
- [ ] 有没有循环依赖？（A依赖B，B又依赖A）
- [ ] 依赖层级清晰吗？
- [ ] 底层模块不依赖高层吗？

**如果发现问题**：重新调整模块划分

---

### 练习 4：单一职责检查（选做）

对每个模块问自己：

| 模块名 | 职责 | 改变的理由（列出所有） | 是否单一？ |
|--------|------|------------------------|-----------|
| 示例：用户模块 | 管理用户信息 | 1. 用户字段变化<br>2. 认证方式变化 | ❌ 太多 |
| 改进：拆分成 | 用户数据 | 1. 用户字段变化 | ✅ 单一 |
|  | 用户认证 | 1. 认证方式变化 | ✅ 单一 |

---

### 练习 5：接口抽象实践（选做）

假设你的项目需要支持多个数据源（数据库、API、文件），设计接口：

```python
# 接口定义
class DataSource:
    def fetch_data(self, query):
        """获取数据"""
        pass

    def save_data(self, data):
        """保存数据"""
        pass

# 实现1：数据库
class DatabaseSource(DataSource):
    def fetch_data(self, query):
        return db.execute(query)

    def save_data(self, data):
        db.insert(data)

# 实现2：API
class APISource(DataSource):
    def fetch_data(self, query):
        return requests.get(f"{api_url}/{query}")

    def save_data(self, data):
        requests.post(api_url, json=data)

# 使用（不关心具体实现）
def process(source: DataSource):
    data = source.fetch_data("something")
    # 处理数据...
    source.save_data(result)
```

**你的设计**：
- 接口名称：__________
- 需要哪些方法：__________
- 有哪些实现：__________

---

## 🤔 思考题

### 1. 为什么 NOFX 要单独拆一个 `pool` 模块？直接在 `decision` 里获取币种不行吗?

<details>
<summary>点击查看答案</summary>

**如果不拆分**：
```go
// decision/engine.go
func GetCandidateCoins() []string {
    if useDefaultCoins {
        return []string{"BTCUSDT", "ETHUSDT", ...}
    } else {
        // 调用 API
        resp := http.Get(coinPoolAPI)
        // 解析...
    }
}
```

**问题**：
- `decision` 模块职责不单一（既管决策，又管币种筛选）
- 其他模块想用币种池怎么办？复制代码？
- 测试困难（必须 mock API）

**拆分后**：
```go
// pool/coin_pool.go - 专门负责币种筛选
func GetCoinPool() []string {
    // 币种筛选逻辑
}

// decision/engine.go - 只负责决策
func GetFullDecision(ctx *Context) {
    coins := pool.GetCoinPool()  // 使用池模块
    // 构建决策...
}
```

**优势**：
- ✅ 职责清晰
- ✅ 可复用（其他模块也能用）
- ✅ 易测试（独立测试币种筛选）
- ✅ 易替换（换成其他筛选策略）

</details>

---

### 2. `AutoTrader` 为什么不是一个模块，而是放在 `trader` 模块里？

<details>
<summary>点击查看答案</summary>

**模块 vs 类**：
- 模块：一组相关功能的集合
- 类：实现特定功能的代码

**`trader` 模块包含**：
- `interface.go` - Trader 接口
- `binance_futures.go` - 交易所实现
- `hyperliquid_trader.go` - 交易所实现
- `auto_trader.go` - **自动交易控制器**

**为什么放在一起**：
- 都是关于"交易"的
- `AutoTrader` 使用 `Trader` 接口
- 高内聚（交易相关代码在一起）

**如果单独拆模块**：
```
autotrader/  # 新模块
└── auto_trader.go

trader/      # 只剩下交易所实现
└── binance_futures.go
```

**反而不好**：
- `autotrader` 模块太小（只有一个文件）
- `autotrader` 和 `trader` 耦合太强（必须一起用）
- 不符合"高内聚"

**结论**：不是所有类都要独立成模块，相关的放在一起。

</details>

---

### 3. 如果要添加"策略系统"（用户可以编写自己的策略），应该怎么设计模块？

<details>
<summary>点击查看答案</summary>

**新增模块**：`strategy/`

```
nofx/
├── strategy/                    # 新增策略模块
│   ├── interface.go             # 策略接口
│   ├── ai_strategy.go           # AI 策略（现有）
│   ├── user_strategy_loader.go  # 用户策略加载器
│   └── examples/
│       ├── rsi_strategy.py      # 示例策略1
│       └── macd_strategy.py     # 示例策略2
│
├── decision/                    # 决策引擎改为使用策略
│   └── engine.go                # 使用 Strategy 接口

```

**接口设计**：

```go
// strategy/interface.go
type Strategy interface {
    // 分析市场，返回决策
    Analyze(ctx *decision.Context) ([]decision.Decision, error)

    // 策略名称
    Name() string
}

// strategy/ai_strategy.go（现有逻辑封装）
type AIStrategy struct {
    mcpClient *mcp.Client
}

func (s *AIStrategy) Analyze(ctx *decision.Context) ([]decision.Decision, error) {
    // 原来 decision/engine.go 的逻辑
}

// strategy/user_strategy_loader.go（新增）
type UserStrategy struct {
    pythonScript string
}

func (s *UserStrategy) Analyze(ctx *decision.Context) ([]decision.Decision, error) {
    // 调用用户的 Python 脚本
    result := executePython(s.pythonScript, ctx)
    return parseDecisions(result), nil
}
```

**使用**：

```go
// AutoTrader 配置使用哪个策略
trader := &AutoTrader{
    strategy: strategy.NewAIStrategy(mcpClient),  // 或 UserStrategy
}

// 决策时
decisions := trader.strategy.Analyze(ctx)
```

**优势**：
- ✅ 符合开闭原则（对扩展开放）
- ✅ AI 策略和用户策略平等对待
- ✅ 易于添加新策略类型

</details>

---

## 📖 本章总结

### 你学到了什么

✅ **核心概念**：
- 模块化是将系统拆分成独立功能单元
- 每个模块有清晰职责和接口
- 好的模块化让系统易维护、易扩展

✅ **思维方法**：
- 按业务功能拆分（推荐）
- 单一职责原则（一个改变理由）
- 高内聚、低耦合
- 依赖倒置（依赖接口）

✅ **实践技能**：
- 能将项目拆分成合理的模块
- 能设计模块接口
- 能分析模块依赖关系
- 能识别和解决设计问题

✅ **案例收获**：
- NOFX 的 8 大模块设计
- Trader 接口的抽象技巧
- 模块间依赖关系管理
- 如何平衡模块粒度

---

### 模块化设计检查清单

- [ ] **单一职责**：每个模块只做一件事
- [ ] **高内聚**：相关功能放在一起
- [ ] **低耦合**：模块间依赖少
- [ ] **接口清晰**：对外接口简单明确
- [ ] **可测试**：每个模块能独立测试
- [ ] **可替换**：实现可以轻松替换
- [ ] **无循环依赖**：A依赖B，B不能依赖A
- [ ] **依赖方向正确**：高层依赖低层

---

### 下一步

完成练习后，你应该有：
- ✅ 你的项目模块划分方案（5-10个模块）
- ✅ 每个模块的职责说明
- ✅ 核心模块的接口定义
- ✅ 模块依赖关系图

准备好后，进入 **第 4 章：配置系统**，学习如何让系统更灵活！

---

## 📚 延伸阅读

- 《代码整洁之道》- Robert C. Martin（单一职责原则）
- 《设计模式》- GoF（接口、抽象）
- 《领域驱动设计》- Eric Evans（业务建模）
- 《SOLID 原则》- 面向对象设计五大原则

---

## ❓ FAQ

**Q1：模块拆得越细越好吗？**
A：不是。太细导致文件太多，反而难管理。**原则**：一个模块 100-500 行代码合适。

**Q2：如何判断是拆成模块还是只是一个类？**
A：
- 模块：多个类，对外提供一组相关功能
- 类：单个功能实现

**Q3：模块间能相互调用吗？**
A：可以，但要注意依赖方向（高层→低层），避免循环依赖。

**Q4：接口一定要用 interface 语法吗？**
A：不是。接口是思想，不是语法。Python 用 ABC，Go 用 interface，思想一样。

**Q5：NOFX 的模块划分是最优的吗？**
A：没有最优，只有合适。NOFX 的划分适合它的场景，你的项目可能需要不同划分。

---

**🎉 恭喜完成第 3 章！**

你已经掌握了模块化设计的核心技能！

记住：**好的模块化，改代码像换零件，加功能像装插件。**

准备好了吗？进入第 4 章！💪
