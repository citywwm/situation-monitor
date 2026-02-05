# Situation Monitor Docker 部署手册（Step-by-Step）

本文档说明如何在本地或服务器上，通过 Docker 与 Docker Compose 一键部署 `situation-monitor`。

## 1. 先决条件

- 一台可访问终端的机器（Linux/macOS/Windows WSL2）
- 已安装 Git
- 已安装 Docker Engine
- 已安装 Docker Compose Plugin（`docker compose`）

> 建议 Docker 版本：24+，Compose 版本：v2+

---

## 2. 安装 Docker（Ubuntu 示例）

如果你已经安装 Docker，可跳到第 3 步。

### 2.1 卸载旧版本（可选）

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 2.2 安装依赖

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
```

### 2.3 添加 Docker 官方 GPG Key 与源

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 2.4 安装 Docker Engine + Compose

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 2.5 验证安装

```bash
docker --version
docker compose version
```

### 2.6 （可选）免 sudo 使用 Docker

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## 3. 获取项目代码

```bash
git clone <你的仓库地址> situation-monitor
cd situation-monitor
```

如果你已在项目目录，可直接进入下一步。

---

## 4. 构建并启动容器

项目根目录已包含以下 Docker 文件：

- `Dockerfile`
- `docker-compose.yml`
- `.dockerignore`
- `nginx/default.conf`

执行：

```bash
docker compose up -d --build
```

说明：

- `--build`：首次或更新代码后重新构建镜像
- `-d`：后台运行

---

## 5. 访问应用

默认将容器 `80` 端口映射到主机 `8080` 端口。

浏览器打开：

```text
http://localhost:8080
```

如果是远程服务器，请改成服务器 IP 或域名：

```text
http://<SERVER_IP>:8080
```

---

## 6. 常用运维命令

### 查看容器状态

```bash
docker compose ps
```

### 查看日志

```bash
docker compose logs -f
```

### 重启服务

```bash
docker compose restart
```

### 停止并删除容器（不删镜像）

```bash
docker compose down
```

### 停止并删除容器 + 镜像（谨慎）

```bash
docker compose down --rmi all
```

---

## 7. 更新版本流程（推荐）

当代码有更新时：

```bash
git pull
docker compose up -d --build
```

---

## 8. 故障排查

### 8.1 端口冲突（8080 被占用）

修改 `docker-compose.yml`：

```yaml
ports:
  - "9090:80"
```

然后重启：

```bash
docker compose up -d --build
```

### 8.2 页面 404（SPA 路由）

本项目已在 `nginx/default.conf` 中配置 `try_files $uri $uri/ /app.html;`，用于处理前端路由直达刷新场景。

### 8.3 容器启动失败

```bash
docker compose logs -f
```

根据日志重点查看：

- Node 构建阶段报错
- 静态文件是否生成在 `/app/build`
- Nginx 配置语法是否正确

---

## 9. 生产环境建议

- 通过 Nginx/Traefik 做 HTTPS 反向代理
- 配置镜像仓库（如 Harbor / Docker Hub / GHCR）
- 建议在 CI/CD 中执行：`docker build` + 安全扫描 + 自动部署
- 使用监控与告警（容器重启、CPU/内存、可用性探测）

