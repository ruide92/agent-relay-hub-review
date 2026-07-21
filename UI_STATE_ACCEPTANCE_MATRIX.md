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

# UI_STATE_ACCEPTANCE_MATRIX.md（UI 状态验收矩阵）

> 本文件是第三包最重要的验收文件，定义 UI 在后端各状态下的**行为契约**与**防伪完成验收**。
> 本文档已通过 Reviewer-2 对 target commit `b74e4076ce59e567de52d67916133a5d9ca26596` 的独立审核；
> 已由 `GOVERNANCE_APPROVAL_0003.md` 批准；
> 当前为 Phase 0 Source of Truth；
> 批准只表示设计契约生效；
> 不表示 UI、代码、组件、skill、依赖或验收已经实现；
> Phase 0 仍为 OPEN / NOT_READY；
> Phase 1 仍为 NOT AUTHORIZED。
> 权威事实仅来自 Event Ledger（ARCHITECTURE.md §2、§4）；UI 状态为派生视图，刷新/重启 MUST 从同一 projection 恢复相同视图。

---

## 1. 主矩阵字段

| UI State ID | Page/Component | Backend Source | Trigger Event | Display State | Allowed Actions | Forbidden Actions | Required Evidence | Exit Condition | Phase | Acceptance Test |

- **Backend Source**：权威显示状态一律来自 **Event Ledger 派生 projection**（ARCHITECTURE.md §2、§4）。可选项：Event Ledger / Projection Builder / Artifact Store。
- **Policy Engine 不是界面事实状态库**：Policy Engine 是授权、预算、路由与 release gate 的**裁决者（decision producer）**；其裁决 MUST 先产生 `policy.decision` 事件进入 Event Ledger，UI 再从 projection 展示该裁决。Policy Engine 仅可出现在 `Policy Check`、`Trigger Event` 或“decision producer”说明中，不得作为权威显示状态的直接持久来源。
- **Trigger Event**：ARP 消息/事件类型（PROTOCOL.md §5），如 `status`、`delivery.status`、`review.finding`、`evidence`、`policy.decision`、`reconciliation.*`、`audit.event`、`capability_change`。
- **Display State**：UI_DESIGN_SYSTEM.md §5 状态视觉之一。
- **Required Evidence**：显示"完成/成功"所 MUST 具备的可验证证据（evidence_id / artifact sha256 / policy_decision_id / reconciliation.result）。
- **Acceptance Test**：Phase 1 用 mock events / mock adapter 演示（§5），不接真实外部系统。

### 1.1 Projection 健康元数据

每个 projection MUST 携带以下健康元数据，UI 据此判断陈旧（见 UI-STALE）：

- `projection_version`
- `last_applied_event_sequence`
- `ledger_head_sequence`
- `last_rebuilt_at`
- `last_successful_sync_at`

### 1.2 本地健康探测例外（非业务事实）

- `app boot` 与 `core unavailable` 可读取**本地实时 liveness / health probe**（本地健康检查、进程/连接可达性）；
- 该探测**只表示当前连接或进程可达性**，不是 workflow 业务事实；
- **不得**根据 health probe 伪造工作流状态、终态、授权或完成；
- Core 恢复后 MUST 与 Event Ledger / projection 重新同步；
- 可记录时，健康状态变化追加脱敏 audit event。

---

## 2. 必须覆盖的状态（主矩阵）

| UI State ID | Page/Component | Backend Source | Trigger Event | Display State | Allowed Actions | Forbidden Actions | Required Evidence | Exit Condition | Phase | Acceptance Test |
|---|---|---|---|---|---|---|---|---|---|---|
| UI-INIT-LOAD | App Shell | Projection（加载中） | app boot | loading (skeleton) | 等待 | 任何写操作 | —— | projection 可用 | 1 | 启动显示 Skeleton，不假进度 |
| UI-EMPTY | 各列表页 | Projection（空） | 无记录 | empty | Control Room / Workflow List 可创建模拟工作流（UX-CREATE-SIM，A2）；其他列表空态仅查看 | 误标"完成" | —— | 有记录 | 1 | 空态文案，无 success |
| UI-LOADING | 各详情页 | Projection（加载） | 导航进入 | loading | 等待 | 写事实 | —— | 数据加载 | 1 | Skeleton，无成功动画 |
| UI-STALE | 各页面 | Projection（派生自 Event Ledger） | 陈旧判据由政策配置（sequence lag = `ledger_head_sequence - last_applied_event_sequence` 或 `now - last_successful_sync_at` 超阈值），不硬编码于 UI | stale (warning) | 刷新、查看、导出诊断(A0) | 当作实时事实；依赖最新事实/授权的写操作 | `projection_version` + `last_applied_event_sequence` + `ledger_head_sequence` + `last_successful_sync_at` | 重放后一致且 lag 回到阈值内 | 1 | 显著陈旧横幅，明确标注「非实时，缓存状态」 |
| UI-CORE-DOWN | Global Health Banner | Core 健康探测 | 核心不可达 | degraded (danger) | 查看诊断(A0) | A1–A5、外部 | 健康检查失败记录 | 核心恢复 | 1 | 核心不可用时降级横幅，不伪装 |
| UI-ADAPT-LOAD | Adapter Detail | Event Ledger（adapter lifecycle projection） | `lifecycle: LOADING` | adapter loading | 查看 | 调度任务 | session_id | → NEGOTIATING | 1 | mock adapter LOADING 显示 |
| UI-ADAPT-NEGOT | Adapter Detail | Event Ledger（adapter lifecycle projection） | `lifecycle: NEGOTIATING` | adapter negotiating | 查看 | 跳过协商 | capability_set | → READY | 1 | 协商显示，协议不兼容拒绝 |
| UI-ADAPT-READY | Adapter Detail | Event Ledger（adapter lifecycle projection） | `lifecycle: READY` | adapter ready | 查看；导航至关联 workflow（NAV） | 提权 | capability_set | → BUSY/STOPPING | 1 | READY 显示，可接收 |
| UI-ADAPT-BUSY | Adapter Detail | Event Ledger（adapter lifecycle projection） | `lifecycle: BUSY` | adapter busy | 查看；导航至关联 workflow（NAV） | 直接重发 | job_id | → READY/DEGRADED/FAILED | 1 | BUSY 显示，心跳不误断 |
| UI-ADAPT-DEGR | Adapter Detail | Event Ledger（adapter lifecycle projection） | `lifecycle: DEGRADED` | adapter degraded | 查看；导航至受影响 workflow（NAV） | 强制提权 | capability_change | → READY/STOPPING/FAILED | 1 | DEGRADED 显著，可降级 |
| UI-ADAPT-FAIL | Adapter Detail | Event Ledger（adapter lifecycle projection） | `lifecycle: FAILED` | adapter failed | 查看、请求重建(UX-RECREATE-MOCK-ADAPTER, A2, 仅 mock) | 同实例 FAILED→READY；加载真实适配器 | 旧 session FAILED 终态 + 新 `session_id` + 新实例从 `LOADING` 起的 lifecycle event | 新 instance 从 LOADING | 1 | FAILED 终态，新 session 重建，禁止同实例恢复 |
| UI-WF-PEND | Workflow Detail | Workflow projection | `status: PENDING` | pending | 启动（UX-START）；取消（UX-CANCEL） | 标完成 | 目标固化事件 | → RUNNING | 1 | PENDING 不显示成功 |
| UI-WF-RUN | Workflow Detail | Workflow projection | `status: RUNNING` | running | 取消请求（UX-CANCEL） | 标完成 | 心跳 + policy decision | → 终态/STALLED | 1 | RUNNING 非完成（resume 仅 STALLED 后，须带 resume token） |
| UI-WF-STALL | Workflow Detail | Workflow projection | `status: STALLED` | stalled | 查看；恢复（UX-RESUME） | 直接重发 | 最后心跳时间 | → RUNNING | 1 | STALLED 显著，可恢复 |
| UI-WORKER-DONE | Workflow Detail | Event Ledger | `result` 事件 | verification pending | 查看证据 | **标完成** | evidence_id（待 verifier） | evidence verified | 1 | worker 自述不显完成 |
| UI-EVID-VER | Evidence Detail | Artifact + Event | `evidence` 事件 + 校验 | verified | 查看、请求 reverify（UX-REQUEST-REVERIFY） | 标终态完成 | evidence_id + sha256 + policy_decision | release gate passed | 1 | verified 非终态完成 |
| UI-FIND-HIGH | Finding Detail | Event Ledger（finding projection） | `review.finding` HIGH | finding HIGH | 查看、请求 reverify（UX-REQUEST-REVERIFY） | 隐藏严重度、自动降级 | finding_id + fingerprint | 未被充分证据排除时必须阻断；verifier 提供充分可验证证据排除后解除该 finding 阻断（insufficient evidence 不得自动降级） | 1 | HIGH 阻断晋级提示 |
| UI-FIND-CRIT | Finding Detail | Event Ledger（finding projection） | `review.finding` CRITICAL | finding CRITICAL | 查看、请求 reverify（UX-REQUEST-REVERIFY） | 隐藏严重度、自动降级、写「永远不得排除」 | finding_id + fingerprint | 未被充分证据排除时必须阻断；verifier 提供充分可验证证据排除后可解除（不得写「永远不得排除」；排除事实须有 evidence + policy decision） | 1 | CRITICAL 未排除必阻断 |
| UI-REL-BLOCK | Workflow Detail | Event Ledger（policy.decision projection） | `policy.decision`: release gate blocked | blocked | 查看；导航至 Policies & Authorization（NAV） | 标完成 | gate decision (policy_decision_id) | 证据排除风险 | 1 | release blocked 不显完成 |
| UI-RECON | Reconciliation | Reconciliation projection | `delivery.status: RECONCILIATION_REQUIRED` | reconciliation required | 查看、查询（UX-VIEW-RECON） | **盲重发** | reconciliation.result (external_state_known) | 确定性证据→VERIFIED/安全终态 | 1 | 0 次盲重试，进对账 |
| UI-BUD-WARN | Budget Meter | Budget projection | 预算≥阈值 | warning | 查看 | 绕过预算 | budget 计数 | 低于阈值/增额 | 1 | 警告色非 success |
| UI-BUD-EXH | Budget Meter | Budget projection | `budget_exhausted` | blocked (exhausted) | 查看；导航至 Policies & Authorization（NAV） | 继续消耗 | budget 计数达上限 | 治理增额+新授权 | 1 | 耗尽进 BLOCKED_BUDGET_EXHAUSTED |
| UI-AUTH-EXP | Workflow Detail | Event Ledger（policy.decision projection） | `policy.decision`: BLOCKED_AUTH_EXPIRED | blocked (auth) | 查看、重新认证(人) | 自动 A1–A5 | auth 失效事件（policy.decision） | 安全重认证+新令牌 | 1 | 过期进安全停止 |
| UI-POL-NEED | Workflow Detail | Event Ledger（policy.decision projection） | `policy.decision`: BLOCKED_NEEDS_POLICY_DECISION | blocked (policy) | 查看；导航至 Policies & Authorization（NAV） | 自动续跑 | 默认决策阶梯记录 | 新政策/owner 决策 | 1 | 阶梯穷尽才阻断 |
| UI-QUAR | Workflow Detail | Event Ledger（policy.decision projection） | `policy.decision`: QUARANTINED_SECURITY_RISK | quarantined | 查看；导航至 Policies & Authorization（NAV） | 解除隔离 | 安全证据 | owner 安全确认 | 1 | 隔离显著，不伪装 |
| UI-SAFE | Safe Mode Panel | Event Ledger（policy.decision projection） | `policy.decision`: POLICY_INTEGRITY_SAFE_MODE | safe mode | 查看、导出诊断（UX-EXPORT-DIAG）；重载批准政策（UX-RELOAD-APPROVED-POLICY） | **A1–A5/外部/模型/编辑或签署 policy** | 校验失败记录；重载时须 bundle 内容哈希+治理批准引用+签名+key_id+key purpose+trust-root 校验结果 | 可信政策重载校验通过并产生 Event Ledger 审计事件（UX-RELOAD-APPROVED-POLICY） | 1 | safe mode 控件全禁；重载失败仍留 safe mode |
| UI-CANCEL-REQ | Workflow Detail | Event Ledger | `cancellation` 请求 | cancellation requested | 查看 | 标已取消 | adapter 最终状态待回报 | adapter 回报 | 1 | 请求非确认 |
| UI-CANCEL-CONF | Workflow Detail | Event Ledger | `CANCELLED_BY_POLICY` | cancelled | 查看 | 自动续跑 | 取消 policy decision | ——(终态) | 1 | 取消为终态 |
| UI-BACKUP-RUN | Backup Page | Event Ledger（backup projection） | `backup.trigger` 事件 | backup running | 查看(A0) | 标恢复成功 | backup set ID + 哈希 | 哈希一致 | 1 | 运行中不显成功 |
| UI-BACKUP-FAIL | Backup Page | Event Ledger（backup projection） | 哈希不一致事件 | error (danger) | 查看(A0)、重试（UX-TRIGGER-BACKUP，A2） | 标成功 | 哈希不一致记录 | 重 backup | 1 | 失败不伪造成功 |
| UI-RESTORE-VER | Backup Page | Event Ledger（recovery projection） | `restore.drill` 事件 | restore verification | 查看(A0) | 标成功 | 恢复成功判据(哈希+重放+对账) | 判据全过 | 1 | 演练未过不显成功 |
| UI-TERM-APPROVED | Workflow Detail | Event Ledger | `COMPLETED_APPROVED` | completed | 查看 | —— | 终态 + evidence + policy_decision | ——(终态) | 1 | 仅终态+证据显完成 |
| UI-TERM-DEFERRED | Workflow Detail | Event Ledger | `COMPLETED_WITH_DEFERRED_LOW_RISK` | completed (deferred) | 查看 | 隐藏 deferred、仅显示绿色「完成」 | ①所有可信候选均低于 HIGH；②无未排除的 HIGH/CRITICAL 候选；③deferred finding 列表；④unresolved reason；⑤defer reason；⑥reevaluation condition；⑦evidence references；⑧release gate policy decision | ——(终态) | 1 | 须显式标注「低风险延期项」并列出 deferred 清单，不得仅显示绿色完成 |
| UI-EXT-FAIL | Workflow Detail | Event Ledger | `FAILED_EXTERNAL_DEPENDENCY` | blocked (external) | 查看；导航至 Evidence / Policy 页面（NAV） | 盲重试 | 外部失败证据 | 依赖恢复+新授权 | 1 | 外部失败进安全停止 |

> 所有 UI State ID 均映射到 ARCHITECTURE.md §6（工作流终态）/ §7（Outbox）/ PROTOCOL.md §5、§7（adapter lifecycle）/ §11（finding/evidence）。**不新增任何 workflow 终态**。
> 注：Pause / `PAUSED` 为延期能力，当前未批准；登记 `Pause is deferred pending a future approved protocol/state-machine change`。Phase 1 不得实现 pause，也不得将 pause 成功映射为 `STALLED`；`STALLED` 仍仅表示心跳丢失。

---

## 3. 防伪完成验收（自动检查）

UI MUST 通过以下自动检查，**不得把非完成展示为完成**：

- adapter `ACK` **不显示完成**（UI-ADAPT-* → 非 completed）；
- worker `result` **不显示完成**（UI-WORKER-DONE → verification pending）；
- `evidence pending` **不显示完成**（UI-WORKER-DONE / UI-EVID-VER）；
- `release gate blocked` **不显示完成**（UI-REL-BLOCK）；
- `reconciliation required` **不显示完成**（UI-RECON）；
- **仅** `terminal event` + `required evidence` 才显示完成（UI-TERM-APPROVED / UI-TERM-DEFERRED）；
- `stale projection` MUST 有显著提示（UI-STALE），不得当作实时事实；
- **UI 刷新不能改变事实状态**：视图仅来自 projection，刷新后状态一致；
- **UI 重启后 MUST 从 projection 恢复相同视图**（ARCHITECTURE.md §2、§8：projection 可重建）。

---

## 4. 权限验收

- **无权限动作**：隐藏还是禁用 MUST 由权限边界决定（UI_INFORMATION_ARCHITECTURE.md §3 Permission Boundary）；敏感动作禁用且带原因，而非静默隐藏导致误操作。
- **A5 默认禁止**：所有 A5 控件默认不可执行，须经 UX-REQUEST-A5 → owner 批准（SECURITY.md §6）。
- **capability token 过期**：令牌过期后相关动作 MUST 禁用，并提示重新授权（UI-AUTH-EXP）。
- **子代理不能扩大权限**：UI 不提供任何提权入口；最终授权由 Policy Engine 裁决（V1.1 §4.2.1、§4.2.2）。
- **UI 不能绕过 Policy Engine**：所有命令经 Core 裁决，UI 不得直写 SQLite / projection / policy bundle / 终态（ARCHITECTURE.md §4）。
- **safe mode 下 A1–A5 控件全部不可执行**（UI-SAFE，SECURITY.md §7）。
- **只读用户（observer）不得触发 command**：所有写操作隐藏/禁用（A0 仅观察，UX_INTERACTION.md §2）。
- **备份/恢复演练权限边界（Phase 1 本地模拟）**：查看备份状态 **A0**；触发一致性备份 **A2**；触发隔离恢复演练 **A2**；**A4 仅保留给未来真实非生产环境部署或恢复操作**，不得用于 Phase 1 mock 触发（UX-TRIGGER-BACKUP / UX-TRIGGER-RESTORE-DRILL）。

---

## 5. Phase 1 验收（mock only）

所有状态 MUST 通过 **mock events** 与 **mock adapters** 演示（ARCHITECTURE.md §1.1、§3：Phase 1 仅加载 mock adapter）。

**不得要求**：

- 真实 Git / GitHub；
- Qoder；
- ChatGPT；
- Playwright；
- UIA（Windows UI Automation）；
- 外部 API；
- NAS；
- STS；
- OKX。

> Phase 1 全部行为在本地模拟环境闭环（ARCHITECTURE.md §1.2）；任何状态若依赖真实外部系统，MUST 延后至对应阶段（Phase 3+）并用 mock 占位。
