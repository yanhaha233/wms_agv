# 学习日志

## 2026-07-10

- 阶段：阶段 0，项目定位与设计。
- 本次切片：把 WMS-AGV 项目的业务故事、边界、角色、流程和核心对象写成仓库文档。
- 练习重点：把真实工作中的出入库流程拆成能落地到表、状态和任务事件的后端模型。
- 当前成果：README、项目故事、业务流程图、角色说明、核心对象与状态。
- 下一步：进入阶段 1，搭建 FastAPI、PostgreSQL、SQLAlchemy、Alembic、Docker Compose 和健康检查。

## 2026-07-11

- 阶段：阶段 0 收尾，阶段 1 准备。
- 本次切片：确认 WSL Ubuntu-24.04、Docker Desktop、Docker Compose、PyCharm 与项目 `.venv` 的开发路线。
- 练习重点：区分 Windows Python、WSL Python 和 Docker 容器环境，选择更接近后续运行环境的开发方式。
- 当前成果：新增开发环境部署记录，明确使用 PyCharm 写代码、WSL Python 跑后端、Docker 跑外部服务。
- 下一步：开始阶段 1-1，创建 FastAPI 后端骨架和 `/health` 健康检查接口。

## 2026-07-15

- 阶段：阶段 1-1，FastAPI 最小后端骨架。
- 本次切片：创建 `app/main.py` 和 `requirements.txt`，让项目第一次作为后端服务运行起来。
- 练习重点：理解 `requirements.txt` 是依赖清单，`.venv` 是项目自己的 Python 环境，FastAPI 负责把 Python 函数暴露成 HTTP 接口。
- 当前成果：`requirements.txt` 使用 `fastapi[standard]`；FastAPI 服务可以启动；当前健康检查接口可以返回 `status is ok`。
- 待改进：统一接口路径大小写，例如后续固定为 `/health` 或 `/api/health`；把响应整理成稳定 JSON 结构，例如 `{"status": "ok"}`；补一个自动化测试验证健康检查接口。
- 下一步：完成阶段 1-1 收尾，再进入阶段 1-2 的基础数据库连接和第一批核心表设计。

## 2026-07-16

- 阶段：阶段 1-1 收尾，开发环境统一。
- 本次切片：删除 Windows 侧 `.venv`，项目后续统一使用 WSL Ubuntu 中的 Python 3.14 和 `.venv-wsl`。
- 练习重点：区分系统 Python、项目 Python 和虚拟环境；不要替换 Ubuntu 自带的 `python3`，项目中显式使用 `python3.14` 创建虚拟环境。
- 当前成果：`.gitignore` 忽略 `.venv-wsl/`；开发环境文档记录 Python 3.14 安装、虚拟环境创建、PyCharm Interpreter 路径和 `/health` 健康检查接口。
- 下一步：提交环境文档更新，然后进入阶段 1-2：PostgreSQL、SQLAlchemy、Alembic 和数据库连接骨架。
