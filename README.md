![42 Badge](https://img.shields.io/badge/42-Project-000000?style=flat&logo=42&logoColor=white)
![Score](https://img.shields.io/badge/Score-125%2F100-brightgreen?style=flat)
![Language](https://img.shields.io/badge/Language-Docker%20%7C%20Shell%20%7C%20YAML-blue?style=flat)
![OS](https://img.shields.io/badge/OS-Debian%20Bookworm-A81D33?style=flat&logo=debian&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat)
![Bonus](https://img.shields.io/badge/Bonus-5%2F5-blueviolet?style=flat)

# 🐳 Inception — 42 Project

A system administration project using Docker and Docker Compose to set up a multi-container infrastructure inside a Virtual Machine.

---

## Architecture

```
Internet → NGINX (443/TLS) → WordPress + PHP-FPM (9000) → MariaDB (3306)
                                        ↕
                                      Redis (6379)

Bonus services:
  Static Website (8081) | Adminer (8080) | Portainer (9000) | FTP (21)
```

All containers communicate through a Docker bridge network. Only NGINX exposes port 443 to the host.

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Base Image | Debian 12 (Bookworm) — penultimate stable |
| Web Server | NGINX with TLSv1.3 |
| CMS | WordPress + PHP-FPM 8.2 |
| Database | MariaDB |
| Cache | Redis |
| Orchestration | Docker Compose |
| Secrets | Docker Secrets |
| Automation | Makefile + Shell scripts |

---

## Project Structure

```
inception/
├── Makefile                  # Entry point: make builds everything
├── README.md                 # Project description (English)
├── USER_DOC.md               # User documentation
├── DEV_DOC.md                # Developer documentation
├── secrets/                  # Passwords (git-ignored)
│   ├── db_password.txt
│   ├── db_root_password.txt
│   ├── wp_admin_password.txt
│   └── wp_user_password.txt
└── srcs/
    ├── .env                  # Environment variables (git-ignored)
    ├── docker-compose.yml    # Service definitions
    └── requirements/
        ├── nginx/
        ├── wordpress/
        ├── mariadb/
        └── bonus/
            ├── redis/
            ├── ftp/
            ├── website/
            ├── adminer/
            └── portainer/
```

---

## Key Concepts

### Docker vs Virtual Machines

| | Docker Container | Virtual Machine |
|---|---|---|
| Isolation | Process-level (shared kernel) | Hardware-level (own kernel) |
| Size | MBs | GBs |
| Startup | Seconds | Minutes |
| Overhead | Minimal | Significant |

### Docker Compose

Orchestrates multiple containers from a single YAML file. Manages service dependencies, networks, volumes, and secrets. One command (`docker compose up`) starts the entire infrastructure.

### Dockerfile

A text file with instructions to build a Docker image. Each instruction creates a layer. This project builds all images from `debian:bookworm` — no pre-built images from DockerHub allowed.

### PID 1

The main process in a container has PID 1. The container lives as long as PID 1 runs. Services must run in the foreground using `exec`:

```bash
exec nginx -g "daemon off;"    # Correct: nginx IS PID 1
exec php-fpm8.2 -F             # Correct: php-fpm IS PID 1
```

Hacks like `tail -f`, `sleep infinity`, `while true` are forbidden.

### Docker Network

An isolated virtual network (bridge mode) connecting all containers. Containers resolve each other by service name (e.g., WordPress connects to `mariadb:3306`). Only NGINX exposes ports to the host.

### Docker Volumes vs Bind Mounts

- **Named Volumes**: Managed by Docker, portable, persist across container lifecycle. Used in this project.
- **Bind Mounts**: Direct host path mapping. Forbidden by the subject for primary storage.

Data is stored at `/home/hguo/data/` on the host, surviving container rebuilds and VM reboots.

### Docker Secrets vs Environment Variables

- **Environment variables**: Visible via `docker inspect`, less secure.
- **Docker Secrets**: Mounted as files at `/run/secrets/` inside containers, never exposed in logs or inspect.

### TLS/SSL

NGINX uses TLSv1.3 with a self-signed certificate. Encrypts communication between browser and server, ensuring confidentiality, integrity, and authentication.

---

## Services

### Mandatory

| Service | Role | Port |
|---------|------|------|
| NGINX | Reverse proxy, TLS termination, sole entry point | 443 |
| WordPress + PHP-FPM | CMS, processes PHP via FastCGI | 9000 (internal) |
| MariaDB | Database for WordPress | 3306 (internal) |

### Bonus

| Service | Role | Port |
|---------|------|------|
| Redis | Object cache for WordPress, reduces DB queries | 6379 (internal) |
| FTP (vsftpd) | File transfer to WordPress volume | 21 |
| Static Website | HTML/CSS showcase site (no PHP) | 8081 |
| Adminer | Lightweight database management UI | 8080 |
| Portainer | Docker container management UI | 9000 |

---

## Subject Rules (Quick Reference)

| Rule | Reason |
|------|--------|
| No `FROM nginx` / `FROM wordpress` | Must build images from scratch |
| No `FROM debian:latest` | Latest tag is forbidden |
| No `network: host` | Must use isolated network |
| No `links:` / `--link` | Deprecated connection method |
| No `tail -f` / `sleep infinity` / `while true` | Fake keep-alive; main process must be PID 1 |
| No passwords in Dockerfiles | Use env vars or secrets |
| No credentials in git repo | secrets/ and .env must be in .gitignore |
| Admin username cannot contain "admin" | Used `hguo_boss` instead |
| No bind mounts for main storage | Must use named volumes |

---

## Commands

```bash
# Project management
make              # Build and start all containers
make down         # Stop all containers
make re           # Rebuild and restart
make clean        # Stop and remove volumes
make fclean       # Full cleanup including images

# Docker
docker ps         # List running containers
docker images     # List images
docker network ls # List networks
docker volume ls  # List volumes
docker logs nginx # View container logs

# Verification
curl -vk https://hguo.42.fr 2>&1 | grep TLS   # Check TLS
docker exec redis redis-cli ping                # Redis health check
docker exec -it mariadb mysql -u root           # Must be denied
```

---

## Debian Version Reference

| Version | Codename | Status (2026) |
|---------|----------|---------------|
| Debian 11 | Bullseye | oldoldstable |
| Debian 12 | Bookworm | oldstable (penultimate stable) ✅ used |
| Debian 13 | Trixie | stable (current) |
| Debian 14 | Forky | testing |

Subject requires "penultimate stable version" → Bookworm is the correct choice.

---
---

# 🇨🇳 中文学习笔记

---

# Inception 学习笔记（自用复习）

---

## 项目概述

Inception 是 42 的系统管理项目，用 Docker 和 Docker Compose 在虚拟机中搭建一个多容器基础设施。

**核心架构：**
```
外部访问 → NGINX (443/TLS) → WordPress+PHP-FPM (9000) → MariaDB (3306)
                                     ↕
                                   Redis (6379)
```

所有容器通过 Docker bridge 网络 `inception` 互相通信，只有 NGINX 暴露端口到宿主机。

---

## 目录结构

```
inception/
├── Makefile                  # 入口：make 启动一切
├── README.md                 # 项目说明（英文，有固定格式要求）
├── USER_DOC.md               # 用户文档
├── DEV_DOC.md                # 开发者文档
├── test.sh                   # 自动化检查脚本（答辩辅助）
├── secrets/                  # 密码文件（.gitignore 忽略）
│   ├── db_password.txt
│   ├── db_root_password.txt
│   ├── wp_admin_password.txt
│   └── wp_user_password.txt
└── srcs/
    ├── .env                  # 环境变量（.gitignore 忽略）
    ├── docker-compose.yml    # 核心配置文件
    └── requirements/
        ├── nginx/            # NGINX 容器
        ├── wordpress/        # WordPress + PHP-FPM 容器
        ├── mariadb/          # MariaDB 容器
        └── bonus/
            ├── redis/        # Redis 缓存
            ├── ftp/          # FTP 服务器
            ├── website/      # 静态网站
            ├── adminer/      # 数据库管理
            └── portainer/    # Docker 管理面板
```

---

## 核心概念

### Docker 是什么

Docker 是容器化工具。容器把应用和依赖打包到隔离环境中运行。

**容器 vs 虚拟机：**
- VM：虚拟化整个硬件 + 完整 OS → 几 GB，启动几分钟
- 容器：共享宿主机内核 → 几 MB，启动几秒
- VM 隔离性更好（独立内核），容器更轻量

**类比：** VM 像独立公寓（自己的水电），容器像合租房（共享基础设施但房间隔离）

### Docker Compose 是什么

多容器编排工具。用一个 YAML 文件（docker-compose.yml）定义所有服务、网络、volumes，一条命令 `docker compose up` 启动全部。

**有 vs 没有 Compose：**
- 没有：每个容器用 `docker run` 手动指定端口、网络、环境变量，逐个启动
- 有：配置集中在 YAML 文件，`depends_on` 管理启动顺序，一键启动/停止

### Dockerfile 是什么

构建 Docker 镜像的指令文件。每条指令创建一层 layer。

```dockerfile
FROM debian:bookworm          # 基础镜像（Debian 12，penultimate stable）
RUN apt-get install -y nginx  # 安装软件
COPY conf/nginx.conf /etc/... # 复制配置文件
EXPOSE 443                    # 声明端口
ENTRYPOINT ["/init.sh"]       # 容器启动时执行的命令
```

**关键规则：**
- `FROM` 只能用 `debian:bookworm` 或 `alpine`，不能用现成镜像（`FROM nginx` 禁止）
- 不能用 `FROM debian:latest`（latest 标签被禁止）
- 不能在 Dockerfile 中写明文密码

### Docker Image vs Container

- Image（镜像）= 只读模板，类似编译好的二进制文件
- Container（容器）= 镜像的运行实例，类似运行中的进程
- 一个镜像可以启动多个容器

### PID 1

容器里的主进程 PID 为 1。容器的生命周期 = PID 1 的生命周期。

**正确做法：** 用 `exec` 让服务成为 PID 1
```bash
exec nginx -g "daemon off;"   # nginx 是 PID 1，前台运行
exec php-fpm8.2 -F            # php-fpm 是 PID 1
exec mysqld --user=mysql       # mysqld 是 PID 1
```

**错误做法（Subject 禁止）：**
```bash
nginx & tail -f /dev/null     # tail 是 PID 1，nginx crash 了容器不知道
sleep infinity                 # 假保活
while true; do sleep 1; done   # 无限循环
```

---

## 网络

### Docker Network

容器之间的虚拟隔离网络。本项目用 bridge 类型。

```yaml
networks:
  inception:
    driver: bridge
```

**特点：**
- 容器用服务名互相访问（WordPress 连 `mariadb:3306`）
- 与宿主机隔离，只有显式映射的端口可从外部访问
- `network: host`（共享宿主机网络）被 Subject 禁止
- `links:` 和 `--link`（旧版连接方式）被 Subject 禁止

### TLS/SSL

加密浏览器和服务器之间的通信。

- TLSv1.3 = 最新版本，更快更安全
- 本项目用自签名证书（开发环境），浏览器会显示警告
- 生产环境用 Let's Encrypt 等 CA 颁发的证书

**验证命令：**
```bash
curl -vk https://hguo.42.fr 2>&1 | grep TLS
```

---

## 存储

### Docker Volumes

持久化存储。容器删了数据还在。

**Named Volume vs Bind Mount：**
- Named Volume：Docker 管理，可移植，本项目使用
- Bind Mount：直接映射宿主机路径，Subject 禁止用于主存储

本项目的 volume 配置用 named volume + driver_opts 指向宿主机目录：

```yaml
volumes:
  db_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /home/hguo/data/db      # 宿主机实际存储位置
  wp_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /home/hguo/data/wordpress
```

**持久化原理：** 数据在 `/home/hguo/data/`，容器删了这个目录不会删。重建容器后 volume 重新挂载，数据恢复。

### Docker Secrets

安全存储敏感信息（密码、API key）。

- 以文件形式挂载到容器内 `/run/secrets/`
- 不暴露在环境变量、日志、docker inspect 中
- 比 `.env` 文件更安全

```yaml
# docker-compose.yml 中定义
secrets:
  db_password:
    file: ../secrets/db_password.txt

# 容器中读取
services:
  mariadb:
    secrets:
      - db_password
```

```bash
# init.sh 中读取密码
MYSQL_PASSWORD=$(cat /run/secrets/db_password)
```

**Secrets vs 环境变量：**
- 环境变量：能用 `docker inspect` 看到，不安全
- Secrets：只在容器内以文件存在，更安全

---

## 各服务详解

### NGINX（主入口）

- 唯一暴露端口的 mandatory 容器（443）
- 处理 TLS 加密
- 反向代理：把 PHP 请求转发给 WordPress 容器的 php-fpm（9000端口）
- 不监听 80 端口

```nginx
# nginx.conf 关键配置
ssl_protocols TLSv1.2 TLSv1.3;
fastcgi_pass wordpress:9000;      # 转发 PHP 到 WordPress 容器
```

### WordPress + PHP-FPM

- PHP-FPM = FastCGI Process Manager，处理 PHP 代码
- 不装 nginx（Subject 要求分离）
- 用 WP-CLI 自动安装 WordPress
- 监听 9000 端口（FastCGI 协议，不是 HTTP）
- 两个用户：hguo_boss（管理员，不含 admin）、hguo_user（普通用户）

### MariaDB

- MySQL 兼容的数据库
- root 必须设密码（无密码登录必须被拒绝）
- 普通用户 wpuser 能连接 wordpress 数据库

**常用 SQL 命令：**
```bash
docker exec -it mariadb mysql -u wpuser -p   # 登录
```
```sql
SHOW DATABASES;        -- 查看所有数据库
USE wordpress;         -- 切换到 wordpress 数据库
SHOW TABLES;           -- 查看所有表
```

### Redis（Bonus）

- 内存缓存数据库
- WordPress 用它缓存数据库查询结果，加速页面加载
- 验证：`docker exec redis redis-cli ping` → 返回 `PONG`

### FTP（Bonus）

- vsftpd 文件传输服务器
- 指向 WordPress volume，可直接上传/下载文件
- 验证：`curl ftp://ftpuser:密码@127.0.0.1/`

### Static Website（Bonus）

- 纯 HTML/CSS，不能用 PHP（Subject 要求）
- 独立容器，独立 nginx
- 端口 8081

### Adminer（Bonus）

- 网页版数据库管理工具
- 一个 PHP 文件，轻量替代 phpMyAdmin
- 端口 8080

### Portainer（Bonus - 自选服务）

- Docker 图形管理面板
- 在浏览器中管理容器、查看日志、重启服务
- 端口 9000
- 选择理由：对多容器环境的监控很有价值

---

## Subject 禁止项速查

| 禁止项 | 原因 |
|--------|------|
| `FROM nginx` / `FROM wordpress` 等 | 必须自己构建镜像 |
| `FROM debian:latest` | 禁止用 latest 标签 |
| `network: host` | 必须用隔离网络 |
| `links:` / `--link` | 旧版连接方式已弃用 |
| `tail -f` / `sleep infinity` / `while true` | 假保活，主进程必须前台运行 |
| Dockerfile 中明文密码 | 用环境变量或 secrets |
| Git repo 中存密码 | secrets/ 和 .env 必须在 .gitignore |
| 管理员用户名含 admin/Admin | 用 hguo_boss 等替代 |
| Bind mount 做主存储 | 必须用 named volume |

---

## 常用命令

### 项目管理
```bash
make              # 构建并启动所有容器
make down         # 停止所有容器
make re           # 重建并重启
make clean        # 停止并删除 volumes
make fclean       # 完全清理（含镜像）
```

### Docker 命令
```bash
docker ps                        # 查看运行中的容器
docker images                    # 查看所有镜像
docker network ls                # 查看网络
docker volume ls                 # 查看 volumes
docker volume inspect srcs_wp_data  # 查看 volume 详情
docker port nginx                # 查看端口映射
docker logs nginx                # 查看容器日志
docker exec -it mariadb bash     # 进入容器 shell
```

### 验证命令
```bash
bash test.sh                                    # 一键验证所有检查项
curl -vk https://hguo.42.fr 2>&1 | grep TLS    # 检查 TLS
curl http://hguo.42.fr                          # 应该无响应
docker exec redis redis-cli ping                # Redis 健康检查
docker exec -it mariadb mysql -u root           # 应该被拒绝
```

### 图形界面
```bash
startx                           # 启动图形界面（在 VirtualBox 窗口中）
# 右键桌面 → Terminal
firefox https://hguo.42.fr &     # 打开浏览器
pkill X                          # 退出图形界面
```

---

## 答辩流程

1. **Preliminaries** — 评审员运行清理命令 → 你运行 `make`
2. **Project Overview** — 口头解释 Docker、Compose、vs VM、目录结构
3. **Simple Setup** — NGINX 只开 443、TLS、WordPress 已安装、HTTP 不通
4. **Docker Basics** — 每个服务有 Dockerfile、FROM debian:bookworm、镜像名=服务名
5. **Docker Network** — docker network ls、解释 bridge 网络
6. **NGINX** — Dockerfile、容器运行、HTTP 不通、TLS 证书
7. **WordPress** — Dockerfile 无 nginx、volume 路径、添加评论、管理员不含 admin、编辑页面
8. **MariaDB** — Dockerfile 无 nginx、volume 路径、root 无密码被拒绝、普通用户能登录
9. **Persistence** — 重启 VM → make → 数据还在
10. **Bonus** — 逐个验证 Redis/FTP/静态网站/Adminer/Portainer

---

## Debian 版本对照

| 版本号 | 代号 | 状态（2026年） |
|--------|------|----------------|
| Debian 11 | bullseye | oldoldstable |
| Debian 12 | bookworm | oldstable（penultimate stable）✅ 项目使用 |
| Debian 13 | trixie | stable（当前最新稳定版） |
| Debian 14 | forky | testing |

Subject 要求 "penultimate stable version" → bookworm 完全符合。
