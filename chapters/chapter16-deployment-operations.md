# 第16章：部署和运维 - 生产环境

> **本章目标**：学会将项目部署到生产环境，并进行日常运维

---

## 📋 本章大纲

1. [开发环境 vs 生产环境](#1-开发环境-vs-生产环境)
2. [服务器选择和配置](#2-服务器选择和配置)
3. [Docker 容器化](#3-docker-容器化)
4. [进程管理](#4-进程管理)
5. [反向代理和HTTPS](#5-反向代理和https)
6. [监控和日志](#6-监控和日志)
7. [备份和恢复](#7-备份和恢复)
8. [NOFX 部署实战](#8-nofx-部署实战)
9. [实战练习](#9-实战练习)

**预计学习时间**：4-5 小时

---

## 1. 开发环境 vs 生产环境

### 1.1 环境对比

| 项目 | 开发环境 | 生产环境 |
|------|----------|----------|
| **目标** | 开发调试 | 稳定运行 |
| **数据** | 测试数据 | 真实数据 |
| **配置** | 调试模式 | 生产模式 |
| **日志** | 详细（DEBUG） | 重要（INFO/ERROR） |
| **性能** | 不重要 | 关键 |
| **安全** | 宽松 | 严格 |
| **重启** | 频繁 | 尽量避免 |

### 1.2 环境配置管理

**不同环境使用不同配置**：

```python
# config.py
import os

class Config:
    DEBUG = False
    DATABASE_URL = os.getenv('DATABASE_URL')
    SECRET_KEY = os.getenv('SECRET_KEY')

class DevelopmentConfig(Config):
    DEBUG = True
    DATABASE_URL = 'sqlite:///dev.db'
    SECRET_KEY = 'dev-secret-key-not-secure'

class ProductionConfig(Config):
    DEBUG = False
    # 从环境变量读取（安全）
    DATABASE_URL = os.getenv('DATABASE_URL')
    SECRET_KEY = os.getenv('SECRET_KEY')

# 根据环境变量选择配置
env = os.getenv('FLASK_ENV', 'development')
if env == 'production':
    config = ProductionConfig()
else:
    config = DevelopmentConfig()
```

**使用环境变量**：

```bash
# 开发环境
export FLASK_ENV=development
python app.py

# 生产环境
export FLASK_ENV=production
export DATABASE_URL=postgresql://user:pass@localhost/mydb
export SECRET_KEY=super-secret-production-key
python app.py
```

**使用 .env 文件**：

```bash
# .env.development
FLASK_ENV=development
DATABASE_URL=sqlite:///dev.db
DEBUG=True

# .env.production
FLASK_ENV=production
DATABASE_URL=postgresql://user:pass@prod-db.example.com/mydb
DEBUG=False
SECRET_KEY=xxx
```

```python
# 加载 .env
from dotenv import load_dotenv

load_dotenv('.env.production')  # 生产环境
# load_dotenv('.env.development')  # 开发环境
```

---

## 2. 服务器选择和配置

### 2.1 服务器类型

**1. VPS（虚拟专用服务器）**
- **优点**：完全控制、价格便宜
- **缺点**：需要自己运维
- **推荐**：DigitalOcean、Vultr、Linode
- **价格**：$5-20/月

**2. 云服务器**
- **优点**：弹性扩展、高可用
- **缺点**：价格较高、学习曲线陡
- **推荐**：AWS EC2、阿里云ECS、腾讯云CVM
- **价格**：$10-100+/月

**3. PaaS（平台即服务）**
- **优点**：零运维、快速部署
- **缺点**：灵活性低、价格高
- **推荐**：Heroku、Render、Railway
- **价格**：$0-50/月

**4. Serverless（无服务器）**
- **优点**：按需付费、无需管理服务器
- **缺点**：冷启动、执行时间限制
- **推荐**：AWS Lambda、Vercel、Netlify
- **价格**：按使用量计费

### 2.2 服务器配置

**初始配置**：

```bash
# 1. 更新系统
sudo apt update && sudo apt upgrade -y

# 2. 创建非 root 用户
sudo adduser deploy
sudo usermod -aG sudo deploy

# 3. 配置 SSH 密钥登录
ssh-copy-id deploy@your-server-ip

# 4. 禁用密码登录（更安全）
sudo nano /etc/ssh/sshd_config
# PasswordAuthentication no
sudo systemctl restart sshd

# 5. 安装基础软件
sudo apt install -y git curl wget vim
```

**安装运行时**：

```bash
# Python
sudo apt install -y python3 python3-pip python3-venv

# Node.js
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Go
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
```

---

## 3. Docker 容器化

### 3.1 为什么使用 Docker

**优势**：
- ✅ **环境一致**：开发、测试、生产完全相同
- ✅ **快速部署**：一条命令启动应用
- ✅ **隔离性**：不同应用互不影响
- ✅ **易于迁移**：随时迁移到其他服务器

### 3.2 Dockerfile

**Python Flask 应用**：

```dockerfile
# Dockerfile
FROM python:3.11-slim

# 设置工作目录
WORKDIR /app

# 复制依赖文件
COPY requirements.txt .

# 安装依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 暴露端口
EXPOSE 5000

# 启动命令
CMD ["python", "app.py"]
```

**Go 应用**：

```dockerfile
# Dockerfile
# 阶段1：编译
FROM golang:1.21 AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# 阶段2：运行（多阶段构建，减小镜像体积）
FROM alpine:latest

WORKDIR /app
COPY --from=builder /app/main .
COPY --from=builder /app/config ./config

EXPOSE 8080
CMD ["./main"]
```

**Node.js 应用**：

```dockerfile
# Dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000
CMD ["node", "index.js"]
```

### 3.3 Docker Compose

**管理多个容器**：

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Web 应用
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped

  # 数据库
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  # Redis
  redis:
    image: redis:7-alpine
    restart: unless-stopped

  # Nginx 反向代理
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - web
    restart: unless-stopped

volumes:
  postgres_data:
```

**启动应用**：

```bash
# 构建并启动
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止
docker-compose down

# 重启特定服务
docker-compose restart web
```

---

## 4. 进程管理

### 4.1 systemd

**创建 systemd 服务**：

```bash
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=deploy
WorkingDirectory=/home/deploy/myapp
Environment="FLASK_ENV=production"
Environment="DATABASE_URL=postgresql://..."
ExecStart=/home/deploy/myapp/venv/bin/python app.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**管理服务**：

```bash
# 启动服务
sudo systemctl start myapp

# 设置开机自启
sudo systemctl enable myapp

# 查看状态
sudo systemctl status myapp

# 重启
sudo systemctl restart myapp

# 查看日志
sudo journalctl -u myapp -f
```

### 4.2 Supervisor

**安装**：

```bash
sudo apt install supervisor
```

**配置**：

```ini
# /etc/supervisor/conf.d/myapp.conf
[program:myapp]
command=/home/deploy/myapp/venv/bin/python app.py
directory=/home/deploy/myapp
user=deploy
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/myapp/app.log
environment=FLASK_ENV="production",DATABASE_URL="postgresql://..."
```

**管理**：

```bash
# 重新加载配置
sudo supervisorctl reread
sudo supervisorctl update

# 启动
sudo supervisorctl start myapp

# 停止
sudo supervisorctl stop myapp

# 查看状态
sudo supervisorctl status
```

### 4.3 PM2（Node.js）

```bash
# 安装
npm install -g pm2

# 启动应用
pm2 start app.js --name myapp

# 查看状态
pm2 status

# 查看日志
pm2 logs myapp

# 重启
pm2 restart myapp

# 开机自启
pm2 startup
pm2 save
```

---

## 5. 反向代理和HTTPS

### 5.1 Nginx 反向代理

**配置文件**：

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name myapp.example.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 静态文件
    location /static {
        alias /home/deploy/myapp/static;
        expires 30d;
    }
}
```

**启用配置**：

```bash
# 创建符号链接
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# 测试配置
sudo nginx -t

# 重启 Nginx
sudo systemctl restart nginx
```

### 5.2 配置 HTTPS（Let's Encrypt）

**安装 Certbot**：

```bash
sudo apt install certbot python3-certbot-nginx
```

**获取证书**：

```bash
sudo certbot --nginx -d myapp.example.com
```

**Nginx 配置（自动更新）**：

```nginx
server {
    listen 80;
    server_name myapp.example.com;
    return 301 https://$server_name$request_uri;  # 强制 HTTPS
}

server {
    listen 443 ssl http2;
    server_name myapp.example.com;

    ssl_certificate /etc/letsencrypt/live/myapp.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**自动续期**：

```bash
# 测试续期
sudo certbot renew --dry-run

# Certbot 会自动创建 cron job 定期续期
```

---

## 6. 监控和日志

### 6.1 日志管理

**应用日志**：

```python
import logging
from logging.handlers import RotatingFileHandler

# 配置日志
handler = RotatingFileHandler(
    'app.log',
    maxBytes=10000000,  # 10MB
    backupCount=10      # 保留10个备份
)
handler.setFormatter(logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
))

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)
logger.addHandler(handler)
```

**集中日志管理**：

```bash
# 使用 Logrotate 管理日志
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 deploy deploy
}
```

### 6.2 监控工具

**1. 系统监控**：

```bash
# 安装 htop
sudo apt install htop

# 查看系统资源
htop
```

**2. 应用监控**：

```python
# 使用 psutil 监控
import psutil

cpu_percent = psutil.cpu_percent(interval=1)
memory = psutil.virtual_memory()
disk = psutil.disk_usage('/')

print(f"CPU: {cpu_percent}%")
print(f"内存: {memory.percent}%")
print(f"磁盘: {disk.percent}%")
```

**3. Prometheus + Grafana**（高级）：

```python
# 暴露 metrics
from prometheus_client import Counter, Histogram, start_http_server

REQUEST_COUNT = Counter('http_requests_total', 'Total HTTP requests')
REQUEST_DURATION = Histogram('http_request_duration_seconds', 'HTTP request duration')

@app.route('/api/users')
@REQUEST_DURATION.time()
def get_users():
    REQUEST_COUNT.inc()
    # ...

# 启动 metrics 服务器
start_http_server(9090)
```

### 6.3 告警

**简单的邮件告警**：

```python
import smtplib
from email.mime.text import MIMEText

def send_alert(subject, message):
    msg = MIMEText(message)
    msg['Subject'] = subject
    msg['From'] = 'alerts@example.com'
    msg['To'] = 'admin@example.com'

    with smtplib.SMTP('smtp.gmail.com', 587) as server:
        server.starttls()
        server.login('user', 'pass')
        server.send_message(msg)

# 使用
try:
    # 执行关键操作
    process_payment()
except Exception as e:
    send_alert('支付失败', f'错误: {e}')
```

---

## 7. 备份和恢复

### 7.1 数据库备份

**PostgreSQL**：

```bash
# 备份
pg_dump -U user -d mydb > backup_$(date +%Y%m%d_%H%M%S).sql

# 恢复
psql -U user -d mydb < backup_20240101_120000.sql
```

**MySQL**：

```bash
# 备份
mysqldump -u user -p mydb > backup_$(date +%Y%m%d_%H%M%S).sql

# 恢复
mysql -u user -p mydb < backup_20240101_120000.sql
```

**自动备份脚本**：

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/home/deploy/backups"
DATE=$(date +%Y%m%d_%H%M%S)
FILENAME="backup_$DATE.sql"

# 创建备份
pg_dump -U user mydb > "$BACKUP_DIR/$FILENAME"

# 压缩
gzip "$BACKUP_DIR/$FILENAME"

# 删除7天前的备份
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +7 -delete

echo "备份完成: $FILENAME.gz"
```

**定时任务（cron）**：

```bash
# 编辑 crontab
crontab -e

# 每天凌晨2点备份
0 2 * * * /home/deploy/backup.sh >> /var/log/backup.log 2>&1
```

### 7.2 文件备份

**使用 rsync**：

```bash
# 同步到远程服务器
rsync -avz /home/deploy/myapp/ user@backup-server:/backups/myapp/

# 排除某些目录
rsync -avz --exclude 'node_modules' --exclude '*.log' /home/deploy/myapp/ user@backup-server:/backups/myapp/
```

**使用云存储（AWS S3）**：

```bash
# 安装 AWS CLI
pip install awscli

# 配置
aws configure

# 上传备份
aws s3 cp backup.sql.gz s3://my-backups/myapp/backup_$(date +%Y%m%d).sql.gz
```

---

## 8. NOFX 部署实战

### 8.1 完整部署流程

**步骤1：准备服务器**

```bash
# SSH 登录服务器
ssh deploy@your-server-ip

# 克隆代码
git clone https://github.com/your-username/nofx.git
cd nofx
```

**步骤2：配置环境**

```bash
# 创建配置文件
cp config/config.example.yaml config/config.yaml
nano config/config.yaml

# 设置环境变量
export NOFX_CONFIG=/home/deploy/nofx/config/config.yaml
```

**步骤3：使用 Docker 部署**

```dockerfile
# Dockerfile
FROM golang:1.21 AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o nofx ./cmd

FROM alpine:latest

WORKDIR /app
COPY --from=builder /app/nofx .
COPY --from=builder /app/config ./config
COPY --from=builder /app/web ./web

EXPOSE 8080
CMD ["./nofx"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  nofx:
    build: .
    ports:
      - "8080:8080"
    volumes:
      - ./config:/app/config
      - ./logs:/app/logs
    restart: unless-stopped
    environment:
      - GIN_MODE=release
```

```bash
# 启动
docker-compose up -d

# 查看日志
docker-compose logs -f nofx
```

**步骤4：配置 Nginx**

```nginx
# /etc/nginx/sites-available/nofx
server {
    listen 80;
    server_name nofx.example.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";  # WebSocket 支持
    }
}
```

**步骤5：配置 HTTPS**

```bash
sudo certbot --nginx -d nofx.example.com
```

### 8.2 更新部署

**使用 Git 拉取更新**：

```bash
# 拉取最新代码
git pull origin main

# 重新构建
docker-compose build

# 重启服务
docker-compose down && docker-compose up -d
```

**自动化部署脚本**：

```bash
#!/bin/bash
# deploy.sh

echo "开始部署..."

# 拉取代码
git pull origin main

# 构建镜像
docker-compose build

# 停止旧容器
docker-compose down

# 启动新容器
docker-compose up -d

# 清理旧镜像
docker image prune -f

echo "部署完成！"

# 查看状态
docker-compose ps
```

---

## 9. 实战练习

### 练习 1：部署第21章的交易系统

将第21章的交易系统部署到云服务器。

**要求**：
- 使用 Docker
- 配置 Nginx 反向代理
- 配置 HTTPS
- 设置自动重启

### 练习 2：监控脚本

编写一个监控脚本，每分钟检查应用是否正常运行。

**要求**：
- 检查进程是否存在
- 检查端口是否监听
- 如果异常，发送告警邮件

```python
# monitor.py
import psutil
import requests
import smtplib

def check_app():
    # 检查进程
    # 检查端口
    # 检查 HTTP 响应
    pass

if __name__ == "__main__":
    check_app()
```

### 练习 3：自动备份

实现自动备份功能。

**要求**：
- 每天备份配置文件
- 备份日志文件
- 上传到云存储（S3 或 OSS）
- 保留最近7天的备份

---

## 本章总结

### 部署流程

1. **准备服务器**：选择合适的服务器类型
2. **配置环境**：安装依赖、配置环境变量
3. **容器化**：使用 Docker 打包应用
4. **进程管理**：使用 systemd/supervisor 管理进程
5. **反向代理**：使用 Nginx 配置 HTTPS
6. **监控日志**：配置日志和监控
7. **备份恢复**：定期备份数据

### 最佳实践

1. **环境隔离**：开发、测试、生产环境分离
2. **配置外部化**：使用环境变量
3. **容器化**：使用 Docker 保证环境一致
4. **自动化**：使用脚本自动化部署
5. **监控告警**：及时发现问题
6. **定期备份**：防止数据丢失

### 常用命令

```bash
# Docker
docker-compose up -d
docker-compose logs -f
docker-compose restart

# systemd
sudo systemctl status myapp
sudo systemctl restart myapp
sudo journalctl -u myapp -f

# Nginx
sudo nginx -t
sudo systemctl restart nginx

# 监控
htop
docker stats
tail -f /var/log/myapp/app.log
```

---

**💡 记住**：部署不是一次性的任务，而是持续的过程。确保有完善的监控、日志和备份机制！

**推荐工具**：
- **服务器**：DigitalOcean、Vultr
- **容器**：Docker、Docker Compose
- **监控**：Prometheus + Grafana
- **日志**：ELK Stack (Elasticsearch, Logstash, Kibana)
- **CI/CD**：GitHub Actions、GitLab CI
