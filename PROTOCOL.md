```text
Document Status: APPROVED
Normative Status: 已批准，当前为 Phase 0 Source of Truth
Approval Target Commit: b8ef21978544e1261081389ecf736e72b80e49d7
Product Baseline: AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md
Product Baseline Commit: c9cc522b4ab01008fed390c282d6bd5a816ee779
Governance Baseline Commit: cf8e66b431b36103fb7c049048521c15df2e3701
Governance Approval Record: GOVERNANCE_APPROVAL_0002.md
Current Phase: Phase 0
Phase 1: NOT AUTHORIZED
Code Status: NO PRODUCT CODE
```

# PROTOCOL.md（规范性协议草案 ARP v1）

> 本文件建立 ARH 统一消息/事件协议的规范性草案（ARP v1）。
> 本文件已通过 Reviewer-2 对 target commit `b8ef21978544e1261081389ecf736e72b80e49d7` 的独立审核，
> 并已由 `GOVERNANCE_APPROVAL_0002.md` 批准，当前为 **Phase 0 Source of Truth**。
> 批准表示设计契约生效，**不表示**产品代码、安全控制或测试已经实现；Phase 0 仍未关闭，Phase 1 仍未授权。
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
| `adapter_id` | 适配器注册周期内稳定 | 该适配器注册 | 不跨注册周期复用 | 控制平面消息必绑（negotiation/lifecycle/heartbeat/health_check/capability_change） |
| `session_id` | 一次加载/连接实例唯一 | 该 adapter 实例 | 不跨重启复用 | 控制平面消息必绑；adapter 进入 `FAILED` 后新建实例 MUST 产生新 `session_id` |

规则：

- 所有标识符 MUST 由生成方保证唯一且不可复用（即使逻辑实体被取消或重做）。
- `idempotency_key` 用于跨 `delivery_attempt` 去重；同一逻辑 message 的多次 attempt MUST 复用相同 `idempotency_key` 与 `correlation_id`。

---

## 3. 时间和序列化

- 消息体与载荷 MUST 为 **UTF-8 编码的 JSON**。
- 所有时间戳 MUST 使用 **UTC，RFC3339** 格式（如 `2026-07-21T00:00:00Z`）。
- 每条消息/事件 MUST 携带 `schema_version` 以标识结构版本。
- **未知字段**：接收方 MUST 保留未知字段用于透传/审计；MUST NOT 因未知字段拒绝合法消息（除非明确 schema 约束禁止）。
- **必填字段（公共必填）**：`protocol_version`、`schema_version`、`message_id`、`sender_role`、`recipient_role`、`message_type`、`created_at`、`security_classification` 为 MUST；缺失即校验失败（字段分类见 §4.1）。
- **canonicalization / hashing 输入规则**：用于完整性校验的哈希 MUST 基于规范化（canonical）JSON（键有序、空白剔除），输入字段集 MUST 在 schema 中固定（见 §4.3）。

---

## 4. 消息 Envelope

### 4.1 字段分类

| Field | Requirement | Applicable Message Types | Meaning |
|---|---|---|---|
| `protocol_version` | 公共必填 | 所有消息 | 协议版本（见 §1） |
| `schema_version` | 公共必填 | 所有消息 | 结构版本 |
| `message_id` | 公共必填 | 所有消息 | 全局唯一，不可复用 |
| `sender_role` | 公共必填 | 所有消息 | 发送方角色 |
| `recipient_role` | 公共必填 | 所有消息 | 接收方角色 |
| `message_type` | 公共必填 | 所有消息 | 消息/事件类型（见 §5） |
| `created_at` | 公共必填 | 所有消息 | UTC RFC3339 时间戳 |
| `security_classification` | 公共必填 | 所有消息 | 安全分类（PUBLIC/INTERNAL/CONFIDENTIAL/RESTRICTED，见 SECURITY.md §14） |
| `workflow_id` | 条件必填（工作流消息） | command / result / status / review.finding / evidence / policy.decision / delivery.status / reconciliation.* / audit.event | 工作流标识 |
| `workflow_version` | 条件必填（工作流消息） | 同上 | 工作流版本 |
| `job_id` | 条件必填（工作流消息） | 同上 | 作业标识 |
| `correlation_id` | 条件必填（工作流消息） | 同上 | 关联上下文 |
| `delivery_attempt_id` | 条件必填（投递尝试） | delivery.status / reconciliation.* | 投递尝试序号 |
| `idempotency_key` | 条件必填（投递尝试） | delivery.status / reconciliation.* | 幂等键 |
| `attempt` | 条件必填（投递尝试） | 同上 | 投递尝试计数 |
| `target` | 可选 / 按类型必填 | command / cancellation / resume | 作用对象（repository / branch / revision 等） |
| `artifacts` | 可选 / 按类型必填 | result / evidence | 附件数组，每个 MUST 带 `sha256` 与 `uri`（内容寻址，V1.1 §11、§12.3） |
| `payload` | 可选 / 按类型必填 | result / command / evidence | 结构化载荷 |
| `expires_at` | 可选 | 所有（建议） | 消息过期时间；过期消息 MUST NOT 被当作新结果处理（V1.1 §11） |
| `resume_token` / `checkpoint` | 可选 / resume 必填 | resume | 恢复令牌 / 检查点 |
| `supersedes_message_id` | 可选 | 修正 / 替代消息 | 关联被替代旧消息（见 §4.4） |
| `integrity` | 公共必填（至少 content_hash） | 所有消息（Phase 1） | 完整性对象（见 §4.3） |
| `adapter_id` | 控制平面/所有消息（条件） | capability negotiation / lifecycle / heartbeat / health_check / capability_change / workflow 消息可选 | 适配器逻辑身份，注册周期内稳定 |
| `session_id` | 控制平面/所有消息（条件） | 同上 | 一次加载/连接实例身份，不跨重启复用 |

说明：

- **所有消息公共必填**：上述 8 个公共字段 + `integrity`（至少 `content_hash`）为每条消息 MUST 携带。
- **工作流消息条件必填**：`workflow_id` / `workflow_version` / `job_id` / `correlation_id`；缺失即校验失败。
- **投递尝试条件必填**：`delivery_attempt_id` / `idempotency_key` / `attempt`；缺失即校验失败。
- **控制平面消息**（capability negotiation、lifecycle、heartbeat、health_check 等）可以没有 `workflow_id` / `job_id`，但 MUST 绑定 `adapter_id` / `session_id`，并携带全部公共必填字段；adapter 进入 `FAILED` 后新建实例 MUST 产生新的 `session_id`（旧 `session_id` 不再复用）。
- **工作流消息**可以同时携带 `adapter_id` / `session_id`（如执行的适配器实例身份），但不是工作流消息的必填项。

### 4.2 公共必填字段语义

- `sender_role` / `recipient_role`：MUST 为已定义角色（如 `worker.primary`、`reviewer.security`、`policy.engine`）。
- `target`：描述作用对象；MUST 在能力 / 政策允许范围内。
- `attempt`：delivery_attempt 序号，从 1 递增。
- `expires_at`：消息过期时间；过期消息 MUST NOT 被当作新结果处理（V1.1 §11）。
- `security_classification`：标记敏感级别；MUST 决定脱敏与存储策略（见 SECURITY.md §14）。
- `integrity`：完整性对象（见 §4.3）。

### 4.3 integrity 完整性对象

每条消息（Phase 1）MUST 携带 `integrity` 对象，至少含规范化内容哈希：

```text
integrity:
  hash_algorithm
  content_hash
  signature_algorithm
  key_id
  signature
  key_purpose
```

规则：

- **哈希预映像（hash preimage）**：canonical hash 的输入为完整 envelope + payload，但 **MUST 排除** `integrity.content_hash` 与 `integrity.signature`：

  ```text
  hash_preimage = canonical_json(
    complete_message_without(
      integrity.content_hash,
      integrity.signature
    )
  )
  ```

- `hash_algorithm` 始终存在，并且 MUST 纳入规范化哈希。
- 除 `integrity.content_hash` 与 `integrity.signature` 外，所有实际存在于消息中的 integrity 元数据 MUST 纳入规范化哈希。
- 当消息要求签名时，`signature_algorithm`、`key_id`、`key_purpose` MUST 在计算 content_hash 前确定，并纳入 hash preimage；`signature` 本身始终排除在 hash preimage 之外。
- 当消息不要求签名时，`signature_algorithm`、`key_id`、`key_purpose`、`signature` MUST 全部省略，因此这些不存在的字段不进入 hash preimage。
- **先计算 `content_hash`**，再对 `content_hash` 及规定的签名元数据签名（如需签名）。
- **MUST NOT** 把 `content_hash` 自身放入其计算输入。
- 未知但被保留的字段仍 MUST 纳入 canonical hash（防止未签名字段被篡改）。
- Phase 1 所有消息 **MUST** 携带 `hash_algorithm` + `content_hash`（必填）。
- `signature_algorithm`、`key_id`、`signature`、`key_purpose` **仅在该信任边界或消息类型要求签名时必填**；不要求签名时这些字段 **MUST 省略**，不得伪造空签名。
- 签名是否必需由**信任边界与消息类型**决定（见 SECURITY.md §8 / §14）；非受限边界的内部消息可仅带哈希。
- 文档 MUST NOT 引用不存在的"签名校验字段"；签名验证以 `integrity.signature` + `key_id` + `key_purpose` 为准（见 SECURITY.md §8）。

### 4.4 消息不可原地修改

- 已接受消息**不可原地改写**。
- 修正或替代消息 MUST 生成新的 `message_id`。
- 使用 `supersedes_message_id` 关联旧消息。
- 旧消息仍保留在 Event Ledger。

### 4.5 校验失败行为

缺失公共必填字段、schema 不匹配或签名 / 摘要失败的信封 MUST 被拒绝并按 §12 记录审计；MUST NOT 猜测或静默丢弃。

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
- `capability_change`：运行期能力增删事件（须含变更前后 `capability_set` / `adapter_id` / `session_id`）。

所有类型 MUST 绑定标识：工作流消息绑 `workflow_id` / `job_id` / 目标版本；控制平面消息绑 `adapter_id` / `session_id`；消息不可原地修改，仅可追加新版本（见 §4.4）。

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
- `delivery_semantics` 取值枚举：**`at_least_once_attempts`** / **`at_most_once_attempt`** / **`unknown`**。
  - `idempotent_preferred` 与 `reconciliation_required` **不得**作为 `delivery_semantics` 取值——它们已由独立布尔能力字段 `supports_idempotency_key` 与 `supports_reconciliation` 表示，二者非互斥投递语义。
  - **MUST NOT** 声明 `exactly_once` 或 `exactly_once_claim`。
- `supports_exactly_once_claim` 为**禁止字段**，任何适配器 MUST NOT 上报。

> ARH 正式保证仍然只是："at-least-once attempts + idempotency + reconciliation + safe stop when uncertain"（V1.1 §16.3）。

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
| DEGRADED | 能力部分丢失但仍可服务 | 恢复关键能力 | → READY / → STOPPING / → FAILED | —— |
| STOPPING | 收到关闭请求 | 正常关闭完成 | → STOPPED | —— |
| STOPPED | 正常关闭完成 | —— | （终态） | —— |
| FAILED | 不可恢复异常 | 创建新 adapter instance/session 重新加载 | （当前 adapter instance 终态） | **不得同实例自动回到 READY** |

- `LOADING` 是 load **正在执行**状态，不是 load 成功后的状态。
- `NEGOTIATING` 在加载**成功后**进行。
- `READY` 才表示可接收任务。
- `DEGRADED` 恢复关键能力后可回到 `READY`，也允许进入 `STOPPING` 或 `FAILED`。
- `FAILED` **MUST NOT** 同实例自动回 `READY`；`FAILED` 是当前 adapter instance 的终态，恢复 MUST 创建新的 adapter instance / session，并从 `LOADING` 开始。
- 运行期能力增删 MUST 主动上报 `capability_change` 事件（见 §5），调度器据以重路由或降级。

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
- `transport`：传输失败（retriable 视上下文；MUST NOT 自动等同可重试）
- `external_state_unknown`：外部状态未知（retriable=false，须进入 RECONCILIATION_REQUIRED）
- `policy_denied`：政策拒绝（retriable=false）
- `budget_exhausted`：预算耗尽（retriable=false，终态）
- `protocol_incompatible`：协议不兼容（retriable=false，拒绝加载）
- `internal`：内部错误（retriable 视上下文）
- `cancelled`：已取消（retriable=false）

规则：

- `external_state_known=false` 时 MUST NOT 自动盲重试；MUST 进入对账或安全终态。
- `terminal_recommendation` 由政策引擎裁决，MUST 映射到 V1.1 §8.2 已有终态之一。

### 8.1 自动重试前置条件

错误类别**不得自动决定**是否重试。自动重试 **ONLY** 在**同时满足**以下条件时才允许：

1. `retriable=true`；
2. `safe_to_retry=true`；
3. 已确认外部动作尚未执行，或适配器提供有效幂等保证；
4. 未超过 `attempt` 上限；
5. 政策引擎批准。

明确：

- `transport` / `timeout` 可能发生在外部副作用**已经执行之后**；
- `external_state_known=false` 时自动盲重试次数 **MUST 为 0**；
- `authentication` 只有在凭据或认证状态变化后才能重新尝试；
- `authorization` 只有在有效政策 / 令牌变化后才能重新尝试；
- 状态未知直接进入查询、对账或安全终态。

---

## 9. Cancel / Resume / Heartbeat / Health

- **cancel 是请求，不是自动确认**：适配器 MUST 在 cancel 后报告已停止或正在执行动作的最终状态，MUST NOT 静默继续（V1.1 §20.1）。
- **adapter 必须报告最终动作状态**：即使 cancel，也 MUST 回报最终状态事件。
- **resume token 与 checkpoint**：resume MUST 携带 resume token 与最近 checkpoint；缺失则 MUST 拒绝。
- **heartbeat 与业务进度分离**：心跳仅表存活与进度；不得因固定轮询超时被打断（V1.1 §16.2）。
- **BUSY 不得因普通轮询超时被打断**：有心跳的长任务 MUST NOT 因轮询超时被重复提交。
- **health_check schema**：至少含 `status`、`last_success_time`、`capability_set`、`version`，格式固定。
- **无心跳先恢复状态，不直接重发**：无心跳时先尝试一次会话恢复 / 状态查询，不直接重发任务（V1.1 §4.3、§16.2）。

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
- **worker 完成只产生 `result` 待验证事件**：worker 完成 MUST 只生成 `result` 待验证事件，不直接改变终态。
- **reviewer 才产生 `review.finding`**：审核发现 MUST 由 reviewer 角色生成，关联 finding_id / fingerprint。
- **evidence_verifier 产生 `evidence`**：可验证证据 MUST 由 evidence_verifier 生成，关联 evidence_id / artifact_id。
- **任意角色自述均不能直接改变 release gate 或终态**：角色声明 / 结果 / 证据只提交待验证事件，由 Policy Engine 裁决。
- **release gate 只接受 verifier 或外部可验证证据**：晋级门禁 MUST 仅接受 evidence_verifier 或外部可验证证据，不接受角色自述（V1.1 §15.3、§21）。

---

## 12. 安全约束

协议信封与载荷 **MUST NOT** 承载以下内容：

- 明文密钥（plaintext secret / API key / token value）；
- 未脱敏 Cookie / browser storage state / 用户目录 / 会话令牌；
- 超出角色必要范围的凭据；
- 未授权路径或域名（须匹配政策白名单）。

- 凭据 MUST 只以引用形式传递；真实值 MUST 存于 OS 凭据库（见 SECURITY.md §9）。
- 安全分类（`security_classification`）决定日志、持久化、脱敏与导出政策（见 SECURITY.md §14）。

### 12.1 非法 / 违规消息的审计语义

> 拒绝的恶意内容不能成为业务事实，但"发生过一次拒绝事件"必须成为审计事实。

- 原始敏感 payload MUST 被拒绝，不得持久化；
- 系统 MUST 向 Event Ledger 追加一条脱敏的 `security.validation_rejected` 或 `audit.event`；
- 该记录 MUST 包含：message_id（如安全）、sender / session、schema / error code、内容哈希、拒绝原因、时间、policy decision ID；
- MUST NOT 记录 secret、Cookie、storage state 或原始敏感 payload；
- 旧消息（如已接受后需替代）仍保留在 Event Ledger，新消息用新 `message_id` + `supersedes_message_id` 关联（见 §4.4）。

---

## 13. 机器可读契约映射（提案 #0007，PROPOSED）

> 本节为 **Phase 0 治理提案 #0007** 的新增内容，状态 **PROPOSED**，未经独立审核与 owner 批准，当前仅为提案映射草案。
> 语义服从已批准 V1.1 与本文件 §1–§12 既有内容，不得与之冲突。

### 13.1 Prose ↔ Schema 映射

| Prose 节 | 对应 Schema | 映射关系 |
|---|---|---|
| §1 协议版本 | `arp-envelope.schema.json` → `protocol_version` | 常量 `1`，Schema 校验确保 |
| §2 标识符 | `arp-envelope.schema.json` → 标识符字段 | 字段定义与唯一性规则一致 |
| §3 时间和序列化 | `common-defs.schema.json` → `utc_rfc3339` / `schema_version_v1` | 时间格式与 schema_version 规则一致 |
| §4 Envelope | `arp-envelope.schema.json` | 公共必填、条件必填、integrity、未知字段保留 |
| §5 消息类型 | `arp-message.schema.json` | 各 message_type 的条件 payload 约束 |
| §6 适配器能力协商 | `adapter-capability.schema.json` | 能力字段封闭枚举；禁止 `supports_exactly_once_claim` |
| §7 生命周期 | `adapter-lifecycle-event.schema.json` | 状态枚举与迁移白名单 |
| §8 错误模型 | `arp-message.schema.json` → `result` message payload | 错误类别枚举（`error_category`）与 `retriable` 语义；错误通过 `result` 消息 payload 中的 `result_status=error` 字段及 `error_category`、`error_message`、`retriable` 传达，不使用独立 `error` 消息类型 |
| §9 Cancel/Resume/Heartbeat/Health | `health-check.schema.json` | 健康检查必含 `capability_set` |
| §10 投递语义 | `arp-message.schema.json` → delivery.status | outbox 状态枚举 |
| §11 Finding/Evidence | `arp-message.schema.json` → review.finding / evidence payload | finding/evidence 必填字段 |
| §12 安全约束 | `arp-envelope.schema.json` → `security_classification` /禁止字段 | 禁止明文密钥 |

### 13.2 Canonicalization 与 content_hash

- Envelope 层 canonical hash 规则见 §4.3（不变）。
- 机器可读契约的 canonicalization 规则见 `MACHINE_READABLE_CONTRACTS.md` §6：envelope 层使用排除 content_hash/signature 的 canonical JSON（§4.3），token 层使用 JCS（RFC 8785）+ JWS，两层各自独立。
- 校验方 MUST NOT 混用 envelope 层与 token 层的 canonicalization。

### 13.3 Capability Token 在 ARP 中的携带位置

- Capability token MUST 通过独立的安全通道或受控信令传递。
- **MUST NOT** 将完整 token 字符串作为 ARP envelope `payload` 的普通字段。
- 在 ARP envelope 中引用 token 时，MUST 仅含 `jti`（token ID）与 token 哈希（`sha256:<hex>`），不得包含完整 token 字符串。
- **MUST NOT** 将完整 token 写入 Event Ledger、日志或截图。
- Token 的 wire format、签名容器、claims 与校验链见 `DECISIONS/ADR-0003-…`（PROPOSED）与 `CONTRACTS/capability-token.schema.json`。

### 13.4 Schema 冲突裁决

- 规范文本与 Schema 冲突时以规范文本（V1.1 → 本 PROTOCOL §1–§12 → `MACHINE_READABLE_CONTRACTS.md` → Schema）为准，不可下层覆盖上层（见 `MACHINE_READABLE_CONTRACTS.md` §9）。
- 本节映射不改变 §1–§12 的任何既有语义。
