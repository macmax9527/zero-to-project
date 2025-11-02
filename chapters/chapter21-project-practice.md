# 第21章：项目实战 - 构建简化版 NOFX

> **本章目标**：综合运用前面所学知识，从零构建一个简化版交易机器人

---

## 📋 本章大纲

1. [项目需求分析](#1-项目需求分析)
2. [架构设计](#2-架构设计)
3. [模块实现](#3-模块实现)
4. [测试与调试](#4-测试与调试)
5. [部署运行](#5-部署运行)
6. [总结与扩展](#6-总结与扩展)

**预计完成时间**：1-2 周（业余时间）

---

## 1. 项目需求分析

### 1.1 项目目标

构建一个**简化版自动交易机器人**，核心功能包括：
- 连接交易所 API
- 获取市场数据
- 简单的交易策略
- Web 界面展示

### 1.2 功能清单

**Must Have（MVP 核心功能）**：
```
✅ 连接币安测试网
✅ 获取 BTC/USDT 实时价格
✅ 简单的移动平均策略
   - 短期均线 > 长期均线 → 买入
   - 短期均线 < 长期均线 → 卖出
✅ 记录交易日志
✅ 简单的 Web 界面显示账户信息
```

**Should Have（后续扩展）**：
```
🟡 支持多个币种
🟡 多种交易策略
🟡 图表展示
🟡 风险控制
```

**Won't Have（明确不做）**：
```
⛔ AI 决策（太复杂）
⛔ 高频交易
⛔ 多交易所支持
⛔ 移动 App
```

### 1.3 技术选型

| 技术 | 选择 | 原因 |
|------|------|------|
| 后端语言 | Python | 简单易学，库丰富 |
| Web 框架 | Flask | 轻量级，适合小项目 |
| 交易所 API | 币安 | 文档完善，测试网免费 |
| 前端 | HTML + JavaScript | 极简实现 |
| 数据存储 | JSON 文件 | 无需数据库 |

---

## 2. 架构设计

### 2.1 整体架构

```
┌─────────────────────────────────────────┐
│           Web 界面（浏览器）               │
│  显示账户、持仓、交易历史                   │
└──────────────┬──────────────────────────┘
               │ HTTP API
               ↓
┌─────────────────────────────────────────┐
│        Flask API 服务器（后端）           │
│  - GET /api/account   账户信息            │
│  - GET /api/positions 持仓信息            │
│  - GET /api/trades    交易历史            │
└──────────────┬──────────────────────────┘
               │
      ┌────────┼────────┐
      ↓        ↓        ↓
 ┌─────────┐ ┌──────┐ ┌────────┐
 │ 交易引擎 │ │ 策略 │ │ 日志   │
 │         │ │ 模块 │ │ 记录器 │
 └────┬────┘ └──┬───┘ └────────┘
      │         │
      ↓         ↓
 ┌─────────────────────┐
 │   币安 API           │
 │  - 获取价格          │
 │  - 执行交易          │
 └─────────────────────┘
```

### 2.2 目录结构

```
simple-trader/
├── main.py                  # 主程序入口
├── config.json              # 配置文件
├── requirements.txt         # Python 依赖
├── exchange/
│   └── binance_client.py    # 币安API客户端
├── strategy/
│   └── ma_strategy.py       # 移动平均策略
├── api/
│   └── server.py            # Flask API服务器
├── web/
│   └── index.html           # Web界面
└── data/
    └── trades.json          # 交易记录
```

---

## 3. 模块实现

### 3.1 配置文件

**config.json**：
```json
{
    "api_key": "your_binance_testnet_key",
    "api_secret": "your_binance_testnet_secret",
    "symbol": "BTCUSDT",
    "trade_amount": 0.001,
    "short_period": 5,
    "long_period": 20,
    "check_interval": 60
}
```

### 3.2 币安客户端

**exchange/binance_client.py**：
```python
import hmac
import hashlib
import time
import requests
from urllib.parse import urlencode

class BinanceClient:
    def __init__(self, api_key, api_secret, testnet=True):
        self.api_key = api_key
        self.api_secret = api_secret
        self.base_url = "https://testnet.binance.vision" if testnet else "https://api.binance.com"

    def _sign(self, params):
        """生成签名"""
        query_string = urlencode(params)
        signature = hmac.new(
            self.api_secret.encode('utf-8'),
            query_string.encode('utf-8'),
            hashlib.sha256
        ).hexdigest()
        params['signature'] = signature
        return params

    def get_price(self, symbol):
        """获取当前价格"""
        url = f"{self.base_url}/api/v3/ticker/price"
        params = {'symbol': symbol}
        response = requests.get(url, params=params)
        data = response.json()
        return float(data['price'])

    def get_account(self):
        """获取账户信息"""
        url = f"{self.base_url}/api/v3/account"
        params = {
            'timestamp': int(time.time() * 1000)
        }
        params = self._sign(params)
        headers = {'X-MBX-APIKEY': self.api_key}
        response = requests.get(url, params=params, headers=headers)
        return response.json()

    def buy(self, symbol, quantity):
        """市价买入"""
        return self._order(symbol, 'BUY', quantity)

    def sell(self, symbol, quantity):
        """市价卖出"""
        return self._order(symbol, 'SELL', quantity)

    def _order(self, symbol, side, quantity):
        """下单"""
        url = f"{self.base_url}/api/v3/order"
        params = {
            'symbol': symbol,
            'side': side,
            'type': 'MARKET',
            'quantity': quantity,
            'timestamp': int(time.time() * 1000)
        }
        params = self._sign(params)
        headers = {'X-MBX-APIKEY': self.api_key}
        response = requests.post(url, params=params, headers=headers)
        return response.json()
```

### 3.3 移动平均策略

**strategy/ma_strategy.py**：
```python
class MAStrategy:
    def __init__(self, short_period=5, long_period=20):
        self.short_period = short_period
        self.long_period = long_period
        self.prices = []

    def add_price(self, price):
        """添加新价格"""
        self.prices.append(price)
        # 只保留需要的数据量
        if len(self.prices) > self.long_period:
            self.prices = self.prices[-self.long_period:]

    def calculate_ma(self, period):
        """计算移动平均"""
        if len(self.prices) < period:
            return None
        return sum(self.prices[-period:]) / period

    def should_buy(self):
        """判断是否应该买入"""
        short_ma = self.calculate_ma(self.short_period)
        long_ma = self.calculate_ma(self.long_period)

        if short_ma is None or long_ma is None:
            return False

        # 短期均线上穿长期均线 → 买入信号
        return short_ma > long_ma and not self.is_holding()

    def should_sell(self):
        """判断是否应该卖出"""
        short_ma = self.calculate_ma(self.short_period)
        long_ma = self.calculate_ma(self.long_period)

        if short_ma is None or long_ma is None:
            return False

        # 短期均线下穿长期均线 → 卖出信号
        return short_ma < long_ma and self.is_holding()

    def is_holding(self):
        """是否持仓（简化实现，实际应查询账户）"""
        # 这里需要与交易引擎集成
        pass
```

### 3.4 交易引擎

**main.py**：
```python
import json
import time
from exchange.binance_client import BinanceClient
from strategy.ma_strategy import MAStrategy

class TradingBot:
    def __init__(self, config_file='config.json'):
        # 加载配置
        with open(config_file) as f:
            self.config = json.load(f)

        # 初始化客户端
        self.client = BinanceClient(
            self.config['api_key'],
            self.config['api_secret'],
            testnet=True
        )

        # 初始化策略
        self.strategy = MAStrategy(
            self.config['short_period'],
            self.config['long_period']
        )

        # 持仓状态
        self.holding = False
        self.trades = []

    def run(self):
        """主循环"""
        print("🤖 交易机器人启动...")

        while True:
            try:
                # 1. 获取当前价格
                price = self.client.get_price(self.config['symbol'])
                print(f"📊 当前价格: {price}")

                # 2. 更新策略
                self.strategy.add_price(price)

                # 3. 判断信号
                if not self.holding and self.strategy.should_buy():
                    self.execute_buy(price)
                elif self.holding and self.strategy.should_sell():
                    self.execute_sell(price)

                # 4. 等待下一个周期
                time.sleep(self.config['check_interval'])

            except Exception as e:
                print(f"❌ 错误: {e}")
                time.sleep(10)

    def execute_buy(self, price):
        """执行买入"""
        print(f"🟢 买入信号，价格: {price}")
        try:
            result = self.client.buy(
                self.config['symbol'],
                self.config['trade_amount']
            )
            self.holding = True
            self.record_trade('BUY', price, self.config['trade_amount'])
            print(f"✅ 买入成功: {result}")
        except Exception as e:
            print(f"❌ 买入失败: {e}")

    def execute_sell(self, price):
        """执行卖出"""
        print(f"🔴 卖出信号，价格: {price}")
        try:
            result = self.client.sell(
                self.config['symbol'],
                self.config['trade_amount']
            )
            self.holding = False
            self.record_trade('SELL', price, self.config['trade_amount'])
            print(f"✅ 卖出成功: {result}")
        except Exception as e:
            print(f"❌ 卖出失败: {e}")

    def record_trade(self, side, price, quantity):
        """记录交易"""
        trade = {
            'timestamp': time.strftime('%Y-%m-%d %H:%M:%S'),
            'side': side,
            'price': price,
            'quantity': quantity
        }
        self.trades.append(trade)

        # 保存到文件
        with open('data/trades.json', 'w') as f:
            json.dump(self.trades, f, indent=2)

if __name__ == '__main__':
    bot = TradingBot()
    bot.run()
```

### 3.5 API 服务器

**api/server.py**：
```python
from flask import Flask, jsonify
from flask_cors import CORS
import json

app = Flask(__name__)
CORS(app)

@app.route('/api/account')
def get_account():
    """获取账户信息"""
    # 实际应该从交易所获取
    return jsonify({
        'balance': 1000,
        'equity': 1050,
        'pnl': 50,
        'pnl_pct': 5.0
    })

@app.route('/api/trades')
def get_trades():
    """获取交易历史"""
    try:
        with open('data/trades.json') as f:
            trades = json.load(f)
        return jsonify(trades)
    except:
        return jsonify([])

if __name__ == '__main__':
    app.run(port=8080)
```

### 3.6 Web 界面

**web/index.html**：
```html
<!DOCTYPE html>
<html>
<head>
    <title>Simple Trader</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        .card { border: 1px solid #ddd; padding: 20px; margin: 10px 0; border-radius: 5px; }
        table { width: 100%; border-collapse: collapse; }
        th, td { padding: 10px; text-align: left; border-bottom: 1px solid #ddd; }
    </style>
</head>
<body>
    <h1>🤖 Simple Trader</h1>

    <div class="card">
        <h2>账户信息</h2>
        <div id="account">加载中...</div>
    </div>

    <div class="card">
        <h2>交易历史</h2>
        <table id="trades">
            <thead>
                <tr>
                    <th>时间</th>
                    <th>类型</th>
                    <th>价格</th>
                    <th>数量</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
    </div>

    <script>
        // 获取账户信息
        async function fetchAccount() {
            const response = await fetch('http://localhost:8080/api/account');
            const data = await response.json();
            document.getElementById('account').innerHTML = `
                <p>余额: ${data.balance} USDT</p>
                <p>净值: ${data.equity} USDT</p>
                <p>盈亏: ${data.pnl} USDT (${data.pnl_pct}%)</p>
            `;
        }

        // 获取交易历史
        async function fetchTrades() {
            const response = await fetch('http://localhost:8080/api/trades');
            const trades = await response.json();
            const tbody = document.querySelector('#trades tbody');
            tbody.innerHTML = trades.map(trade => `
                <tr>
                    <td>${trade.timestamp}</td>
                    <td>${trade.side}</td>
                    <td>${trade.price}</td>
                    <td>${trade.quantity}</td>
                </tr>
            `).join('');
        }

        // 定时更新
        fetchAccount();
        fetchTrades();
        setInterval(() => {
            fetchAccount();
            fetchTrades();
        }, 5000);
    </script>
</body>
</html>
```

---

## 4. 测试与调试

### 4.1 单元测试

**test_strategy.py**：
```python
import pytest
from strategy.ma_strategy import MAStrategy

def test_ma_calculation():
    strategy = MAStrategy(short_period=3, long_period=5)

    # 添加价格
    prices = [100, 102, 104, 103, 105]
    for price in prices:
        strategy.add_price(price)

    # 测试短期均线
    short_ma = strategy.calculate_ma(3)
    assert abs(short_ma - 104) < 0.1  # (104 + 103 + 105) / 3 ≈ 104

    # 测试长期均线
    long_ma = strategy.calculate_ma(5)
    assert abs(long_ma - 102.8) < 0.1  # (100 + 102 + 104 + 103 + 105) / 5 = 102.8
```

### 4.2 集成测试

在币安测试网测试：
1. 注册币安测试网账号
2. 获取测试网 API Key
3. 运行机器人，观察是否正常交易
4. 检查日志文件

### 4.3 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| API 签名错误 | 时间戳不同步 | 检查系统时间 |
| 余额不足 | 测试网余额用完 | 重新申请测试网资产 |
| 网络超时 | 网络问题 | 添加重试机制 |
| 策略不执行 | 数据不足 | 等待足够的价格数据 |

---

## 5. 部署运行

### 5.1 本地运行

```bash
# 1. 安装依赖
pip install -r requirements.txt

# 2. 配置 config.json

# 3. 启动交易机器人
python main.py

# 4. 启动 API 服务器（新终端）
python api/server.py

# 5. 打开浏览器访问 web/index.html
```

### 5.2 云服务器部署

**使用 Supervisor 保持运行**：

```bash
# 安装 supervisor
sudo apt-get install supervisor

# 配置文件 /etc/supervisor/conf.d/trader.conf
[program:trader]
command=/usr/bin/python3 /path/to/main.py
directory=/path/to/simple-trader
autostart=true
autorestart=true
stderr_logfile=/var/log/trader.err.log
stdout_logfile=/var/log/trader.out.log

# 启动
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start trader
```

---

## 6. 总结与扩展

### 6.1 学到了什么

通过这个项目，你应该掌握了：

✅ **需求分析**：从想法到功能清单
✅ **架构设计**：模块化、职责分离
✅ **API 集成**：调用第三方 API
✅ **策略实现**：交易算法
✅ **前后端交互**：API 服务器 + Web 界面
✅ **测试部署**：单元测试、集成测试、服务器部署

### 6.2 扩展方向

**功能扩展**：
```
1. 添加更多策略
   - RSI 策略
   - MACD 策略
   - 布林带策略

2. 风险控制
   - 止损止盈
   - 最大回撤限制
   - 单日亏损限制

3. 数据可视化
   - 收益曲线
   - K 线图
   - 实时行情

4. 通知功能
   - 交易通知（Telegram/邮件）
   - 异常告警

5. 回测系统
   - 历史数据回测
   - 策略优化
```

**技术优化**：
```
1. 使用数据库
   - SQLite 存储交易记录
   - 历史数据查询

2. 异步处理
   - asyncio 提升性能
   - WebSocket 实时数据

3. 前端升级
   - React 组件化
   - Echarts 图表库
   - 响应式设计

4. 安全加固
   - API Key 加密存储
   - JWT 认证
   - HTTPS 部署
```

### 6.3 对比 NOFX

| 特性 | 简化版 | NOFX |
|------|--------|------|
| 语言 | Python | Go |
| 策略 | 移动平均 | AI 决策 |
| 交易所 | 币安 | 币安 + Hyperliquid |
| 前端 | 纯 HTML/JS | React |
| 存储 | JSON 文件 | 内存 + API |
| 复杂度 | ⭐⭐ | ⭐⭐⭐⭐⭐ |

**简化版优势**：
- 易于理解和学习
- 快速上手
- 适合新手

**NOFX 优势**：
- 性能更好（Go 语言）
- AI 决策更智能
- 功能更完善

---

## 实战练习

### 练习 1：完成基础版本

按照本章代码，实现一个可运行的简化版交易机器人

**要求**：
1. 在币安测试网运行
2. 至少执行一次完整交易（买入→卖出）
3. Web 界面能显示交易记录

### 练习 2：添加止损功能

为机器人添加简单的止损机制

**要求**：
- 持仓亏损超过 5% 自动平仓
- 记录止损事件

### 练习 3：回测系统

下载历史数据，测试策略表现

**提示**：
- 使用币安 API 获取历史 K 线数据
- 模拟执行交易
- 计算总盈亏和胜率

---

**💡 总结**：动手实践是最好的学习方式。通过构建这个简化版交易机器人，你将深刻理解 NOFX 的设计思想。记住：先把基础版做出来，再逐步完善！
