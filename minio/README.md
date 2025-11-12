# Minio 对象存储服务
Minio 是一款开源的对象存储服务，它兼容亚马逊 S3 接口，可以作为云存储服务使用。

## 目录结构
- `docker-compose.yml`: MinIO 的编排定义
- `data/`: 映射到容器内 `/data` 的持久化数据目录（桶、对象等）

## 先决条件
- 已安装 `Docker` 与 `Docker Compose`（v2 语法，命令为 `docker compose`）

## 快速开始
在当前目录执行：

```bash
# 后台启动: -d 后台运行，不进入容器
docker compose up -d

# 查看运行状态
docker compose ps
```

停止与移除容器（不删除数据）：

```bash
docker compose down
```

如需同时清除匿名网络、已停止容器等资源（仍不删除 `./data` 内数据）：

```bash
docker compose down --remove-orphans
```

## 访问方式
- API 端口（S3 兼容）：`http://localhost:9000`
- 控制台（Web Console）：`http://localhost:9001`

默认管理员凭据（来自 `docker-compose.yml` 的环境变量，建议尽快修改）：
- 用户名：`admin`
- 密码：`rust123456`

## 常用操作
- 创建存储桶（Bucket）：登录控制台 → Buckets → Create bucket
- 上传/下载对象：登录控制台 → 选择桶 → Upload/Download
- 访问策略：Buckets → 具体桶 → Access Policy

## 自定义配置

### 图床Buckets的Access Policy设置
设置为custom，并添加以下内容：
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "*"
                ]
            },
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::blog/*"
            ]
        }
    ]
}
```
> 注意：此处的 `blog` 请替换为你自己的 Bucket 名称。
> 访问时路径参考： `http://localhost:9000/bucket/hello.txt`