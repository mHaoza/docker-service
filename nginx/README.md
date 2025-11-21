# Nginx 反向代理服务

这是一个基于 Docker 的 Nginx 反向代理配置，用于代理通过 FRP 内网穿透出来的服务。

## 目录结构

```
nginx/
├── docker-compose.yaml    # Docker Compose 配置文件
├── conf.d/                # Nginx 配置文件目录
│   └── default.conf       # 主配置文件
├── ssl/                   # SSL 证书目录
│   ├── .gitkeep
│   ├── iice.fun.crt       # 主站证书（需手动放置）
│   ├── iice.fun.key       # 主站私钥（需手动放置）
│   ├── img.iice.fun.crt   # 图床证书（需手动放置）
│   └── img.iice.fun.key   # 图床私钥（需手动放置）
├── .gitignore             # Git 忽略配置
└── README.md              # 本文件
```

## 代理服务说明

当前配置了两个反向代理服务：

| 域名         | 目标端口 | 用途 | 说明                                   |
| ------------ | -------- | ---- | -------------------------------------- |
| iice.fun     | 9900     | 主站 | 代理通过 FRP TCP 代理出来的网站        |
| img.iice.fun | 9901     | 图床 | 代理通过 FRP 内网穿透出来的 MinIO 图床 |

## 快速开始

### 1. 启动服务

```bash
# 启动 Nginx 服务
docker compose up -d

# 查看日志
docker compose logs -f

# 停止服务
docker compose down
```

### 2. 检查服务状态

```bash
# 查看容器状态
docker compose ps

# 查看 Nginx 配置是否正确
docker exec nginx nginx -t

# 重新加载配置（修改配置文件后）
docker exec nginx nginx -s reload
```

## SSL 证书配置

### 配置 HTTPS

1. **放置证书文件**

   将证书文件放到 `ssl/` 目录：

   ```bash
   # 主站证书
   ssl/iice.fun.crt
   ssl/iice.fun.key

   # 图床证书
   ssl/img.iice.fun.crt
   ssl/img.iice.fun.key
   ```

2. **启用 SSL 配置**

   编辑 `conf.d/default.conf`，取消注释 SSL 相关的 server 块：

   ```nginx
   server {
       listen 443 ssl http2;
       server_name iice.fun;
       ssl_certificate /etc/nginx/ssl/iice.fun.crt;
       ssl_certificate_key /etc/nginx/ssl/iice.fun.key;
       # ... 其他配置
   }
   ```

3. **重新加载配置**
   ```bash
   docker exec nginx nginx -t
   docker exec nginx nginx -s reload
   ```

## 添加新的代理配置

如果需要添加新的域名代理：

1. **编辑配置文件** `conf.d/default.conf`

2. **添加新的 server 块**：

```nginx
server {
    listen 80;
    server_name newdomain.com;

    access_log /var/log/nginx/newdomain_access.log;
    error_log /var/log/nginx/newdomain_error.log;

    location / {
        proxy_pass http://127.0.0.1:PORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

3. **重新加载配置**：

```bash
docker exec nginx nginx -t
docker exec nginx nginx -s reload
```

## 常见问题排查

### 1. 无法访问服务

**检查项：**

- FRP 服务是否正常运行
- 目标端口（9900/9901）是否正常监听
- DNS 解析是否正确指向服务器 IP
- 防火墙是否开放 80/443 端口

```bash
# 检查端口监听
netstat -tlnp | grep -E ':(80|443|9900|9901)'

# 检查 DNS 解析
nslookup iice.fun
nslookup img.iice.fun

# 测试本地连接
curl -I http://127.0.0.1:9900
curl -I http://127.0.0.1:9901
```

### 2. 502 Bad Gateway

**可能原因：**

- 后端服务（9900/9901）未启动
- 防火墙阻止了内部通信
- 使用了 `network_mode: host`，确保后端服务监听在 127.0.0.1 或 0.0.0.0

**解决方法：**

```bash
# 查看 Nginx 错误日志
docker compose logs nginx

# 检查后端服务
curl http://127.0.0.1:9900
curl http://127.0.0.1:9901
```

### 3. 配置文件修改后未生效

```bash
# 检查配置语法
docker exec nginx nginx -t

# 重新加载配置
docker exec nginx nginx -s reload

# 如果还不行，重启容器
docker compose restart
```

### 4. 查看日志

```bash
# 查看 Nginx 容器日志
docker compose logs -f nginx

# 进入容器查看访问日志
docker exec -it nginx tail -f /var/log/nginx/access.log

# 查看错误日志
docker exec -it nginx tail -f /var/log/nginx/error.log
```

## 维护建议

1. **定期检查日志**

   ```bash
   docker compose logs --tail=100 nginx
   ```

2. **监控磁盘空间**（日志文件可能占用空间）

3. **SSL 证书续期**
   - Let's Encrypt 证书有效期 90 天
   - 建议设置自动续期

## 相关服务

- [FRP 服务配置](../frp/README.md)
- [MinIO 图床配置](../minio/README.md)

## 参考资源

- [Nginx 官方文档](https://nginx.org/en/docs/)
- [Nginx 反向代理配置指南](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
