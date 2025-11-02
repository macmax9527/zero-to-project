# 第9章：前端展示层 - 用户界面设计

> **本章目标**：理解前端开发基础，学会用 React 构建用户界面

---

## 📋 本章大纲

1. [前端的作用](#1-前端的作用)
2. [React 快速入门](#2-react-快速入门)
3. [组件化思维](#3-组件化思维)
4. [状态管理](#4-状态管理)
5. [与后端 API 交互](#5-与后端-api-交互)
6. [NOFX 前端架构](#6-nofx-前端架构)
7. [实战练习](#7-实战练习)

**预计学习时间**：3-4 小时

---

## 1. 前端的作用

### 1.1 前端在项目中的位置

```
用户 ←→ 前端 ←→ 后端 ←→ 数据库

用户看到的：      前端负责：         后端负责：
┌──────────┐     ┌──────────┐     ┌──────────┐
│ 按钮、表格 │ ←→ │ 渲染界面  │ ←→ │ 业务逻辑  │
│ 图表、表单 │     │ 处理交互  │     │ 数据处理  │
│ 导航菜单  │     │ 发送请求  │     │ 存储数据  │
└──────────┘     └──────────┘     └──────────┘
```

### 1.2 前端三大核心技术

```
HTML  → 结构（骨架）
CSS   → 样式（外观）
JavaScript → 行为（交互）

就像造房子：
HTML = 墙壁、门窗（结构）
CSS = 刷漆、装修（美化）
JavaScript = 开关、电梯（功能）
```

---

## 2. React 快速入门

### 2.1 什么是 React？

React 是一个用于构建用户界面的 JavaScript 库

**核心理念**：组件化 + 数据驱动视图

```jsx
// 传统方式（手动操作 DOM）
document.getElementById('count').innerHTML = count;

// React 方式（数据驱动）
function Counter() {
    const [count, setCount] = useState(0);  // 数据
    return <div>{count}</div>;  // 视图自动更新
}
```

### 2.2 JSX 语法

JSX = JavaScript + XML，在 JS 中写 HTML

```jsx
// ✅ JSX（推荐）
function Hello() {
    return <h1>你好，世界！</h1>;
}

// 等价于（编译后）
function Hello() {
    return React.createElement('h1', null, '你好，世界！');
}
```

**JSX 规则**：

```jsx
// 1. 必须有一个根元素
// ❌ 错误
function App() {
    return (
        <h1>标题</h1>
        <p>内容</p>
    );
}

// ✅ 正确
function App() {
    return (
        <div>
            <h1>标题</h1>
            <p>内容</p>
        </div>
    );
}

// 2. 使用 {} 嵌入 JavaScript 表达式
function Greeting({ name }) {
    return <h1>你好，{name}！</h1>;
}

// 3. className 代替 class
<div className="container">内容</div>

// 4. 事件处理用驼峰命名
<button onClick={handleClick}>点击</button>
```

### 2.3 函数组件和 Hooks

```jsx
import React, { useState, useEffect } from 'react';

function Counter() {
    // useState：管理组件状态
    const [count, setCount] = useState(0);

    // useEffect：副作用（如 API 请求）
    useEffect(() => {
        document.title = `点击了 ${count} 次`;
    }, [count]);  // count 变化时执行

    const increment = () => {
        setCount(count + 1);
    };

    return (
        <div>
            <p>当前计数：{count}</p>
            <button onClick={increment}>+1</button>
        </div>
    );
}
```

---

## 3. 组件化思维

### 3.1 什么是组件化？

**比喻**：组件就像**乐高积木**

```
大组件 = 小组件组合

页面（Page）
  ├─ 导航栏（Navbar）
  ├─ 侧边栏（Sidebar）
  │   ├─ 菜单项（MenuItem）
  │   └─ 菜单项（MenuItem）
  └─ 主内容（Content）
      ├─ 卡片（Card）
      │   ├─ 标题（Title）
      │   └─ 内容（Body）
      └─ 卡片（Card）
```

### 3.2 组件设计原则

**1. 单一职责**：一个组件只做一件事

```jsx
// ✅ 好的设计
function UserCard({ user }) {
    return (
        <div className="card">
            <Avatar url={user.avatar} />
            <UserName name={user.name} />
            <UserBio bio={user.bio} />
        </div>
    );
}

// ❌ 不好的设计（职责过多）
function UserProfile({ user }) {
    // 包含用户信息、订单列表、评论等
    return (
        <div>
            {/* 500 行代码 */}
        </div>
    );
}
```

**2. 可复用**：同一个组件可以多次使用

```jsx
function Button({ text, onClick, type = 'primary' }) {
    return (
        <button
            className={`btn btn-${type}`}
            onClick={onClick}
        >
            {text}
        </button>
    );
}

// 使用
<Button text="保存" onClick={handleSave} type="primary" />
<Button text="取消" onClick={handleCancel} type="secondary" />
<Button text="删除" onClick={handleDelete} type="danger" />
```

**3. Props 传递数据**

```jsx
// 父组件 → 子组件传递数据
function Parent() {
    const data = { name: '张三', age: 25 };

    return <Child userData={data} />;
}

function Child({ userData }) {
    return (
        <div>
            <p>姓名：{userData.name}</p>
            <p>年龄：{userData.age}</p>
        </div>
    );
}
```

---

## 4. 状态管理

### 4.1 本地状态（useState）

```jsx
function TodoList() {
    // 状态：待办事项列表
    const [todos, setTodos] = useState([
        { id: 1, text: '学习 React', done: false },
        { id: 2, text: '做项目', done: false },
    ]);

    // 添加待办
    const addTodo = (text) => {
        const newTodo = {
            id: Date.now(),
            text: text,
            done: false
        };
        setTodos([...todos, newTodo]);  // 不可变更新
    };

    // 切换完成状态
    const toggleTodo = (id) => {
        setTodos(todos.map(todo =>
            todo.id === id ? { ...todo, done: !todo.done } : todo
        ));
    };

    return (
        <div>
            {todos.map(todo => (
                <div key={todo.id}>
                    <input
                        type="checkbox"
                        checked={todo.done}
                        onChange={() => toggleTodo(todo.id)}
                    />
                    <span>{todo.text}</span>
                </div>
            ))}
        </div>
    );
}
```

### 4.2 全局状态（Context）

当多个组件需要共享状态时使用

```jsx
// 创建 Context
const UserContext = React.createContext();

// Provider 组件
function App() {
    const [user, setUser] = useState({ name: '张三', role: 'admin' });

    return (
        <UserContext.Provider value={{ user, setUser }}>
            <Header />
            <Dashboard />
        </UserContext.Provider>
    );
}

// 消费 Context
function Header() {
    const { user } = useContext(UserContext);
    return <div>欢迎，{user.name}！</div>;
}

function Dashboard() {
    const { user } = useContext(UserContext);
    return <div>你的角色：{user.role}</div>;
}
```

---

## 5. 与后端 API 交互

### 5.1 使用 fetch 获取数据

```jsx
function UserList() {
    const [users, setUsers] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        // 获取用户列表
        fetch('http://localhost:8080/api/users')
            .then(response => {
                if (!response.ok) {
                    throw new Error('请求失败');
                }
                return response.json();
            })
            .then(data => {
                setUsers(data);
                setLoading(false);
            })
            .catch(err => {
                setError(err.message);
                setLoading(false);
            });
    }, []);  // 空数组表示只在组件挂载时执行

    if (loading) return <div>加载中...</div>;
    if (error) return <div>错误：{error}</div>;

    return (
        <ul>
            {users.map(user => (
                <li key={user.id}>{user.name}</li>
            ))}
        </ul>
    );
}
```

### 5.2 使用 axios（推荐）

```jsx
import axios from 'axios';

// 配置 axios
const api = axios.create({
    baseURL: 'http://localhost:8080/api',
    headers: {
        'Content-Type': 'application/json'
    }
});

function UserList() {
    const [users, setUsers] = useState([]);

    useEffect(() => {
        api.get('/users')
            .then(response => {
                setUsers(response.data);
            })
            .catch(error => {
                console.error('获取用户列表失败:', error);
            });
    }, []);

    return (
        <ul>
            {users.map(user => (
                <li key={user.id}>{user.name}</li>
            ))}
        </ul>
    );
}
```

---

## 6. NOFX 前端架构

### 6.1 NOFX 前端技术栈

查看 NOFX 的 `web/package.json` 可以看到：

```json
{
    "dependencies": {
        "react": "^18.x",           // UI 框架
        "react-dom": "^18.x",       // DOM 渲染
        "recharts": "^2.x",         // 图表库
        "axios": "^1.x",            // HTTP 客户端
        "tailwindcss": "^3.x"       // CSS 框架
    }
}
```

### 6.2 NOFX 前端特点

1. **实时数据更新**：定时轮询 API 获取最新数据
2. **数据可视化**：使用 Recharts 展示收益曲线
3. **响应式设计**：使用 Tailwind CSS 适配不同屏幕
4. **组件化**：每个功能模块是独立组件

### 6.3 NOFX 前端学习要点

**关键文件**：
- `web/src/App.jsx` - 主应用组件
- `web/src/components/` - 可复用组件
- `web/src/pages/` - 页面组件

**学习建议**：
1. 先看 `App.jsx` 了解整体结构
2. 分析各个组件如何获取和展示数据
3. 学习如何使用 Recharts 绘制图表
4. 理解如何通过 axios 调用 NOFX API

---

## 7. 实战练习

### 练习 1：创建简单计数器

创建一个计数器组件，包含 +1、-1、重置按钮

<details>
<summary>参考答案</summary>

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    return (
        <div>
            <h1>计数：{count}</h1>
            <button onClick={() => setCount(count + 1)}>+1</button>
            <button onClick={() => setCount(count - 1)}>-1</button>
            <button onClick={() => setCount(0)}>重置</button>
        </div>
    );
}
```

</details>

### 练习 2：获取并显示用户列表

从 `/api/users` 获取用户列表并显示，包含加载状态和错误处理

### 练习 3：创建待办事项应用

实现添加、删除、标记完成功能

### 练习 4：分析 NOFX 前端

1. 运行 NOFX 前端（`cd web && npm run dev`）
2. 找到账户信息展示组件
3. 分析数据如何从 API 获取并渲染
4. 尝试修改样式或添加新字段

---

## 本章总结

### 核心概念

- **组件化**：UI 拆分为可复用的组件
- **状态管理**：useState 管理本地状态，Context 管理全局状态
- **数据驱动**：状态改变，视图自动更新
- **API 交互**：使用 fetch 或 axios 与后端通信

### 学习路径

1. HTML/CSS/JavaScript 基础
2. React 基础（JSX、组件、Props、State）
3. Hooks（useState、useEffect、useContext）
4. API 集成和数据管理
5. 项目实战（分析 NOFX 前端）

---

**💡 提示**：前端开发的核心是组件化思维。学会拆分组件、管理状态、调用 API，你就能构建任何 Web 应用！
