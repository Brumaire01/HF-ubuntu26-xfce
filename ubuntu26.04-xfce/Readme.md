# Ubuntu Web Desktop (Docker)

基于 [linuxserver/webtop](https://github.com/linuxserver/docker-webtop) 的 Ubuntu XFCE 云端桌面环境，通过浏览器即可使用完整的 Linux 桌面。

## 特性

- **Ubuntu 26.04 LTS** + **XFCE** 轻量级桌面环境
- **简体中文** 系统语言，开箱即用
- **Fcitx5 中文输入法**：拼音 + 五笔（86大字集 / 98版五笔拼音混输）
- **中文字体**：Noto CJK、文泉驿微米黑、文泉驿正黑
- **Web 访问**：浏览器打开即用，无需安装任何客户端
- **自定义首页**：7860 端口提供导航首页，通过二级目录 `/desktop/` 进入桌面
- **共享内存 4GB**：适合运行大型 GUI 应用（如浏览器、IDE）

## 快速启动

### 1. 构建镜像

```bash
cd docker-ubuntu
docker build -t ubuntu-console:latest .
```

### 2. 运行容器

```bash
docker run -d \
  --name ubuntu-web \
  -p 7860:7860 \
  -e PUID=0 \
  -e PGID=0 \
  -e TZ=Asia/Shanghai \
  --shm-size=4gb \
  ubuntu-console:latest
```
```bash
docker run -d --name ubuntu-web -p 7860:7860 -e PUID=0 -e PGID=0 -e TZ=Asia/Shanghai --shm-size=4gb ubuntu-console:latest # 快速测试
```

> **说明**：
> - `PUID=0 / PGID=0`：以 root 用户运行桌面（生产环境建议改为普通用户 UID/GID）
> - `TZ=Asia/Shanghai`：设置时区为北京时间
> - `--shm-size=4gb`：增大共享内存，保证 Chromium/Firefox 等应用正常运行
> - `CUSTOM_USER`：Web 认证自定义用户名（可留空，留空则无认证）
> - `PASSWORD`：Web 认证自定义密码（可留空，留空则无认证）

### 3. 访问

| 地址 | 说明 |
|------|------|
| `http://<服务器IP>:7860/` | 导航首页 |
| `http://<服务器IP>:7860/.console/` | XFCE 桌面环境 |

## 输入法使用

进入桌面后，打开终端执行以下命令启动输入法：

```bash
fcitx5 &
```

- **切换中英文**：`Ctrl + Space`
- **切换输入法**：点击系统托盘中的 fcitx5 图标，或右键进行配置
- 默认启用**拼音**输入法，可在 fcitx5 配置中切换为五笔

## 可用环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `PUID` | `911` | 容器内运行用户 UID |
| `PGID` | `911` | 容器内运行用户 GID |
| `TZ` | `UTC` | 时区，建议设为 `Asia/Shanghai` |
| `CUSTOM_PORT` | `3000` | 桌面 Web 服务端口（内部） |
| `PASSWORD` | (空) | 设置后访问桌面需 HTTP 基本认证 |

更多变量请参考 [linuxserver/webtop 文档](https://github.com/linuxserver/docker-webtop)。

## 文件结构

```
docker-ubuntu/
├── Dockerfile              # 镜像构建文件
├── index.html              # 7860 端口首页
├── proxy-server.conf       # Nginx 7860 端口 server 配置
├── init-nginx-run          # 覆盖 webtop 的 init-nginx 脚本
└── Readme.md               # 本文件
```

## 架构说明

- Nginx 监听 **7860** 端口，提供首页 + 桌面反代
- 首页（`/`）展示欢迎页，引导用户进入桌面
- 桌面（`/.console/`）反向代理到 `127.0.0.1:3000`（webtop 内建的 KasmVNC Web 服务）
- 原 webtop 的 3000 端口逻辑不受影响
