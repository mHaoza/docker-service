# FRP 内网穿透 Docker 一键启动（单文件·Profile 隔离）

本项目使用 **Docker Compose** 运行最新版 FRP（v0.61+），支持 **TOML** 格式配置。  
通过 **profile 机制**强制「显式选择」启动服务端或客户端，彻底防止误操作一次性拉起全部容器。

---

## 文件结构

```
.
├── docker-compose.yaml       # 已内置 profile 隔离
├── frps.example.toml        # 服务端示例配置
├── frpc.example.toml        # 客户端示例配置
└── README.md
```

---

## 1. 准备配置

```bash
# 公网服务器：生成服务端配置
cp frps.example.toml frps.toml
vim frps.toml               # 修改 token、端口等

# 内网机器：生成客户端配置
cp frpc.example.toml frpc.toml
vim frpc.toml               # 修改 serverAddr、token、本地映射等
```

---

## 2. 按需启动（必须带 profile）

```bash
# 仅启动服务端
docker compose --profile server up -d

# 仅启动客户端
docker compose --profile client up -d

# 查看状态
docker compose ps

# 实时日志
docker compose logs -f frps   # 或 frpc
```

> 直接 `docker compose up` **不会启动任何服务**，避免误操作。

---

## 3. 停止 / 重启 / 升级

```bash
# 停止并删除容器（保留 *.toml）
docker compose down

# 更新镜像后重新启动
docker compose pull
docker compose --profile <server|client> up -d
```

---

## 4. 默认端口说明

| 端口      | 作用             | 需防火墙放行 |
| --------- | ---------------- | ------------ |
| 7000      | 控制通信         | ✅            |
| 7500      | 管理面板（可选） | 按需         |
| 6000-6100 | 示例预留映射段   | 按需         |

管理面板：  
浏览器访问 `http://<服务器IP>:7500`  
默认账号/密码：`admin / admin`（请在 `frps.toml` 中修改）

---

## 5. 常见问题
1. **客户端连不上服务端**  
   → 检查服务器安全组是否放行 7000；确认 `auth.token` 一致。
2. **想暴露更多本地端口**  
   → 在 `frpc.toml` 继续添加 `[[proxies]]` 段落即可。

---

## 6. 参考链接

* FRP 官方仓库：<https://github.com/fatedier/frp>  
* TOML 语法速查：<https://toml.io/cn/>
