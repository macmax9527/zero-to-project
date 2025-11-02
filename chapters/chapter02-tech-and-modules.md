# 第2章深度补充②：技术栈全景图 & 模块划分实战指南

> **本文目标**：让小白理解所有主流技术，掌握模块划分的底层逻辑

---

## 📖 目录

**Part 1: 技术栈全景图**
1. [前端技术完全解析](#part-1-技术栈全景图)
2. [后端技术完全解析](#12-后端技术完全解析)
3. [数据库技术完全解析](#13-数据库技术完全解析)
4. [移动端技术完全解析](#14-移动端技术完全解析)
5. [其他重要技术](#15-其他重要技术)
6. [技术选型决策树](#16-技术选型决策树)

**Part 2: 模块划分实战指南**
1. [模块的本质](#part-2-模块划分实战指南)
2. [通用模块详解](#22-通用模块详解)
3. [电商系统模块划分](#23-电商系统模块划分)
4. [博客系统模块划分](#24-博客系统模块划分)
5. [交易系统模块划分](#25-交易系统模块划分)
6. [社交APP模块划分](#26-社交app模块划分)
7. [新闻推送模块划分](#27-新闻推送系统模块划分)
8. [如何判断该不该拆分模块](#28-如何判断该不该拆分模块)

---

# Part 1: 技术栈全景图

## 1.1 前端技术完全解析

### 什么是前端？

**类比：餐厅的前厅**

```
前端 = 餐厅前厅
- 菜单（界面）
- 服务员（交互）
- 装修（样式）

后端 = 厨房
- 厨师（业务逻辑）
- 食材（数据）
- 配方（算法）

用户只看到前厅，但前厅背后是厨房在工作
```

**前端就是**：用户能看到和操作的部分

---

### 前端技术栈层次

```
┌──────────────────────────────────────┐
│  第3层：框架/库 (React/Vue/Angular)   │ ← 提高开发效率
├──────────────────────────────────────┤
│  第2层：JavaScript (逻辑)             │ ← 实现交互
├──────────────────────────────────────┤
│  第1层：HTML + CSS (结构+样式)        │ ← 最基础
└──────────────────────────────────────┘
```

---

### HTML（结构）

**是什么**：网页的骨架

**类比**：房子的框架（柱子、墙）

```html
<!-- HTML 示例 -->
<html>
  <head>
    <title>我的网站</title>
  </head>
  <body>
    <h1>欢迎来到我的网站</h1>
    <p>这是一段文字</p>
    <button>点击我</button>
  </body>
</html>
```

**用途**：
- 定义网页内容（标题、段落、按钮、图片）
- 网页的结构

**关键点**：
- ✅ 所有网页都需要 HTML
- ✅ 浏览器能直接理解
- ❌ 只有结构，没有样式和交互

---

### CSS（样式）

**是什么**：网页的外观

**类比**：房子的装修（油漆、壁纸、家具）

```css
/* CSS 示例 */
h1 {
  color: blue;          /* 蓝色 */
  font-size: 32px;      /* 字体大小 */
  text-align: center;   /* 居中 */
}

button {
  background-color: red; /* 红色背景 */
  padding: 10px 20px;    /* 内边距 */
  border-radius: 5px;    /* 圆角 */
}
```

**用途**：
- 控制颜色、字体、大小
- 控制布局（居中、左对齐、响应式）
- 动画效果

**关键点**：
- ✅ 让网页好看
- ✅ 响应式设计（手机、平板、电脑自适应）
- ❌ 只有样式，没有逻辑

---

### JavaScript（逻辑）

**是什么**：网页的行为

**类比**：房子的电器（灯开关、空调遥控器）

```javascript
// JavaScript 示例
button.addEventListener('click', function() {
  alert('你点击了按钮！');

  // 可以改变内容
  document.querySelector('h1').textContent = '标题变了！';

  // 可以改变样式
  document.querySelector('h1').style.color = 'red';

  // 可以发送请求
  fetch('/api/data')
    .then(response => response.json())
    .then(data => console.log(data));
});
```

**用途**：
- 响应用户操作（点击、输入）
- 动态改变页面内容
- 与后端通信（AJAX）
- 复杂逻辑（表单验证、计算）

**关键点**：
- ✅ 所有交互都需要 JavaScript
- ✅ 浏览器原生支持
- ✅ 可以操作 HTML 和 CSS

---

### React（前端框架）

**是什么**：用组件的方式构建界面的 JavaScript 库

**类比**：乐高积木

```
传统方式：
用 HTML + CSS + JS 一点点拼

React 方式：
用现成的积木（组件）拼
```

**示例**：

```jsx
// React 组件
function Button({ text, onClick }) {
  return (
    <button
      className="my-button"
      onClick={onClick}
    >
      {text}
    </button>
  );
}

// 使用组件（复用）
<Button text="提交" onClick={handleSubmit} />
<Button text="取消" onClick={handleCancel} />
<Button text="删除" onClick={handleDelete} />
```

**为什么用 React？**

```
❌ 传统方式的问题：
function updateUI() {
  document.getElementById('count').innerHTML = count;
  document.getElementById('message').innerHTML = message;
  // 手动操作 DOM，容易出错
  // 代码重复
}

✅ React 方式：
function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>计数：{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
  // 状态变化自动更新界面
  // 代码简洁
}
```

**优点**：
- ✅ 组件化（复用代码）
- ✅ 虚拟 DOM（性能好）
- ✅ 生态强大（库多）
- ✅ Facebook 维护

**缺点**：
- ❌ 学习曲线稍陡
- ❌ 需要编译（不能直接在浏览器运行）

**适用场景**：
- ✅ 复杂的单页应用（SPA）
- ✅ 交互多的网站
- ✅ 企业级应用
- ❌ 简单的静态网站（杀鸡用牛刀）

---

### Vue（前端框架）

**是什么**：渐进式 JavaScript 框架

**类比**：React 的简化版（更容易学）

```vue
<!-- Vue 组件 -->
<template>
  <div>
    <p>计数：{{ count }}</p>
    <button @click="count++">+1</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>
```

**React vs Vue 对比**：

| 特性 | React | Vue |
|------|-------|-----|
| 学习曲线 | 中等 | 简单 ⭐ |
| 生态 | 最强 ⭐ | 强 |
| 中文文档 | 一般 | 优秀 ⭐ |
| 公司支持 | Facebook | 个人+社区 |
| 灵活性 | 高 ⭐ | 中 |
| 适合 | 大型项目 | 中小型项目 ⭐ |

**何时选 Vue？**
- ✅ 团队经验少（Vue 更容易）
- ✅ 中文文档好（中国团队）
- ✅ 快速开发（语法简单）

**何时选 React？**
- ✅ 大型复杂项目
- ✅ 需要最强生态
- ✅ 国际化团队

---

### Angular（前端框架）

**是什么**：完整的前端框架（全家桶）

**类比**：全套西装（什么都有，但笨重）

**特点**：
- ✅ Google 维护
- ✅ TypeScript 原生支持
- ✅ 完整方案（路由、状态管理都内置）
- ❌ 学习曲线陡峭
- ❌ 笨重（文件大）

**何时用**：
- ✅ 企业级应用（特别是传统企业）
- ✅ 团队有 Java 背景（概念相似）
- ❌ 小项目（太重）

---

### Tailwind CSS（CSS 框架）

**是什么**：原子化 CSS 框架

**传统 CSS**：
```css
/* style.css */
.button {
  background-color: blue;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
}
```

```html
<button class="button">点击</button>
```

**Tailwind CSS**：
```html
<button class="bg-blue-500 text-white px-5 py-2 rounded">
  点击
</button>
```

**优点**：
- ✅ 不用写 CSS 文件
- ✅ 类名即样式（直观）
- ✅ 体积小（只打包用到的）

**缺点**：
- ❌ HTML 类名很长
- ❌ 需要记住类名

**何时用**：
- ✅ 快速开发
- ✅ 不想写 CSS
- ✅ NOFX 就用了 Tailwind

---

### 前端技术总结

```
入门必学：
1. HTML (1-2天)
2. CSS (3-5天)
3. JavaScript (2-4周)

进阶选择：
1. React (主流，生态强)
2. Vue (简单，中文好)
3. Angular (企业级)

辅助工具：
1. Tailwind CSS (快速写样式)
2. TypeScript (类型安全的 JS)
3. Vite (构建工具)
```

---

## 1.2 后端技术完全解析

### 什么是后端？

**后端就是**：处理业务逻辑、存储数据、提供 API 的部分

**用户看不到，但必须有**

---

### Python（后端语言）

**是什么**：简单易学的通用语言

**类比**：瑞士军刀（什么都能干）

```python
# Python 示例
@app.route('/api/users/<id>')
def get_user(id):
    user = database.query(f"SELECT * FROM users WHERE id={id}")
    return jsonify(user)
```

**优点**：
- ✅ 简单易学（语法接近自然语言）
- ✅ 库超级多（几乎什么都有现成的）
- ✅ 数据科学、AI 首选
- ✅ 快速开发

**缺点**：
- ❌ 性能一般（比 Go/Java 慢）
- ❌ 并发弱（GIL 锁）
- ❌ 部署麻烦（需要环境）

**适用场景**：
- ✅ 数据分析、机器学习
- ✅ 快速原型开发
- ✅ 脚本、自动化
- ✅ Web 后端（中小型）
- ❌ 高并发系统

**流行框架**：
- **Django**：全功能框架（自带管理后台）
- **Flask**：轻量框架（灵活）
- **FastAPI**：现代框架（性能好、自动生成文档）

---

### Go（后端语言）

**是什么**：Google 开发的高性能语言

**类比**：跑车（快、但需要学会开）

```go
// Go 示例
func getUser(c *gin.Context) {
    id := c.Param("id")
    var user User
    db.First(&user, id)
    c.JSON(200, user)
}
```

**优点**：
- ✅ 性能强（接近 C/C++）
- ✅ 并发友好（goroutine）
- ✅ 部署简单（编译成单文件）
- ✅ 类型安全（编译时检查）
- ✅ 标准库强大

**缺点**：
- ❌ 学习曲线（相比 Python）
- ❌ 生态不如 Python/Java 成熟
- ❌ 语法啰嗦（错误处理 if err != nil）

**适用场景**：
- ✅ 高并发服务（微服务）
- ✅ 网络编程
- ✅ 云原生应用
- ✅ API 服务
- ✅ **NOFX 用的就是 Go**

**为什么 NOFX 选 Go？**
1. 多 Trader 并发运行（goroutine 完美）
2. 性能好（处理市场数据快）
3. 部署简单（一个可执行文件）

---

### Node.js（后端运行时）

**是什么**：在服务器运行 JavaScript

**类比**：前端工程师也能写后端

```javascript
// Node.js 示例
app.get('/api/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user);
});
```

**优点**：
- ✅ 前后端同语言（JavaScript）
- ✅ 异步友好（适合 I/O 密集）
- ✅ npm 生态丰富
- ✅ 实时应用（WebSocket）

**缺点**：
- ❌ 单线程（CPU 密集型弱）
- ❌ 回调地狱（虽然有 async/await）
- ❌ 错误一个请求崩溃全服务

**适用场景**：
- ✅ 实时应用（聊天、游戏）
- ✅ API 服务（I/O 多）
- ✅ 前端工具链
- ❌ CPU 密集计算

**流行框架**：
- **Express**：老牌框架（简单）
- **Koa**：现代框架（优雅）
- **NestJS**：企业级框架（类似 Angular）

---

### Java（后端语言）

**是什么**：企业级后端首选

**类比**：坦克（笨重但稳）

```java
// Java 示例（Spring Boot）
@GetMapping("/api/users/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));
}
```

**优点**：
- ✅ 成熟稳定（30年历史）
- ✅ 生态最强（各种库）
- ✅ 企业级特性（事务、安全）
- ✅ 跨平台（JVM）
- ✅ 招人容易

**缺点**：
- ❌ 啰嗦（代码量大）
- ❌ 笨重（启动慢）
- ❌ 学习曲线陡峭
- ❌ 过度设计（各种抽象）

**适用场景**：
- ✅ 大型企业应用
- ✅ 金融、电商
- ✅ 安卓开发
- ❌ 小项目（杀鸡用牛刀）

**流行框架**：
- **Spring Boot**：最流行（全家桶）
- **Quarkus**：云原生（轻量）

---

### PHP（后端语言）

**是什么**：Web 开发老牌语言

**类比**：老爷车（能用但过时）

```php
// PHP 示例
<?php
function getUser($id) {
    $user = mysqli_query($conn, "SELECT * FROM users WHERE id=$id");
    return json_encode($user);
}
?>
```

**优点**：
- ✅ 简单易学
- ✅ 虚拟主机支持好
- ✅ WordPress 等 CMS
- ✅ 部署简单

**缺点**：
- ❌ 语言设计缺陷多
- ❌ 现代特性少
- ❌ 性能一般
- ❌ 逐渐被淘汰

**适用场景**：
- ✅ WordPress 网站
- ✅ 传统 Web 项目维护
- ❌ 新项目不推荐

---

### Rust（后端语言）

**是什么**：内存安全的系统级语言

**类比**：战斗机（极致性能，但难开）

**优点**：
- ✅ 性能极强（接近 C）
- ✅ 内存安全（无 GC）
- ✅ 并发安全
- ✅ 现代语言特性

**缺点**：
- ❌ 学习曲线陡峭（最难）
- ❌ 编译慢
- ❌ 生态不成熟
- ❌ 开发慢

**适用场景**：
- ✅ 系统编程
- ✅ 高性能服务
- ✅ 区块链
- ❌ 普通 Web 开发（杀鸡用牛刀）

---

### 后端语言对比总结

| 语言 | 学习难度 | 性能 | 生态 | 适合场景 | 推荐指数 |
|------|---------|------|------|----------|----------|
| **Python** | ⭐ 简单 | ⭐⭐ 中 | ⭐⭐⭐ 强 | 数据/AI/快速开发 | ⭐⭐⭐⭐ |
| **Go** | ⭐⭐ 中等 | ⭐⭐⭐⭐ 强 | ⭐⭐⭐ 中 | 微服务/高并发 | ⭐⭐⭐⭐⭐ |
| **Node.js** | ⭐ 简单 | ⭐⭐⭐ 中上 | ⭐⭐⭐⭐ 强 | 实时应用 | ⭐⭐⭐⭐ |
| **Java** | ⭐⭐⭐ 难 | ⭐⭐⭐⭐ 强 | ⭐⭐⭐⭐⭐ 最强 | 企业级 | ⭐⭐⭐⭐ |
| **PHP** | ⭐ 简单 | ⭐⭐ 弱 | ⭐⭐⭐ 中 | WordPress | ⭐⭐ |
| **Rust** | ⭐⭐⭐⭐⭐ 最难 | ⭐⭐⭐⭐⭐ 最强 | ⭐⭐ 弱 | 系统级 | ⭐⭐⭐ |

---

## 1.3 数据库技术完全解析

### 什么是数据库？

**类比**：图书馆

```
数据库 = 图书馆
- 书架 = 表（Table）
- 书 = 记录（Row）
- 书的属性 = 字段（Column）
- 索引 = 书的目录
```

---

### 关系型数据库 vs 非关系型数据库

```
关系型数据库（SQL）：
- 数据有固定结构（表格）
- 支持复杂查询
- 支持事务

非关系型数据库（NoSQL）：
- 数据结构灵活
- 查询相对简单
- 性能更好（通常）
```

---

### MySQL（关系型数据库）

**是什么**：最流行的开源数据库

**类比**：Excel 表格（但更强大）

```sql
-- MySQL 示例
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP
);

-- 插入数据
INSERT INTO users (name, email) VALUES ('张三', 'zhang@example.com');

-- 查询数据
SELECT * FROM users WHERE name = '张三';

-- 关联查询
SELECT orders.*, users.name
FROM orders
JOIN users ON orders.user_id = users.id;
```

**优点**：
- ✅ 成熟稳定（30年）
- ✅ 生态丰富
- ✅ 支持复杂查询
- ✅ 事务支持
- ✅ 免费

**缺点**：
- ❌ 大数据量性能下降
- ❌ 水平扩展难

**适用场景**：
- ✅ 电商（订单、用户）
- ✅ 后台管理系统
- ✅ 绝大多数 Web 应用
- ❌ 大数据分析

---

### PostgreSQL（关系型数据库）

**是什么**：最先进的开源数据库

**类比**：MySQL 的增强版

**优点**：
- ✅ 功能最强（窗口函数、全文搜索）
- ✅ 性能好
- ✅ 数据一致性强
- ✅ 支持 JSON

**缺点**：
- ❌ 稍微复杂
- ❌ 资源占用多

**何时选 PostgreSQL 而不是 MySQL**：
- ✅ 需要复杂查询
- ✅ 需要 JSON 支持
- ✅ 数据一致性要求高

---

### MongoDB（非关系型数据库）

**是什么**：文档型数据库

**类比**：JSON 文件数据库

```javascript
// MongoDB 示例
// 插入数据（JSON 格式）
db.users.insertOne({
  name: "张三",
  email: "zhang@example.com",
  address: {
    city: "北京",
    street: "朝阳路123号"
  },
  hobbies: ["游泳", "看书"]
});

// 查询
db.users.find({ name: "张三" });

// 更新
db.users.updateOne(
  { name: "张三" },
  { $set: { email: "new@example.com" } }
);
```

**优点**：
- ✅ 灵活（不需要固定表结构）
- ✅ 易于扩展（水平扩展）
- ✅ 性能好（简单查询）
- ✅ JSON 原生支持

**缺点**：
- ❌ 不支持关联查询（JOIN）
- ❌ 不支持事务（早期版本）
- ❌ 数据一致性弱

**适用场景**：
- ✅ 内容管理（博客、文章）
- ✅ 日志存储
- ✅ 实时分析
- ✅ 快速原型
- ❌ 金融（需要强一致性）

---

### Redis（缓存数据库）

**是什么**：内存数据库（超快）

**类比**：记事本（临时快速记录）

```python
# Redis 示例
# 设置缓存（键值对）
redis.set("user:1", json.dumps(user))

# 获取缓存
user_json = redis.get("user:1")

# 设置过期时间（60秒后自动删除）
redis.setex("session:abc123", 60, session_data)

# 列表操作（消息队列）
redis.lpush("tasks", task1)
redis.rpop("tasks")  # 取出任务
```

**优点**：
- ✅ 极快（内存操作）
- ✅ 多种数据结构（字符串、列表、集合、哈希）
- ✅ 发布订阅
- ✅ 分布式锁

**缺点**：
- ❌ 内存限制（贵）
- ❌ 数据丢失风险（断电）

**适用场景**：
- ✅ 缓存（最常用）
- ✅ 会话存储
- ✅ 排行榜
- ✅ 消息队列（简单）
- ✅ 限流

**NOFX 可以用 Redis**：
```
问题：频繁调用 Binance API 获取 K 线
      → API 限流
      → 慢

解决：Redis 缓存 K 线数据
      → 5 秒内重复请求直接从缓存拿
      → 快，不会被限流
```

---

### SQLite（嵌入式数据库）

**是什么**：文件数据库（无需服务器）

**类比**：Excel 文件（但支持 SQL）

```python
# SQLite 示例
import sqlite3

conn = sqlite3.connect('myapp.db')  # 就是一个文件
cursor = conn.cursor()

cursor.execute('CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)')
cursor.execute('INSERT INTO users (name) VALUES ("张三")')
conn.commit()
```

**优点**：
- ✅ 零配置（不需要服务器）
- ✅ 单文件（方便备份）
- ✅ 轻量
- ✅ 移动端支持

**缺点**：
- ❌ 并发写入弱
- ❌ 不适合大数据

**适用场景**：
- ✅ 移动 APP
- ✅ 桌面应用
- ✅ 小网站
- ✅ 测试
- ❌ 高并发 Web

---

### 数据库选择决策树

```
需要关系型吗？（有复杂关联查询）
├─ 是 → MySQL 或 PostgreSQL
│        ├─ 简单项目 → MySQL
│        └─ 复杂查询/JSON → PostgreSQL
│
└─ 否 → 考虑 NoSQL
         ├─ 需要灵活结构 → MongoDB
         ├─ 需要极速读取 → Redis（缓存）
         └─ 单机小项目 → SQLite

实际项目常用组合：
- MySQL/PostgreSQL（主数据）+ Redis（缓存）
- MongoDB（文档）+ Redis（缓存）
```

---

## 1.4 移动端技术完全解析

### 移动端开发方式对比

```
原生开发：
- iOS: Swift/Objective-C
- Android: Kotlin/Java
优点：性能最好
缺点：要写两套代码

跨平台开发：
- React Native
- Flutter
优点：一套代码，两个平台
缺点：性能略差
```

---

### Swift（iOS 原生）

**是什么**：苹果开发的 iOS 语言

**类比**：只能开苹果车的驾照

```swift
// Swift 示例
struct ContentView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("计数：\(count)")
            Button("增加") {
                count += 1
            }
        }
    }
}
```

**优点**：
- ✅ 性能最好
- ✅ 苹果官方支持
- ✅ 最新 iOS 功能优先

**缺点**：
- ❌ 只能开发 iOS
- ❌ 需要 Mac 电脑
- ❌ 学习成本高

**何时用**：
- ✅ 纯 iOS 应用
- ✅ 性能要求极高
- ✅ 需要最新 iOS 特性

---

### Kotlin（Android 原生）

**是什么**：Google 推荐的 Android 语言

**优点**：
- ✅ 现代语言（比 Java 好）
- ✅ Android 官方推荐
- ✅ 性能好

**缺点**：
- ❌ 只能开发 Android

---

### React Native（跨平台）

**是什么**：用 React 写移动应用

**类比**：用网页技术写 APP

```jsx
// React Native 示例
function App() {
  const [count, setCount] = useState(0);

  return (
    <View>
      <Text>计数：{count}</Text>
      <Button
        title="增加"
        onPress={() => setCount(count + 1)}
      />
    </View>
  );
}
```

**优点**：
- ✅ 一套代码，iOS + Android
- ✅ 热更新（不用提交审核）
- ✅ 前端技能复用（React）
- ✅ Facebook 支持

**缺点**：
- ❌ 性能不如原生（但够用）
- ❌ 依赖原生模块（有些功能需要原生实现）

**何时用**：
- ✅ 快速开发
- ✅ 预算有限（一套代码）
- ✅ 团队有前端经验

---

### Flutter（跨平台）

**是什么**：Google 的跨平台框架

**优点**：
- ✅ 性能好（接近原生）
- ✅ UI 美观（Material Design）
- ✅ 一套代码多平台（iOS、Android、Web）

**缺点**：
- ❌ Dart 语言（需要学新语言）
- ❌ 包大小大

**React Native vs Flutter**：

| 特性 | React Native | Flutter |
|------|-------------|---------|
| 语言 | JavaScript | Dart |
| 性能 | 中上 | 接近原生 |
| 学习曲线 | 低（前端可复用） | 中（新语言） |
| 生态 | 强 | 成长中 |
| 热更新 | 支持 | 部分支持 |

**何时选 Flutter**：
- ✅ 性能要求高
- ✅ UI 要求高
- ✅ 需要支持 Web

---

## 1.5 其他重要技术

### WordPress（CMS 系统）

**是什么**：内容管理系统（不需要写代码）

**类比**：搭积木建网站

**优点**：
- ✅ 不需要编程（拖拽建站）
- ✅ 插件超多（几乎什么功能都有）
- ✅ 主题丰富
- ✅ SEO 友好

**缺点**：
- ❌ 性能一般
- ❌ 安全性（插件漏洞多）
- ❌ 灵活性差

**适用场景**：
- ✅ 博客
- ✅ 企业官网
- ✅ 新闻网站
- ❌ 复杂应用

---

### Docker（容器技术）

**是什么**：打包应用和环境

**类比**：集装箱（把应用和环境一起打包）

```dockerfile
# Dockerfile 示例
FROM python:3.9
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

**解决什么问题**：

```
❌ 传统部署：
开发环境能跑，生产环境报错
"在我电脑上能跑啊！"

✅ Docker：
把代码+环境打包成镜像
哪里都能跑
```

**何时用**：
- ✅ 部署（必用）
- ✅ 开发环境统一
- ✅ 微服务

---

### Nginx（Web 服务器）

**是什么**：高性能 Web 服务器

**作用**：
1. **反向代理**（转发请求）
2. **负载均衡**（分发流量）
3. **静态文件服务**（前端文件）

```
用户请求
   ↓
Nginx (端口 80)
   ├→ 后端1 (端口 8001)
   ├→ 后端2 (端口 8002)
   └→ 后端3 (端口 8003)
```

**何时用**：
- ✅ 生产环境（必用）
- ✅ HTTPS 配置
- ✅ 静态文件服务

---

### Git（版本控制）

**是什么**：代码版本管理

**类比**：Word 的修订历史（但强大得多）

**核心概念**：
```
工作区 → git add → 暂存区 → git commit → 本地仓库 → git push → 远程仓库

分支：
main ────────────────→
  \
   feature-login ────→ 合并回 main
```

**何时用**：
- ✅ 所有项目（必用）

---

## 1.6 技术选型决策树

### 决策树：我应该用什么技术？

#### 前端技术选型

```
你要做什么类型的网站？

1. 简单静态网站（几个页面）
   → HTML + CSS + 少量 JS
   → 或者用 WordPress

2. 交互复杂的网站（后台管理、电商）
   → React（生态强，大项目）
   → Vue（简单，中小项目）

3. 移动APP
   → React Native（前端团队）
   → Flutter（性能要求高）
   → 原生（iOS/Android）

4. 需要 SEO 的网站（新闻、博客）
   → Next.js（React SSR）
   → Nuxt.js（Vue SSR）
```

#### 后端技术选型

```
考虑因素：

1. 团队技能
   ├─ 有 Python 经验 → Python (FastAPI/Django)
   ├─ 有 JS 经验 → Node.js (Express)
   └─ 学新的 → Go（推荐）

2. 项目类型
   ├─ 快速原型 → Python
   ├─ 高并发 → Go / Java
   ├─ 实时应用 → Node.js
   ├─ 企业级 → Java
   └─ 数据/AI → Python

3. 性能要求
   ├─ < 100 QPS → 任意
   ├─ 100-1000 QPS → Python/Node/Go
   └─ > 1000 QPS → Go/Java/Rust

推荐组合：
- 初创团队：Python + PostgreSQL + Redis
- 中型团队：Go + PostgreSQL + Redis
- 大型企业：Java + MySQL + Redis + Kafka
```

#### 数据库选型

```
数据类型：

1. 结构化数据（用户、订单、商品）
   → MySQL（简单项目）
   → PostgreSQL（复杂查询）

2. 文档型数据（文章、日志）
   → MongoDB

3. 缓存
   → Redis（必用）

4. 时序数据（监控、日志）
   → InfluxDB / TimescaleDB

5. 全文搜索
   → Elasticsearch

实际组合：
- 电商：MySQL + Redis + Elasticsearch
- 博客：PostgreSQL + Redis
- 大数据：MongoDB + Redis + Elasticsearch
```

---

### 常见项目技术栈推荐

#### 1. 博客/个人网站

```
前端：Next.js (React) 或 WordPress
后端：Node.js / Python
数据库：PostgreSQL
部署：Vercel / Netlify
```

#### 2. 电商网站

```
前端：React + Tailwind CSS
后端：Go / Java (Spring Boot)
数据库：MySQL + Redis + Elasticsearch
支付：Stripe / 支付宝
部署：AWS / 阿里云
```

#### 3. 社交APP

```
前端：React Native / Flutter
后端：Go (微服务)
数据库：PostgreSQL + MongoDB + Redis
实时：WebSocket / Socket.io
CDN：七牛云 / OSS
```

#### 4. SaaS 平台

```
前端：React + TypeScript
后端：Go / Java
数据库：PostgreSQL + Redis
队列：RabbitMQ / Kafka
部署：Kubernetes
```

#### 5. 数据分析平台

```
前端：React + ECharts
后端：Python (FastAPI)
数据库：PostgreSQL + ClickHouse
数据处理：Pandas / Spark
可视化：Tableau / Grafana
```

---

# Part 2: 模块划分实战指南

## 2.1 模块的本质

### 什么是模块？

**类比 1：公司的部门**

```
公司（软件系统）
├── 人事部（用户模块）
│   └── 负责：招聘、员工管理
├── 财务部（支付模块）
│   └── 负责：报销、工资
├── 销售部（订单模块）
│   └── 负责：销售、客户
└── IT部（基础设施模块）
    └── 负责：服务器、网络

每个部门：
- 有明确职责
- 独立运作
- 通过流程沟通（接口）
```

**类比 2：乐高积木**

```
大城堡（项目）由小积木（模块）组成
- 每块积木有固定接口（凸起和凹槽）
- 可以单独制造
- 可以替换
- 可以复用
```

---

### 为什么要拆分模块？

#### 场景：做一个电商网站

**❌ 不拆分模块（所有代码混在一起）**：

```python
# app.py (5000行代码)
def main():
    # 用户登录
    user = authenticate(username, password)

    # 添加商品到购物车
    cart = add_to_cart(user_id, product_id)

    # 计算价格
    total = calculate_price(cart)

    # 处理支付
    payment = process_payment(total)

    # 创建订单
    order = create_order(user, cart, payment)

    # 更新库存
    update_inventory(order)

    # 发送邮件
    send_email(user.email, order_details)

    # 生成发票
    generate_invoice(order)

    # ... 更多功能
```

**问题**：
1. **一个文件太长**：5000 行代码，翻到哪都不知道
2. **职责不清**：登录、支付、库存、邮件都混在一起
3. **难以维护**：改一个地方可能影响其他地方
4. **无法复用**：其他项目想用支付功能？复制粘贴代码？
5. **测试困难**：必须启动整个系统
6. **多人协作难**：3 个人都在改同一个文件

---

**✅ 拆分模块（按职责组织）**：

```
ecommerce/
├── user/                    # 用户模块
│   ├── auth.py              # 认证
│   ├── profile.py           # 个人资料
│   └── permissions.py       # 权限
│
├── product/                 # 商品模块
│   ├── catalog.py           # 商品目录
│   ├── search.py            # 搜索
│   └── inventory.py         # 库存
│
├── cart/                    # 购物车模块
│   └── cart_service.py
│
├── order/                   # 订单模块
│   ├── order_service.py     # 订单处理
│   └── invoice.py           # 发票
│
├── payment/                 # 支付模块
│   ├── payment_gateway.py   # 支付网关
│   └── refund.py            # 退款
│
└── notification/            # 通知模块
    ├── email.py             # 邮件
    └── sms.py               # 短信
```

**优势**：
- ✅ 每个模块职责单一（好找）
- ✅ 可以独立开发（3 个人分别做 user、product、order）
- ✅ 可以复用（payment 模块可以用在其他项目）
- ✅ 容易测试（单独测试 payment 模块）
- ✅ 容易维护（改支付只看 payment 目录）

---

### 模块划分的原则

#### 原则 1：单一职责

**一个模块只做一件事**

```
❌ 坏例子：
utils/  # 工具模块
├── email.py          # 发邮件
├── payment.py        # 处理支付
├── image.py          # 图片处理
├── auth.py           # 认证
└── database.py       # 数据库

问题：什么都有，等于没有分类

✅ 好例子：
user/         # 用户相关
payment/      # 支付相关
media/        # 媒体相关
notification/ # 通知相关
```

#### 原则 2：高内聚

**相关的东西放在一起**

```
❌ 低内聚：
project/
├── models/
│   ├── user.py
│   └── order.py
├── services/
│   ├── user_service.py
│   └── order_service.py
└── controllers/
    ├── user_controller.py
    └── order_controller.py

问题：修改用户功能要改 3 个目录

✅ 高内聚：
project/
├── user/
│   ├── model.py
│   ├── service.py
│   └── controller.py
└── order/
    ├── model.py
    ├── service.py
    └── controller.py

优势：改用户功能只看 user/ 目录
```

#### 原则 3：低耦合

**模块间依赖尽量少**

```
❌ 高耦合：
order 模块直接调用 user 模块内部函数
order 模块直接访问 user 模块的数据库

问题：user 模块改了，order 模块也要改

✅ 低耦合：
order 模块通过接口调用 user 模块
user 模块对外提供统一 API
```

---

## 2.2 通用模块详解

### 几乎所有项目都有的模块

```
任何 Web 应用基本都有：
1. 用户模块 (User)
2. 认证模块 (Auth)
3. 配置模块 (Config)
4. 日志模块 (Logger)
5. 数据库模块 (Database)
6. API 模块 (API)
```

---

### 模块 1：用户模块 (User)

**职责**：
- 用户注册
- 用户资料管理
- 用户查询

**为什么需要**：
几乎所有应用都有"用户"概念

**典型文件结构**：

```
user/
├── models.py           # 数据模型
│   └── class User:
│           id
│           username
│           email
│           password_hash
│           created_at
│
├── service.py          # 业务逻辑
│   └── def create_user(username, email, password)
│       def get_user_by_id(id)
│       def update_profile(user_id, data)
│
├── repository.py       # 数据访问
│   └── def save_user(user)
│       def find_user_by_email(email)
│
└── api.py              # API 接口
    └── POST /api/users          # 创建用户
        GET  /api/users/:id      # 获取用户
        PUT  /api/users/:id      # 更新用户
```

---

### 模块 2：认证模块 (Auth)

**职责**：
- 登录
- 退出
- Token 管理
- 权限检查

**为什么独立？**
- 认证逻辑复杂（JWT、Session、OAuth）
- 安全敏感（单独审计）
- 多处使用（复用）

**典型文件结构**：

```
auth/
├── login.py
│   └── def login(username, password):
│           验证密码
│           生成 Token
│           return token
│
├── jwt.py              # JWT Token
│   └── def generate_token(user_id)
│       def verify_token(token)
│
├── permissions.py      # 权限
│   └── def check_permission(user, resource)
│
└── middleware.py       # 中间件
    └── def auth_required(func):  # 装饰器
            验证 Token
            提取用户信息
```

---

### 模块 3：配置模块 (Config)

**职责**：
- 加载配置文件
- 环境变量管理
- 配置验证

**为什么需要**：
所有环境相关的参数集中管理

**典型文件结构**：

```
config/
├── config.py
│   └── class Config:
│           DATABASE_URL
│           SECRET_KEY
│           DEBUG
│
├── development.py      # 开发环境
├── production.py       # 生产环境
└── test.py             # 测试环境
```

---

### 模块 4：数据库模块 (Database)

**职责**：
- 数据库连接
- ORM 配置
- 迁移管理

**典型文件结构**：

```
database/
├── connection.py       # 连接池
├── models.py           # 基础模型
└── migrations/         # 数据库迁移
    ├── 001_create_users_table.sql
    └── 002_add_email_to_users.sql
```

---

## 2.3 电商系统模块划分

### 完整的电商系统架构

```
ecommerce/
├── user/               # 👤 用户模块
│   ├── auth/           # 认证登录
│   ├── profile/        # 个人资料
│   └── address/        # 收货地址
│
├── product/            # 📦 商品模块
│   ├── catalog/        # 商品目录
│   ├── category/       # 分类
│   ├── search/         # 搜索
│   └── inventory/      # 库存管理
│
├── cart/               # 🛒 购物车模块
│   └── cart_service.py
│
├── order/              # 📋 订单模块
│   ├── order_service/  # 订单处理
│   ├── order_status/   # 订单状态
│   └── invoice/        # 发票
│
├── payment/            # 💳 支付模块
│   ├── gateway/        # 支付网关（支付宝、微信）
│   ├── refund/         # 退款
│   └── transaction/    # 交易记录
│
├── shipping/           # 🚚 物流模块
│   ├── shipping_method/# 配送方式
│   ├── tracking/       # 物流追踪
│   └── address/        # 地址验证
│
├── review/             # ⭐ 评价模块
│   ├── product_review/
│   └── seller_review/
│
├── promotion/          # 🎁 促销模块
│   ├── coupon/         # 优惠券
│   ├── discount/       # 折扣
│   └── campaign/       # 营销活动
│
├── notification/       # 📧 通知模块
│   ├── email/
│   ├── sms/
│   └── push/
│
├── analytics/          # 📊 数据分析模块
│   ├── sales_report/   # 销售报表
│   ├── user_behavior/  # 用户行为
│   └── inventory_report/ # 库存报表
│
└── admin/              # 🔧 后台管理模块
    ├── dashboard/      # 仪表盘
    ├── user_management/
    └── product_management/
```

---

### 为什么要这些模块？

#### 用户模块 (User)

**为什么需要**：
- 用户注册、登录
- 管理收货地址
- 查看订单历史

**核心功能**：
```python
# 用户注册
POST /api/users/register
{
    "username": "zhangsan",
    "email": "zhang@example.com",
    "password": "123456"
}

# 添加收货地址
POST /api/users/addresses
{
    "name": "张三",
    "phone": "13800138000",
    "address": "北京市朝阳区xxx"
}
```

---

#### 商品模块 (Product)

**为什么需要**：
- 展示商品信息
- 商品搜索
- 库存管理

**核心功能**：
```python
# 获取商品列表
GET /api/products?category=手机&page=1

# 搜索商品
GET /api/products/search?q=iPhone

# 获取商品详情
GET /api/products/123
{
    "id": 123,
    "name": "iPhone 15",
    "price": 5999,
    "stock": 100,
    "images": [...],
    "description": "..."
}
```

**为什么库存在商品模块**：
- 库存是商品的属性
- 商品和库存紧密关联
- 查询商品时需要知道库存

---

#### 购物车模块 (Cart)

**为什么独立**：
- 购物车逻辑复杂（价格计算、优惠券）
- 临时数据（用户可能不下单）
- 高频操作（频繁修改）

**核心功能**：
```python
# 添加到购物车
POST /api/cart/items
{
    "product_id": 123,
    "quantity": 2
}

# 查看购物车
GET /api/cart
{
    "items": [
        {
            "product_id": 123,
            "name": "iPhone 15",
            "price": 5999,
            "quantity": 2,
            "subtotal": 11998
        }
    ],
    "total": 11998
}

# 更新数量
PUT /api/cart/items/123
{
    "quantity": 3
}
```

**为什么不放在订单模块**：
- 购物车 ≠ 订单（购物车可能不结账）
- 购物车可以随意修改，订单不能

---

#### 订单模块 (Order)

**为什么需要**：
- 记录用户购买
- 追踪订单状态
- 生成发票

**核心功能**：
```python
# 创建订单（从购物车）
POST /api/orders
{
    "cart_id": "abc123",
    "address_id": 456,
    "payment_method": "alipay"
}

# 订单状态流转
待支付 → 待发货 → 待收货 → 已完成

# 查询订单
GET /api/orders/789
{
    "order_id": 789,
    "status": "待发货",
    "items": [...],
    "total": 11998,
    "address": {...},
    "created_at": "2024-01-15 10:30:00"
}
```

---

#### 支付模块 (Payment)

**为什么独立**：
- 支持多种支付方式（支付宝、微信、银行卡）
- 安全敏感（单独审计）
- 可复用（其他项目也能用）

**核心功能**：
```python
# 创建支付
POST /api/payments
{
    "order_id": 789,
    "amount": 11998,
    "method": "alipay"
}
→ 返回支付链接

# 支付回调（支付宝通知）
POST /api/payments/callback
{
    "trade_no": "xxx",
    "status": "success"
}
→ 更新订单状态为"待发货"
```

**为什么不放在订单模块**：
- 支付逻辑复杂（各种支付方式）
- 支付可能失败需要重试
- 支付模块可以给其他业务用（充值、会员）

---

#### 物流模块 (Shipping)

**为什么需要**：
- 选择配送方式
- 追踪物流
- 计算运费

**核心功能**：
```python
# 获取配送方式
GET /api/shipping/methods
[
    {"id": 1, "name": "顺丰速运", "price": 15, "days": "1-2天"},
    {"id": 2, "name": "中通快递", "price": 8, "days": "3-5天"}
]

# 查询物流
GET /api/shipping/track/SF1234567890
{
    "status": "运输中",
    "timeline": [
        {"time": "2024-01-15 10:00", "desc": "已揽收"},
        {"time": "2024-01-15 14:00", "desc": "到达北京分拨中心"},
        {"time": "2024-01-16 08:00", "desc": "派送中"}
    ]
}
```

---

#### 评价模块 (Review)

**为什么独立**：
- 评价数据量大
- 需要审核（防止恶意评价）
- 可以给推荐系统用

```python
# 发表评价
POST /api/reviews
{
    "product_id": 123,
    "rating": 5,
    "content": "非常好用！",
    "images": [...]
}

# 查看商品评价
GET /api/products/123/reviews
```

---

#### 促销模块 (Promotion)

**为什么独立**：
- 促销活动复杂（满减、折扣、优惠券）
- 计算逻辑复杂
- 经常变化（活动更新频繁）

```python
# 使用优惠券
POST /api/cart/apply-coupon
{
    "coupon_code": "NEW2024"
}
→ 减免 100 元

# 活动商品
GET /api/promotions/double11
→ 返回参与双11的商品
```

---

### 电商模块依赖关系

```
                    [用户]
                      ↓
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       [商品]      [购物车]    [订单]
          ↑           ↓           ↓
          │      [促销模块]   [支付模块]
          │                       ↓
          │                   [物流模块]
          │                       ↓
          └────────────────→  [评价]

说明：
- 箭头表示依赖方向
- 购物车依赖商品（需要商品信息）
- 订单依赖购物车、支付、物流
- 评价依赖订单（必须购买后才能评价）
```

---

## 2.4 博客系统模块划分

### 完整的博客系统架构

```
blog/
├── user/               # 👤 用户模块
│   ├── auth/           # 认证
│   ├── profile/        # 个人主页
│   └── follow/         # 关注/粉丝
│
├── article/            # 📝 文章模块
│   ├── editor/         # 编辑器
│   ├── draft/          # 草稿箱
│   ├── publish/        # 发布
│   └── category/       # 分类
│
├── comment/            # 💬 评论模块
│   ├── comment_service/
│   └── reply/          # 回复
│
├── like/               # ❤️ 点赞模块
│   └── like_service/
│
├── tag/                # 🏷️ 标签模块
│   └── tag_service/
│
├── search/             # 🔍 搜索模块
│   ├── article_search/
│   └── user_search/
│
├── media/              # 🖼️ 媒体模块
│   ├── image_upload/
│   └── file_manager/
│
├── notification/       # 🔔 通知模块
│   └── notification_service/
│
└── analytics/          # 📊 统计模块
    ├── view_count/     # 浏览量
    └── trending/       # 热门文章
```

---

### 为什么博客系统需要这些模块？

#### 文章模块 (Article)

**核心功能**：
```python
# 创建文章
POST /api/articles
{
    "title": "我的第一篇博客",
    "content": "...",
    "category": "技术",
    "tags": ["Python", "Web"],
    "status": "draft"  # 草稿
}

# 发布文章
PUT /api/articles/123/publish

# 获取文章列表
GET /api/articles?category=技术&page=1
```

**为什么需要草稿功能**：
- 用户可能写一半保存
- 发布前需要预览

---

#### 评论模块 (Comment)

**为什么独立**：
- 评论有复杂的嵌套（评论的回复）
- 需要审核（防止垃圾评论）
- 数据量大（可能比文章多10倍）

```python
# 评论结构（支持嵌套）
{
    "article_id": 123,
    "comments": [
        {
            "id": 1,
            "user": "张三",
            "content": "写得好！",
            "replies": [
                {
                    "id": 2,
                    "user": "李四",
                    "content": "同意！"
                }
            ]
        }
    ]
}
```

---

#### 标签模块 (Tag)

**为什么独立**：
- 标签可以给文章分类
- 用户可以关注标签
- 标签有统计（多少文章）

```python
# 热门标签
GET /api/tags/popular
[
    {"name": "Python", "count": 1234},
    {"name": "React", "count": 890}
]

# 通过标签找文章
GET /api/tags/Python/articles
```

---

## 2.5 交易系统模块划分

### NOFX 的模块划分（实际案例）

```
nofx/
├── config/             # ⚙️ 配置模块
│   └── config.go       # 加载 config.json
│
├── trader/             # 💱 交易执行模块
│   ├── interface.go    # Trader 接口
│   ├── binance_futures.go
│   ├── hyperliquid_trader.go
│   ├── aster_trader.go
│   └── auto_trader.go  # 自动交易主控
│
├── market/             # 📊 市场数据模块
│   └── data.go         # K线、技术指标
│
├── pool/               # 🎲 币种池模块
│   └── coin_pool.go    # 候选币种筛选
│
├── mcp/                # 🤖 AI 通信模块
│   └── client.go       # DeepSeek/Qwen API
│
├── decision/           # 🧠 决策引擎模块
│   └── engine.go       # 提示词构建、解析
│
├── logger/             # 📝 日志模块
│   └── decision_logger.go  # 决策记录、性能追踪
│
├── manager/            # 👔 管理模块
│   └── trader_manager.go   # 管理多个 Trader
│
├── api/                # 🌐 API 服务模块
│   └── server.go       # Gin HTTP 服务器
│
└── main.go             # 🎯 入口
```

---

### 为什么 NOFX 需要这些模块？

#### trader 模块（交易执行）

**为什么独立**：
- 支持多个交易所（Binance、Hyperliquid、Aster）
- 交易逻辑复杂（开仓、平仓、止损止盈）
- 接口统一（通过 Trader interface）

**职责**：
```go
type Trader interface {
    GetAccount() (Account, error)      // 获取账户
    GetPositions() ([]Position, error) // 获取持仓
    OpenLong(...)                      // 开多
    CloseLong(...)                     // 平多
}
```

---

#### market 模块（市场数据）

**为什么独立**：
- 数据来源复杂（不同交易所 API 不同）
- 需要计算技术指标（RSI、MACD、EMA）
- 数据可以缓存（减少 API 调用）

```go
// 获取市场数据
data := market.FetchData("BTCUSDT", "binance")

// 包含完整指标
data.RSI3m      // 3分钟 RSI
data.MACD3m     // 3分钟 MACD
data.EMA20_4h   // 4小时 EMA20
```

---

#### decision 模块（决策引擎）

**为什么独立**：
- 决策逻辑复杂（构建提示词）
- AI 模型可能更换（DeepSeek → Qwen）
- 提示词需要经常调整

```go
// 决策流程
func GetFullDecision(ctx *Context) (*FullDecision, error) {
    // 1. 构建提示词
    systemPrompt := buildSystemPrompt()
    userPrompt := buildUserPrompt(ctx)

    // 2. 调用 AI
    response := mcpClient.Chat(systemPrompt, userPrompt)

    // 3. 解析决策
    decisions := parseDecisions(response)

    return decisions, nil
}
```

---

#### logger 模块（日志和性能）

**为什么独立**：
- 记录完整决策过程（调试）
- 计算性能指标（胜率、盈亏比）
- 提供历史反馈（AI 自学习）

**数据流**：
```
决策 → logger.LogDecision() → JSON 文件
平仓 → logger.RecordClose() → 计算盈亏 → 更新统计
查询 → logger.GetPerformance() → 返回胜率、盈亏比
```

---

#### manager 模块（多实例管理）

**为什么需要**：
- 支持多个 Trader 同时运行（竞赛模式）
- 统一管理生命周期（启动、停止）
- 提供查询接口（给 API 用）

```go
// 创建管理器
manager := NewTraderManager()

// 添加 Trader
manager.AddTrader(config1)
manager.AddTrader(config2)

// 启动所有
manager.StartAll()

// API 查询
trader := manager.GetTrader("trader1")
```

---

### 为什么不把某些模块合并？

#### 为什么 decision 和 trader 分开？

```
❌ 如果合并：
trader/auto_trader.go
├── 交易执行逻辑（500行）
├── AI 决策逻辑（500行）
└── 提示词构建（500行）
→ 1500 行代码，太长

✅ 分开后：
trader/auto_trader.go（500行）
  └── 只管交易执行
decision/engine.go（500行）
  └── 只管 AI 决策
```

**好处**：
- 每个文件更短（易读）
- 职责清晰
- 修改提示词不影响交易逻辑

---

#### 为什么 pool（币种池）独立？

```
如果放在 decision 里：
decision/engine.go
├── 构建提示词
├── 调用 AI
├── 获取币种池  ← 混在一起

如果独立：
pool/coin_pool.go
  └── 专门管理币种筛选

decision/engine.go
  └── 调用 pool.GetCoinPool()
```

**好处**：
- 币种池逻辑可能换（AI500 → 其他来源）
- decision 模块更简单
- pool 可以给其他地方用

---

## 2.6 社交APP模块划分

### 完整的社交APP架构

```
social-app/
├── user/               # 👤 用户模块
│   ├── profile/        # 个人资料
│   ├── settings/       # 设置
│   └── privacy/        # 隐私设置
│
├── auth/               # 🔐 认证模块
│   ├── login/
│   ├── oauth/          # 第三方登录（微信、QQ）
│   └── session/
│
├── post/               # 📱 动态模块
│   ├── create_post/
│   ├── feed/           # 动态流
│   └── timeline/       # 时间线
│
├── follow/             # 👥 关注模块
│   ├── follow_service/
│   ├── follower/       # 粉丝
│   └── following/      # 关注的人
│
├── like/               # ❤️ 点赞模块
│   └── like_service/
│
├── comment/            # 💬 评论模块
│   └── comment_service/
│
├── message/            # 💌 私信模块
│   ├── chat/           # 聊天
│   ├── group_chat/     # 群聊
│   └── conversation/   # 会话列表
│
├── notification/       # 🔔 通知模块
│   ├── push/           # 推送通知
│   └── activity/       # 活动通知
│
├── media/              # 🖼️ 媒体模块
│   ├── image/
│   ├── video/
│   └── cdn/            # CDN 上传
│
├── recommendation/     # 🎯 推荐模块
│   ├── friend_recommend/ # 好友推荐
│   └── content_recommend/ # 内容推荐
│
└── search/             # 🔍 搜索模块
    ├── user_search/
    └── content_search/
```

---

### 为什么社交APP需要这些模块？

#### 关注模块 (Follow)

**为什么独立**：
- 关系复杂（A 关注 B，B 关注 C，A 和 C 是互关吗？）
- 查询频繁（查看关注列表、粉丝列表）
- 影响推荐（推荐好友）

```python
# 关注用户
POST /api/users/123/follow

# 取消关注
DELETE /api/users/123/follow

# 获取关注列表
GET /api/users/123/following

# 获取粉丝列表
GET /api/users/123/followers

# 检查关系
GET /api/users/123/relationship?target=456
{
    "following": true,   # 我关注了他
    "followed_by": false # 他没关注我
}
```

---

#### 动态流模块 (Feed)

**为什么独立**：
- 算法复杂（按时间排序 vs 推荐算法）
- 性能要求高（刷新频繁）
- 需要缓存

```python
# 获取动态流
GET /api/feed
→ 返回关注的人的最新动态

算法：
1. 获取我关注的所有人
2. 获取他们最近的动态
3. 按时间/热度排序
4. 缓存结果（减少查询）
```

---

#### 私信模块 (Message)

**为什么独立**：
- 实时性要求高（WebSocket）
- 数据量大（聊天记录）
- 单独的数据库（MongoDB 存消息）

```python
# 发送消息
POST /api/messages
{
    "to_user_id": 123,
    "content": "你好！",
    "type": "text"
}

# 获取会话列表
GET /api/conversations
[
    {
        "user_id": 123,
        "last_message": "你好！",
        "unread_count": 3,
        "updated_at": "2024-01-15 10:30"
    }
]

# 获取聊天记录
GET /api/conversations/123/messages
```

---

#### 推荐模块 (Recommendation)

**为什么独立**：
- 算法复杂（协同过滤、内容推荐）
- 计算密集（离线计算）
- 可能用机器学习

```python
# 推荐好友
GET /api/recommendations/friends
算法：
- 共同好友
- 共同兴趣
- 地理位置

# 推荐内容
GET /api/recommendations/posts
算法：
- 用户历史行为
- 热门内容
- 相似用户喜欢的
```

---

## 2.7 新闻推送系统模块划分

### 完整的新闻系统架构

```
news-app/
├── content/            # 📰 内容模块
│   ├── article/        # 文章
│   ├── video/          # 视频
│   └── category/       # 分类（科技、体育、娱乐）
│
├── editor/             # ✍️ 编辑模块
│   ├── cms/            # 内容管理系统
│   └── review/         # 审核
│
├── recommendation/     # 🎯 推荐模块
│   ├── algorithm/      # 推荐算法
│   └── personalization/ # 个性化
│
├── user/               # 👤 用户模块
│   ├── preference/     # 偏好设置
│   └── history/        # 阅读历史
│
├── comment/            # 💬 评论模块
│   └── comment_service/
│
├── notification/       # 🔔 通知模块
│   ├── push/           # 推送
│   └── breaking_news/  # 突发新闻
│
├── search/             # 🔍 搜索模块
│   └── elasticsearch/
│
└── analytics/          # 📊 数据分析模块
    ├── view_tracking/  # 浏览追踪
    ├── hot_topics/     # 热门话题
    └── user_behavior/  # 用户行为
```

---

### 为什么新闻系统需要推荐模块？

**传统新闻**：
- 所有人看一样的首页
- 编辑决定什么重要

**现代新闻（今日头条模式）**：
- 每个人看不同的首页
- 算法推荐你感兴趣的

```python
# 推荐算法
def recommend_news(user_id):
    # 1. 获取用户历史
    history = get_user_history(user_id)

    # 2. 分析兴趣
    interests = analyze_interests(history)
    # 例如：用户经常看科技新闻

    # 3. 推荐相似内容
    recommendations = find_similar_articles(interests)

    # 4. 混入热门内容（防止信息茧房）
    trending = get_trending_news()

    # 5. 混合排序
    return merge_and_rank(recommendations, trending)
```

---

## 2.8 如何判断该不该拆分模块

### 判断标准

#### 标准 1：文件行数

```
< 200 行：不用拆
200-500 行：考虑拆
> 500 行：必须拆
> 1000 行：立即拆！
```

#### 标准 2：职责数量

```
问自己：这个模块在做几件事？

1 件事 → 不用拆
2-3 件事 → 考虑拆
> 3 件事 → 必须拆

示例：
user 模块做：
1. 用户注册
2. 用户资料
3. 用户登录  ← 应该拆成 auth 模块
4. 权限管理  ← 应该拆成 permission 模块
```

#### 标准 3：改动频率

```
经常一起改 → 放一起（高内聚）
独立改动 → 分开

示例：
支付模块：
- 支付宝接口经常变
- 微信接口也经常变
- 但两者互不影响
→ 应该分成 alipay/、wechat/ 两个子模块
```

#### 标准 4：复用性

```
能被其他项目用 → 独立成模块

示例：
email 发送功能：
- 注册要发邮件
- 订单要发邮件
- 其他项目也可能用
→ 独立成 email/ 模块
```

---

### 实战：重构一个混乱的项目

#### 现状（所有代码混在一起）

```python
# app.py (2000 行)
def main():
    # 用户注册
    user = register_user(username, email, password)

    # 发送邮件
    send_welcome_email(user.email)

    # 创建博客文章
    article = create_article(title, content)

    # 上传图片
    image_url = upload_image(file)

    # 发表评论
    comment = add_comment(article_id, content)

    # 点赞
    like(article_id)

    # ... 更多功能
```

**问题**：
- 2000 行，找代码要翻半天
- 功能混在一起
- 难以测试

---

#### 第一步：识别功能

```
列出所有功能：
1. 用户注册、登录
2. 发邮件
3. 博客文章 CRUD
4. 图片上传
5. 评论
6. 点赞
```

---

#### 第二步：分组

```
相关的功能分组：

组1：用户相关
- 注册
- 登录

组2：文章相关
- 创建文章
- 编辑文章
- 删除文章

组3：互动相关
- 评论
- 点赞

组4：基础服务
- 发邮件
- 上传文件
```

---

#### 第三步：拆分模块

```
blog/
├── user/
│   ├── auth.py         # 注册、登录
│   └── profile.py      # 个人资料
│
├── article/
│   ├── service.py      # 文章 CRUD
│   └── model.py        # 文章数据模型
│
├── interaction/
│   ├── comment.py      # 评论
│   └── like.py         # 点赞
│
├── media/
│   └── upload.py       # 文件上传
│
└── notification/
    └── email.py        # 邮件发送
```

---

#### 第四步：定义接口

```python
# user/auth.py
def register_user(username, email, password):
    """注册用户"""
    # 1. 验证输入
    # 2. 创建用户
    # 3. 发送欢迎邮件
    pass

def login_user(username, password):
    """登录用户"""
    # 1. 验证密码
    # 2. 生成 Token
    pass

# article/service.py
def create_article(title, content, author_id):
    """创建文章"""
    pass

def get_article(article_id):
    """获取文章"""
    pass

# 其他模块类似...
```

---

#### 第五步：模块间通信

```python
# main.py（变得很简洁）
from user import auth
from article import service as article_service
from notification import email

def handle_register(username, email, password):
    # 1. 注册用户（user 模块）
    user = auth.register_user(username, email, password)

    # 2. 发送邮件（notification 模块）
    email.send_welcome_email(user.email)

    return user

def handle_create_article(title, content, author_id):
    # 调用 article 模块
    article = article_service.create_article(title, content, author_id)
    return article
```

---

### 模块拆分的演进过程

```
阶段1：单文件（< 500 行）
main.py

阶段2：按功能拆分（500-2000 行）
├── user.py
├── article.py
└── comment.py

阶段3：模块化（2000-5000 行）
├── user/
│   ├── auth.py
│   └── profile.py
├── article/
│   └── service.py
└── comment/
    └── service.py

阶段4：分层模块化（> 5000 行）
├── api/           # API 层
├── services/      # 业务层
├── repository/    # 数据层
└── models/        # 模型
```

---

## 🎯 总结：技术选型和模块划分的本质

### 技术选型的本质

**不是选"最好"的技术，而是选"最合适"的技术**

```
考虑因素优先级：
1. 团队能力（最重要）
2. 项目需求
3. 性能要求
4. 长期维护
5. 个人喜好（最不重要）
```

**决策框架**：
```
1. 列出所有候选方案
2. 评估每个方案（团队、性能、生态、成本）
3. 权衡取舍（没有完美方案）
4. 做出决策
5. 记录原因（ADR）
```

---

### 模块划分的本质

**不是拆得越细越好，而是职责清晰**

```
好的模块：
- 职责单一（一个模块做一件事）
- 高内聚（相关的放一起）
- 低耦合（模块间依赖少）
- 可测试（能独立测试）
- 可复用（其他项目能用）
```

**判断标准**：
```
该不该拆分模块？

文件 > 500 行 → 考虑拆
做 > 3 件事 → 必须拆
经常独立改 → 应该拆
能被复用 → 应该拆
```

---

### 从 NOFX 学到的经验

**1. 够用原则**
- 不用微服务（单体够用）
- 不用数据库（JSON 够用）
- 不用 WebSocket（轮询够用）

**2. 接口抽象**
- Trader 接口（支持多交易所）
- 易于扩展（加交易所不改代码）

**3. 模块清晰**
- 每个模块职责明确
- 修改一个不影响其他

**4. 配置驱动**
- 多 Trader 靠配置（不是复制代码）

---

## 📚 推荐学习路径

### 技术栈学习顺序

```
1. 基础（2-3 个月）
   HTML → CSS → JavaScript

2. 前端框架（1-2 个月）
   React 或 Vue

3. 后端语言（2-3 个月）
   Python 或 Go

4. 数据库（1 个月）
   MySQL + Redis

5. 综合项目（1-2 个月）
   完整的 Web 应用
```

### 模块划分学习方法

```
1. 看优秀开源项目
   - NOFX
   - Django
   - Rails

2. 分析模块划分
   - 为什么这样拆？
   - 如果是我会怎么拆？

3. 实践
   - 从单文件开始
   - 觉得混乱时拆分
   - 重构优化

4. 总结经验
   - 记录决策原因
   - 对比不同方案
```

---

**🎉 恭喜你完成第二个深度补充！**

现在你应该能够：
- ✅ 理解所有主流技术的用途
- ✅ 判断什么项目用什么技术
- ✅ 知道如何拆分模块
- ✅ 看懂任何项目的模块结构

**记住**：
- **技术选型**：选最合适的，不是最新的
- **模块划分**：职责清晰，不是越多越好

准备好了吗？告诉我你的想法！💪
