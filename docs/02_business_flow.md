# 业务流程图

## 总体链路

```mermaid
flowchart LR
    ERP["模拟 ERP"] -->|基础资料/入库单/出库单| WMS["WMS 执行层"]
    WMS -->|库存校验/任务生成| Task["任务中心"]
    Task -->|AGV 任务| AGV["AGV 仿真系统"]
    AGV -->|状态回传| Task
    Task -->|任务事件| Timeline["任务时间线"]
    WMS -->|库存变化| Inventory["库存余额/库存流水"]
    WMS -->|仓库执行账| Callback["ERP 回传记录"]
    Callback --> ERP
    Timeline --> Board["2D 实时看板"]
```

## 出库流程

业务目标：ERP 下发出库需求后，WMS 确认可出库库存，AGV 完成搬运，WMS 扣减库存并回传 ERP 仓库执行账。

```mermaid
sequenceDiagram
    participant ERP as 模拟 ERP
    participant WMS as WMS
    participant Operator as WMS 操作员
    participant AGV as AGV 仿真系统
    participant Ledger as 库存与执行账

    ERP->>WMS: 下发出库单
    WMS->>WMS: 保存出库单和明细
    Operator->>WMS: 人工下发出库单
    WMS->>WMS: 校验可用库存
    WMS->>Ledger: 记录出库分配
    WMS->>AGV: 创建 AGV 出库任务
    AGV->>WMS: 回传已接单
    AGV->>WMS: 回传搬运中
    AGV->>WMS: 回传已送达
    WMS->>Ledger: 扣减库存并记录库存流水
    WMS->>Ledger: 生成仓库执行账
    WMS->>ERP: 回传出库执行结果
```

出库最小验收示例：

- 初始库存 100。
- 出库单需要出库 20。
- WMS 下发后生成 1 条 AGV 出库任务。
- AGV 完成后库存变成 80。
- 系统生成库存流水、任务时间线和 ERP 回传记录。

## 入库流程

业务目标：ERP 下发入库需求后，现场扫码确认到货，叉车放到交接点，WMS 生成 AGV 上架任务，AGV 完成后库存入正式库位。

```mermaid
sequenceDiagram
    participant ERP as 模拟 ERP
    participant WMS as WMS
    participant Operator as 现场操作员
    participant Forklift as 叉车人员
    participant AGV as AGV 仿真系统
    participant Inventory as 库存

    ERP->>WMS: 下发入库单
    WMS->>WMS: 保存入库单和明细
    Operator->>WMS: 扫码确认到货物料/批次/数量
    Forklift->>WMS: 确认货物放到交接点或电梯点
    WMS->>WMS: 校验是否超收
    WMS->>Inventory: 记录暂存库存
    WMS->>AGV: 创建 AGV 入库上架任务
    AGV->>WMS: 回传已接单
    AGV->>WMS: 回传搬运中
    AGV->>WMS: 回传已上架
    WMS->>Inventory: 增加正式库位库存
    WMS->>Inventory: 记录库存流水
```

入库最小验收示例：

- 入库单应收 100。
- 第一次扫码到货 60。
- 货物放到 HANDOVER-IN-01。
- WMS 生成 1 条 AGV 入库上架任务。
- AGV 完成后目标库位库存增加 60。
- 系统记录暂存、上架、库存流水和任务时间线。

## 任务事件与看板关系

第一版看板不直接读取复杂业务表，而是读取任务事件或可视化事件。

```mermaid
flowchart TD
    Business["业务状态变化"] --> Event["任务事件"]
    Event --> Visual["可视化事件"]
    Visual --> Board["2D 看板"]

    BusinessExamples["例：AGV 已接单/搬运中/已完成"] --> Event
    VisualExamples["例：AGV-01 从等待区移动到交接点"] --> Board
```

看板后期支持两种模式：

- 实时模式：后端产生事件后推送或被前端轮询，画面即时变化。
- 回放模式：按入库单号、出库单号或 AGV 任务号读取历史事件，按时间顺序重播。
