# Docker Images Pusher

使用 Github Action 将国外的 Docker 镜像转存到阿里云私有仓库，供国内服务器使用。

- **多平台支持**：支持单个镜像标签（Tag）包含多个架构（Multi-Arch），不再产生繁琐的前缀。
- **全量同步**：默认同步源仓库所有可用的平台镜像。
- **极速分发**：利用 Docker Buildx 引擎，直接在仓库间同步 Manifest，无需下载镜像层到本地，极大节省时间和空间。
- **超大镜像**：支持最大 40GB 的大型镜像同步。

视频教程：https://www.bilibili.com/video/BV1Zn4y19743/

作者：**[技术爬爬虾](https://github.com/tech-shrimp/me)**

---

## 使用方式

### 1. 配置阿里云
1. 登录 [阿里云容器镜像服务](https://cr.console.aliyun.com/)。
2. 启用个人实例，创建一个**命名空间**（`ALIYUN_NAME_SPACE`）。
3. 在“访问凭证”中设置固定密码，并获取以下信息：
   - 用户名 (`ALIYUN_REGISTRY_USER`)
   - 密码 (`ALIYUN_REGISTRY_PASSWORD`)
   - 仓库地址 (`ALIYUN_REGISTRY`)：例如 `registry.cn-hangzhou.aliyuncs.com`

### 2. Fork 本项目
1. Fork 本项目到你的账号下。
2. 进入 `Settings -> Secrets and variables -> Actions -> New Repository secret`。
3. 配置以下四个环境变量：
   - `ALIYUN_REGISTRY`
   - `ALIYUN_NAME_SPACE`
   - `ALIYUN_REGISTRY_USER`
   - `ALIYUN_REGISTRY_PASSWORD`

### 3. 添加镜像
打开 `images.txt` 文件，添加你想要的镜像。
- **默认全量同步**：直接写镜像名，会同步该 Tag 下的所有平台架构。
- **指定平台同步**：使用 `--platform` 参数限制同步的架构（多个用逗号隔开）。
- **注释**：使用 `#` 开头。

**示例：**
```text
# 同步所有架构 (linux/amd64, linux/arm64, etc.)
nginx:latest

# 只同步指定的架构
--platform=linux/amd64,linux/arm64 node:22-bookworm-slim

# 同步特定私库镜像
gcr.io/kaniko-project/executor:latest
```
文件提交后，将自动触发同步流程。

---

## 平台支持说明

在使用 `--platform` 参数时，常用的合法字符串包括：

| 平台字符串 | 描述 |
| :--- | :--- |
| `linux/amd64` | 常见的 64 位 PC 架构 (x86_64) |
| `linux/arm64` | 64 位 ARM 架构 (如 Apple M1/M2, 华为鲲鹏) |
| `linux/arm/v7` | 32 位 ARM 架构 (如 树莓派 3B) |
| `linux/arm/v6` | 旧版 32 位 ARM (如 树莓派 Zero) |
| `linux/386` | 32 位 x86 架构 |
| `linux/ppc64le` | PowerPC 64 位架构 |
| `linux/s390x` | IBM System z 架构 |

### 错误处理
- **平台不存在**：如果指定的平台在源仓库中不存在，脚本会**报错并停止执行**。
- **部分存在**：如果你指定了多个平台（如 `amd64,arm64`），但源仓库只存在其中一个，脚本同样会**报错退出**，不会部分同步。

---

## 使用镜像
同步成功后，在阿里云后台将镜像仓库设为“公开”，即可直接拉取。

```bash
# 格式: docker pull [ALIYUN_REGISTRY]/[ALIYUN_NAME_SPACE]/[镜像名]:[标签]
docker pull registry.cn-hangzhou.aliyuncs.com/shrimp-images/node:22-bookworm-slim
```

---

## 镜像重名处理
如果 `images.txt` 中存在同名但不同命名空间的镜像，例如：
```text
xhofe/alist
xiaoyaliu/alist
```
脚本会自动在阿里云镜像名中添加源命名空间作为前缀，变为：
- `.../xhofe_alist:latest`
- `.../xiaoyaliu_alist:latest`

## 定时执行
修改 `.github/workflows/docker.yaml` 文件中的 `on:` 部分，添加 `schedule` 即可实现自动更新。
