# 第14章：迭代扩展 - 版本规划

> **本章目标**：学会规划项目迭代，有序扩展功能

---

## 📋 本章大纲

1. [为什么要迭代开发](#1-为什么要迭代开发)
2. [版本规划方法](#2-版本规划方法)
3. [功能优先级排序](#3-功能优先级排序)
4. [向后兼容](#4-向后兼容)
5. [数据库迁移](#5-数据库迁移)
6. [NOFX 的版本演进](#6-nofx-的版本演进)
7. [实战练习](#7-实战练习)

**预计学习时间**：3-4 小时

---

## 1. 为什么要迭代开发

### 1.1 一次性开发的问题

```
❌ 瀑布模式（一次性开发）

需求分析 → 设计 → 开发 → 测试 → 发布
     ↓
  6个月后才能看到产品
  需求可能已经变化
  发现问题时成本巨大
```

```
✅ 迭代模式

v1.0 (2周) → v1.1 (2周) → v1.2 (2周) → ...
  ↓            ↓            ↓
 核心功能     优化         新功能
 快速验证     用户反馈     持续改进
```

### 1.2 迭代开发的优势

**1. 快速验证想法**
```
第1周：开发基础版本
第2周：给用户试用
第3周：根据反馈调整
第4周：发布正式版
```

**2. 降低风险**
- 早期发现问题
- 及时调整方向
- 分散开发成本

**3. 持续交付价值**
- 用户每2周看到新功能
- 保持团队动力
- 避免"最后一刻"压力

### 1.3 迭代周期

**常见周期**：
- **Sprint（冲刺）**：1-2周
- **Release（发布）**：4-6周
- **Major Version（大版本）**：3-6个月

**示例**：
```
Sprint 1 (2周)：用户登录、基础UI
Sprint 2 (2周)：数据展示、图表
Sprint 3 (2周)：导出功能、通知
  ↓
Release v1.0 (6周后)
  ↓
Sprint 4 (2周)：性能优化
Sprint 5 (2周)：新功能A
Sprint 6 (2周)：新功能B
  ↓
Release v1.1 (12周后)
```

---

## 2. 版本规划方法

### 2.1 语义化版本（Semantic Versioning）

**格式**：`Major.Minor.Patch`

```
v1.2.3
 │ │ │
 │ │ └─ Patch (修复): 1.2.3 → 1.2.4
 │ │    - 修复bug
 │ │    - 向后兼容
 │ │
 │ └─── Minor (功能): 1.2.0 → 1.3.0
 │      - 新增功能
 │      - 向后兼容
 │
 └───── Major (重大): 1.0.0 → 2.0.0
        - 破坏性变更
        - 不兼容旧版本
```

**示例**：
```
v0.1.0: 初始版本（MVP）
v0.2.0: 添加用户管理功能
v0.2.1: 修复登录bug
v0.3.0: 添加数据导出功能
v1.0.0: 正式发布（稳定版）
v1.1.0: 添加通知功能
v1.1.1: 修复通知延迟问题
v2.0.0: 重构架构，API不兼容
```

### 2.2 版本规划模板

**v1.0 规划**：

```markdown
## v1.0 - 核心功能（目标：2024-03-01）

### Must Have（必须有）
- [x] 用户注册/登录
- [x] 数据列表展示
- [x] 基础CRUD操作
- [ ] 数据持久化

### Should Have（应该有）
- [ ] 搜索功能
- [ ] 数据导出

### Could Have（可以有）
- [ ] 邮件通知
- [ ] 高级筛选

### Won't Have（暂不做）
- 多语言支持
- 第三方集成
```

### 2.3 Roadmap（路线图）

```
2024 产品路线图

Q1 (1-3月)
├─ v1.0: 核心功能上线
├─ v1.1: 用户反馈优化
└─ v1.2: 性能优化

Q2 (4-6月)
├─ v1.3: 高级搜索
├─ v1.4: 数据分析
└─ v2.0: 架构升级

Q3 (7-9月)
├─ v2.1: 移动端支持
├─ v2.2: API开放平台
└─ v2.3: 企业版功能

Q4 (10-12月)
├─ v3.0: AI功能集成
└─ v3.1: 国际化
```

---

## 3. 功能优先级排序

### 3.1 MoSCoW 方法

**复习第13章**：
- **M**ust have：必须有
- **S**hould have：应该有
- **C**ould have：可以有
- **W**on't have：暂不做

### 3.2 价值 vs 成本矩阵

```
      高价值
        ↑
快速实现 │ 优先做 │ 计划做
────────┼────────┼────────→ 高成本
慢速实现 │ 考虑做 │ 不做
        │
      低价值
```

**示例**：

| 功能 | 价值 | 成本 | 优先级 |
|------|------|------|--------|
| 用户登录 | 高 | 低 | 🔴 P0（立即做） |
| 数据导出 | 高 | 低 | 🔴 P0 |
| 实时通知 | 中 | 高 | 🟡 P1（计划） |
| 深色模式 | 低 | 低 | 🟢 P2（考虑） |
| AI推荐 | 高 | 高 | 🟡 P1 |
| 多语言 | 低 | 高 | ⚪ P3（不做） |

### 3.3 RICE 评分法

**公式**：`RICE = (Reach × Impact × Confidence) / Effort`

- **Reach**（覆盖面）：影响多少用户？
- **Impact**（影响力）：对用户的价值？(3=高, 2=中, 1=低)
- **Confidence**（信心）：有多确定？(100%=1.0, 50%=0.5)
- **Effort**（工作量）：需要多少人天？

**示例**：

| 功能 | Reach | Impact | Confidence | Effort | RICE | 排序 |
|------|-------|--------|------------|--------|------|------|
| 搜索 | 1000 | 3 | 0.8 | 5 | 480 | 1 |
| 通知 | 800 | 2 | 0.9 | 3 | 480 | 1 |
| 深色 | 500 | 1 | 1.0 | 2 | 250 | 3 |
| AI | 1200 | 3 | 0.5 | 20 | 90 | 4 |

---

## 4. 向后兼容

### 4.1 什么是向后兼容

**向后兼容**：新版本能运行旧版本的代码/数据

```python
# v1.0 API
def get_user(id):
    return {"id": id, "name": "张三"}

# v2.0 API（向后兼容）
def get_user(id, include_email=False):
    user = {"id": id, "name": "张三"}
    if include_email:
        user["email"] = "zhang@example.com"
    return user

# 旧代码仍然可以运行
user = get_user(1)  # ✅ 正常工作
```

```python
# v2.0 API（不兼容）
def get_user(id, include_email):  # 新增必需参数
    # ...

# 旧代码会报错
user = get_user(1)  # ❌ TypeError: missing argument 'include_email'
```

### 4.2 API 版本管理

**方法1：URL版本**

```python
# v1 API
@app.route('/api/v1/users/<id>')
def get_user_v1(id):
    return {"id": id, "name": "张三"}

# v2 API
@app.route('/api/v2/users/<id>')
def get_user_v2(id):
    return {"id": id, "name": "张三", "email": "zhang@example.com"}

# 旧客户端继续使用 /api/v1/users
# 新客户端使用 /api/v2/users
```

**方法2：请求头版本**

```python
@app.route('/api/users/<id>')
def get_user(id):
    version = request.headers.get('API-Version', 'v1')

    user = {"id": id, "name": "张三"}

    if version == 'v2':
        user["email"] = "zhang@example.com"

    return user
```

### 4.3 数据格式兼容

**添加字段（兼容）**：

```python
# v1.0 数据格式
{
    "id": 1,
    "name": "张三"
}

# v2.0 数据格式（添加字段，兼容）
{
    "id": 1,
    "name": "张三",
    "email": "zhang@example.com",  # 新字段
    "created_at": "2024-01-01"     # 新字段
}

# v1.0 客户端仍能正常读取 id 和 name
```

**修改字段（不兼容）**：

```python
# v1.0
{
    "user_id": 1,      # 字段名
    "full_name": "张三"
}

# v2.0（不兼容）
{
    "id": 1,           # 改名：user_id → id
    "name": "张三"     # 改名：full_name → name
}

# v1.0 客户端会找不到 user_id 字段
```

### 4.4 废弃（Deprecation）策略

**分阶段废弃**：

```python
# v2.0: 标记废弃
@deprecated("getUserById 将在 v3.0 移除，请使用 getUser")
def getUserById(id):
    return get_user(id)

# v2.1: 警告
def getUserById(id):
    warnings.warn("getUserById 已废弃，请使用 getUser", DeprecationWarning)
    return get_user(id)

# v3.0: 移除
# getUserById 函数完全删除
```

**时间线**：
```
v2.0 (2024-01)：标记废弃，文档说明
v2.1 (2024-04)：添加警告
v2.2 (2024-07)：最后兼容版本
v3.0 (2024-10)：完全移除
```

---

## 5. 数据库迁移

### 5.1 为什么需要迁移

**场景**：添加新字段

```sql
-- v1.0 数据库
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- v1.1 需要添加 email 字段
-- 如何在不丢失现有数据的情况下更新结构？
```

### 5.2 迁移工具

**Python - Alembic**：

```bash
# 安装
pip install alembic

# 初始化
alembic init migrations

# 创建迁移
alembic revision -m "add email to users"

# 应用迁移
alembic upgrade head

# 回滚
alembic downgrade -1
```

**迁移文件示例**：

```python
# migrations/versions/001_add_email.py

def upgrade():
    # 升级逻辑
    op.add_column('users', sa.Column('email', sa.String(255), nullable=True))

def downgrade():
    # 降级逻辑
    op.drop_column('users', 'email')
```

**Go - golang-migrate**：

```bash
# 安装
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest

# 创建迁移
migrate create -ext sql -dir migrations -seq add_email_to_users

# 应用迁移
migrate -path migrations -database "postgres://localhost/mydb" up

# 回滚
migrate -path migrations -database "postgres://localhost/mydb" down 1
```

**迁移文件**：

```sql
-- migrations/000001_add_email_to_users.up.sql
ALTER TABLE users ADD COLUMN email VARCHAR(255);

-- migrations/000001_add_email_to_users.down.sql
ALTER TABLE users DROP COLUMN email;
```

### 5.3 迁移最佳实践

**1. 小步迁移**

```
❌ 大迁移（危险）
v1.0 → v2.0：重构整个数据库结构

✅ 小步迭代（安全）
v1.0 → v1.1：添加 email 字段
v1.1 → v1.2：添加索引
v1.2 → v1.3：迁移数据
v1.3 → v2.0：删除旧字段
```

**2. 提供回滚**

```python
def upgrade():
    # 升级必须有对应的降级
    op.add_column('users', sa.Column('email', sa.String(255)))

def downgrade():
    # 确保可以回滚
    op.drop_column('users', 'email')
```

**3. 数据迁移**

```python
from alembic import op
from sqlalchemy.sql import table, column

def upgrade():
    # 1. 添加新字段
    op.add_column('users', sa.Column('full_name', sa.String(255)))

    # 2. 迁移数据
    users = table('users',
        column('first_name', sa.String),
        column('last_name', sa.String),
        column('full_name', sa.String)
    )

    op.execute(
        users.update().values(
            full_name=users.c.first_name + ' ' + users.c.last_name
        )
    )

    # 3. 删除旧字段（可选，分阶段）
    # op.drop_column('users', 'first_name')
    # op.drop_column('users', 'last_name')
```

---

## 6. NOFX 的版本演进

### 6.1 实际演进历程

**v0.1.0 - MVP**（2024-01）
- ✅ 单个交易所支持（Binance）
- ✅ 简单移动平均策略
- ✅ 基础Web界面
- ❌ 无持久化（内存存储）

**v0.2.0 - 多策略**（2024-02）
- ✅ 添加 RSI 策略
- ✅ 策略可配置
- ✅ 日志优化
- 🔧 重构策略接口

**v0.3.0 - 多交易所**（2024-03）
- ✅ OKX 支持
- ✅ Hyperliquid 支持
- ✅ 统一交易接口
- 🗄️ 添加配置数据库

**v1.0.0 - 正式版**（2024-04）
- ✅ 完整的Web管理界面
- ✅ 风险管理模块
- ✅ 性能监控
- ✅ 文档完善
- 🧪 测试覆盖率 > 70%

**v1.1.0 - 性能优化**（2024-05）
- ⚡ 并发优化
- ⚡ API调用缓存
- 📊 实时图表
- 🐛 修复已知bug

### 6.2 未来规划

**v1.2.0 - 机器学习**（规划中）
- 🤖 ML策略集成
- 📈 预测模型
- 🧠 自动参数优化

**v2.0.0 - 重构**（规划中）
- 🏗️ 微服务架构
- 🐳 Docker 部署
- ☸️ Kubernetes 支持
- 💾 PostgreSQL 替代内存存储

### 6.3 变更日志（CHANGELOG）

**格式**：

```markdown
# Changelog

## [1.1.0] - 2024-05-01

### Added
- 实时图表功能
- API调用缓存

### Changed
- 优化并发性能
- 改进日志格式

### Fixed
- 修复持仓计算错误
- 修复 WebSocket 重连问题

### Deprecated
- `GetPositionById` 将在 v2.0 移除，请使用 `GetPosition`

## [1.0.0] - 2024-04-01

### Added
- 完整的 Web 管理界面
- 风险管理模块
- 性能监控

...
```

---

## 7. 实战练习

### 练习 1：规划你的项目版本

为你的项目（或第21章的交易系统）规划3个版本。

**要求**：
- 使用语义化版本号
- 列出每个版本的功能
- 使用 MoSCoW 分类
- 估计开发时间

**模板**：
```markdown
## v0.1.0 - MVP (2周)
### Must Have
- [ ] ...
### Should Have
- [ ] ...

## v0.2.0 - 优化 (2周)
...

## v1.0.0 - 正式版 (4周)
...
```

### 练习 2：API 版本兼容

设计一个用户API，支持两个版本。

**要求**：
- v1 返回基本信息
- v2 返回详细信息
- 保证 v1 客户端继续工作

```python
# 实现这两个版本
@app.route('/api/v1/users/<id>')
def get_user_v1(id):
    pass

@app.route('/api/v2/users/<id>')
def get_user_v2(id):
    pass
```

### 练习 3：数据库迁移

为用户表添加 `phone` 字段。

**要求**：
- 使用 Alembic 或 golang-migrate
- 编写 upgrade 和 downgrade
- 测试迁移和回滚

---

## 本章总结

### 迭代开发原则

1. **小步快跑**：每2周发布一次
2. **用户反馈**：根据真实需求调整
3. **持续改进**：没有完美的 v1.0
4. **向后兼容**：保护现有用户

### 版本规划技巧

1. **语义化版本**：Major.Minor.Patch
2. **MoSCoW 优先级**：Must/Should/Could/Won't
3. **RICE 评分**：量化优先级
4. **路线图**：让团队知道方向

### 数据迁移要点

1. **小步迁移**：降低风险
2. **提供回滚**：出错时可恢复
3. **分阶段废弃**：给用户时间适应
4. **自动化工具**：使用 Alembic/migrate

---

**💡 记住**：好的产品是迭代出来的，不是一次开发完美的。v1.0 只是起点！

**推荐阅读**：
- [Semantic Versioning](https://semver.org/)
- [Alembic Documentation](https://alembic.sqlalchemy.org/)
- [golang-migrate](https://github.com/golang-migrate/migrate)
