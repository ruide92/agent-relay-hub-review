```text
Document Status: APPROVED
Normative Status: 已批准，当前为 Phase 0 Source of Truth
Approval Target Commit: b74e4076ce59e567de52d67916133a5d9ca26596
Product Baseline: AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md
Product Baseline Commit: c9cc522b4ab01008fed390c282d6bd5a816ee779
Architecture Baseline Approval Target: b8ef21978544e1261081389ecf736e72b80e49d7
Governance Approval Record: GOVERNANCE_APPROVAL_0003.md
Current Phase: Phase 0
Phase 1: NOT AUTHORIZED
Code Status: NO PRODUCT CODE
Implementation Status: DESIGN ONLY
Last Revised: 2026-07-21 (Reviewer-2 Round 2, mechanical cleanup)
```

# UX_INTERACTION.md（用户体验与交互契约）

> 本文档定义 ARH 的整体用户体验与交互契约。
> 本文档已通过 Reviewer-2 对 target commit `b74e4076ce59e567de52d67916133a5d9ca26596` 的独立审核；
> 已由 `GOVERNANCE_APPROVAL_0003.md` 批准；
> 当前为 Phase 0 Source of Truth；
> 批准只表示设计契约生效；
> 不表示 UI、代码、组件、skill、依赖或验收已经实现；
> Phase 0 仍为 OPEN / NOT_READY；
> Phase 1 仍为 NOT AUTHORIZED。
> 所有内容必须服从已批准 V1.1、ARCHITECTURE.md、PROTOCOL.md、SECURITY.md，且不与它们冲突。
> 权威事实仅来自 Event Ledger（ARCHITECTURE.md §2、§4）；UI 只是派生视图与本地操作入口。

---

## 1. 产品交互定位

ARH 是**自治智能体控制中心（autonomous agent control center）**，而不是聊天客户端。

- 首页**不是**聊天输入框；不提供"问任意问题"的自由文本主入口（与 V1.1 §6 角色模型一致——外部文本不能修改权限、循环、预算和目标）。
- 用户主要负责：**设定目标、配置政策、查看结果**，日常中继由系统自动完成。
- 普通运行**不得不断向用户提问**：心跳等待、传输重试、适配器切换、测试失败退回、finding 去重、修复循环、审核仲裁、证据保存、摘要生成均由系统自动处理（见 §3）。
- 界面必须始终回答六个问题：
  1. 系统**正在做什么**；
  2. 为什么**这么做**（policy decision / 当前阶段）；
  3. 是否**真的完成**（是否有 verifier/外部可验证证据）；
  4. 有什么**证据**（evidence_id / artifact_id / content hash）；
  5. 出错后**停在哪里**（终态 / 安全停止点）；
  6. 是否**需要用户介入**（A5 / 凭据重认证 / 治理事件）。

---

## 2. 目标用户（人类 UI 用户）

> 机器角色（worker / reviewer / evidence_verifier / messenger / repository / adapter / model）**不是**人类 UI 用户，它们走 ARP 通道（PROTOCOL.md §5），不出现在本节的角色权限中。

| 角色 | 看得见什么 | 能执行什么 | 不能执行什么 | 权限档位 |
|---|---|---|---|---|
| **owner** | 全部视图、治理、批准、凭据引用 | 批准、阶段关闭/授权、查看/导出脱敏诊断、A5 授权 | 不得绕过 Policy Engine 直接改事实；不得伪造完成 | 最高（含 A5 授权） |
| **governance approver** | 治理记录、政策、批准视图 | 受委派的治理批准（如 `GOVERNANCE_APPROVAL_*`） | 不得执行 A1–A5 外部动作；不得改工作流事实 | 治理档（非操作档） |
| **operator** | Control Room、Workflows、Evidence、Reconciliation、Adapters、Budgets、Audit、Safe Mode | 创建模拟工作流、启动、取消、恢复（仅 STALLED 后且须 resume token/checkpoint）、查看证据与 finding、触发备份/恢复演练、请求 A5 | 不得直接写 SQLite / projection / policy bundle / 终态；A5 默认禁止，须请求 | A0–A4（按预授权与令牌） |
| **reviewer** | 待审 workflow、finding、evidence | 提交 `review.finding`、请求重新验证、查看 evidence / verifier 结论 | 不得直接标 evidence 为 verified；不得直接确认 release gate；不得直写 workflow 终态；不得代替 evidence_verifier 生成可验证 evidence；不得执行外部动作 | A1–A3（审核/规划/候选） |
| **observer** | 只读视图（Control Room、Workflow List、Evidence、Audit） | 查看、导出脱敏诊断（A0） | 不触发任何 command（只读，见 UI_STATE_ACCEPTANCE_MATRIX.md §4） | A0 仅观察 |

权限档位 A0–A5 定义见 SECURITY.md §6；最终授权上限 = min(自报能力, 政策上限, 本次 capability token)，由 Policy Engine 裁决（V1.1 §4.2.1）。

> **职责分离（固定，不得越权）**：
> - **reviewer** → 仅提交 `review.finding`；
> - **evidence_verifier** → 仅产出 `evidence`；
> - **Policy Engine** → 仅做 release gate decision（裁决先产生 `policy.decision` 事件）；
> - **Event Ledger** → 记录事实；
> - **UI** → 仅展示 projection。
>
> reviewer 不得直接“确认证据”或批准终态；该职责由 evidence_verifier 与 Policy Engine 分别承担。

---

## 3. 无人值守交互原则

### 3.1 不得询问用户的事项（系统自动处理）

以下情形**不形成用户待办**，由系统按政策自动完成：

- 有心跳任务是否继续等待；
- 普通传输重试（满足 PROTOCOL.md §8.1 五条件时）；
- 备用适配器切换（在 policy 范围内）；
- 普通测试失败退回；
- finding 去重（按 fingerprint，V1.1 §9）；
- 常规修复循环（≤ 上限，V1.1 §9.1）；
- 审核冲突仲裁（在默认决策阶梯内，V1.1 §4.3.1）；
- 自动保存证据；
- 生成摘要；
- 预算内的正常模型选择；
- 默认政策已经覆盖的选择。

### 3.2 允许形成用户待办的情形

1. **A5 不可逆操作**（支付 / 删重要数据 / 合并受保护主分支 / 实盘 / 解除安全门禁，SECURITY.md §6）；
2. **policy integrity / trust root 失败**（进入 `POLICY_INTEGRITY_SAFE_MODE`，SECURITY.md §7）；
3. **默认决策阶梯执行后仍无合法路径**（进入 `BLOCKED_NEEDS_POLICY_DECISION`）；
4. **治理批准、阶段关闭或阶段授权**（须 owner/governance approver，GOVERNANCE_APPROVAL_*）；
5. **凭据必须由人重新认证且无安全替代方案**（SECURITY.md §9）。

### 3.3 用户介入时必备信息（不得只弹空对话框）

即使需要用户介入，界面 MUST 展示：

- 发生了什么；
- 当前安全停止点；
- 已采取的动作；
- 禁止继续的原因；
- 推荐处理方案；
- 精确权限范围（A 档 / 令牌范围）；
- 不处理的后果。

### 3.4 禁止的弹窗文案

禁止只弹出以下无意义内容：

- "是否继续？"
- "请选择一个工具。"
- "任务失败了。"
- "需要人工干预。"

---

## 4. UI 命令契约

每个用户操作 MUST 使用统一表格；UI 命令**不是事实**，按钮点击后**不能直接显示成功**，必须等待 Event Ledger 事件、Policy Decision 与所需 Evidence（见 §6）。

| Interaction ID | User Intent | Preconditions | UI Action | Core Command | Policy Check | Pending State | Required Evidence | Success Event | Failure/Safe State | Audit Event |
|---|---|---|---|---|---|---|---|---|---|---|
| UX-CREATE-SIM | 创建模拟工作流 | A2 范围内、mock adapter 可用 | 打开创建向导，填目标/政策 | `workflow.create_simulation` | A2（沙箱/任务分支），Phase 1 仅 mock | `COMMAND_SUBMITTED` | 目标固化事件 + policy decision | `status: PENDING` 落 Event Ledger | `BLOCKED_*` / `POLICY_INTEGRITY_SAFE_MODE` | `audit.event` |
| UX-START | 启动工作流 | `PENDING` + 授权预检通过 | 点击启动（先显示「启动请求已提交」） | `workflow.start` | A1–A3 + capability token | `AWAITING_LEDGER_EVENT` | 权限预检 policy decision | `status: RUNNING`（仅收到权威事件后显示运行中） | `BLOCKED_AUTH_EXPIRED` / `BLOCKED_BUDGET_EXHAUSTED` | `audit.event` |
| UX-CANCEL | 取消请求 | 任意非终态 | 发起取消请求 | `workflow.cancel_request` | A1；A5 取消受保护须升级 | `CANCELLATION_REQUESTED` | adapter 报告最终动作状态（PROTOCOL §9） | `CANCELLED_BY_POLICY` | adapter 仍在执行→ `RECONCILIATION_REQUIRED` | `audit.event` |
| UX-RESUME | 恢复（仅 `STALLED` 后） | `STALLED`（须 resume token/checkpoint） | 提交 resume token/checkpoint | `workflow.resume` | A1 | `RESUME_REQUESTED` | resume token + checkpoint（PROTOCOL §9） | 恢复成功事件 | 缺失 token→拒绝，保持原态 | `audit.event` |
| UX-VIEW-EVIDENCE | 查看证据 | 有 evidence_id | 打开 Evidence Detail | `evidence.query` | A0 | 只读视图 | evidence_id / artifact_id / content hash | 派生视图加载 | 证据缺失→空态提示 | （只读，无写） |
| UX-VIEW-FINDING | 查看 finding | 有 finding_id | 打开 Finding Detail | `review.finding.query` | A0/A1 | 只读视图 | finding_id / fingerprint | 派生视图加载 | 无→空态 | （只读） |
| UX-REQUEST-REVERIFY | 请求重新验证 | finding 待裁决 | 发起重新验证 | `evidence.reverify_request` | A1 | `AWAITING_LEDGER_EVENT` | verifier 输出 | `evidence` 事件 + policy decision | 仍高危→ `release blocked` | `audit.event` |
| UX-VIEW-RECON | 查看对账 | 有 `RECONCILIATION_REQUIRED` | 打开 Reconciliation Case | `reconciliation.query` | A0 | 只读视图 | reconciliation.result | 派生视图 | 状态未知→保持对账 | （只读） |
| UX-EXPORT-DIAG | 导出脱敏诊断 | A0 或 safe mode 内 | 导出脱敏包 | `diagnostic.export` | A0；safe mode 仅本地（SECURITY §7） | `COMMAND_SUBMITTED` | 脱敏后的诊断数据 | 导出完成事件 | 含 secret→拒绝（SECURITY §12） | `audit.event` |
| UX-VIEW-POLICY | 查看政策 | 有签名 policy 引用 | 打开 Policy Viewer | `policy.query` | A0 | 只读视图 | policy version / key_id | 派生视图 | 不可读→ safe mode 提示 | （只读） |
| UX-REQUEST-A5 | 请求 A5 批准 | A5 操作待授权 | 提交 A5 请求 + 范围 | `auth.a5_request` | A1 提交本地申请；目标操作 A5 默认禁止，须 owner 批准和 Policy Engine capability token | `AWAITING_POLICY_DECISION` | 精确权限范围说明 | owner APPROVED record + policy.decision | owner REJECTED / request expired / no action executed | `audit.event` |
| UX-VIEW-BUDGET | 查看预算 | 有 budget 计数 | 打开 Budget Detail | `budget.query` | A0 | 只读视图 | budget 计数 / 上限 | 派生视图 | 耗尽→ `BLOCKED_BUDGET_EXHAUSTED` 提示 | （只读） |
| UX-TRIGGER-BACKUP | 触发备份 | A2 预授权 | 触发一致性备份 | `backup.trigger` | A2 | `BACKUP_REQUESTED` | backup set ID + 哈希 | backup 完成事件 | 哈希不一致→安全终态（ARCHITECTURE §8） | `audit.event` |
| UX-TRIGGER-RESTORE-DRILL | 触发恢复演练 | A2 预授权 | 触发恢复演练 | `restore.drill` | A2 | `RESTORE_DRILL_REQUESTED` | 恢复成功判据（哈希+重放+对账） | 演练通过事件 | 不一致→不宣称成功 | `audit.event` |
| UX-RELOAD-APPROVED-POLICY | 重新加载已批准 policy bundle | 当前 `POLICY_INTEGRITY_SAFE_MODE` | 选择/发现 owner 控制的可信治理工具已提供的批准 bundle | `policy.reload_approved_bundle` | 仅本地 safe-mode 恢复路径，不提升 A1–A5 | `AWAITING_POLICY_VALIDATION` | bundle 内容哈希 + 治理批准引用 + 签名 + key_id + key purpose + trust-root 校验结果 | 可信政策验证通过并产生 Event Ledger 审计事件 | 验证失败→保持 `POLICY_INTEGRITY_SAFE_MODE` | 脱敏的 policy reload validation event |
| UX-RECREATE-MOCK-ADAPTER | 重建 mock adapter 实例 | adapter 当前 instance=`FAILED`（仅 Phase 1 mock） | 请求重建 mock 实例 | `adapter.recreate_mock_instance` | A2（仅 mock，禁止真实适配器） | `AWAITING_NEW_ADAPTER_SESSION` | 旧 session FAILED 终态 + 新 `session_id` + 新实例从 `LOADING` 起的 lifecycle event | 新 instance 进入 `LOADING`→`NEGOTIATING` | 失败→保持/重回 `FAILED` | `audit.event` |

规则：

- **UI command 不是事实**：Core 处理后须产生 Event Ledger 事件（带前序哈希）才算权威状态变更。
- **cancel 是请求，不是自动完成**：须等待 adapter 回报最终动作状态（PROTOCOL.md §9）。
- **UI 不得直接写** SQLite、projection、policy bundle 或终态；只能提交本地操作经 Core 裁决。

### 4.1 本地交互等待态（UI 临时态）

按钮点击后 UI 只能进入**前端临时交互状态**，绝不代表业务事实：

- `COMMAND_SUBMITTED`
- `AWAITING_POLICY_DECISION`
- `AWAITING_LEDGER_EVENT`
- `CANCELLATION_REQUESTED`
- `RESUME_REQUESTED`
- `BACKUP_REQUESTED`
- `RESTORE_DRILL_REQUESTED`
- `AWAITING_POLICY_VALIDATION`
- `AWAITING_NEW_ADAPTER_SESSION`
- `REQUEST_TIMEOUT`

规则：

- 这些状态**仅属 UI 临时交互状态**，不是 workflow / adapter / Outbox 的权威状态；
- **不得写入 projection 冒充业务事实**；
- 收到 Event Ledger 事件后，UI 临时态 MUST 被权威 projection 替换；
- 例：点击 Start 后**不得立即显示 `RUNNING`**，先显示「启动请求已提交」；仅收到权威 `status: RUNNING` 事件后才显示运行中。
- `REQUEST_TIMEOUT` 是 UI 本地展示态：**不是** workflow、adapter、Outbox 或 Event Ledger 状态；
- UI 请求超时只能进入本地 `REQUEST_TIMEOUT` 或 `UI-STALE` 展示；
- UI 超时**不得自行生成** `BLOCKED_*`、`FAILED_*`、`CANCELLED_*` 或其他领域终态；
- 只有收到 Event Ledger 中的权威阻塞/失败事件后，才能显示对应 `BLOCKED_*` 或 `FAILED_*`；
- 重新查询成功后，以最新 projection 替换本地超时状态；
- **不得无限 loading**。

---

## 5. 工作流详情交互（证据化时间线）

定义统一证据化时间线（与 V1.1 §5 流程一致）：

```text
目标固化 → 权限预检 → 规划 → 执行 → 测试 → 审核 → 证据验证
→ 修复 → 复审 → 发布候选 → 终态
```

每个节点 MUST 至少展示：

- actor / adapter（角色身份，区分人机）；
- 开始和结束时间（UTC RFC3339，PROTOCOL.md §3）；
- input revision（输入版本 / target）；
- output artifact（产物引用 + SHA256）；
- attempt（投递尝试序号）；
- budget（已用/上限）；
- policy decision（policy_decision_id + 政策版本）；
- evidence（evidence_id / artifact_id）；
- finding（finding_id / fingerprint / 严重度候选）；
- 当前状态（来自 projection，非权威）；
- 是否可验证（verifier 输出 / 外部可验证）；
- 阻塞原因（如 `release blocked`）。

高级节点详情折叠展示（见 UI_INFORMATION_ARCHITECTURE.md §6）：message_id、workflow_id、job_id、policy_decision_id、adapter/session ID、hashes、raw schema、audit chain。

---

## 6. 完成语义（防伪完成）

必须严格区分以下层级，**不得把低层当作完成**：

| 层级 | 含义 | 是否可展示为"完成" |
|---|---|---|
| `actor reported complete` | 角色自述完成 | **否**（PROTOCOL §11：自述不能改终态） |
| `result received` | 收到 `result` 事件 | **否**（仅待验证） |
| `evidence pending` | 待 verifier 输出 | **否** |
| `evidence verified` | `evidence` 事件 + 校验通过 | **否**（仍须 release gate） |
| `release gate passed` | Policy Engine 裁决通过 | **条件**（仍待终态） |
| `workflow terminal completed` | 进入 `COMPLETED_APPROVED` / `COMPLETED_WITH_DEFERRED_LOW_RISK` | **是**（终态 + 所需证据） |

**禁止展示为"任务已完成"的内容**：

- worker 自述（无论文字如何肯定）；
- 消息发送成功；
- adapter ACK；
- 文件存在；
- UI 动画结束。

---

## 7. 异常与恢复体验

每种异常状态 MUST 定义七个要素：用户看到的标题、一句话解释、系统自动做了什么、当前是否安全、可用操作、禁止操作、需要的证据、如何离开该状态。

| 状态 | 标题 | 解释 | 自动动作 | 安全？ | 可用操作 | 禁止操作 | 证据 | 离开条件 |
|---|---|---|---|---|---|---|---|---|
| `STALLED` | 已停滞 | 心跳丢失超阈值 | 尝试一次会话恢复/状态查询 | 是 | 查看、恢复 | 直接重发任务 | 最后心跳时间 | 恢复成功→`RUNNING` |
| `DEGRADED` | 已降级 | adapter 能力部分丢失 | 重路由/降级服务 | 是 | 查看；导航至受影响的工作流（NAV） | 强制提权 | capability_change 事件 | 恢复关键能力→`READY` |
| `FAILED_EXTERNAL_DEPENDENCY` | 外部依赖失败 | 外部系统确定失败 | 标记终态、写摘要 | 是 | 查看、请求重审 | 盲重试 | 外部失败证据 | 依赖恢复+新授权 |
| `BLOCKED_AUTH_EXPIRED` | 授权已过期 | 凭据/认证失效且无安全恢复 | 停止外部动作 | 是 | 重新认证（人） | 自动 A1–A5 | auth 失效事件 | 安全重认证+新令牌 |
| `BLOCKED_BUDGET_EXHAUSTED` | 预算耗尽 | 预算计数达上限 | 进入终态 | 是 | 查看、请求增额（治理） | 绕过预算 | budget 计数 | 治理增额+新授权 |
| `BLOCKED_NEEDS_POLICY_DECISION` | 需政策决策 | 默认阶梯无合法路径 | 停止、写摘要 | 是 | 查看、提交治理 | 自动续跑 | 决策阶梯记录 | 新政策/owner 决策 |
| `BLOCKED_REDESIGN_REQUIRED` | 需重新设计 | 修复/循环上限耗尽 | 进入终态 | 是 | 查看、提交重设计 | 继续修复 | 循环计数 | 新设计+授权 |
| `QUARANTINED_SECURITY_RISK` | 安全隔离 | 安全风险/副作用不确定 | 隔离、停止外部动作 | 是 | 查看（脱敏）、上报 | 解除隔离 | 安全证据 | owner 安全确认 |
| `RECONCILIATION_REQUIRED` | 需对账 | 外部状态未知 | 进入对账、0 次盲重试 | 是 | 查看对账、查询 | 盲重发 | reconciliation.result | 确定性证据→`VERIFIED` 或安全终态 |
| `POLICY_INTEGRITY_SAFE_MODE` | 安全模式 | 政策/信任根校验失败 | 仅本地 A0 诊断 | 是 | 查看、导出诊断、重新加载已批准政策(本地只读验证, 不提权) | A1–A5、外部调用、编辑/签署/修复 policy、自动生成 trust root | 校验失败记录 + 重载时 bundle 哈希/批准引用/签名/key_id/key purpose/trust-root 校验结果 | 可信政策重载校验通过（UX-RELOAD-APPROVED-POLICY） |
| `CANCELLED_BY_POLICY` | 已取消 | 政策主动取消 | 停止、写摘要 | 是 | 查看 | 自动续跑 | 取消 policy decision | ——（终态） |

---

## 8. 通知策略

五级通知（与 V1.1 §16 进度/告警一致）：

| 级别 | 名称 | 触发 | 是否打断 |
|---|---|---|---|
| `Silent` | 静默 | 普通进度、心跳 | 否 |
| `In-app` | 应用内 | 状态变更、节点完成 | 否 |
| `Summary` | 摘要 | 夜间非关键问题合并 | 否（次日摘要） |
| `Attention Required` | 需关注 | A5 请求、blocked、quarantine | 是（横幅） |
| `Critical Security` | 关键安全 | safe mode、approval forgery 风险、A5 执行 | 是（强提醒） |

规则：

- 普通进度**不推送**；相同问题**去重**（按 fingerprint）；
- 夜间非关键问题进入**摘要**；
- `security safe mode`、`A5 请求`、`治理事件` 可以打断；
- 通知本身**不得包含 secret 或敏感 payload**（SECURITY.md §12、§14：`RESTRICTED` 仅含引用/哈希）。
