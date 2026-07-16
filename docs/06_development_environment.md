# 开发环境部署记录

本文记录 WMS-AGV 项目进入阶段 1 前的本机开发环境部署步骤。目标是让开发、运行、容器服务使用同一套清晰环境，减少后续 FastAPI、PostgreSQL、Docker Compose 调试时的差异。

## 环境选择

推荐组合：

```text
PyCharm 负责写代码
WSL Ubuntu-24.04 负责运行 Python
Docker Desktop 负责运行容器服务
```

这样做的原因：

- PyCharm 适合写代码、看项目结构、调试、使用 Git。
- WSL Ubuntu 的 Python 环境更接近 Docker 容器里的 Linux 运行环境。
- 后续 PostgreSQL、Docker Compose、脚本命令、日志路径都更适合统一放在 Linux 侧处理。
- 避免 Windows Python、WSL Python、Docker Python 三套环境混在一起导致“本地能跑，容器里不行”。

项目目录暂时继续使用：

```text
D:\wms_agv
```

在 WSL 中对应路径是：

```bash
/mnt/d/wms_agv
```

## 已确认的基础环境

本机环境已经确认过：

- Windows 支持 WSL 2。
- CPU 虚拟化已开启。
- Docker Desktop 已安装。
- Docker Engine 可以正常响应。
- Docker Compose 可以正常响应。
- WSL 默认发行版已切换到 Ubuntu-24.04。
- Ubuntu-24.04 版本为 Ubuntu 24.04 LTS。
- Ubuntu 中 Python 版本为 Python 3.12.3。

如果需要再次检查 WSL 发行版：

```powershell
wsl -l -v
```

预期能看到类似结果：

```text
* Ubuntu-24.04    Running/Stopped    2
  docker-desktop  Running/Stopped    2
```

## WSL Ubuntu 初始化

进入 Ubuntu-24.04：

```powershell
wsl -d Ubuntu-24.04
```

第一次进入时需要创建 Linux 用户名和密码。密码输入时不会显示，这是正常现象。

进入后更新软件源并安装 Python 开发工具：

```bash
sudo apt update
sudo apt install -y python3-pip python3-venv git curl
```

检查 Python：

```bash
python3 --version
python3 -m pip --version
```

## Docker Desktop WSL 集成

打开 Docker Desktop，进入：

```text
Settings -> Resources -> WSL Integration
```

开启：

```text
Enable integration with my default WSL distro
Ubuntu-24.04
```

然后点击：

```text
Apply & Restart
```

在 Ubuntu-24.04 中检查：

```bash
docker --version
docker compose version
```

如果能看到版本号，说明 Docker 已经可以从 WSL 中调用。

## 项目虚拟环境

进入项目目录：

```bash
cd /mnt/d/wms_agv
```

创建项目虚拟环境：

```bash
python3 -m venv .venv
```

启用虚拟环境：

```bash
source .venv/bin/activate
```

启用成功后，命令行前面会出现：

```text
(.venv)
```

检查虚拟环境中的 Python 和 pip：

```bash
python --version
pip --version
```

## 阶段 1 基础依赖

阶段 1-1 的目标是先搭建 FastAPI 后端骨架，并提供 `/health` 健康检查接口。

当前依赖写在项目根目录的 `requirements.txt` 中。在已经启用 `.venv` 的前提下安装基础依赖：

```bash
python -m pip install -r requirements.txt
```

当前 `requirements.txt` 先保持最小：

```text
fastapi[standard]
```

注意：`requirements.txt` 中不要给 `fastapi[standard]` 加引号；只有在命令行里直接安装单个包时才需要根据 shell 情况加引号。

如果在 Ubuntu 中看到 `externally-managed-environment`，说明命令打到了系统 Python，而不是项目 `.venv`。先执行：

```bash
cd /mnt/d/wms_agv
source .venv/bin/activate
```

再用 `python -m pip install -r requirements.txt` 安装。

## PyCharm 配置

PyCharm 仍然打开 Windows 项目目录：

```text
D:\wms_agv
```

Python Interpreter 建议配置为 WSL 里的项目虚拟环境：

```text
WSL: Ubuntu-24.04
/mnt/d/wms_agv/.venv/bin/python
```

推荐设置路径：

```text
Settings -> Project: wms_agv -> Python Interpreter -> Add Interpreter -> On WSL
```

选择：

```text
Ubuntu-24.04
Existing environment
/mnt/d/wms_agv/.venv/bin/python
```

## PyCharm 终端配置

为了让 PyCharm 下方终端直接进入 WSL，可以设置：

```text
Settings -> Tools -> Terminal
```

Shell path：

```text
wsl.exe -d Ubuntu-24.04
```

以后在 PyCharm Terminal 中执行：

```bash
cd /mnt/d/wms_agv
source .venv/bin/activate
```

## 常见提示

### localhost 代理提示

进入 WSL 时可能看到：

```text
wsl: 检测到 localhost 代理配置，但未镜像到 WSL。NAT 模式下的 WSL 不支持 localhost 代理。
```

这个提示不代表 WSL 安装失败。只要 `sudo apt update`、`docker --version`、`docker compose version` 可以正常执行，就可以先忽略。

### Windows Python 和 WSL Python

Windows 本地可以有 Python，PyCharm 也可以使用 Windows `.venv`。但本项目推荐使用 WSL 中的 `.venv`，因为后续 Docker 容器、PostgreSQL、Linux 路径和脚本命令都会更接近 WSL 环境。

推荐原则：

```text
PyCharm 写代码
WSL Python 跑后端
Docker 跑外部服务
```

## 阶段 1-1 启动检查

每次准备开发时：

```bash
cd /mnt/d/wms_agv
source .venv/bin/activate
python --version
pip --version
docker --version
docker compose version
```

如果这些命令都正常，说明可以开始阶段 1-1：

```text
创建 FastAPI 后端骨架
创建 GET /health 健康检查接口
运行 uvicorn
用浏览器或 curl 验证服务可用
```

当前最小后端启动命令：

```bash
python -m uvicorn app.main:app --reload
```

当前健康检查接口：

```text
GET /api/Health
```

浏览器或接口工具能看到 `status is ok`，说明 FastAPI 服务已经跑通。
