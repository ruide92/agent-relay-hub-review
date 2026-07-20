```text
Document Status: PROPOSED
Normative Status: 未批准，当前不是 Source of Truth
Product Baseline: AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md
Product Baseline Commit: c9cc522b4ab01008fed390c282d6bd5a816ee779
Architecture Baseline Approval Target: b8ef21978544e1261081389ecf736e72b80e49d7
Governance Approval Record: GOVERNANCE_APPROVAL_0002.md
Current Phase: Phase 0
Phase 1: NOT AUTHORIZED
Code Status: NO PRODUCT CODE
Implementation Status: DESIGN ONLY
```

# UI_STATE_ACCEPTANCE_MATRIX.md（UI 状态验收矩阵）

> 本文件是第三包最重要的验收文件，定义 UI 在后端各状态下的**行为契约**与**防伪完成验收**。
> 当前为 `PROPOSED`，未经 Reviewer-2 审核与 owner 批准前**不属于 Source of Truth**；
> 不得描述为已批准、已实现或已测试；不得因完成 UI 设计就关闭 Phase 0。
> 权威事实仅来自 Event Ledger（ARCHITECTURE.md §2、§4）；UI 状态为派生视图，刷新/重启 MUST 从同一 projection 恢复相同视图。

---

## 1. 主矩阵字段

| UI State ID | Page/Component | Backend Source | Trigger Event | Display State | Allowed Actions | Forbidden Actions | Required Evidence | Exit Condition | Phase | Acceptance Test |

- **Backend Source**：Event Ledger / Projection Builder / Policy Engine / Artifact Store（ARCHITECTURE.md §3、§5）。
- **Trigger Event**：ARP 消息/事件类型（PROTOCOL.md §5），如 `status`、`delivery.status`、`review.finding`、`evidence`、`policy.decision`、`reconciliation.*`、`audit.event`、`capability_change`。
- **Display State**：UI_DESIGN_SYSTEM.md §5 状态视觉之一。
- **Required Evidence**：显示"完成/成功"所 MUST 具备的可验证证据（evidence_id / artifact sha256 / policy_decision_id / reconciliation.result）。
- **Acceptance Test**：Phase 1 用 mock events / mock adapter 演示（§5），不接真实外部系统。

---

## 2. 必须覆盖的状态（主矩阵）

| UI State ID | Page/Component | Backend Source | Trigger Event | Display State | Allowed Actions | Forbidden Actions | Required Evidence | Exit Condition | Phase | Acceptance Test |
|---|---|---|---|---|---|---|---|---|---|---|
| UI-INIT-LOAD | App Shell | Projection（加载中） | app boot | loading (skeleton) | 等待 | 任何写操作 | —— | projection 可用 | 1 | 启动显示 Skeleton，不假进度 |
| UI-EMPTY | 各列表页 | Projection（空） | 无记录 | empty | 创建（限 A2） | 误标"完成" | —— | 有记录 | 1 | 空态文案，无 success |
| UI-LOADING | 各详情页 | Projection（加载） | 导航进入 | loading | 等待 | 写事实 | —— | 数据加载 | 1 | Skeleton，无成功动画 |
| UI-STALE | 各页面 | Projection（陈旧） | 超过陈旧阈值 | stale (warning) | 刷新、查看 | 当作实时事实 | 最后事件时间戳 | 重放后一致 | 1 | 显著陈旧提示 |
| UI-CORE-DOWN | Global Health Banner | Core 健康探测 | 核心不可达 | degraded (danger) | 查看诊断(A0) | A1–A5、外部 | 健康检查失败记录 | 核心恢复 | 1 | 核心不可用时降级横幅，不伪装 |
| UI-ADAPT-LOAD | Adapter Detail | Adapter lifecycle | `lifecycle: LOADING` | adapter loading | 查看 | 调度任务 | session_id | → NEGOTIATING | 1 | mock adapter LOADING 显示 |
| UI-ADAPT-NEGOT | Adapter Detail | Adapter lifecycle | `lifecycle: NEGOTIATING` | adapter negotiating | 查看 | 跳过协商 | capability_set | → READY | 1 | 协商显示，协议不兼容拒绝 |
| UI-ADAPT-READY | Adapter Detail | Adapter lifecycle | `lifecycle: READY` | adapter ready | 查看、调度 | 提权 | capability_set | → BUSY/STOPPING | 1 | READY 显示，可接收 |
| UI-ADAPT-BUSY | Adapter Detail | Adapter lifecycle | `lifecycle: BUSY` | adapter busy | 查看、取消请求 | 直接重发 | job_id | → READY/DEGRADED/FAILED | 1 | BUSY 显示，心跳不误断 |
| UI-ADAPT-DEGR | Adapter Detail | Adapter lifecycle | `lifecycle: DEGRADED` | adapter degraded | 查看、暂停 | 强制提权 | capability_change | → READY/STOPPING/FAILED | 1 | DEGRADED 显著，可降级 |
| UI-ADAPT-FAIL | Adapter Detail | Adapter lifecycle | `lifecycle: FAILED` | adapter failed | 查看、请求重建 | 同实例回 READY | 失败事件 + 新 session_id 要求 | 新 instance 从 LOADING | 1 | FAILED 终态，新 session 重建 |
| UI-WF-PEND | Workflow Detail | Workflow projection | `status: PENDING` | pending | 启动、取消 | 标完成 | 目标固化事件 | → RUNNING | 1 | PENDING 不显示成功 |
| UI-WF-RUN | Workflow Detail | Workflow projection | `status: RUNNING` | running | 暂停/取消/恢复 | 标完成 | 心跳 + policy decision | → 终态/STALLED | 1 | RUNNING 非完成 |
| UI-WF-STALL | Workflow Detail | Workflow projection | `status: STALLED` | stalled | 查看、恢复 | 直接重发 | 最后心跳时间 | → RUNNING | 1 | STALLED 显著，可恢复 |
| UI-WORKER-DONE | Workflow Detail | Event Ledger | `result` 事件 | verification pending | 查看证据 | **标完成** | evidence_id（待 verifier） | evidence verified | 1 | worker 自述不显完成 |
| UI-EVID-VER | Evidence Detail | Artifact + Event | `evidence` 事件 + 校验 | verified | 查看、请求 reverify | 标终态完成 | evidence_id + sha256 + policy_decision | release gate passed | 1 | verified 非终态完成 |
| UI-FIND-HIGH | Finding Detail | Finding projection | `review.finding` HIGH | finding HIGH | 查看、reverify | 隐藏严重度 | finding_id + fingerprint | 排除或阻断 | 1 | HIGH 阻断晋级提示 |
| UI-FIND-CRIT | Finding Detail | Finding projection | `review.finding` CRITICAL | finding CRITICAL | 查看、reverify | 隐藏严重度 | finding_id + fingerprint | 阻断（不得排除） | 1 | CRITICAL 必阻断 |
| UI-REL-BLOCK | Workflow Detail | Policy Engine | release gate blocked | blocked | 查看、提交治理 | 标完成 | gate decision (policy_decision_id) | 证据排除风险 | 1 | release blocked 不显完成 |
| UI-RECON | Reconciliation | Reconciliation projection | `delivery.status: RECONCILIATION_REQUIRED` | reconciliation required | 查看、查询 | **盲重发** | reconciliation.result (external_state_known) | 确定性证据→VERIFIED/安全终态 | 1 | 0 次盲重试，进对账 |
| UI-BUD-WARN | Budget Meter | Budget projection | 预算≥阈值 | warning | 查看 | 绕过预算 | budget 计数 | 低于阈值/增额 | 1 | 警告色非 success |
| UI-BUD-EXH | Budget Meter | Budget projection | `budget_exhausted` | blocked (exhausted) | 查看、请求增额(治理) | 继续消耗 | budget 计数达上限 | 治理增额+新授权 | 1 | 耗尽进 BLOCKED_BUDGET_EXHAUSTED |
| UI-AUTH-EXP | Workflow Detail | Policy Engine | `BLOCKED_AUTH_EXPIRED` | blocked (auth) | 查看、重新认证(人) | 自动 A1–A5 | auth 失效事件 | 安全重认证+新令牌 | 1 | 过期进安全停止 |
| UI-POL-NEED | Workflow Detail | Policy Engine | `BLOCKED_NEEDS_POLICY_DECISION` | blocked (policy) | 查看、提交治理 | 自动续跑 | 默认决策阶梯记录 | 新政策/owner 决策 | 1 | 阶梯穷尽才阻断 |
| UI-QUAR | Workflow Detail | Policy Engine | `QUARANTINED_SECURITY_RISK` | quarantined | 查看(脱敏)、上报 | 解除隔离 | 安全证据 | owner 安全确认 | 1 | 隔离显著，不伪装 |
| UI-SAFE | Safe Mode Panel | Policy Engine | `POLICY_INTEGRITY_SAFE_MODE` | safe mode | 查看、导出诊断(A0) | **A1–A5/外部/模型** | 校验失败记录 | 可信政策重载校验 | 1 | safe mode 控件全禁 |
| UI-CANCEL-REQ | Workflow Detail | Event Ledger | `cancellation` 请求 | cancellation requested | 查看 | 标已取消 | adapter 最终状态待回报 | adapter 回报 | 1 | 请求非确认 |
| UI-CANCEL-CONF | Workflow Detail | Event Ledger | `CANCELLED_BY_POLICY` | cancelled | 查看 | 自动续跑 | 取消 policy decision | ——(终态) | 1 | 取消为终态 |
| UI-BACKUP-RUN | Backup Page | Backup Manager | backup trigger | backup running | 查看 | 标恢复成功 | backup set ID + 哈希 | 哈希一致 | 1 | 运行中不显成功 |
| UI-BACKUP-FAIL | Backup Page | Backup Manager | 哈希不一致 | error (danger) | 查看、重试 | 标成功 | 哈希不一致记录 | 重 backup | 1 | 失败不伪造成功 |
| UI-RESTORE-VER | Backup Page | Backup/Recovery | restore.drill | restore verification | 查看 | 标成功 | 恢复成功判据(哈希+重放+对账) | 判据全过 | 1 | 演练未过不显成功 |
| UI-TERM-APPROVED | Workflow Detail | Event Ledger | `COMPLETED_APPROVED` | completed | 查看 | —— | 终态 + evidence + policy_decision | ——(终态) | 1 | 仅终态+证据显完成 |
| UI-TERM-DEFERRED | Workflow Detail | Event Ledger | `COMPLETED_WITH_DEFERRED_LOW_RISK` | completed (deferred) | 查看 | 隐藏 deferred | 终态 + 候选严重度 + 未决原因 | ——(终态) | 1 | deferred 须显未决原因 |
| UI-EXT-FAIL | Workflow Detail | Event Ledger | `FAILED_EXTERNAL_DEPENDENCY` | blocked (external) | 查看、请求重审 | 盲重试 | 外部失败证据 | 依赖恢复+新授权 | 1 | 外部失败进安全停止 |

> 所有 UI State ID 均映射到 ARCHITECTURE.md §6（工作流终态）/ §7（Outbox）/ PROTOCOL.md §5、§7（adapter lifecycle）/ §11（finding/evidence）。**不新增任何 workflow 终态**。

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
