# 第6章：业务逻辑层 - 核心算法设计

> **本章目标**：学会设计业务逻辑层，这是系统的"大脑"，包含核心算法和决策逻辑

---

## 🎯 核心概念

### 什么是业务逻辑层？

**类比：餐厅运作**

```
表现层（前厅）：接待客人、展示菜单、端菜上桌
                ↓
业务逻辑层（厨师）：决定怎么做菜、调味、摆盘 ← 核心技术
                ↓
数据层（仓库）：存储食材、调料
```

**业务逻辑层就是**：
- 系统的**核心智能**
- 包含所有**业务规则**和**算法**
- 决定**"怎么做"**，而不是"存什么"或"显示什么"

### 业务逻辑 vs 技术实现

很多初学者容易混淆！

#### 业务逻辑（What to do）

```python
# ✅ 业务逻辑：交易规则
def should_open_position(account, signal):
    """决定是否开仓"""

    # 规则1：资金不足不开仓
    if account.available_balance < 100:
        return False

    # 规则2：信号强度不够不开仓
    if signal.confidence < 70:
        return False

    # 规则3：已有持仓不重复开
    if account.has_position(signal.symbol):
        return False

    return True
```

#### 技术实现（How to do）

```python
# ❌ 这不是业务逻辑，这是技术实现
def save_to_database(data):
    """保存数据"""
    conn = sqlite3.connect("data.db")
    cursor = conn.cursor()
    cursor.execute("INSERT INTO ... VALUES ...")
    conn.commit()
```

**区别**：
- **业务逻辑**：反映业务规则，换技术栈也不变
  - "资金不足不开仓"在任何系统都成立
- **技术实现**：完成技术任务，可能随技术栈变化
  - 用 MySQL 还是 MongoDB？这不是业务问题

---

## 🧠 思维方法

### 方法一：业务流程建模

**步骤**：

1. **列出所有业务步骤**
2. **识别决策点**（需要判断的地方）
3. **明确输入输出**
4. **定义业务规则**

#### 示例：NOFX 的决策流程

```
[开始决策循环]
        ↓
┌───────────────────┐
│ 1. 获取账户状态    │ → 输入：无  输出：Account
│    - 余额          │
│    - 持仓          │
│    - 保证金使用率   │
└─────────┬─────────┘
          ↓
┌─────────────────────┐
│ 2. 获取候选币种      │ → 输入：无  输出：[]Symbol
│    - AI500 池       │
│    - OI Top 池      │
└─────────┬───────────┘
          ↓
┌──────────────────────┐
│ 3. 获取市场数据       │ → 输入：[]Symbol  输出：MarketData
│    - K线数据         │
│    - 技术指标        │
└─────────┬────────────┘
          ↓
┌──────────────────────┐  ❓决策点1
│ 4. AI 分析决策        │ → 是否有交易机会？
│    - 分析所有数据     │
│    - 输出决策列表     │
└─────────┬────────────┘
          ↓
   有机会？ ┌───┐
          │ 否 │ → [等待下次循环]
          └───┘
          │ 是
          ↓
┌─────────────────────┐  ❓决策点2
│ 5. 风控检查          │ → 是否符合风控规则？
│    - 仓位限制        │
│    - 保证金检查      │
│    - 止损止盈比例    │
└─────────┬───────────┘
          ↓
   通过？ ┌───┐
          │ 否 │ → [拒绝交易]
          └───┘
          │ 是
          ↓
┌─────────────────────┐
│ 6. 执行交易          │ → 输入：Decision  输出：Result
│    - 下单            │
│    - 设置止损止盈     │
└─────────┬───────────┘
          ↓
┌─────────────────────┐
│ 7. 记录日志          │ → 输入：Result  输出：Log
│    - 决策过程        │
│    - 执行结果        │
└─────────────────────┘
          ↓
      [等待3分钟]
          ↓
      [下一次循环]
```

### 方法二：状态机思维

**很多业务逻辑可以用状态机建模**

#### 示例：订单状态

```
         [创建订单]
              ↓
        ┌──────────┐
        │  待支付   │ ──超时──→ [已取消]
        └─────┬────┘
              │ 支付
              ↓
        ┌──────────┐
        │  待发货   │ ──取消──→ [已取消]
        └─────┬────┘
              │ 发货
              ↓
        ┌──────────┐
        │  待收货   │ ──退货──→ [退货中]
        └─────┬────┘
              │ 确认收货
              ↓
        ┌──────────┐
        │  已完成   │
        └──────────┘
```

#### NOFX 的持仓状态

```
        [无持仓]
            ↓ 开仓
      ┌──────────┐
      │ 持仓中    │
      └─────┬────┘
            │
     ┌──────┼──────┐
     │      │      │
  触发止损 触发止盈 手动平仓
     │      │      │
     └──────┴──────┘
            ↓
       [已平仓]
```

### 方法三：规则引擎模式

**将业务规则抽象成可配置的规则**

#### 传统硬编码

```python
# ❌ 规则都写死在代码里
def check_risk(account, order):
    if account.available_balance < order.value:
        return False

    if order.leverage > 20:
        return False

    if account.margin_used_pct > 90:
        return False

    return True
```

**问题**：
- 修改规则要改代码
- 添加新规则要改代码
- 不同用户不能用不同规则

#### 规则引擎

```python
# ✅ 规则可配置
class Rule:
    def check(self, context):
        raise NotImplementedError

class BalanceRule(Rule):
    def __init__(self, min_balance):
        self.min_balance = min_balance

    def check(self, context):
        if context.account.available_balance < self.min_balance:
            return False, "余额不足"
        return True, "OK"

class LeverageRule(Rule):
    def __init__(self, max_leverage):
        self.max_leverage = max_leverage

    def check(self, context):
        if context.order.leverage > self.max_leverage:
            return False, f"杠杆超过{self.max_leverage}倍"
        return True, "OK"

# 规则引擎
class RiskEngine:
    def __init__(self):
        self.rules = []

    def add_rule(self, rule):
        self.rules.append(rule)

    def check_all(self, context):
        for rule in self.rules:
            passed, message = rule.check(context)
            if not passed:
                return False, message
        return True, "所有规则通过"

# 使用（可配置）
engine = RiskEngine()
engine.add_rule(BalanceRule(min_balance=100))
engine.add_rule(LeverageRule(max_leverage=20))
engine.add_rule(MarginRule(max_margin_pct=90))

# 检查
passed, message = engine.check_all(context)
```

**优势**：
- ✅ 规则可以动态添加
- ✅ 规则可以单独测试
- ✅ 不同用户可以有不同规则集

### 方法四：策略模式

**将算法封装成可替换的策略**

```python
# 策略接口
class TradingStrategy:
    def analyze(self, market_data):
        raise NotImplementedError

# 策略1：RSI 策略
class RSIStrategy(TradingStrategy):
    def analyze(self, market_data):
        if market_data.rsi < 30:
            return "买入"  # 超卖
        elif market_data.rsi > 70:
            return "卖出"  # 超买
        return "持有"

# 策略2：MACD 策略
class MACDStrategy(TradingStrategy):
    def analyze(self, market_data):
        if market_data.macd > market_data.signal:
            return "买入"  # 金叉
        elif market_data.macd < market_data.signal:
            return "卖出"  # 死叉
        return "持有"

# 策略3：AI 策略
class AIStrategy(TradingStrategy):
    def __init__(self, ai_client):
        self.ai_client = ai_client

    def analyze(self, market_data):
        prompt = f"分析这个市场数据：{market_data}"
        decision = self.ai_client.ask(prompt)
        return decision

# 交易员可以切换策略
class Trader:
    def __init__(self, strategy: TradingStrategy):
        self.strategy = strategy

    def set_strategy(self, strategy):
        """随时切换策略"""
        self.strategy = strategy

    def make_decision(self, market_data):
        return self.strategy.analyze(market_data)

# 使用
trader = Trader(RSIStrategy())
decision = trader.make_decision(market_data)

# 切换策略
trader.set_strategy(AIStrategy(ai_client))
decision = trader.make_decision(market_data)
```

---

## 📚 NOFX 案例分析

### NOFX 的核心业务逻辑

#### 1. 决策引擎（Decision Engine）

**位置**：`decision/engine.go`

**职责**：
- 收集所有输入数据
- 构建 AI 提示词
- 调用 AI 获取决策
- 解析 JSON 响应

**核心代码**：

```go
// decision/engine.go
package decision

// Context 决策上下文（所有输入数据）
type Context struct {
    CurrentTime     string
    RuntimeMinutes  int
    CallCount       int
    Account         AccountInfo
    Positions       []PositionInfo
    CandidateCoins  []CandidateCoin
    MarketDataMap   map[string]*market.Data
    Performance     *logger.PerformanceAnalysis
    BTCETHLeverage  int
    AltcoinLeverage int
}

// Decision AI 的决策
type Decision struct {
    Symbol          string  `json:"symbol"`
    Action          string  `json:"action"`  // open_long, close_long, etc.
    Leverage        int     `json:"leverage"`
    PositionSizeUSD float64 `json:"position_size_usd"`
    StopLoss        float64 `json:"stop_loss"`
    TakeProfit      float64 `json:"take_profit"`
    Confidence      int     `json:"confidence"`
    Reasoning       string  `json:"reasoning"`
}

// GetFullDecision 核心决策流程
func GetFullDecision(ctx *Context, mcpClient *mcp.Client) (*FullDecision, error) {
    // 1. 获取所有币种的市场数据
    if err := fetchMarketDataForContext(ctx); err != nil {
        return nil, err
    }

    // 2. 构建 System Prompt（规则说明）
    systemPrompt := buildSystemPrompt(
        ctx.Account.TotalEquity,
        ctx.BTCETHLeverage,
        ctx.AltcoinLeverage,
    )

    // 3. 构建 User Prompt（具体数据）
    userPrompt := buildUserPrompt(ctx)

    // 4. 调用 AI
    response, err := mcpClient.Chat(systemPrompt, userPrompt)
    if err != nil {
        return nil, fmt.Errorf("AI调用失败: %w", err)
    }

    // 5. 解析 JSON 响应
    var decisions []Decision
    if err := parseDecisions(response, &decisions); err != nil {
        return nil, fmt.Errorf("解析AI响应失败: %w", err)
    }

    return &FullDecision{
        UserPrompt: userPrompt,
        CoTTrace:   response,
        Decisions:  decisions,
        Timestamp:  time.Now(),
    }, nil
}
```

#### 2. System Prompt 构建

**System Prompt = 告诉 AI "你是谁，要做什么，规则是什么"**

```go
func buildSystemPrompt(totalEquity float64, btcEthLev, altcoinLev int) string {
    return fmt.Sprintf(`你是一个专业的加密货币期货交易员。

**你的任务**：
分析市场数据和账户状态，做出交易决策（开仓/平仓/观望）。

**交易规则**：
1. 杠杆限制：
   - BTC/ETH 最高 %d 倍杠杆
   - 其他币种最高 %d 倍杠杆

2. 仓位限制：
   - 单个山寨币仓位不超过账户权益的 1.5 倍
   - BTC/ETH 单个仓位不超过账户权益的 10 倍

3. 保证金管理：
   - 总保证金使用率不超过 90%%

4. 风险控制：
   - 必须设置止损和止盈
   - 止损止盈比例至少 1:2（风险1，收益2）

5. 防止重复开仓：
   - 同一币种同一方向已有持仓，不能重复开

**输出格式**：
返回 JSON 数组，每个决策包含：
{
    "symbol": "BTCUSDT",
    "action": "open_long|open_short|close_long|close_short|hold|wait",
    "leverage": 10,
    "position_size_usd": 500.0,
    "stop_loss": 95000.0,
    "take_profit": 100000.0,
    "confidence": 75,
    "reasoning": "简短解释原因"
}

**思考过程**：
1. 分析市场：趋势、指标、持仓量
2. 分析账户：余额、持仓、保证金
3. 参考历史：哪些币种表现好/坏
4. 做出决策：是否有高胜率机会
5. 风控检查：是否符合规则
`,btcEthLev, altcoinLev)
}
```

#### 3. User Prompt 构建

**User Prompt = 告诉 AI "当前的具体情况"**

```go
func buildUserPrompt(ctx *Context) string {
    var prompt strings.Builder

    // 1. 时间和系统状态
    prompt.WriteString(fmt.Sprintf(`## 当前时间
%s

## 系统运行状态
- 运行时长：%d 分钟
- 决策次数：%d

`, ctx.CurrentTime, ctx.RuntimeMinutes, ctx.CallCount))

    // 2. 历史表现（AI 自学习）
    if ctx.Performance != nil {
        prompt.WriteString(fmt.Sprintf(`## 📊 历史表现反馈

### 总体统计
- 总交易：%d 笔（盈利 %d | 亏损 %d）
- 胜率：%.1f%%
- 平均盈利：%.2f USDT | 平均亏损：%.2f USDT
- 盈亏比：%.2f:1

### 最近5笔交易
%s

### 币种表现统计
%s

**学习建议**：
- 继续增加胜率高的币种权重
- 避免连续亏损的币种
- 参考历史最佳交易时机

`,
            ctx.Performance.TotalTrades,
            ctx.Performance.WinCount,
            ctx.Performance.LossCount,
            ctx.Performance.WinRate,
            ctx.Performance.AvgProfit,
            ctx.Performance.AvgLoss,
            ctx.Performance.ProfitFactor,
            formatRecentTrades(ctx.Performance.RecentTrades),
            formatCoinStats(ctx.Performance.CoinStats),
        ))
    }

    // 3. 账户状态
    prompt.WriteString(fmt.Sprintf(`## 💰 账户状态
- 总权益：%.2f USDT
- 可用余额：%.2f USDT
- 已用保证金：%.2f USDT（%.1f%%）
- 当前持仓数：%d

`,
        ctx.Account.TotalEquity,
        ctx.Account.AvailableBalance,
        ctx.Account.MarginUsed,
        ctx.Account.MarginUsedPct,
        len(ctx.Positions),
    ))

    // 4. 现有持仓（如果有）
    if len(ctx.Positions) > 0 {
        prompt.WriteString("## 📍 现有持仓\n\n")
        for _, pos := range ctx.Positions {
            duration := calculateDuration(pos.UpdateTime)
            prompt.WriteString(fmt.Sprintf(`### %s %s
- 开仓价：%.4f → 当前价：%.4f
- 盈亏：%.2f USDT（%.2f%%）
- 数量：%.8f | 杠杆：%dx
- 持仓时长：%s
- 清算价：%.4f

**市场数据**：
%s

`,
                pos.Symbol,
                strings.ToUpper(pos.Side),
                pos.EntryPrice,
                pos.MarkPrice,
                pos.UnrealizedPnL,
                pos.UnrealizedPnLPct,
                pos.Quantity,
                pos.Leverage,
                duration,
                pos.LiquidationPrice,
                formatMarketData(ctx.MarketDataMap[pos.Symbol]),
            ))
        }
    }

    // 5. 候选币种市场数据
    prompt.WriteString(fmt.Sprintf("## 🎯 候选币种市场数据（共 %d 个）\n\n", len(ctx.CandidateCoins)))

    for _, coin := range ctx.CandidateCoins {
        data := ctx.MarketDataMap[coin.Symbol]
        if data == nil {
            continue
        }

        prompt.WriteString(fmt.Sprintf(`### %s
**3分钟级别**：
- 当前价：%.4f
- RSI(7)：%.2f %s
- MACD：%.4f（信号：%.4f，柱状图：%.4f）
- EMA20：%.4f %s

**4小时级别**：
- RSI(14)：%.2f %s
- MACD：%.4f
- EMA20：%.4f | EMA50：%.4f %s
- ATR：%.4f（波动率）

**持仓量**：%.0f USD（变化：%s）

`,
            coin.Symbol,
            data.CurrentPrice,
            data.RSI3m, interpretRSI(data.RSI3m),
            data.MACD3m.MACD, data.MACD3m.Signal, data.MACD3m.Hist,
            data.EMA20_3m, comparePriceEMA(data.CurrentPrice, data.EMA20_3m),
            data.RSI4h, interpretRSI(data.RSI4h),
            data.MACD4h.MACD,
            data.EMA20_4h, data.EMA50_4h, compareEMA(data.EMA20_4h, data.EMA50_4h),
            data.ATR4h,
            data.OIValue,
            formatOIChange(data.OIDeltaPct),
        ))
    }

    // 6. 请求 AI 决策
    prompt.WriteString(`
## 🎯 请给出你的决策

基于以上所有信息，请：
1. 分析每个持仓是否应该平仓
2. 分析候选币种是否有开仓机会
3. 严格遵守风控规则
4. 返回 JSON 格式决策列表

如果没有合适机会，返回 []
`)

    return prompt.String()
}
```

**设计亮点**：
- ✅ 完整的上下文：账户、持仓、市场、历史
- ✅ 结构化清晰：用 Markdown 格式组织
- ✅ AI 自学习：包含历史表现反馈
- ✅ 持仓时长：帮助 AI 判断何时止盈

#### 4. 风控规则检查

```go
// trader/auto_trader.go
func (at *AutoTrader) executeDecisions(decisions []decision.Decision) {
    for _, d := range decisions {
        // 风控检查
        if err := at.checkRiskRules(d); err != nil {
            log.Printf("风控拒绝：%s - %v", d.Symbol, err)
            continue
        }

        // 执行交易
        at.executeTrade(d)
    }
}

func (at *AutoTrader) checkRiskRules(d decision.Decision) error {
    account, _ := at.trader.GetAccount()

    // 规则1：余额检查
    requiredMargin := d.PositionSizeUSD / float64(d.Leverage)
    if account.AvailableBalance < requiredMargin {
        return fmt.Errorf("可用余额不足")
    }

    // 规则2：仓位限制
    maxPositionSize := at.getMaxPositionSize(d.Symbol, account.TotalEquity)
    if d.PositionSizeUSD > maxPositionSize {
        return fmt.Errorf("仓位超限：%.2f > %.2f", d.PositionSizeUSD, maxPositionSize)
    }

    // 规则3：保证金使用率
    newMarginUsed := account.MarginUsed + requiredMargin
    if newMarginUsed / account.TotalEquity > 0.9 {
        return fmt.Errorf("保证金使用率将超过90%%")
    }

    // 规则4：止损止盈比例
    riskRewardRatio := at.calculateRiskRewardRatio(d)
    if riskRewardRatio < 2.0 {
        return fmt.Errorf("风险回报比不足1:2（当前%.2f）", riskRewardRatio)
    }

    // 规则5：防止重复开仓
    if at.hasPosition(d.Symbol, d.Action) {
        return fmt.Errorf("已有持仓，不能重复开")
    }

    return nil
}

func (at *AutoTrader) getMaxPositionSize(symbol string, equity float64) float64 {
    // BTC/ETH: 最大10倍权益
    if symbol == "BTCUSDT" || symbol == "ETHUSDT" {
        return equity * 10.0
    }
    // 其他：最大1.5倍权益
    return equity * 1.5
}
```

---

## 💪 实战练习

### 练习 1：画出业务流程图（必做）

为你的项目画出核心业务流程：

```
[开始]
   ↓
[步骤1：xxx]
   ↓
❓决策点：xxx？
   ├─是→ [xxx]
   └─否→ [xxx]
   ↓
[结束]
```

**要求**：
- 标出所有决策点
- 写清楚每步的输入输出
- 识别关键业务规则

---

### 练习 2：定义业务规则（必做）

列出你项目的核心业务规则：

```markdown
## 业务规则清单

### 规则1：xxx
- 描述：
- 条件：
- 结果：

### 规则2：xxx
- 描述：
- 条件：
- 结果：
```

---

### 练习 3：实现规则引擎（选做）

用代码实现简单的规则引擎：

```python
class Rule:
    def check(self, data):
        pass

class AgeRule(Rule):
    def __init__(self, min_age):
        self.min_age = min_age

    def check(self, data):
        if data.age < self.min_age:
            return False, f"年龄必须≥{self.min_age}"
        return True, "OK"

# 添加更多规则...
```

---

### 练习 4：状态机建模（选做）

为你项目的某个对象设计状态机：

```
[状态A]
   ├─事件1→ [状态B]
   └─事件2→ [状态C]
```

---

## 🤔 思考题

### 1. 为什么 NOFX 要把历史表现反馈给 AI？

<details>
<summary>点击查看答案</summary>

**AI 自学习机制**：

```
传统策略：
- 固定规则
- 不会从错误中学习
- 重复同样的错误

AI 自学习：
- 看到自己的历史表现
- 知道哪些币种胜率高
- 避免重复亏损的币种
- 强化成功的模式
```

**具体例子**：

```
AI 看到历史：
- SOLUSDT：连续3次亏损，胜率25%
- BTCUSDT：5次交易4次盈利，胜率80%

AI 学到：
- "SOLUSDT 最近状态不好，暂时避开"
- "BTCUSDT 策略有效，继续使用"
```

**效果**：
- 胜率逐渐提高
- 减少重复错误
- 动态适应市场

</details>

---

### 2. 风控规则应该在哪里执行？AI决策前还是决策后？

<details>
<summary>点击查看答案</summary>

**两个地方都需要**！

**1. AI 决策前（System Prompt）**

告诉 AI 规则，让 AI 自己遵守：

```
System Prompt:
- 杠杆不超过20倍
- 仓位不超过权益1.5倍
- 必须设置止损止盈
```

**好处**：
- AI 输出的决策已经符合规则
- 减少被拒绝的决策

**2. AI 决策后（代码检查）**

代码再次验证，确保安全：

```python
if decision.leverage > max_leverage:
    reject("杠杆超限")
```

**好处**：
- 双重保险
- 防止 AI 出错
- 防止 AI 理解偏差

**为什么双重检查？**
- AI 可能误解规则
- AI 可能计算错误
- 涉及资金安全，不能冒险

</details>

---

### 3. 如何测试业务逻辑？

<details>
<summary>点击查看答案</summary>

**方法1：单元测试（测试单个规则）**

```python
def test_balance_rule():
    account = Account(available_balance=50)
    order = Order(required_margin=100)

    result = check_balance_rule(account, order)

    assert result == False  # 余额不足应该拒绝
```

**方法2：集成测试（测试完整流程）**

```python
def test_decision_flow():
    # 准备测试数据
    ctx = create_test_context()

    # 执行决策
    decisions = get_full_decision(ctx)

    # 验证结果
    assert len(decisions) > 0
    assert all(d.leverage <= 20 for d in decisions)
```

**方法3：场景测试（测试业务场景）**

```python
def test_scenario_low_balance():
    """场景：余额不足时不应该开仓"""
    ctx = Context(
        account=Account(available_balance=10),
        market_data=strong_buy_signal()  # 即使信号很强
    )

    decisions = get_full_decision(ctx)

    # 应该返回空决策或观望
    assert len(decisions) == 0 or decisions[0].action == "wait"
```

**方法4：Mock AI（不依赖真实 AI）**

```python
def test_with_mock_ai():
    # Mock AI 返回固定决策
    mock_ai = Mock()
    mock_ai.ask.return_value = '{"action": "open_long", ...}'

    # 测试后续流程
    result = execute_decision(mock_ai)
```

</details>

---

## 📖 本章总结

### 你学到了什么

✅ **核心概念**：
- 业务逻辑层是系统的"大脑"
- 业务逻辑 vs 技术实现的区别
- 核心算法和决策流程

✅ **思维方法**：
- 业务流程建模（流程图）
- 状态机思维（状态转换）
- 规则引擎模式（可配置规则）
- 策略模式（可替换算法）

✅ **实践技能**：
- 能画出业务流程图
- 能定义业务规则
- 能实现规则引擎
- 能设计决策系统

✅ **案例收获**：
- NOFX 的决策引擎设计
- System/User Prompt 构建技巧
- 风控规则实现
- AI 自学习机制

---

### 业务逻辑层检查清单

- [ ] **流程清晰**：能用流程图表达
- [ ] **规则明确**：业务规则文档化
- [ ] **可测试**：每个规则能独立测试
- [ ] **可扩展**：易于添加新规则
- [ ] **错误处理**：异常情况有处理
- [ ] **日志完整**：决策过程可追溯
- [ ] **性能合理**：核心流程不能太慢

---

### 下一步

完成练习后，你应该有：
- ✅ 项目的业务流程图
- ✅ 核心业务规则清单
- ✅ 简单的规则引擎实现

到此为止，你已经完成了前 6 章的学习！

**第一阶段**（1-3章）：项目思维基础 ✅
**第二阶段前半部分**（4-6章）：核心模块设计 ✅

接下来准备学习：
- 第7章：数据持久化
- 第8章：API 服务层
- 第9章：前端展示层

---

## 📚 延伸阅读

- 《领域驱动设计》- Eric Evans
- 《企业应用架构模式》- Martin Fowler
- 《设计模式》- GoF（策略模式、状态模式）
- 《规则引擎实战》

---

## ❓ FAQ

**Q1：业务逻辑一定要单独一层吗？**
A：
- 小项目：可以和其他层混在一起
- 中大型项目：强烈建议分离
- **标准**：业务规则超过5个 → 单独成层

**Q2：业务规则写在配置还是代码？**
A：
- 简单规则：代码
- 经常变化的规则：配置
- 复杂规则：代码+配置结合

**Q3：如何让业务逻辑易于修改？**
A：
- 使用规则引擎（规则可配置）
- 使用策略模式（算法可替换）
- 完善测试（修改时能快速验证）

**Q4：AI 决策算业务逻辑吗？**
A：
- AI 是决策工具
- 调用 AI、解析结果、应用规则 → 这是业务逻辑
- AI 模型本身 → 外部依赖

---

**🎉 恭喜完成第 6 章！**

你已经掌握了业务逻辑层设计！

记住：**好的业务逻辑让系统智能，坏的业务逻辑让系统愚蠢。**

准备好了吗？告诉我继续！💪
