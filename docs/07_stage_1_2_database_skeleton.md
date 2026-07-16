# 阶段 1-2：数据库连接骨架

## 目标

阶段 1-2 的目标是把后端项目从“只有 FastAPI 健康检查”推进到“具备正式数据库连接骨架”。

这一阶段只做基础设施，不创建 WMS 业务表。物料、库位、交接点位、库存余额、库存流水等对象放到阶段 2 再实现。

## 本阶段要解决的问题

- PostgreSQL 通过 Docker Compose 本地启动。
- 项目通过环境变量读取数据库连接配置。
- SQLAlchemy 可以创建 engine 和 session。
- Alembic 可以连接数据库并执行迁移。
- 现有 `/health` 测试继续通过。

## 建议新增或修改的文件

```text
requirements.txt
docker-compose.yml
.env.example
app/core/__init__.py
app/core/config.py
app/db/__init__.py
app/db/base.py
app/db/session.py
alembic.ini
alembic/env.py
alembic/versions/
```

## 依赖建议

阶段 1-2 建议在 `requirements.txt` 中补充：

```text
sqlalchemy
alembic
psycopg[binary]
pydantic-settings
```

说明：

- `sqlalchemy`：ORM 和数据库连接核心。
- `alembic`：数据库迁移工具。
- `psycopg[binary]`：PostgreSQL 驱动。
- `pydantic-settings`：读取 `.env` 和环境变量配置。

## 环境变量建议

`.env.example` 建议先记录开发环境配置，不提交真实密码：

```text
POSTGRES_DB=wms_agv
POSTGRES_USER=wms_agv
POSTGRES_PASSWORD=wms_agv_dev
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
DATABASE_URL=postgresql+psycopg://wms_agv:wms_agv_dev@localhost:5432/wms_agv
```

第一版可以先让 `DATABASE_URL` 成为 SQLAlchemy 和 Alembic 共用的连接字符串。

## Docker Compose 建议

PostgreSQL 镜像不要使用 `latest`。第一版建议固定一个明确版本，例如：

```text
postgres:16
```

容器服务建议保持简单：

- 容器名：`wms_agv_postgres`
- 数据库：`wms_agv`
- 用户：`wms_agv`
- 开发密码：`wms_agv_dev`
- 本机端口：`5432`
- 数据卷：使用命名 volume，避免容器删除后数据马上丢失。

## Alembic 第一阶段策略

阶段 1-2 只需要让 Alembic 能连接数据库并执行迁移。

可以先生成一个空迁移，用来验证迁移链路：

```bash
alembic revision -m "init database skeleton"
alembic upgrade head
```

此时不需要建业务表。等阶段 2 开始，再围绕物料、库位、交接点位和库存余额创建第一批模型和迁移。

## 验收命令

在 WSL 中执行：

```bash
cd /mnt/d/wms_agv
source .venv-wsl/bin/activate
python --version
python -m pip install -r requirements.txt
docker compose up -d
docker compose ps
alembic upgrade head
pytest -q
```

如果所有命令通过，阶段 1-2 可以认为完成。

## 不在本阶段做的事

- 不创建物料表。
- 不创建库位表。
- 不创建库存余额表。
- 不写入库、出库、AGV 业务流程。
- 不做前端看板。
- 不接真实 ERP 或真实 AGV。

本阶段只验证数据库基础设施是否可靠。
