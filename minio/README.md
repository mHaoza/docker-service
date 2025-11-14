# Minio 对象存储服务
Minio 是一款开源的对象存储服务，它兼容亚马逊 S3 接口，可以作为云存储服务使用。

## 目录结构
- `docker-compose.yaml`: MinIO 的编排定义
- `Dockerfile.minio`: MinIO 主服务的 Dockerfile
- `Dockerfile.backup`: 备份服务的 Dockerfile
- `Dockerfile.restore`: 恢复服务的 Dockerfile
- `.env`: 环境变量配置文件（包含 MinIO 管理员凭据）
- `data/`: 映射到容器内 `/data` 的持久化数据目录（桶、对象等）
- `backup/`: 备份数据存储目录（通过备份服务生成）

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

默认管理员凭据（来自 `.env` 文件，建议尽快修改）：
- 用户名：`admin`
- 密码：`rust123456`

> 注意：可以通过编辑 `.env` 文件来修改管理员凭据。

## 常用操作
- 创建存储桶（Bucket）：登录控制台 → Buckets → Create bucket
- 上传/下载对象：登录控制台 → 选择桶 → Upload/Download
- 访问策略：Buckets → 具体桶 → Access Policy

## 数据备份

### 手动热备份

使用 MinIO Client (mc) 工具执行热备份，备份过程中 MinIO 服务保持运行状态：

```bash
# 执行备份（备份到 ./backup 目录）
docker compose --profile tools run --rm backup
```

备份说明：
- 备份服务使用 `tools` profile，不会在 `docker compose up` 时自动启动
- 备份服务会自动等待 MinIO 服务就绪后再开始备份
- 使用 `mc mirror` 命令进行备份，支持增量更新
- 备份数据存储在 `./backup` 目录中
- 备份完成后容器会自动退出（`--rm` 参数会自动清理容器）

### 恢复数据

使用 MinIO Client (mc) 工具从备份恢复数据：

```bash
# 执行恢复（从 ./backup 目录恢复数据到 MinIO）
docker compose --profile tools run --rm restore
```

恢复说明：
- 恢复服务使用 `tools` profile，不会在 `docker compose up` 时自动启动
- 恢复服务会自动等待 MinIO 服务就绪后再开始恢复
- 使用 `mc mirror` 命令将备份数据恢复到 MinIO
- 恢复数据来源为 `./backup` 目录
- 恢复操作会覆盖 MinIO 中的现有数据，请谨慎操作
- 如果备份目录为空或不存在，恢复操作会失败并提示错误

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

## 搭配picList使用

1. 下载并安装 [picList](https://piclist.cn)
2. 配置AWS S3图床，参考：[配置AWS S3图床](https://piclist.cn/configure.html#%E5%86%85%E7%BD%AEaws-s3)

AWS S3图床参考配置如下：

- 配置名: 任意名称
- 设定AccessKeyld: MinIO的AccessKey
- 设定SecretAccessKey: MinIO的SecretAccessKey
- 设定Bucket： `blog`
- 设定上传路径: {year}/{month}/{day}/{md5}.{extName}
- 设定自定义节点: http://127.0.0.1:9000
- 启用s3ForcePathStyle: 勾选
- 设定上传资源的访问策略: public-read