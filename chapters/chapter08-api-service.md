# 第8章：API 服务层 - 前后端协作

> **本章目标**：学会设计 RESTful API，实现前后端分离架构

---

## 📋 本章大纲

1. [什么是 API 服务层](#1-什么是-api-服务层)
2. [RESTful API 设计原则](#2-restful-api-设计原则)
3. [HTTP 基础知识](#3-http-基础知识)
4. [API 开发实战](#4-api-开发实战)
5. [CORS 跨域处理](#5-cors-跨域处理)
6. [认证和授权](#6-认证和授权)
7. [NOFX 的 API 设计](#7-nofx-的-api-设计)
8. [实战练习](#8-实战练习)

**预计学习时间**：3-4 小时

---

## 1. 什么是 API 服务层

### 1.1 API 服务层的作用

**比喻**：API 就像是**餐厅的服务员**

```
前端（顾客）                 后端（厨房）
    │                           │
    │  1. "我要一份炒饭"          │
    ├────────────────────────→  │
    │      (HTTP Request)        │
    │                           │
    │                        2. 厨师做饭
    │                           │
    │  3. "您的炒饭好了"          │
    │←────────────────────────  │
    │      (HTTP Response)       │
    │                           │

API服务层 = 服务员
- 接收顾客点单（请求）
- 传达给厨房（业务逻辑）
- 送餐给顾客（响应）
```

**在前后端分离架构中**：

```
┌─────────────┐      HTTP      ┌─────────────┐
│   前端      │ ─────────────→ │   后端      │
│  (React)    │    API 请求     │  (Go/Node)  │
│             │ ←───────────── │             │
│             │   JSON 响应     │             │
└─────────────┘                 └─────────────┘

前端职责：
- 展示界面
- 处理用户交互
- 发送 API 请求

后端职责：
- 处理业务逻辑
- 操作数据库
- 返回数据
```

### 1.2 为什么需要 API 服务层？

| 好处 | 说明 | 举例 |
|------|------|------|
| **前后端分离** | 前端和后端独立开发和部署 | 前端换框架（Vue→React）不影响后端 |
| **多端复用** | 一套 API 供多个客户端使用 | Web、iOS、Android 用同一个 API |
| **清晰职责** | 前端负责展示，后端负责业务 | 前端不关心数据如何存储 |
| **易于测试** | API 可以独立测试 | 用 Postman 测试 API |
| **版本管理** | API 可以有多个版本 | `/api/v1/users`、`/api/v2/users` |

---

## 2. RESTful API 设计原则

### 2.1 什么是 REST？

**REST** = Representational State Transfer（表现层状态转移）

**简单理解**：用 HTTP 方法操作资源

```
资源（Resource）：用户、文章、订单等
HTTP 方法：GET、POST、PUT、DELETE
URL：资源的地址

操作用户的例子：
GET    /api/users          - 获取用户列表
GET    /api/users/123      - 获取ID为123的用户
POST   /api/users          - 创建新用户
PUT    /api/users/123      - 更新ID为123的用户
DELETE /api/users/123      - 删除ID为123的用户
```

### 2.2 HTTP 方法的语义

| HTTP 方法 | 用途 | 是否幂等 | 示例 |
|----------|------|---------|------|
| **GET** | 读取资源 | ✅ 是 | 获取文章列表 |
| **POST** | 创建资源 | ❌ 否 | 发布新文章 |
| **PUT** | 更新资源（完整） | ✅ 是 | 更新文章全部内容 |
| **PATCH** | 更新资源（部分） | ❌ 否 | 只更新文章标题 |
| **DELETE** | 删除资源 | ✅ 是 | 删除文章 |

**幂等性**：多次执行结果相同

```
# GET 是幂等的
GET /api/users/123  → 获取用户
GET /api/users/123  → 获取同一个用户（结果相同）

# POST 不是幂等的
POST /api/users {"name": "张三"}  → 创建用户（ID=1）
POST /api/users {"name": "张三"}  → 创建新用户（ID=2）（结果不同）

# DELETE 是幂等的
DELETE /api/users/123  → 删除用户
DELETE /api/users/123  → 用户已删除（结果相同）
```

### 2.3 URL 设计规范

**✅ 好的设计**：

```
GET    /api/articles              - 获取文章列表
GET    /api/articles/123          - 获取特定文章
GET    /api/articles/123/comments - 获取文章的评论
POST   /api/articles              - 创建文章
PUT    /api/articles/123          - 更新文章
DELETE /api/articles/123          - 删除文章

特点：
- 使用复数名词（articles 而不是 article）
- URL 只包含名词，不包含动词
- 层级清晰（文章 → 评论）
```

**❌ 不好的设计**：

```
GET  /api/getArticles           - ❌ URL中有动词
POST /api/createArticle         - ❌ URL中有动词
GET  /api/article/delete/123    - ❌ 应该用DELETE方法
GET  /api/article_comments/123  - ❌ 层级关系不清晰

应该改为：
GET    /api/articles
POST   /api/articles
DELETE /api/articles/123
GET    /api/articles/123/comments
```

### 2.4 响应状态码

**常用状态码**：

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| **200 OK** | 成功 | GET、PUT 成功 |
| **201 Created** | 创建成功 | POST 创建资源成功 |
| **204 No Content** | 成功但无内容 | DELETE 成功 |
| **400 Bad Request** | 请求参数错误 | 缺少必填字段 |
| **401 Unauthorized** | 未授权 | 未登录或 Token 失效 |
| **403 Forbidden** | 禁止访问 | 没有权限 |
| **404 Not Found** | 资源不存在 | 请求的ID不存在 |
| **500 Internal Server Error** | 服务器错误 | 代码异常 |

**示例**：

```javascript
// ✅ 正确使用状态码
// 获取成功
GET /api/users/123
Response: 200 OK
{
    "id": 123,
    "name": "张三"
}

// 创建成功
POST /api/users
Response: 201 Created
{
    "id": 456,
    "name": "李四"
}

// 未找到
GET /api/users/999
Response: 404 Not Found
{
    "error": "用户不存在"
}

// 参数错误
POST /api/users
{
    "name": ""  // 名字为空
}
Response: 400 Bad Request
{
    "error": "name 字段不能为空"
}
```

---

## 3. HTTP 基础知识

### 3.1 HTTP 请求结构

```
GET /api/users/123 HTTP/1.1                      ← 请求行
Host: api.example.com                             ← 请求头
Authorization: Bearer token123
Content-Type: application/json
                                                  ← 空行
{"key": "value"}                                  ← 请求体（仅POST/PUT等）
```

**组成部分**：

1. **请求行**：方法 + URL + 协议版本
2. **请求头**：附加信息（认证、内容类型等）
3. **请求体**：发送的数据（GET 请求通常没有）

### 3.2 HTTP 响应结构

```
HTTP/1.1 200 OK                                   ← 状态行
Content-Type: application/json                    ← 响应头
Access-Control-Allow-Origin: *
                                                  ← 空行
{                                                 ← 响应体
    "id": 123,
    "name": "张三"
}
```

### 3.3 查询参数 vs 请求体

**查询参数（Query Parameters）**：

```
GET /api/users?page=1&limit=10&sort=name

用途：
- 过滤数据（筛选条件）
- 分页（page、limit）
- 排序（sort）

特点：
- 出现在 URL 中
- 适合 GET 请求
- 有长度限制
```

**请求体（Request Body）**：

```
POST /api/users
Content-Type: application/json

{
    "name": "张三",
    "email": "zhang@example.com",
    "age": 25
}

用途：
- 创建资源（POST）
- 更新资源（PUT/PATCH）

特点：
- 不出现在 URL 中
- 无长度限制
- 适合大量数据
```

---

## 4. API 开发实战

### 4.1 选择 Web 框架

**Go 语言框架对比**：

| 框架 | 特点 | 学习曲线 | 性能 | 推荐场景 |
|------|------|---------|------|---------|
| **Gin** | 轻量、快速、中间件丰富 | 低 | ⭐⭐⭐⭐⭐ | 中小型API |
| **Echo** | 类似Gin，更现代 | 低 | ⭐⭐⭐⭐⭐ | 中小型API |
| **Fiber** | 类似Express.js | 低 | ⭐⭐⭐⭐⭐ | 熟悉Node.js的开发者 |
| **Chi** | 纯标准库风格 | 中 | ⭐⭐⭐⭐ | 标准化项目 |

**Node.js 框架对比**：

| 框架 | 特点 | 学习曲线 | 生态 | 推荐场景 |
|------|------|---------|------|---------|
| **Express** | 最流行，中间件丰富 | 低 | ⭐⭐⭐⭐⭐ | 任何项目 |
| **Koa** | 现代、async/await | 中 | ⭐⭐⭐⭐ | 熟悉ES6+ |
| **Fastify** | 高性能、类型安全 | 中 | ⭐⭐⭐⭐ | 性能敏感项目 |
| **NestJS** | 企业级、TypeScript | 高 | ⭐⭐⭐⭐⭐ | 大型项目 |

### 4.2 使用 Gin 开发 API

**基础示例**：

```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

// User 用户模型
type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

// 模拟数据库
var users = []User{
    {ID: 1, Name: "张三", Email: "zhang@example.com"},
    {ID: 2, Name: "李四", Email: "li@example.com"},
}

func main() {
    // 创建 Gin 路由
    r := gin.Default()

    // 设置路由
    setupRoutes(r)

    // 启动服务器
    r.Run(":8080")  // 监听 8080 端口
}

func setupRoutes(r *gin.Engine) {
    api := r.Group("/api")
    {
        // 用户相关路由
        api.GET("/users", getUsers)         // 获取用户列表
        api.GET("/users/:id", getUserByID)  // 获取单个用户
        api.POST("/users", createUser)      // 创建用户
        api.PUT("/users/:id", updateUser)   // 更新用户
        api.DELETE("/users/:id", deleteUser) // 删除用户
    }
}

// 获取用户列表
func getUsers(c *gin.Context) {
    // 支持分页查询
    page := c.DefaultQuery("page", "1")
    limit := c.DefaultQuery("limit", "10")

    c.JSON(http.StatusOK, gin.H{
        "page":  page,
        "limit": limit,
        "data":  users,
        "total": len(users),
    })
}

// 获取单个用户
func getUserByID(c *gin.Context) {
    id := c.Param("id")  // 从 URL 路径获取参数

    // 查找用户
    for _, user := range users {
        if user.ID == parseID(id) {
            c.JSON(http.StatusOK, user)
            return
        }
    }

    // 未找到
    c.JSON(http.StatusNotFound, gin.H{
        "error": "用户不存在",
    })
}

// 创建用户
func createUser(c *gin.Context) {
    var newUser User

    // 绑定 JSON 数据到结构体
    if err := c.ShouldBindJSON(&newUser); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{
            "error": "请求参数错误: " + err.Error(),
        })
        return
    }

    // 验证必填字段
    if newUser.Name == "" || newUser.Email == "" {
        c.JSON(http.StatusBadRequest, gin.H{
            "error": "name 和 email 不能为空",
        })
        return
    }

    // 生成 ID
    newUser.ID = len(users) + 1

    // 保存用户
    users = append(users, newUser)

    // 返回创建的用户
    c.JSON(http.StatusCreated, newUser)
}

// 更新用户
func updateUser(c *gin.Context) {
    id := c.Param("id")

    var updatedUser User
    if err := c.ShouldBindJSON(&updatedUser); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{
            "error": "请求参数错误",
        })
        return
    }

    // 查找并更新用户
    for i, user := range users {
        if user.ID == parseID(id) {
            users[i].Name = updatedUser.Name
            users[i].Email = updatedUser.Email
            c.JSON(http.StatusOK, users[i])
            return
        }
    }

    c.JSON(http.StatusNotFound, gin.H{
        "error": "用户不存在",
    })
}

// 删除用户
func deleteUser(c *gin.Context) {
    id := c.Param("id")

    // 查找并删除用户
    for i, user := range users {
        if user.ID == parseID(id) {
            users = append(users[:i], users[i+1:]...)
            c.JSON(http.StatusOK, gin.H{
                "message": "删除成功",
            })
            return
        }
    }

    c.JSON(http.StatusNotFound, gin.H{
        "error": "用户不存在",
    })
}

func parseID(id string) int {
    // 简化版本，实际应该处理错误
    var result int
    fmt.Sscanf(id, "%d", &result)
    return result
}
```

**测试 API**：

```bash
# 获取所有用户
curl http://localhost:8080/api/users

# 获取特定用户
curl http://localhost:8080/api/users/1

# 创建新用户
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"王五","email":"wang@example.com"}'

# 更新用户
curl -X PUT http://localhost:8080/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"张三更新","email":"zhangsan@example.com"}'

# 删除用户
curl -X DELETE http://localhost:8080/api/users/1
```

### 4.3 中间件（Middleware）

**什么是中间件？**

中间件就是**在请求处理前后执行的函数**

```
请求流程：
Request → 中间件1 → 中间件2 → 处理函数 → 中间件2 → 中间件1 → Response
          ↓         ↓         ↓         ↑         ↑
        日志记录   身份验证   业务逻辑   格式化    添加Header
```

**日志中间件示例**：

```go
// 日志中间件
func LoggerMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 请求前
        start := time.Now()
        path := c.Request.URL.Path

        // 处理请求
        c.Next()

        // 请求后
        duration := time.Since(start)
        status := c.Writer.Status()

        log.Printf("[%s] %s %d %v",
            c.Request.Method,
            path,
            status,
            duration,
        )
    }
}

// 使用中间件
r := gin.New()
r.Use(LoggerMiddleware())
r.Use(gin.Recovery())  // panic 恢复中间件
```

**认证中间件示例**：

```go
// 认证中间件
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 从 Header 获取 Token
        token := c.GetHeader("Authorization")

        if token == "" {
            c.JSON(http.StatusUnauthorized, gin.H{
                "error": "缺少认证token",
            })
            c.Abort()  // 阻止后续处理
            return
        }

        // 验证 Token
        if !isValidToken(token) {
            c.JSON(http.StatusUnauthorized, gin.H{
                "error": "无效的token",
            })
            c.Abort()
            return
        }

        // Token 有效，继续处理
        c.Next()
    }
}

// 应用到特定路由
api := r.Group("/api")
api.Use(AuthMiddleware())  // 整个 /api 路由组都需要认证
{
    api.GET("/users", getUsers)
    // ...
}

// 或者应用到单个路由
r.GET("/api/admin/users", AuthMiddleware(), getAdminUsers)
```

---

## 5. CORS 跨域处理

### 5.1 什么是 CORS？

**CORS** = Cross-Origin Resource Sharing（跨源资源共享）

**问题场景**：

```
前端运行在：   http://localhost:3000   (React开发服务器)
后端运行在：   http://localhost:8080   (API服务器)

由于端口不同，浏览器认为是"跨域"，默认会阻止请求

❌ 错误信息：
Access to fetch at 'http://localhost:8080/api/users' from origin
'http://localhost:3000' has been blocked by CORS policy
```

**什么是同源？**

| 条件 | 是否同源 |
|------|---------|
| `http://example.com:80/a` vs `http://example.com:80/b` | ✅ 同源 |
| `http://example.com` vs `https://example.com` | ❌ 协议不同 |
| `http://example.com` vs `http://api.example.com` | ❌ 域名不同 |
| `http://example.com:80` vs `http://example.com:8080` | ❌ 端口不同 |

### 5.2 解决 CORS 问题

**方法1：在后端添加 CORS 头**

```go
// CORS 中间件
func CORSMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 允许所有来源（开发环境）
        c.Writer.Header().Set("Access-Control-Allow-Origin", "*")

        // 允许的 HTTP 方法
        c.Writer.Header().Set("Access-Control-Allow-Methods",
            "GET, POST, PUT, DELETE, OPTIONS")

        // 允许的请求头
        c.Writer.Header().Set("Access-Control-Allow-Headers",
            "Content-Type, Authorization")

        // 预检请求（OPTIONS）直接返回
        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(http.StatusOK)
            return
        }

        c.Next()
    }
}

// 使用
r := gin.Default()
r.Use(CORSMiddleware())
```

**方法2：使用 CORS 库**

```go
import "github.com/gin-contrib/cors"

r := gin.Default()

// 默认配置（允许所有来源）
r.Use(cors.Default())

// 或者自定义配置
r.Use(cors.New(cors.Config{
    AllowOrigins:     []string{"http://localhost:3000"},  // 只允许这个来源
    AllowMethods:     []string{"GET", "POST", "PUT", "DELETE"},
    AllowHeaders:     []string{"Origin", "Content-Type", "Authorization"},
    ExposeHeaders:    []string{"Content-Length"},
    AllowCredentials: true,  // 允许携带Cookie
    MaxAge:           12 * time.Hour,
}))
```

**生产环境配置**：

```go
// ❌ 不安全（开发环境可以用）
c.Writer.Header().Set("Access-Control-Allow-Origin", "*")

// ✅ 安全（生产环境）
allowedOrigins := []string{
    "https://myapp.com",
    "https://www.myapp.com",
}

origin := c.Request.Header.Get("Origin")
for _, allowed := range allowedOrigins {
    if origin == allowed {
        c.Writer.Header().Set("Access-Control-Allow-Origin", origin)
        break
    }
}
```

---

## 6. 认证和授权

### 6.1 认证 vs 授权

```
认证（Authentication）：你是谁？
- 登录验证
- 验证用户名和密码
- 验证 Token

授权（Authorization）：你能做什么？
- 权限检查
- 管理员可以删除用户
- 普通用户只能查看
```

### 6.2 JWT 认证

**JWT** = JSON Web Token

**工作流程**：

```
1. 用户登录
   POST /api/login
   {
       "username": "zhang",
       "password": "123456"
   }

2. 服务器验证密码，生成 Token
   Response:
   {
       "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
   }

3. 前端保存 Token（localStorage/Cookie）

4. 后续请求带上 Token
   GET /api/users
   Headers:
     Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

5. 服务器验证 Token，返回数据
```

**Go 实现 JWT**：

```go
import "github.com/golang-jwt/jwt/v4"

// JWT 密钥（应该放在环境变量中）
var jwtSecret = []byte("your-secret-key")

// Claims 结构
type Claims struct {
    UserID   int    `json:"user_id"`
    Username string `json:"username"`
    jwt.RegisteredClaims
}

// 生成 Token
func GenerateToken(userID int, username string) (string, error) {
    claims := Claims{
        UserID:   userID,
        Username: username,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),  // 24小时过期
            IssuedAt:  jwt.NewNumericDate(time.Now()),
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(jwtSecret)
}

// 验证 Token
func ValidateToken(tokenString string) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(token *jwt.Token) (interface{}, error) {
        return jwtSecret, nil
    })

    if err != nil {
        return nil, err
    }

    if claims, ok := token.Claims.(*Claims); ok && token.Valid {
        return claims, nil
    }

    return nil, fmt.Errorf("invalid token")
}

// 登录接口
func login(c *gin.Context) {
    var req struct {
        Username string `json:"username"`
        Password string `json:"password"`
    }

    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "参数错误"})
        return
    }

    // 验证用户名密码（这里简化处理）
    if req.Username != "zhang" || req.Password != "123456" {
        c.JSON(http.StatusUnauthorized, gin.H{"error": "用户名或密码错误"})
        return
    }

    // 生成 Token
    token, err := GenerateToken(1, req.Username)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "生成token失败"})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "token": token,
    })
}

// JWT 中间件
func JWTAuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 从 Header 获取 Token
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "缺少认证token"})
            c.Abort()
            return
        }

        // 提取 Token（格式：Bearer <token>）
        parts := strings.Split(authHeader, " ")
        if len(parts) != 2 || parts[0] != "Bearer" {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "token格式错误"})
            c.Abort()
            return
        }

        // 验证 Token
        claims, err := ValidateToken(parts[1])
        if err != nil {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "无效的token"})
            c.Abort()
            return
        }

        // 将用户信息存入上下文
        c.Set("user_id", claims.UserID)
        c.Set("username", claims.Username)

        c.Next()
    }
}

// 使用中间件
api := r.Group("/api")
{
    // 公开接口（无需认证）
    api.POST("/login", login)

    // 需要认证的接口
    api.Use(JWTAuthMiddleware())
    api.GET("/users", getUsers)
    api.POST("/users", createUser)
}
```

---

## 7. NOFX 的 API 设计

### 7.1 NOFX API 架构分析

文件：`api/server.go`

**整体结构**：

```go
type Server struct {
    router        *gin.Engine              // Gin 路由引擎
    traderManager *manager.TraderManager   // Trader 管理器
    port          int                      // 端口号
}

func NewServer(traderManager *manager.TraderManager, port int) *Server {
    // 设置为 Release 模式（减少日志）
    gin.SetMode(gin.ReleaseMode)

    router := gin.Default()

    // 启用 CORS
    router.Use(corsMiddleware())

    s := &Server{
        router:        router,
        traderManager: traderManager,
        port:          port,
    }

    s.setupRoutes()
    return s
}
```

### 7.2 NOFX 的路由设计

```go
func (s *Server) setupRoutes() {
    // 健康检查
    s.router.Any("/health", s.handleHealth)

    // API 路由组
    api := s.router.Group("/api")
    {
        // 竞赛总览
        api.GET("/competition", s.handleCompetition)

        // Trader 列表
        api.GET("/traders", s.handleTraderList)

        // 指定 trader 的数据（使用 query 参数 ?trader_id=xxx）
        api.GET("/status", s.handleStatus)
        api.GET("/account", s.handleAccount)
        api.GET("/positions", s.handlePositions)
        api.GET("/decisions", s.handleDecisions)
        api.GET("/decisions/latest", s.handleLatestDecisions)
        api.GET("/statistics", s.handleStatistics)
        api.GET("/equity-history", s.handleEquityHistory)
        api.GET("/performance", s.handlePerformance)
    }
}
```

**API 设计特点**：

1. **RESTful 风格**：使用 GET 方法获取资源
2. **路由分组**：所有 API 在 `/api` 路径下
3. **查询参数**：使用 `?trader_id=xxx` 指定 trader
4. **资源清晰**：每个端点返回特定资源

### 7.3 NOFX 的 CORS 处理

文件：`api/server.go:42`

```go
func corsMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 允许所有来源
        c.Writer.Header().Set("Access-Control-Allow-Origin", "*")

        // 允许的方法
        c.Writer.Header().Set("Access-Control-Allow-Methods",
            "GET, POST, PUT, DELETE, OPTIONS")

        // 允许的请求头
        c.Writer.Header().Set("Access-Control-Allow-Headers",
            "Content-Type, Authorization")

        // 处理预检请求
        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(http.StatusOK)
            return
        }

        c.Next()
    }
}
```

**为什么允许所有来源 (`*`)?**

- NOFX 是本地运行的工具，不需要严格的跨域限制
- 前端可能运行在不同端口（开发环境）
- 简化部署和使用

### 7.4 NOFX 的处理函数示例

**获取账户信息**：

文件：`api/server.go:152`

```go
func (s *Server) handleAccount(c *gin.Context) {
    // 1. 从查询参数获取 trader_id
    _, traderID, err := s.getTraderFromQuery(c)
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    // 2. 获取 Trader 实例
    trader, err := s.traderManager.GetTrader(traderID)
    if err != nil {
        c.JSON(http.StatusNotFound, gin.H{"error": err.Error()})
        return
    }

    // 3. 获取账户信息
    log.Printf("📊 收到账户信息请求 [%s]", trader.GetName())
    account, err := trader.GetAccountInfo()
    if err != nil {
        log.Printf("❌ 获取账户信息失败 [%s]: %v", trader.GetName(), err)
        c.JSON(http.StatusInternalServerError, gin.H{
            "error": fmt.Sprintf("获取账户信息失败: %v", err),
        })
        return
    }

    // 4. 记录日志
    log.Printf("✓ 返回账户信息 [%s]: 净值=%.2f, 可用=%.2f, 盈亏=%.2f (%.2f%%)",
        trader.GetName(),
        account["total_equity"],
        account["available_balance"],
        account["total_pnl"],
        account["total_pnl_pct"])

    // 5. 返回 JSON 响应
    c.JSON(http.StatusOK, account)
}
```

**设计亮点**：

1. **清晰的错误处理**：参数错误返回 400，资源不存在返回 404
2. **详细的日志**：记录请求和响应，方便调试
3. **统一的响应格式**：成功返回数据，失败返回 `{"error": "..."}`

### 7.5 NOFX API 完整列表

| 端点 | 方法 | 功能 | 响应 |
|------|------|------|------|
| `/health` | GET | 健康检查 | `{"status": "ok"}` |
| `/api/competition` | GET | 所有 trader 对比数据 | 各 trader 的盈亏、持仓 |
| `/api/traders` | GET | Trader 列表 | trader ID、名称、AI模型 |
| `/api/status` | GET | 系统状态 | 运行状态、调用次数 |
| `/api/account` | GET | 账户信息 | 净值、余额、盈亏 |
| `/api/positions` | GET | 持仓列表 | 所有持仓详情 |
| `/api/decisions` | GET | 决策日志 | 历史决策记录 |
| `/api/decisions/latest` | GET | 最新决策 | 最近5条决策 |
| `/api/statistics` | GET | 统计信息 | 胜率、盈亏比 |
| `/api/equity-history` | GET | 收益率历史 | 净值变化曲线 |
| `/api/performance` | GET | AI 表现分析 | 成功/失败交易 |

---

## 8. 实战练习

### 练习 1：实现用户管理 API

**需求**：设计并实现用户管理的完整 CRUD API

**要求**：
1. 使用 Gin 框架（或你熟悉的框架）
2. 实现以下端点：
   - `GET /api/users` - 获取用户列表（支持分页）
   - `GET /api/users/:id` - 获取单个用户
   - `POST /api/users` - 创建用户
   - `PUT /api/users/:id` - 更新用户
   - `DELETE /api/users/:id` - 删除用户
3. 包含参数验证和错误处理
4. 使用合适的 HTTP 状态码

### 练习 2：添加 JWT 认证

**任务**：为练习 1 的 API 添加 JWT 认证

**要求**：
1. 实现登录接口 `POST /api/login`
2. 实现 JWT 中间件
3. 保护所有 `/api/users` 端点（除了登录）
4. 从 Token 中提取用户信息

### 练习 3：为 NOFX 添加新端点

**任务**：为 NOFX 添加一个新端点，返回指定时间范围内的交易历史

**要求**：
1. 端点：`GET /api/trades?trader_id=xxx&start_date=2024-01-01&end_date=2024-12-31`
2. 参数验证：检查日期格式是否正确
3. 错误处理：trader 不存在、日期范围无效等
4. 返回格式：
   ```json
   {
       "trader_id": "qwen",
       "start_date": "2024-01-01",
       "end_date": "2024-12-31",
       "trades": [
           {
               "symbol": "BTCUSDT",
               "side": "buy",
               "quantity": 0.5,
               "price": 50000,
               "timestamp": "2024-06-15 10:30:00"
           }
       ],
       "total_count": 100
   }
   ```

### 练习 4：API 文档

**任务**：为练习 1 的 API 编写文档

**要求**：
1. 列出所有端点
2. 说明请求参数
3. 提供请求示例（curl 命令）
4. 说明响应格式
5. 列出可能的错误码

<details>
<summary>参考格式</summary>

```markdown
# 用户管理 API 文档

## 获取用户列表

**端点**: `GET /api/users`

**查询参数**:
- `page` (int, 可选): 页码，默认1
- `limit` (int, 可选): 每页数量，默认10

**请求示例**:
```bash
curl http://localhost:8080/api/users?page=1&limit=10
```

**成功响应** (200 OK):
```json
{
    "page": 1,
    "limit": 10,
    "total": 100,
    "data": [
        {
            "id": 1,
            "name": "张三",
            "email": "zhang@example.com"
        }
    ]
}
```

**错误响应**:
- `400 Bad Request`: 参数错误
- `500 Internal Server Error`: 服务器错误
```

</details>

---

## 9. 本章总结

### 核心概念

| 概念 | 说明 | 关键点 |
|------|------|--------|
| **RESTful** | API 设计风格 | 使用 HTTP 方法操作资源 |
| **HTTP 方法** | GET/POST/PUT/DELETE | GET 查询，POST 创建，PUT 更新，DELETE 删除 |
| **状态码** | 表示请求结果 | 200 成功，400 参数错误，401 未认证，404 未找到，500 服务器错误 |
| **CORS** | 跨域资源共享 | 前后端分离必须处理 |
| **JWT** | 认证 Token | 无状态认证方案 |
| **中间件** | 请求处理链 | 日志、认证、CORS 等 |

### RESTful API 设计原则

1. **使用名词而非动词**：`/api/users` 而不是 `/api/getUsers`
2. **使用复数名词**：`/api/users` 而不是 `/api/user`
3. **使用 HTTP 方法表示操作**：GET 查询，POST 创建，PUT 更新，DELETE 删除
4. **使用正确的状态码**：成功 2xx，客户端错误 4xx，服务器错误 5xx
5. **支持过滤、排序、分页**：`?page=1&limit=10&sort=name`
6. **提供清晰的错误信息**：`{"error": "用户名不能为空"}`

### NOFX 的启示

1. **简洁的设计**：只实现必要的端点，不过度设计
2. **清晰的日志**：每个请求都有日志，方便调试
3. **统一的响应格式**：成功返回数据，失败返回 `{"error": "..."}`
4. **CORS 友好**：开发环境允许所有来源
5. **合理的路由分组**：所有 API 在 `/api` 下

---

## 📚 扩展阅读

- [RESTful API 设计最佳实践](https://restfulapi.net/)
- [Gin 框架官方文档](https://gin-gonic.com/docs/)
- [HTTP 状态码完整列表](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [JWT 介绍和使用](https://jwt.io/introduction)

---

## 📌 下一章预告

**第9章：前端展示层 - 用户界面设计**
- React 基础
- 组件化思维
- 状态管理
- NOFX 前端架构分析

---

**💡 记住**：API 设计要简单明了，让前端开发者一眼就能看懂如何使用。好的 API 文档和清晰的错误信息能节省大量调试时间。NOFX 的 API 设计虽然简单，但非常实用，值得学习！
