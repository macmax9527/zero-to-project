# 第4章：配置系统 - 灵活性设计

> **本章目标**：学会设计灵活的配置系统，让系统可以适应不同环境和需求

---

## 🎯 核心概念

### 什么是配置？

**类比：手机设置**

```
手机出厂时：
- 默认铃声
- 默认壁纸
- 默认语言

你买到手机后：
- 改成自己喜欢的铃声
- 换成自己的照片
- 选择自己的语言

→ 这就是"配置"：不改代码，只改设置
```

**软件配置就是**：
- 不修改代码就能改变行为的参数
- 区分不同环境（开发/测试/生产）
- 保护敏感信息（API密钥）
- 让用户自定义系统

### 配置 vs 硬编码

#### ❌ 硬编码（Hard-coded）

```python
# main.py
def main():
    api_key = "sk-abc123xyz"  # 直接写死在代码里
    api_url = "https://api.binance.com"
    scan_interval = 180  # 3分钟

    trader = Trader(api_key, api_url)
    while True:
        trader.run()
        time.sleep(scan_interval)
```

**问题**：
- 🔴 换个API密钥 → 改代码重新部署
- 🔴 测试环境用不同URL → 改代码
- 🔴 想改扫描间隔 → 改代码
- 🔴 API密钥暴露在代码里 → 安全风险
- 🔴 多人用不同配置 → 每个人一份代码？

#### ✅ 配置化（Configurable）

```python
# config.json（配置文件）
{
    "api_key": "sk-abc123xyz",
    "api_url": "https://api.binance.com",
    "scan_interval_minutes": 3
}

# main.py（代码）
def main():
    config = load_config("config.json")
    trader = Trader(config["api_key"], config["api_url"])

    while True:
        trader.run()
        time.sleep(config["scan_interval_minutes"] * 60)
```

**优势**：
- ✅ 换API密钥 → 只改config.json
- ✅ 测试环境 → 用config.test.json
- ✅ 调整间隔 → 改数字，不改代码
- ✅ API密钥 → config.json不提交到Git
- ✅ 多人使用 → 各自的config.json

### 配置的三种形式

| 形式 | 示例 | 优点 | 缺点 | 适用场景 |
|------|------|------|------|----------|
| **配置文件** | config.json<br>config.yaml | 可读性好<br>易于编辑 | 需要解析 | 复杂配置 ⭐ |
| **环境变量** | API_KEY=xxx | 安全性高<br>云平台友好 | 不直观 | 敏感信息 |
| **命令行参数** | --port 8080 | 灵活 | 参数多了难记 | 临时覆盖 |

**NOFX 的选择**：以配置文件为主，结合环境变量

---

## 🧠 思维方法

### 方法一：分层配置

**配置的三个层次**：

```
┌─────────────────────────────────────┐
│  1. 默认配置（代码内）                │
│  - 扫描间隔默认 3 分钟                │
│  - 端口默认 8080                     │
└────────────────┬────────────────────┘
                 │ 被覆盖
┌────────────────▼────────────────────┐
│  2. 配置文件（config.json）          │
│  - 用户设置扫描间隔 5 分钟            │
│  - 用户设置端口 3000                 │
└────────────────┬────────────────────┘
                 │ 被覆盖
┌────────────────▼────────────────────┐
│  3. 环境变量/命令行参数               │
│  - API_KEY环境变量                   │
│  - --port 9000 命令行参数            │
└─────────────────────────────────────┘

最终生效：
扫描间隔 = 5 分钟（来自配置文件）
端口 = 9000（来自命令行，优先级最高）
```

**优先级**：
```
命令行参数 > 环境变量 > 配置文件 > 默认值
```

**代码示例**：

```python
# 默认配置
DEFAULT_CONFIG = {
    "port": 8080,
    "scan_interval": 3,
}

# 加载配置
def load_config(config_file):
    # 1. 从默认配置开始
    config = DEFAULT_CONFIG.copy()

    # 2. 读取配置文件（覆盖默认值）
    if os.path.exists(config_file):
        with open(config_file) as f:
            file_config = json.load(f)
            config.update(file_config)

    # 3. 读取环境变量（覆盖文件配置）
    if os.getenv("API_KEY"):
        config["api_key"] = os.getenv("API_KEY")

    # 4. 读取命令行参数（最高优先级）
    if args.port:
        config["port"] = args.port

    return config
```

### 方法二：配置验证

**问题**：用户填错了怎么办？

```json
{
    "api_key": "",               // 忘记填
    "scan_interval_minutes": -5, // 填了负数
    "port": "abc"                // 填了字符串
}
```

**解决**：加载配置后立即验证

```python
def validate_config(config):
    errors = []

    # 必填项检查
    if not config.get("api_key"):
        errors.append("api_key 不能为空")

    # 类型检查
    if not isinstance(config.get("port"), int):
        errors.append("port 必须是整数")

    # 范围检查
    if config.get("scan_interval_minutes", 0) <= 0:
        errors.append("scan_interval_minutes 必须大于0")

    if config.get("port", 0) < 1024 or config.get("port", 0) > 65535:
        errors.append("port 必须在 1024-65535 之间")

    # 如果有错误，提前退出
    if errors:
        print("配置错误：")
        for err in errors:
            print(f"  - {err}")
        sys.exit(1)

    return config
```

### 方法三：敏感信息保护

**敏感信息**：API 密钥、数据库密码、私钥

#### ❌ 错误做法：提交到 Git

```bash
# ❌ 危险！
git add config.json  # config.json 包含 API 密钥
git commit -m "添加配置"
git push  # 密钥泄露！
```

**后果**：
- API密钥被盗用
- 账户被盗刷
- 安全事故

#### ✅ 正确做法

**1. 使用配置模板**

```json
// config.json.example（提交到 Git）
{
    "api_key": "YOUR_API_KEY_HERE",
    "api_secret": "YOUR_API_SECRET_HERE",
    "scan_interval_minutes": 3
}

// .gitignore
config.json  ← 真实配置不提交
```

**用户使用流程**：
```bash
# 1. 复制模板
cp config.json.example config.json

# 2. 填写真实密钥
nano config.json

# 3. config.json 不会被提交到 Git
```

**2. 使用环境变量**

```bash
# .env 文件（不提交到 Git）
API_KEY=sk-real-key-abc123
API_SECRET=secret-xyz789

# 加载环境变量
export $(cat .env | xargs)

# 或使用 python-dotenv
from dotenv import load_dotenv
load_dotenv()
```

**3. 使用密钥管理服务**（生产环境）

- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault

### 方法四：配置的可读性

**格式对比**：

#### JSON
```json
{
    "traders": [
        {
            "id": "trader1",
            "api_key": "xxx"
        }
    ]
}
```

**优点**：
- ✅ 标准格式，所有语言都支持
- ✅ 严格，不容易出错

**缺点**：
- ❌ 不支持注释
- ❌ 语法严格（多个逗号就报错）

#### YAML
```yaml
traders:
  - id: trader1
    api_key: xxx  # 这是注释
    enabled: true
```

**优点**：
- ✅ 支持注释
- ✅ 更简洁

**缺点**：
- ❌ 缩进敏感（空格错了就报错）
- ❌ 解析稍慢

#### TOML
```toml
[[traders]]
id = "trader1"
api_key = "xxx"  # 注释
enabled = true
```

**优点**：
- ✅ 支持注释
- ✅ 语法清晰

**缺点**：
- ❌ 知名度低

**NOFX 选择 JSON**：因为 Go 原生支持好

---

## 📚 NOFX 案例分析

### NOFX 的配置结构

```json
{
    "traders": [
        {
            "id": "trader1",
            "name": "Binance DeepSeek Trader",
            "enabled": true,
            "ai_model": "deepseek",
            "exchange": "binance",

            "binance_api_key": "xxx",
            "binance_secret_key": "yyy",

            "deepseek_key": "sk-zzz",

            "initial_balance": 1000.0,
            "scan_interval_minutes": 3
        }
    ],

    "leverage": {
        "btc_eth_leverage": 5,
        "altcoin_leverage": 5
    },

    "use_default_coins": true,
    "default_coins": ["BTCUSDT", "ETHUSDT", ...],

    "api_server_port": 8080,

    "max_daily_loss": 10.0,
    "max_drawdown": 20.0,
    "stop_trading_minutes": 60
}
```

### 配置分组设计

**1. Trader 配置（数组，支持多个）**

```json
"traders": [
    {
        // 基本信息
        "id": "trader1",           // 唯一标识
        "name": "My Trader",       // 显示名称
        "enabled": true,           // 是否启用

        // 模型选择
        "ai_model": "deepseek",    // deepseek|qwen|custom
        "exchange": "binance",     // binance|hyperliquid|aster

        // 交易所配置
        "binance_api_key": "...",
        "binance_secret_key": "...",

        // AI 配置
        "deepseek_key": "sk-...",

        // 运行参数
        "initial_balance": 1000.0,
        "scan_interval_minutes": 3
    }
]
```

**设计亮点**：
- ✅ 数组结构：支持多 Trader 竞赛
- ✅ `enabled` 字段：可以暂时禁用某个 Trader
- ✅ 交易所和 AI 分离：灵活组合

**2. 全局配置**

```json
// 杠杆配置（所有 Trader 共享）
"leverage": {
    "btc_eth_leverage": 5,
    "altcoin_leverage": 5
},

// 币种池配置
"use_default_coins": true,
"default_coins": ["BTCUSDT", "ETHUSDT"],

// API 服务器
"api_server_port": 8080,

// 风控参数
"max_daily_loss": 10.0,
"max_drawdown": 20.0
```

**设计亮点**：
- ✅ 全局参数：避免重复配置
- ✅ 分类清晰：杠杆、币种、风控分开

### 配置加载代码

```go
// config/config.go
package config

import (
    "encoding/json"
    "os"
    "fmt"
)

// Config 总配置
type Config struct {
    Traders         []TraderConfig  `json:"traders"`
    Leverage        LeverageConfig  `json:"leverage"`
    UseDefaultCoins bool            `json:"use_default_coins"`
    DefaultCoins    []string        `json:"default_coins"`
    APIServerPort   int             `json:"api_server_port"`
    MaxDailyLoss    float64         `json:"max_daily_loss"`
    MaxDrawdown     float64         `json:"max_drawdown"`
}

// TraderConfig 单个 Trader 配置
type TraderConfig struct {
    ID                  string  `json:"id"`
    Name                string  `json:"name"`
    Enabled             bool    `json:"enabled"`
    AIModel             string  `json:"ai_model"`
    Exchange            string  `json:"exchange"`

    // Binance
    BinanceAPIKey       string  `json:"binance_api_key"`
    BinanceSecretKey    string  `json:"binance_secret_key"`

    // Hyperliquid
    HyperliquidPrivateKey string `json:"hyperliquid_private_key"`

    // AI
    DeepSeekKey         string  `json:"deepseek_key"`
    QwenKey             string  `json:"qwen_key"`

    InitialBalance      float64 `json:"initial_balance"`
    ScanIntervalMinutes int     `json:"scan_interval_minutes"`
}

// LoadConfig 加载配置
func LoadConfig(filepath string) (*Config, error) {
    // 1. 读取文件
    data, err := os.ReadFile(filepath)
    if err != nil {
        return nil, fmt.Errorf("读取配置文件失败: %w", err)
    }

    // 2. 解析 JSON
    var config Config
    if err := json.Unmarshal(data, &config); err != nil {
        return nil, fmt.Errorf("解析配置文件失败: %w", err)
    }

    // 3. 设置默认值
    setDefaults(&config)

    // 4. 验证配置
    if err := validate(&config); err != nil {
        return nil, fmt.Errorf("配置验证失败: %w", err)
    }

    return &config, nil
}

// setDefaults 设置默认值
func setDefaults(config *Config) {
    // 全局默认值
    if config.APIServerPort == 0 {
        config.APIServerPort = 8080
    }

    if len(config.DefaultCoins) == 0 {
        config.DefaultCoins = []string{"BTCUSDT", "ETHUSDT", "SOLUSDT"}
    }

    // Trader 默认值
    for i := range config.Traders {
        if config.Traders[i].ScanIntervalMinutes == 0 {
            config.Traders[i].ScanIntervalMinutes = 3
        }

        if config.Traders[i].InitialBalance == 0 {
            config.Traders[i].InitialBalance = 1000.0
        }
    }

    // 智能默认：如果没有币种池 API，自动使用默认币种
    if config.CoinPoolAPIURL == "" {
        config.UseDefaultCoins = true
    }
}

// validate 验证配置
func validate(config *Config) error {
    // 检查是否至少有一个 Trader
    if len(config.Traders) == 0 {
        return fmt.Errorf("至少需要配置一个 trader")
    }

    // 验证每个 Trader
    for i, trader := range config.Traders {
        // 必填字段
        if trader.ID == "" {
            return fmt.Errorf("trader[%d]: id 不能为空", i)
        }

        if trader.AIModel == "" {
            return fmt.Errorf("trader[%d]: ai_model 不能为空", i)
        }

        if trader.Exchange == "" {
            return fmt.Errorf("trader[%d]: exchange 不能为空", i)
        }

        // 交易所配置检查
        if trader.Exchange == "binance" {
            if trader.BinanceAPIKey == "" {
                return fmt.Errorf("trader[%d]: binance_api_key 不能为空", i)
            }
            if trader.BinanceSecretKey == "" {
                return fmt.Errorf("trader[%d]: binance_secret_key 不能为空", i)
            }
        }

        // AI 配置检查
        if trader.AIModel == "deepseek" && trader.DeepSeekKey == "" {
            return fmt.Errorf("trader[%d]: deepseek_key 不能为空", i)
        }

        // 参数范围检查
        if trader.ScanIntervalMinutes < 1 {
            return fmt.Errorf("trader[%d]: scan_interval_minutes 必须 >= 1", i)
        }

        if trader.InitialBalance <= 0 {
            return fmt.Errorf("trader[%d]: initial_balance 必须 > 0", i)
        }
    }

    // 杠杆检查
    if config.Leverage.BTCETHLeverage < 1 || config.Leverage.BTCETHLeverage > 125 {
        return fmt.Errorf("btc_eth_leverage 必须在 1-125 之间")
    }

    // 端口检查
    if config.APIServerPort < 1024 || config.APIServerPort > 65535 {
        return fmt.Errorf("api_server_port 必须在 1024-65535 之间")
    }

    return nil
}
```

### 配置的扩展性设计

#### 1. 支持多种交易所（通过 exchange 字段）

```json
{
    "traders": [
        {
            "id": "binance_trader",
            "exchange": "binance",
            "binance_api_key": "xxx"
        },
        {
            "id": "hyperliquid_trader",
            "exchange": "hyperliquid",
            "hyperliquid_private_key": "yyy"
        }
    ]
}
```

#### 2. 支持多种 AI（通过 ai_model 字段）

```json
{
    "traders": [
        {
            "ai_model": "deepseek",
            "deepseek_key": "sk-xxx"
        },
        {
            "ai_model": "qwen",
            "qwen_key": "sk-yyy"
        },
        {
            "ai_model": "custom",
            "custom_api_url": "https://my-ai.com",
            "custom_api_key": "xxx"
        }
    ]
}
```

#### 3. 支持单/多 Trader（数组结构）

```json
// 单 Trader（初学者）
{
    "traders": [
        { "id": "my_trader", ... }
    ]
}

// 多 Trader 竞赛（高级用户）
{
    "traders": [
        { "id": "trader1", ... },
        { "id": "trader2", ... },
        { "id": "trader3", ... }
    ]
}
```

### 配置文件的演进

#### v1.0（最简单）

```json
{
    "api_key": "xxx",
    "api_secret": "yyy",
    "deepseek_key": "sk-zzz"
}
```

**问题**：只支持单个 Trader，只支持 Binance

#### v1.5（支持多交易所）

```json
{
    "exchange": "binance",  // 新增
    "api_key": "xxx",
    "deepseek_key": "sk-zzz"
}
```

**问题**：还是只支持单个 Trader

#### v2.0（支持多 Trader）⭐ 当前版本

```json
{
    "traders": [  // 改成数组
        {
            "id": "trader1",
            "exchange": "binance",
            "api_key": "xxx"
        }
    ]
}
```

**优势**：灵活、可扩展

---

## 💪 实战练习

### 练习 1：设计你的配置文件（必做）

根据你的项目需求，设计配置文件：

```json
{
    // 你的项目需要哪些配置？
    // 分成几组？
    // 哪些是必填？
    // 哪些有默认值？
}
```

**提示**：
- 参考 NOFX 的分组方式
- 敏感信息单独标注
- 提供示例值

---

### 练习 2：编写配置加载代码（必做）

用你熟悉的语言实现：

```python
def load_config(filepath):
    """
    1. 读取文件
    2. 解析 JSON
    3. 设置默认值
    4. 验证配置
    5. 返回配置对象
    """
    pass
```

---

### 练习 3：配置验证（必做）

为你的配置编写验证函数：

```python
def validate_config(config):
    errors = []

    # 必填项检查
    if not config.get("xxx"):
        errors.append("xxx 不能为空")

    # 类型检查
    # 范围检查
    # ...

    if errors:
        # 打印错误
        # 退出程序

    return config
```

---

### 练习 4：配置模板（必做）

创建三个文件：

1. **config.json.example**（模板，提交到 Git）
```json
{
    "api_key": "YOUR_API_KEY_HERE",
    "database_url": "YOUR_DATABASE_URL"
}
```

2. **.gitignore**（忽略真实配置）
```
config.json
.env
```

3. **README 使用说明**
```markdown
## 配置步骤

1. 复制配置模板
   ```bash
   cp config.json.example config.json
   ```

2. 填写真实配置
   编辑 config.json，替换所有 YOUR_XXX_HERE

3. 启动程序
   ```bash
   python main.py
   ```
```

---

### 练习 5：环境变量支持（选做）

支持通过环境变量覆盖配置：

```python
def load_config(filepath):
    config = load_from_file(filepath)

    # 环境变量覆盖
    if os.getenv("API_KEY"):
        config["api_key"] = os.getenv("API_KEY")

    if os.getenv("PORT"):
        config["port"] = int(os.getenv("PORT"))

    return config
```

---

## 🤔 思考题

### 1. 为什么 NOFX 用 JSON 而不是 YAML？

<details>
<summary>点击查看答案</summary>

**原因**：
1. **Go 原生支持好**：`encoding/json` 标准库强大
2. **严格语法**：不容易因为缩进出错（YAML 缩进敏感）
3. **性能**：JSON 解析更快
4. **普及度**：所有语言都支持 JSON

**为什么不用 YAML**：
- YAML 缩进容易出错
- 需要第三方库
- 初学者不熟悉

**什么时候用 YAML**：
- Kubernetes 配置（行业标准）
- Docker Compose（行业标准）
- 配置项特别多，需要注释

</details>

---

### 2. `enabled` 字段有什么用？为什么不直接删除 Trader 配置？

<details>
<summary>点击查看答案</summary>

**`enabled: false` 的用途**：
1. **临时禁用**：测试时暂时关闭某个 Trader
2. **保留配置**：不用删除配置，随时可以重新启用
3. **快速切换**：改一个 true/false 比删除整段配置方便

**场景**：
```json
{
    "traders": [
        {
            "id": "trader1",
            "enabled": true   // 正在运行
        },
        {
            "id": "trader2",
            "enabled": false  // 暂时禁用，但保留配置
        }
    ]
}
```

**如果没有 enabled**：
- 要停用 trader2 → 删除整段配置
- 要重新启用 → 重新填写所有配置（容易出错）

</details>

---

### 3. 配置文件应该多详细？所有参数都放进去吗？

<details>
<summary>点击查看答案</summary>

**原则**：
1. **常改的**：放配置文件（API密钥、扫描间隔）
2. **不常改的**：代码里写死（端口默认值、超时时间）
3. **高级的**：代码里写死，文档里说明

**反例**：

```json
// ❌ 太详细了
{
    "http_timeout_seconds": 30,
    "json_indent_spaces": 2,
    "log_timestamp_format": "2006-01-02 15:04:05",
    "max_retries": 3,
    "retry_delay_ms": 1000,
    // ... 100 个配置项
}
```

**问题**：
- 用户看晕了，不知道哪些重要
- 配置错了容易出问题

**正确做法**：

```json
// ✅ 只暴露必要的
{
    "api_key": "xxx",           // 必须配置
    "scan_interval": 3,         // 常改
    // 其他用默认值
}

// 高级配置在代码里
const HTTP_TIMEOUT = 30
const MAX_RETRIES = 3
```

**指导原则**：
- 80%的用户需要的 → 放配置文件
- 20%的高级用户需要的 → 代码里写死，文档说明

</details>

---

## 📖 本章总结

### 你学到了什么

✅ **核心概念**：
- 配置 vs 硬编码
- 配置的三种形式（文件、环境变量、命令行）
- 配置优先级

✅ **思维方法**：
- 分层配置（默认 → 文件 → 环境变量 → 命令行）
- 配置验证（早发现问题）
- 敏感信息保护（.gitignore）
- 可读性设计（JSON vs YAML）

✅ **实践技能**：
- 能设计合理的配置结构
- 能编写配置加载和验证代码
- 能保护敏感信息
- 能提供配置模板

✅ **案例收获**：
- NOFX 的配置演进过程
- 如何支持多 Trader
- 如何做配置验证
- 配置的扩展性设计

---

### 配置系统检查清单

- [ ] **易用性**：用户能轻松理解和填写
- [ ] **安全性**：敏感信息不提交到 Git
- [ ] **验证**：加载时检查配置正确性
- [ ] **默认值**：合理的默认值
- [ ] **文档**：提供 example 和说明
- [ ] **扩展性**：方便添加新配置项
- [ ] **向后兼容**：旧配置能继续用

---

### 下一步

完成练习后，你应该有：
- ✅ 你的项目配置文件设计
- ✅ config.json.example 模板
- ✅ 配置加载和验证代码
- ✅ .gitignore 配置

准备好后，进入 **第 5 章：数据获取层**，学习如何对接外部 API！

---

## 📚 延伸阅读

- [The Twelve-Factor App - Config](https://12factor.net/config)
- [环境变量最佳实践](https://github.com/motdotla/dotenv)
- Go 配置库：Viper
- Python 配置库：python-dotenv, pydantic

---

## ❓ FAQ

**Q1：配置文件放哪个目录？**
A：
- 简单项目：根目录 `config.json`
- 复杂项目：`config/` 目录

**Q2：配置文件要不要加密？**
A：
- 本地开发：不需要
- 生产环境：敏感信息用环境变量或密钥管理服务

**Q3：配置改了要重启程序吗？**
A：
- 简单做法：要重启
- 高级做法：监听文件变化，热重载（复杂度高）

**Q4：多环境怎么办？**
A：
```
config.dev.json    # 开发环境
config.test.json   # 测试环境
config.prod.json   # 生产环境

# 启动时指定
python main.py --config config.prod.json
```

**Q5：用户不会填 JSON 怎么办？**
A：
- 提供详细注释版 example
- 提供配置生成工具（问答式）
- 提供 Web 界面配置

---

**🎉 恭喜完成第 4 章！**

你已经掌握了配置系统设计！

记住：**好的配置让系统灵活，坏的配置让用户抓狂。**

准备好了吗？进入第 5 章！💪
