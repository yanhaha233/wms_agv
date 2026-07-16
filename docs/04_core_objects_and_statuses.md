# 核心对象与状态

## 核心对象

| 对象 | 用途 | 第一版关键字段 |
| --- | --- | --- |
| 物料 | 描述仓库管理的物品 | material_code、name、unit、enabled |
| 库位 | 表示正式库存所在位置 | location_code、area、location_type、enabled |
| 交接点位 | 表示叉车和 AGV 交接位置 | point_code、point_type、enabled |
| 库存余额 | 表示某物料在某库位或点位的数量 | material_code、location_code、qty、inventory_status |
| 库存流水 | 记录每次库存变化 | txn_no、material_code、qty_change、source_doc_no、reason |
| 入库单 | ERP 下发的入库需求 | inbound_no、status、source_system |
| 入库单明细 | 入库物料和数量 | inbound_no、line_no、material_code、expected_qty、received_qty |
| 入库到货扫码记录 | 现场确认实际到货 | scan_no、inbound_no、material_code、batch_no、qty、handover_point |
| 入库上架任务 | WMS 生成的 AGV 上架任务 | putaway_task_no、inbound_no、from_point、to_location、status |
| 出库单 | ERP 下发的出库需求 | outbound_no、status、source_system |
| 出库单明细 | 出库物料和数量 | outbound_no、line_no、material_code、required_qty |
| 出库分配记录 | WMS 分配的出库来源库存 | allocation_no、outbound_no、material_code、from_location、allocated_qty |
| AGV 任务 | 入库或出库的搬运任务 | agv_task_no、task_type、business_doc_no、status |
| AGV 任务事件 | AGV 状态变化记录 | event_no、agv_task_no、event_type、occurred_at |
| 可视化事件 | 给 2D 看板使用的事件 | event_type、object_type、object_code、from_point、to_point |
| ERP 回传记录 | WMS 回传 ERP 的结果 | callback_no、doc_no、callback_type、status、retry_count |
| 仓库执行账 | WMS 侧确认已完成的仓库业务结果 | ledger_no、doc_no、material_code、qty、completed_at |
| 操作日志 | 人工操作留痕 | operator、action、target_no、occurred_at |
| 接口日志 | 模拟外部系统交互留痕 | interface_name、request_id、status、error_message |
| 异常事件 | 记录业务异常 | exception_no、exception_type、severity、status |

## 入库单状态

| 状态 | 含义 |
| --- | --- |
| pending_arrival | 待到货 |
| partial_arrived | 部分到货 |
| arrived | 已到货 |
| pending_putaway | 待生成上架任务 |
| putting_away | 上架中 |
| completed | 已完成 |
| exception | 异常 |

## 入库上架任务状态

| 状态 | 含义 |
| --- | --- |
| pending_create | 待生成 |
| waiting_agv_accept | 待 AGV 接单 |
| agv_accepted | AGV 已接单 |
| moving | 搬运中 |
| putaway_done | 已上架 |
| failed | 失败 |
| canceled | 取消 |

## 出库单状态

| 状态 | 含义 |
| --- | --- |
| received | 已接收 |
| waiting_release | 待下发 |
| released | 已下发 |
| allocating | 分配中 |
| waiting_agv | 待 AGV 执行 |
| shipping | 出库搬运中 |
| completed | 已完成 |
| callback_pending | 待回传 ERP |
| callback_done | ERP 回传完成 |
| exception | 异常 |

## AGV 任务状态

| 状态 | 含义 |
| --- | --- |
| created | 已创建 |
| accepted | AGV 已接单 |
| moving_to_pick | 前往取货点 |
| picked | 已取货 |
| moving_to_drop | 前往放货点 |
| dropped | 已放货 |
| completed | 已完成 |
| failed | 失败 |
| canceled | 取消 |

## 库存状态

| 状态 | 含义 |
| --- | --- |
| expected | 预期库存，来自 ERP 入库单应收数量 |
| temporary | 暂存库存，扫码后位于交接点或电梯点 |
| in_transit | 在途库存，AGV 已接单并搬运中 |
| available | 可用库存，已进入正式库位 |
| allocated | 已分配库存，被出库单占用但尚未扣减 |

## 异常类型

第一版需要能表达这些异常：

- insufficient_inventory：库存不足。
- unknown_material：未知物料。
- unknown_location：未知库位。
- unknown_handover_point：未知交接点位。
- duplicate_document：重复单据。
- over_receipt：入库超收。
- agv_timeout：AGV 超时。
- agv_failed：AGV 任务失败。
- invalid_status_transition：状态流转错误。
- erp_callback_failed：ERP 回传失败。

## 幂等规则

为了接近真实接口系统，第一版从设计上保留幂等意识：

- ERP 单据以 source_system + doc_no 作为唯一来源。
- 入库和出库明细以 doc_no + line_no 作为唯一明细。
- AGV 任务以 agv_task_no 作为唯一任务号。
- 接口请求可以带 request_id，重复请求不能重复生成业务数据。
- 库存流水一旦生成，不直接修改，纠错通过反向流水或异常处理完成。
