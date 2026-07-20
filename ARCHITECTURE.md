```text
Document Status: PROPOSED
Normative Status: 未批准，当前不是 Source of Truth
Product Baseline: AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md
Product Baseline Commit: c9cc522b4ab01008fed390c282d6bd5a816ee779
Governance Baseline Commit: cf8e66b431b36103fb7c049048521c15df2e3701
Governance Approval Record: GOVERNANCE_APPROVAL_0001.md
Current Phase: Phase 0
Phase 1: NOT AUTHORIZED
Code Status: NO PRODUCT CODE
```

# ARCHITECTURE.md（Phase 1 Core Prototype 实施架构契约）

> 本文档定义 **Phase 1 Core Prototype** 的实施架构契约（implementation architecture contract）。
> 它**不是**完整产品实现说明，也不是 Phase 3+ 生产架构。本文档当前为 `PROPOSED`，
> 未经 Reviewer-2 审核与 owner 批准前**不属于 Source of Truth**；不得描述为已批准、已实现或已测试；
> 不得自动关闭 Phase 0。所有内容必须服从已批准 V1.1，且不与 V1.1 冲突。

---

## 1. 范围与非目标

### 1.1 Phase 1 仅包含

- 状态机（Workflow State Machine）
- SQLite 事件账本（Event Ledger，含事务型 Outbox）
- projection（可重建派生视图）
- 政策引擎（Policy Engine，含三层授权裁决）
- Transactional Outbox（同事务提交内部状态与投递意图）
- Artifact Store（内容寻址产物仓库）
- 模拟适配器（Mock Adapter）
- 模拟工作流（Mock Workflow）
- 备份、恢复与自动测试

### 1.2 明确不包含（非 Phase 1 范围）

- Git / GitHub 真实适配器
- Qoder
- ChatGPT
- WorkBuddy
- Playwright
- UIA（Windows UI Automation）
- 真实网络、模型与外部系统调用
- NAS
- STS
- OKX
- 生产部署

> Phase 1 的全部行为在本地模拟环境内闭环，不接入任何真实外部工具（见 V1.1 §12.5、§13.1）。

---

## 2. 架构原则

固定以下原则（与 V1.1 §5、§12.4 一致）：

- **modular monolith 优先**：Phase 1 以单机模块化单体交付，不引入分布式组件。
- **内核领域逻辑与 UI、数据库、适配器实现解耦**：核心领域（状态机、政策引擎、事件账本）不依赖具体 UI、SQLite 访问细节或适配器实现。
- **事件账本是事实 Source of Truth**：工作流事实、外部动作意图、投递结果、审计事件唯一权威在 Event Ledger。
- **projection 可删除重建**：当前状态视图、仪表盘、查询表均为派生视图，不得反向覆盖事件事实。
- **签名政策包是授权 Source of Truth**：已签名政策包是授权边界与默认决策规则的唯一权威。
- **政策引擎是唯一裁决者**：权限、预算、工具路由、发布门禁由 Policy Engine 裁决，但不保存或伪造工作流事实。
- **外部或模拟角色只能提交声明、结果和证据**：worker / reviewer / messenger / repository / model 不能直接改写权威状态或政策。
- **fail-closed**：任何不确定或失败默认进入安全终态或安全降级，而不是伪造完成。
- **不宣称跨第三方 exactly-once**：对任意外部系统只承诺"至少一次尝试、幂等优先、状态对账、无法确认时安全停止"（V1.1 §16.3）。

---

## 3. 逻辑组件

每个组件定义职责、输入、输出与禁止事项。

| 组件 | 职责 | 输入 | 输出 | 禁止事项 |
|---|---|---|---|---|
| Workflow Engine | 编排工作流生命周期与状态迁移 | 事件、政策决策、调度指令 | 状态变更事件、终态归档 | 不得伪造事实；不得绕过政策引擎裁决 |
| State Machine | 维护工作流状态与允许迁移 | 迁移请求、政策决定 | 状态事件 | 不得进入未在 V1.1 定义的终态；不得停留在无限 RUNNING |
| Policy Engine | 三层授权裁决、预算/路由/门禁 | 能力自报、政策包、令牌请求 | 授权上限、决策记录 | 不得被插件/模型自行提权；不得保存工作流事实 |
| Event Ledger | 追加式事件存储，事实 SoT | 状态变更、外部动作五状态 | 事件流、可重放 | 不得覆盖历史；不得反向被 projection 覆盖 |
| Projection Builder | 从事件重建派生视图 | 事件流 | 当前状态视图 | 不得写回权威；不得成为事实来源 |
| Transactional Outbox | 内部状态与投递意图同事务 | 外部动作意图 | Outbox 记录（PREPARED→…） | 不得盲目重发；不得宣称 exactly-once |
| Delivery Coordinator | 管理 delivery_attempt 与对账 | Outbox 状态、适配器能力 | 投递指令、对账请求 | 未知外部状态不得盲重试（次数 0） |
| Reconciliation Manager | 对状态未知动作对账 | DISPATCHING/ACKNOWLEDGED 未决动作 | 对账结论、RECONCILIATION_REQUIRED | 不得猜测外部状态；不得绕过政策引擎 |
| Artifact Store | 内容寻址产物存储 | 产物字节、SHA256 | artifact 引用 | 不得保存明文凭据；不得混用非产物数据 |
| Adapter Host | 加载与管理适配器进程 | 适配器清单、能力自报 | 适配器生命周期事件 | Phase 1 **仅加载 mock adapter**；不得加载真实适配器 |
| Mock Adapter | 模拟外部角色与崩溃点 | 模拟任务、注入故障 | 模拟结果/证据 | 不得发起真实网络/模型/外部调用 |
| Scheduler / Timer Service | 定时触发、超时与心跳检测 | 调度配置、心跳 | 超时事件、STALLED 标记 | 不得替代政策引擎裁决 |
| Budget / Loop Limit Enforcer | 强制执行预算与循环上限 | 计数请求、预算配置 | 预算耗尽终态、超阈熔断 | 数值不得被提示词修改；重启不归零 |
| Backup and Recovery Manager | 一致性备份与重放恢复 | 快照触发、WAL checkpoint | backup set、恢复判据 | 哈希不一致不得宣称恢复成功 |
| Observability / Audit | 结构化日志与审计哈希链 | 领域事件 | 审计记录、脱敏诊断 | 不得记录明文 secret；不得关闭审计 |
| Local Management UI | WebView2 管理界面 | 用户查看/配置 | 本地展示 | 不得成为状态权威；不得执行 A1–A5 外部动作 |

> Phase 1 中 **Adapter Host 只能加载 mock adapter**（V1.1 §13.1）。真实适配器在 Phase 3 引入。

---

## 4. 进程与故障边界

- **Core 进程**：Workflow Engine / Policy Engine / Event Ledger / Projection / Outbox / Delivery / Reconciliation / Budget / Backup / Observability 同属核心域，但内部以模块化边界隔离。
- **mock adapter 独立进程或隔离宿主**：通过 JSON-RPC 2.0 over stdio 或本地受限 WebSocket 与核心通信（V1.1 §10.4）。
- **UI 与核心服务边界**：WebView2 管理界面只读取派生视图与提交本地操作；不得直接写事件账本或政策包。
- **SQLite 与 Artifact Store 边界**：事件账本（SQLite WAL）与产物仓库（内容寻址）使用同一 backup set ID，但逻辑隔离。
- **单组件崩溃不得拖垮整个系统**：插件/适配器崩溃由 Adapter Host 隔离；核心域故障进入安全终态或降级模式（V1.1 §10.4）。
- **插件不得直接读取核心 SQLite**：只能经核心 API 提交声明/结果与证据（V1.1 §10.4、§12.4）。
- **UI 不得成为状态权威**：权威状态仅来自 Event Ledger。

---

## 5. 数据所有权

| 数据类型 | 权威所有者 | 派生视图 | 谁可写入 | 恢复来源 |
|---|---|---|---|---|
| workflow facts | Event Ledger | projection / 仪表盘 | Core（Workflow/State Machine） | 事件重放 + backup set |
| jobs | Event Ledger | 查询表 | Core | 事件重放 + backup set |
| messages | Event Ledger | 消息索引 | Core | 事件重放 |
| delivery attempts | Event Ledger（Outbox） | 投递视图 | Core（Outbox/Delivery） | 事件重放 |
| findings | Event Ledger | 审核面板 | Core（reviewer 提交） | 事件重放 |
| evidence | Artifact Store + Event Ledger 引用 | 证据视图 | Core（verifier 提交） | artifact 内容寻址 |
| policy decisions | Policy Engine（签名政策包为 SoT）+ Event Ledger 记录 | 决策面板 | Policy Engine | 重新加载签名政策包 |
| budgets | Policy Engine + Event Ledger 计数 | 预算视图 | Budget Enforcer（Core） | 事件重放 |
| artifacts | Artifact Store | 产物引用 | Core | 内容寻址 + backup set |
| projections | （派生，无权威） | —— | Projection Builder（重建） | 从 Event Ledger 重建 |
| credentials | Credential Store（OS 凭据库） | 凭据引用 | OS / Vault（适配器仅得引用） | OS 凭据库 |
| policy bundle | Policy Engine（签名，授权 SoT） | —— | owner / governance（签名发布） | 优先恢复最后已批准且签名有效的 policy bundle；修复后的新 bundle 须经治理批准方可签发；不得对未知/被篡改内容直接重新签名；信任根失败不得自动生成新根 |

---

## 6. 工作流状态机

依据 V1.1 §8.2 与 §4.3。

### 6.1 非终态（派生，须与 V1.1 一致）

- `PENDING`：已创建，等待调度。
- `RUNNING`：执行中（含心跳更新）。
- `STALLED`：RUNNING 且无心跳超过阈值（V1.1 §16.2）。

### 6.2 终态清单（V1.1 §8.2，权威）

- `COMPLETED_APPROVED`
- `COMPLETED_WITH_DEFERRED_LOW_RISK`
- `BLOCKED_REDESIGN_REQUIRED`
- `BLOCKED_AUTH_EXPIRED`
- `BLOCKED_BUDGET_EXHAUSTED`
- `BLOCKED_NEEDS_POLICY_DECISION`（须先走 §4.3.1 默认决策阶梯）
- `QUARANTINED_SECURITY_RISK`
- `FAILED_EXTERNAL_DEPENDENCY`
- `CANCELLED_BY_POLICY`
- `POLICY_INTEGRITY_SAFE_MODE`

### 6.3 允许迁移

- `PENDING → RUNNING`
- `RUNNING → STALLED`（心跳丢失）
- `STALLED → RUNNING`（恢复成功）
- `RUNNING/STALLED →` 任意终态（由政策引擎裁决）
- `PENDING →` 任意终态（如预算/授权前置失败）

### 6.4 禁止迁移

- 终态不得再迁移到非终态（终态不可逆）。
- 不得 `RUNNING → RUNNING` 无限保持（无无限 RUNNING，V1.1 §8.2）。
- 不得由外部/模拟角色直接写入终态（只能提交结果，由政策引擎裁决）。

### 6.5 每次迁移的必要事件与政策决定

- 每次状态变更必须产生 Event Ledger 事件（含前序哈希）。
- 进入 `BLOCKED_*` / `QUARANTINED_*` / `CANCELLED_*` 必须由 Policy Engine 决策记录支撑。
- 进入 `BLOCKED_NEEDS_POLICY_DECISION` 前必须先执行默认决策阶梯（V1.1 §4.3.1）。

### 6.6 强制终态规则（确定性映射）

最大执行时间到达时，MUST 依据触发原因进入下列确定终态，不得无限 `RUNNING`：

| Condition | Required Terminal |
|---|---|
| 预算耗尽 | `BLOCKED_BUDGET_EXHAUSTED` |
| 授权/认证过期且无安全恢复 | `BLOCKED_AUTH_EXPIRED` |
| 修复/循环/重新设计上限耗尽 | `BLOCKED_REDESIGN_REQUIRED` |
| 外部依赖确定失败 | `FAILED_EXTERNAL_DEPENDENCY` |
| 发现安全风险或副作用状态存在安全不确定性 | `QUARANTINED_SECURITY_RISK` |
| 政策未覆盖且完成默认决策阶梯后仍无法继续 | `BLOCKED_NEEDS_POLICY_DECISION` |
| 政策可信根失败 | `POLICY_INTEGRITY_SAFE_MODE` |
| 政策主动取消 | `CANCELLED_BY_POLICY` |

- 上表终态均来自 V1.1 §8.2（6.2 清单），**不新增任何 workflow 终态**。
- **预算耗尽后**：进入 `BLOCKED_BUDGET_EXHAUSTED`（V1.1 §8.2、§9.1）。
- **reconciliation 终态**：外部动作 `RECONCILIATION_REQUIRED` 无法确认时，工作流映射到上表确定终态（如 `BLOCKED_NEEDS_POLICY_DECISION` / `QUARANTINED_SECURITY_RISK`）。
- **safe mode 下**：仅允许本地 A0 诊断；不得产生任何 A1–A5 状态迁移（V1.1 §15.5）。

### 6.7 不变量

- 所有未被取消的 workflow 必须到达 6.2 中明确终态。
- 不得编造与 V1.1 冲突的新状态。

---

## 7. Outbox 与恢复

Outbox 状态（V1.1 §12.2）：

```text
PREPARED
DISPATCHING
ACKNOWLEDGED
VERIFIED
RECONCILIATION_REQUIRED
```

允许迁移（确定性）：

```text
PREPARED → DISPATCHING
DISPATCHING → ACKNOWLEDGED
ACKNOWLEDGED → VERIFIED
DISPATCHING → RECONCILIATION_REQUIRED
ACKNOWLEDGED → RECONCILIATION_REQUIRED
```

规则：

- `VERIFIED` 为终态。
- `RECONCILIATION_REQUIRED` **不得盲目返回** `DISPATCHING`；只有确定性证据证明外部动作未执行，且政策引擎批准新 attempt，才能创建新的 `delivery_attempt`。
- 对账证明已执行后进入 `VERIFIED`。
- 无法证明外部动作状态时，工作流进入安全终态（由政策引擎裁决，见 §6.6）。
- 内部状态变更与"待投递意图"在**同一数据库事务**提交（Transactional Outbox）。
- 逻辑 message 唯一（message_id 不可复用；同一逻辑 message 只创建一次）。
- delivery_attempt 可受控重试（相同 idempotency_key + 递增 attempt 序号，默认上限 3、绝对上限 10，V1.1 §9）。
- 状态未知的外部写操作 **0 次自动盲重试**；先对账，无法确认则进入 `RECONCILIATION_REQUIRED`。
- Phase 1 使用 **mock adapter 注入所有崩溃点**（不依赖真实外部系统）。
- 恢复后先对账再决定；不得盲目重发。
- 不保证 exactly-once（V1.1 §16.3）。

---

## 8. 存储与备份

依据 V1.1 §12.1、§12.5：

- **SQLite WAL** 模式；核心状态持久化于本地 SQLite。
- **事件追加**：Event Ledger 只追加，不覆盖历史，每事件带前序哈希。
- **projection 重建**：删除 projection 后可从事件账本重建相同状态。
- **backup set ID**：数据库快照、事件序号、产物 manifest 使用同一 backup set ID。
- **数据库与 artifact manifest 一致性**：备份时同步记录并共享 backup set ID。
- **WAL checkpoint**：备份前执行受控 checkpoint。
- **backup API**：使用 SQLite backup API 或等价一致性备份机制。
- **恢复顺序**：校验哈希 → 恢复数据库与产物 → 重放至指定事件序号 → 重建 projection → 对账未决动作。
- **哈希校验**：所有产物使用 SHA256 内容寻址。
- **未决动作对账**：恢复后对所有 `DISPATCHING`/`ACKNOWLEDGED` 动作对账。
- **恢复成功判据**：哈希校验通过 + 事件重放一致 + projection 重建一致 + 未决动作对账完成。
- **失败安全终态**：状态或哈希不一致时进入安全终态，不宣称恢复成功。

---

## 9. 技术栈

固定（V1.1 §13.3，V1.1 修订 `.NET 8` → `.NET 10 LTS`）：

- **.NET 10 LTS** / ASP.NET Core（核心与 API）
- **SQLite**（本地状态，WAL 模式）
- **JSON / YAML + JSON Schema**（配置与契约）
- **JSON-RPC 2.0**（插件/适配器通信）
- **WebView2** 仅作为 ARH 自身管理界面（不得用于第三方网页自动化；第三方自动化由 Phase 3 独立 Playwright Browser Runner 执行）
- **Phase 1 不引入 Playwright 或 UIA**

> 明确：**不得写 `.NET 8`**；全文技术栈以 `.NET 10 LTS` 为准。

---

## 10. 演进边界

- **Phase 2**：只做一次性隔离探针（feasibility probe），不进入生产组件（见 GOVERNANCE_APPROVAL_0001.md 非阻塞解释）。
- **Phase 3**：才引入正式适配器（Git/GitHub、Qoder CLI/SDK、Playwright Browser Runner 等）与首个 Product MVP。
- **Phase 4**：才引入多审核（双 reviewer / evidence_verifier / adjudicator）与夜间无人值守。
- **Phase 1 架构不得将 Qoder、ChatGPT 或 WorkBuddy 写成内核依赖**：它们是未来的外部角色/适配器，Phase 1 内核只认识 mock adapter 与标准角色契约。
- 演进架构（NAS Control Plane / Windows Runner / Cloud Runners）见 V1.1 §13.2，不属于 Phase 1 范围。
