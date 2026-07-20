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

# PROTOCOL.md（规范性协议草案 ARP v1）

> 本文件建立 ARH 统一消息/事件协议的规范性草案（ARP v1）。
> 当前为 `PROPOSED`，未经 Reviewer-2 审核与 owner 批准前**不属于 Source of Truth**；
> 不得描述为已批准、已实现或已测试；不得自动关闭 Phase 0。
> 协议语义必须服从已批准 V1.1，且不与 V1.1 冲突。

---

## 关键词含义（RFC 2119 风格）

- **MUST / MUST NOT**：绝对要求 / 绝对禁止；违反即协议错误。
- **SHOULD / SHOULD NOT**：强烈建议 / 不建议；例外须有记录理由。
- **MAY**：可选；实现可自由取舍。

---

## 1. 协议版本

- 每条消息 MUST 携带 `protocol_version`（当前 `1`）。
- 核心与适配器 MUST 在连接建立时交换 `supported_protocol_versions` 并协商共同版本区间。
- 支持的协议版本范围 MUST 在能力协商中声明（见 §6）。
- 当双方支持区间**不重叠**时，接收方 MUST 拒绝加载并告警；**MUST NOT** 静默降级到不兼容版本。
- 协议升级 MUST 满足：可迁移、可回放、可向后兼容（V1.1 §11）。
- 旧版本消息重放 MUST 能被新版本解析；新字段 MUST 在不被理解时被安全忽略（见 §3 未知字段处理）。

---

## 2. 标识符

| 标识符 | 唯一性 | 作用域 | 复用规则 | 关联 |
|---|---|---|---|---|
| `workflow_id` | 全局唯一 | ARH 实例 | 不可复用 | 关联 `workflow_version` |
| `workflow_version` | 与 workflow_id 组合唯一 | 该 workflow | 变更后新版本 | 绑定 workflow_id |
| `job_id` | 全局唯一 | 该 workflow | 不可复用 | 隶属 workflow_id |
| `message_id` | 全局唯一 | 全系统 | 不可复用 | 关联 correlation_id / idempotency_key |
| `delivery_attempt_id` | 同 message 内唯一 | 该 message | 不可复用 | 序号递增 |
| `correlation_id` | 关联一组相关 message | 该业务上下文 | 同上下文内稳定 | 跨 attempt 保持 |
| `idempotency_key` | 同逻辑动作唯一 | 该动作 | 不可复用 | 跨 delivery_attempt 一致 |
| `artifact_id` | 全局唯一 | Artifact Store | 不可复用 | 绑定 SHA256 |
| `finding_id` | 全局唯一 | 审核上下文 | 不可复用 | 关联 evidence_id |
| `evidence_id` | 全局唯一 | 证据上下文 | 不可复用 | 关联 artifact_id |
| `policy_decision_id` | 全局唯一 | 政策引擎 | 不可复用 | 引用政策版本 |

规则：

- 所有标识符 MUST 由生成方保证唯一且不可复用（即使逻辑实体被取消或重做）。
- `idempotency_key` 用于跨 `delivery_attempt` 去重；同一逻辑 message 的多次 attempt MUST 复用相同 `idempotency_key` 与 `correlation_id`。

---

## 3. 时间和序列化

- 消息体与载荷 MUST 为 **UTF-8 编码的 JSON**。
- 所有时间戳 MUST 使用 **UTC，RFC3339** 格式（如 `2026-07-21T00:00:00Z`）。
- 每条消息/事件 MUST 携带 `schema_version` 以标识结构版本。
- **未知字段**：接收方 MUST 保留未知字段用于透传/审计；MUST NOT 因未知字段拒绝合法消息（除非明确 schema 约束禁止）。
- **必填字段**：`protocol_version`、`message_id`、`sender_role`、`recipient_role`、`message_type`、`created_at` 为 MUST；缺失即校验失败。
- **canonicalization / hashing 输入规则**：用于完整性校验的哈希 MUST 基于规范化（canonical）JSON（键有序、空白剔除），输入字段集 MUST 在 schema 中固定。

---

## 4. 消息 Envelope

每条消息 MUST 包含以下信封字段：

```text
protocol_version
schema_version
workflow_id
workflow_version
job_id
message_id
correlation_id
idempotency_key
sender_role
recipient_role
message_type
target
attempt
artifacts
payload
created_at
expires_at
security_classification
```

语义与校验：

- `sender_role` / `recipient_role`：MUST 为已定义角色（如 `worker.primary`、`reviewer.security`、`policy.engine`）。
- `target`：描述作用对象（repository / branch / revision 等）；MUST 在能力/政策允许范围内。
- `attempt`：delivery_attempt 序号，从 1 递增。
- `artifacts`：附件数组，每个 MUST 带 `sha256` 与 `uri`（内容寻址，V1.1 §11、§12.3）。
- `expires_at`：消息过期时间；过期消息 MUST NOT 被当作新结果处理（V1.1 §11）。
- `security_classification`：标记敏感级别；MUST 决定脱敏与存储策略（见 SECURITY.md §12）。
- **校验失败行为**：缺失必填字段、schema 不匹配或签名/摘要失败的信封 MUST 被拒绝并记录审计；MUST NOT 猜测或静默丢弃。

---

## 5. 消息与事件类型

至少覆盖：

- `command`：调度指令（如执行、取消）。
- `result`：角色执行结果（结构化，须符合 Schema）。
- `status`：状态更新（如 RUNNING / STALLED）。
- `heartbeat`：存活与业务进度信号（与业务进度分离，见 §9）。
- `cancellation`：取消请求（非自动确认，见 §9）。
- `resume`：恢复请求（须携带 resume token / checkpoint，见 §9）。
- `review.finding`：审核发现（关联 finding_id / fingerprint）。
- `evidence`：可验证证据引用（关联 evidence_id / artifact_id）。
- `policy.decision`：政策引擎裁决（关联 policy_decision_id / 政策版本）。
- `delivery.status`：Outbox 投递状态（PREPARED/DISPATCHING/ACKNOWLEDGED/VERIFIED/RECONCILIATION_REQUIRED）。
- `reconciliation.request`：对账请求（状态未知动作）。
- `reconciliation.result`：对账结论。
- `audit.event`：审计事件（追加至 Event Ledger 哈希链）。

所有类型 MUST 绑定 `workflow_id`/`job_id`/目标版本，且消息不可原地修改，只能追加新版本（V1.1 §11）。

---

## 6. 适配器能力协商

连接建立时适配器 MUST 上报能力；**MUST NOT** 恢复已删除的 `supports_exactly_once_claim`（V1.1 §10.2）。

允许的能力字段：

```text
send
receive
stream
upload_artifact
download_artifact
execute
cancel
resume
health_check
structured_output
requires_unlocked_session
requires_human_auth
supports_idempotency_key
supports_status_query
supports_external_correlation_id
supports_reconciliation
delivery_semantics
max_payload_size
supported_protocol_versions
```

规则：

- 工作流引擎 MUST 先检查能力再调度，MUST NOT 依赖运行时猜测。
- `delivery_semantics` 只能是：`at_least_once` / `idempotent_preferred` / `reconciliation_required` 之一；MUST NOT 声明 `exactly_once`。
- `supports_exactly_once_claim` 为**禁止字段**，任何适配器 MUST NOT 上报。

---

## 7. 生命周期

完整状态（V1.1 §20.1）：

```text
LOADING
NEGOTIATING
READY
BUSY
DEGRADED
STOPPING
STOPPED
FAILED
```

进入/退出条件与迁移：

| 状态 | 进入条件 | 退出条件 | 允许迁移 | 禁止迁移 |
|---|---|---|---|---|
| LOADING | 初始/开始加载 | 加载成功 | → NEGOTIATING | 不得直接 READY |
| NEGOTIATING | 加载成功，协议/能力协商 | 协商成功且协议兼容 | → READY | 不得跳过 READY |
| READY | 协商成功且协议兼容 | 接收任务 | → BUSY / → STOPPING | —— |
| BUSY | 接收任务 | 任务正常结束 | → READY / → DEGRADED / → STOPPING / → FAILED | —— |
| DEGRADED | 能力部分丢失但仍可服务 | 恢复关键能力 | → READY | —— |
| STOPPING | 收到关闭请求 | 正常关闭完成 | → STOPPED | —— |
| STOPPED | 正常关闭完成 | —— | （终态） | —— |
| FAILED | 不可恢复异常 | 重新加载或政策决定替换 | （终态） | **不得自动回到 READY** |

- `LOADING` 是 load **正在执行**状态，不是 load 成功后的状态。
- `NEGOTIATING` 在加载**成功后**进行。
- `READY` 才表示可接收任务。
- `DEGRADED` 恢复关键能力后可回到 `READY`。
- `FAILED` **MUST NOT** 自动回 `READY`；必须重新加载或由政策决定替换。
- 运行期能力增删 MUST 主动上报 `capability change event`，调度器据以重路由或降级。

---

## 8. 错误模型

每个错误 MUST 包含：

```text
error_code
category
retriable
retry_after
safe_to_retry
external_state_known
details
evidence
terminal_recommendation
```

错误类别（category）至少覆盖：

- `validation`：输入/schema 校验失败（retriable=false）
- `authentication`：认证失败（retriable=false）
- `authorization`：授权被拒（retriable=false）
- `capability`：能力不支持（retriable=false）
- `timeout`：超时（retriable 视是否幂等）
- `transport`：传输失败（retriable=true，受 attempt 上限约束）
- `external_state_unknown`：外部状态未知（retriable=false，须进入 RECONCILIATION_REQUIRED）
- `policy_denied`：政策拒绝（retriable=false）
- `budget_exhausted`：预算耗尽（retriable=false，终态）
- `protocol_incompatible`：协议不兼容（retriable=false，拒绝加载）
- `internal`：内部错误（retriable 视上下文）
- `cancelled`：已取消（retriable=false）

规则：

- `external_state_known=false` 时 MUST NOT 自动盲重试；MUST 进入对账或安全终态。
- `terminal_recommendation` 由政策引擎裁决，MUST 映射到 V1.1 §8.2 已有终态之一。

---

## 9. Cancel / Resume / Heartbeat / Health

- **cancel 是请求，不是自动确认**：适配器 MUST 在 cancel 后报告已停止或正在执行动作的最终状态，MUST NOT 静默继续（V1.1 §20.1）。
- **adapter 必须报告最终动作状态**：即使 cancel，也 MUST 回报最终状态事件。
- **resume token 与 checkpoint**：resume MUST 携带 resume token 与最近 checkpoint；缺失则 MUST 拒绝。
- **heartbeat 与业务进度分离**：心跳仅表存活与进度；不得因固定轮询超时被打断（V1.1 §16.2）。
- **BUSY 不得因普通轮询超时被打断**：有心跳的长任务 MUST NOT 因轮询超时被重复提交。
- **health_check schema**：至少含 `status`、`last_success_time`、`capability_set`、`version`，格式固定。
- **无心跳先恢复状态，不直接重发**：无心跳时先尝试一次会话恢复/状态查询，不直接重发任务（V1.1 §4.3、§16.2）。

---

## 10. 投递语义

- **at-least-once attempts**：仅承诺"至少一次尝试"，不保证 exactly-once。
- **一个逻辑 message 只创建一次**：`message_id` 唯一，逻辑消息 MUST NOT 重复生成。
- **多个 delivery attempt 使用相同 idempotency_key**：attempt 序号递增（默认上限 3，绝对上限 10）。
- **状态未知时自动盲重试次数为 0**：未知外部状态 MUST NOT 盲重发；先 status query / reconciliation。
- **status query / reconciliation**：适配器支持时 MUST 先查询并对账。
- **`RECONCILIATION_REQUIRED`**：无法确定外部动作是否已执行时 MUST 进入该状态或对应安全终态。
- **不保证任意第三方副作用 exactly-once**（V1.1 §16.3）。

---

## 11. Finding / Evidence / Release Gate

- **finding fingerprint**：finding MUST 使用稳定指纹跨版本跟踪（V1.1 §9）。
- **severity candidates**：finding 须带严重度候选（HIGH / CRITICAL 等）。
- **evidence references**：finding MUST 关联可验证 evidence（evidence_id / artifact_id）。
- **unresolved reason**：未解决 finding MUST 记录未解决原因。
- **reevaluation condition**：须定义重新评估触发条件（如目标 SHA 变化）。
- **HIGH/CRITICAL 无法排除即阻断**：HIGH/CRITICAL 候选不可排除时 MUST 阻断晋级。
- **worker 的完成只生成待验证事件**：worker 完成 MUST 只生成 `result` / `review.finding` 待验证事件，不直接改变终态。
- **release gate 只接受 verifier 或外部可验证证据**：晋级门禁 MUST 仅接受 evidence_verifier 或外部可验证证据，不接受角色自述（V1.1 §15.3、§21）。

---

## 12. 安全约束

协议信封与载荷 **MUST NOT** 承载以下内容：

- 明文密钥（plaintext secret / API key / token value）；
- 未脱敏 Cookie / browser storage state / 用户目录 / 会话令牌；
- 超出角色必要范围的凭据；
- 未授权路径或域名（须匹配政策白名单）。

- 凭据 MUST 只以引用形式传递；真实值 MUST 存于 OS 凭据库（见 SECURITY.md §9）。
- 违反安全约束的消息 MUST 被拒绝并记录审计，MUST NOT 进入事件账本作为事实。
