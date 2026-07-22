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

# UI_INFORMATION_ARCHITECTURE.md（信息架构）

> 本文档定义 ARH 的导航、页面、信息层级与低保真结构。
> 本文档已通过 Reviewer-2 对 target commit `b74e4076ce59e567de52d67916133a5d9ca26596` 的独立审核；
> 已由 `GOVERNANCE_APPROVAL_0003.md` 批准；
> 当前为 Phase 0 Source of Truth；
> 批准只表示设计契约生效；
> 不表示 UI、代码、组件、skill、依赖或验收已经实现；
> Phase 0 当前为 OPEN / READY_FOR_CLOSURE（尚未关闭）；
> Phase 1 仍为 NOT AUTHORIZED。
> 所有内容必须服从已批准基线；UI 权威数据仅来自 Event Ledger 派生 projection（ARCHITECTURE.md §2、§4）。
> 不得生成图片或外部设计链接；本文件仅用 Markdown ASCII / Mermaid 表达结构。

---

## 1. 一级导航

固定候选一级导航（Phase 1 只实现 mock 所需部分；Phase 3/4 页面须标注"未来阶段"，不得宣称 Phase 1 实现）：

1. **Control Room**（首页）
2. **Workflows**
3. **Evidence & Findings**
4. **Reconciliation**
5. **Adapters**
6. **Policies & Authorization**
7. **Budgets & Limits**
8. **Audit & Recovery**
9. **Settings**

> Phase 1 实际范围以 ARCHITECTURE.md §1.1 / §1.2 为界；仅 mock 所需页面进入 Phase 1 实现，其余标注未来阶段。

---

## 2. Control Room（首页）

首页 MUST 首先展示（信息优先级见 §5）：

- 全局健康（Global Health Banner）；
- safe mode 状态（若进入 `POLICY_INTEGRITY_SAFE_MODE` 显著提示）；
- 运行中工作流（RUNNING / STALLED）；
- waiting verification（evidence pending 数）；
- reconciliation（RECONCILIATION_REQUIRED 数）；
- blocked / quarantined（BLOCKED_* / QUARANTINED_*）；
- adapter health（READY/BUSY/DEGRADED/FAILED）；
- budget（已用/上限）；
- 最近终态；
- 昨夜/最近自动完成摘要（Summary 级）。

**首页不得以大聊天框作为主入口**（UX_INTERACTION.md §1）。

主要操作限制为：

- 创建模拟工作流（UX-CREATE-SIM）；
- 查看异常（跳转 Evidence / Reconciliation / Adapters）；
- 查看结果（Workflow Detail）；
- 查看证据（Evidence Detail）。

---

## 3. 页面目录

每个页面 MUST 定义如下字段。Phase 1 仅实现标注 Phase 1 的页面。

| Page ID | Route | Phase | Primary Purpose | Projection Source | Primary Actions | Empty State | Loading State | Degraded State | Permission Boundary |
|---|---|---|---|---|---|---|---|---|---|
| PAGE-CTRL | `/control-room` | 1 | 全局态势总览 | Workflow/Adapter/Budget projection | 创建模拟、查看异常/结果/证据 | 无运行工作流提示 | Skeleton | 核心不可用时 Global Health Banner 降级 | A0 可见 |
| PAGE-WF-LIST | `/workflows` | 1 | 工作流列表与筛选 | Workflow projection | 创建、打开详情、取消 | 空列表提示 | Skeleton | 投影陈旧提示 | A0 可见 |
| PAGE-WF-DETAIL | `/workflows/:id` | 1 | 证据化时间线详情 | Workflow + Evidence + Finding projection | 取消/恢复(仅 STALLED 后)/查看证据 | 无节点提示 | Skeleton | stale projection 提示 | A0 可见；命令受 A 档限制 |
| PAGE-EVID | `/evidence/:id` | 1 | 证据详情 | Evidence + Artifact projection | 查看(A0)、请求重新验证(A1) | 证据缺失提示 | Skeleton | 校验失败提示 | A0 查看证据；A1 请求重新验证；不涉及 A5 |
| PAGE-FIND | `/findings/:id` | 1 | finding 详情 | Finding projection | 查看、请求 reverify | 无 finding 提示 | Skeleton | —— | A0/A1（reviewer） |
| PAGE-RECON | `/reconciliation/:id` | 1 | 对账案例 | Reconciliation projection | 查看、查询 | 无对账提示 | Skeleton | —— | A0 可见 |
| PAGE-ADAPT | `/adapters/:id` | 1 | 适配器详情 | Adapter lifecycle projection | 查看健康、查看能力；FAILED 时可请求重建 mock adapter | 无 adapter 提示 | Skeleton | adapter FAILED 显著 | 查看 A0；重建 mock adapter A2（仅 Phase 1 mock，不得加载真实 adapter，不得同 session FAILED→READY） |
| PAGE-POLICY | `/policies` | 1 | 政策查看 | Policy projection（签名引用） | 查看政策版本 | 无政策提示 | Skeleton | safe mode 提示 | A0 可见 |
| PAGE-AUTH | `/authorization/request` | 1/3 | A5 授权请求（Phase 1 仅 mock 流程） | Policy decision projection | 提交 A5 请求（mock，无真实外部副作用） | 无请求提示 | Skeleton | —— | A1 可提交本地授权申请；提交申请本身不授予 A5；目标 A5 操作仍默认禁止，须 owner 批准且由 Policy Engine 签发有效 capability token；Phase 1 仅 mock，无真实外部副作用 |
| PAGE-BUDGET | `/budgets` | 1 | 预算详情 | Budget projection | 查看 | 无预算提示 | Skeleton | 耗尽提示 | A0 可见 |
| PAGE-AUDIT | `/audit` | 1 | 审计浏览 | Audit hash-chain projection | 查询、导出脱敏 | 无记录提示 | Skeleton | —— | A0 可见 |
| PAGE-BACKUP | `/audit/backup` | 1 | 备份与恢复演练（Phase 1 本地 mock） | Backup/Recovery projection | 触发备份/演练 | 无备份提示 | Skeleton | 哈希不一致提示 | A0 查看；A2 触发备份/恢复演练；A4 保留给未来真实非生产部署 |
| PAGE-SAFE | `/safe-mode` | 1 | 安全模式面板 | Safe Mode projection（派生自 Event Ledger policy.decision / audit events） | 查看、导出脱敏诊断、重新加载外部提供的已批准 policy bundle | 未进入提示 | Skeleton | —— | 查看/导出本地 A0；Reload 为 safe-mode 本地验证路径，不授予 A1–A5，不编辑/生成/签署/修复 policy，不创建 trust root |
| PAGE-SETTINGS | `/settings` | 1 | 设置 | 本地配置 projection | 查看（改配置须治理） | 默认提示 | Skeleton | —— | A0 可见 |

### 3.1 Action Traceability（操作追溯）

每个可见操作 MUST 追溯到 Interaction ID；只读/导航链接标记为 NAV，不要求 Core Command。

| Page/Component | Visible Action | Interaction ID | Core Command | Required Permission | Phase |
|---|---|---|---|---|---|
| 一级导航 | 各导航项跳转 | NAV | （无） | A0 | 1 |
| Workflow List | [Cancel] | UX-CANCEL | `workflow.cancel_request` | A1 | 1 |
| Workflow List | [Create Simulation Workflow] | UX-CREATE-SIM | `workflow.create_simulation` | A2 | 1 |
| Control Room | [Create Simulation Workflow] | UX-CREATE-SIM | `workflow.create_simulation` | A2 | 1 |
| Control Room | [View Anomalies] / [View Evidence] | NAV | （无） | A0 | 1 |
| Workflow Detail | [Cancel] | UX-CANCEL | `workflow.cancel_request` | A1 | 1 |
| Workflow Detail | [View Evidence] | UX-VIEW-EVIDENCE | `evidence.query` | A0 | 1 |
| Workflow Detail | [Resume]（仅 STALLED 后） | UX-RESUME | `workflow.resume` | A1 | 1 |
| Evidence Detail | [Request Reverify] | UX-REQUEST-REVERIFY | `evidence.reverify_request` | A1 | 1 |
| Findings Detail | [Request Reverify] | UX-REQUEST-REVERIFY | `evidence.reverify_request` | A1 | 1 |
| Reconciliation | [Query Status] | UX-VIEW-RECON | `reconciliation.query` | A0 | 1 |
| Reconciliation | [View Result] | NAV | （无） | A0 | 1 |
| Adapter Detail | [View Health] | NAV | （无） | A0 | 1 |
| Adapter Detail | [Recreate Mock Adapter] | UX-RECREATE-MOCK-ADAPTER | `adapter.recreate_mock_instance` | A2 | 1 |
| Policy & Authorization | [Submit A5 Request] | UX-REQUEST-A5 | `auth.a5_request` | A1 提交申请；owner 批准后目标操作才可能取得 A5 | 1/3 |
| Policy Viewer | [View Policy] | UX-VIEW-POLICY | `policy.query` | A0 | 1 |
| Budget | [View Budget] | UX-VIEW-BUDGET | `budget.query` | A0 | 1 |
| Audit & Recovery | [Export Diagnostic] | UX-EXPORT-DIAG | `diagnostic.export` | A0 | 1 |
| Audit & Recovery | [Trigger Backup] | UX-TRIGGER-BACKUP | `backup.trigger` | A2 | 1 |
| Audit & Recovery | [Trigger Restore Drill] | UX-TRIGGER-RESTORE-DRILL | `restore.drill` | A2 | 1 |
| Safe Mode | [Export Diagnostic] | UX-EXPORT-DIAG | `diagnostic.export` | A0 | 1 |
| Safe Mode | [Reload Approved Policy] | UX-RELOAD-APPROVED-POLICY | `policy.reload_approved_bundle` | 本地 safe-mode 恢复（不提权） | 1 |

---

## 4. 页面低保真结构（ASCII / Mermaid）

仅用文本结构，不生成图片。

### 4.1 Control Room

```text
+----------------------------------------------------------+
| Global Health Banner [OK | DEGRADED | SAFE MODE]         |
+----------------------------------------------------------+
| Running: 3   Waiting Verif: 2   Reconciliation: 1         |
| Blocked: 0   Quarantined: 0   Budget: 64%                |
| Adapters: 4 Ready / 1 Degraded                           |
+----------------------------------------------------------+
| Recent Terminals (last 5)                                |
|  WF-102  COMPLETED_APPROVED       2026-07-20 23:10Z      |
|  WF-101  BLOCKED_BUDGET_EXHAUSTED 2026-07-20 22:48Z      |
+----------------------------------------------------------+
| [Create Simulation Workflow]  [View Anomalies]           |
+----------------------------------------------------------+
```

### 4.2 Workflow Detail（证据化时间线）

```text
WF-102  RUNNING
┌──────────────────────────────────────────────────────┐
| 目标固化 ✓  权限预检 ✓  规划 ✓  执行 ✓  测试 ✓         |
| 审核 ◐  证据验证 ○  修复 ○  复审 ○  发布候选 ○  终态 ○  |
└──────────────────────────────────────────────────────┘
Node: 执行
  actor: worker.primary (adapter mock-01 / session s-9)
  start: 2026-07-20T22:00Z  end: 22:40Z
  artifact: artifact-77 (sha256:…)  attempt: 2
  policy_decision: pd-33  budget: 12/50
  evidence: pending  finding: none
[Cancel] [View Evidence]  (collapsed: message_id/job_id/hashes)
```

### 4.3 Evidence & Findings

```text
Evidence Detail
  evidence_id: ev-12
  verification: PENDING → VERIFIED (verifier output)
  artifact: artifact-77 (sha256:…)
  policy_decision: pd-34
Findings (related)
  finding-05  HIGH  fingerprint:f3a1  status: OPEN
  [Request Reverify]
```

### 4.4 Reconciliation Center

```text
Reconciliation Case RC-03
  delivery_attempt: da-9  idempotency_key: ik-09
  state: RECONCILIATION_REQUIRED
  external_state_known: false
  auto blind retry: 0
  [Query Status] [View Result]
```

### 4.5 Adapter Detail

```text
Adapter: mock-01
  lifecycle: READY  session: s-9
  capabilities: send, receive, supports_idempotency_key, …
  last_health: 2026-07-20T23:00Z
[View Health]  [Recreate Mock Adapter]  (FAILED 时须新 session 重建，UX-RECREATE-MOCK-ADAPTER, A2, 仅 mock)
```

### 4.6 Policy & Authorization

```text
Policy Viewer
  version: policy-2.1  key_id: k-policy-01
  signed: true  status: LOADED
Authorization Request (A5)
  scope: merge protected branch X
  status: WAITING_OWNER
[Submit A5 Request]
```

### 4.7 Audit & Recovery

```text
Audit Explorer
  hash-chain verified: true
  recent events: …
Backup & Restore
  last backup: bs-12 (hash ok)
  [Trigger Backup] [Trigger Restore Drill]
```

### 4.8 Safe Mode

```text
Safe Mode Panel
  state: POLICY_INTEGRITY_SAFE_MODE
  allowed: view / export diagnostic (A0 only) / reload approved policy (local read-only validation, no privilege escalation)
  blocked: A1–A5 / external / model / edit or sign policy / auto new trust root
[Export Diagnostic] [Reload Approved Policy]
  (reload: select owner-controlled trusted governance tool's provided approved bundle;
   verify hash + governance approval ref + signature + key_id + key purpose + trust-root;
   on failure stay in safe mode)
```

---

## 5. 信息优先级（首页与详情通用）

固定顺序（不得让模型对话文本压过安全状态与证据）：

1. 安全状态（safe mode / quarantine / blocked）；
2. 当前终态 / 阻塞；
3. 是否已经验证（evidence verified?）；
4. 当前自动动作；
5. 需要用户介入的原因；
6. 成本与时间（budget / duration）；
7. 详细日志（折叠）。

---

## 6. 详情展开策略

默认展示（人能理解）：

- 摘要；
- 当前状态；
- 原因；
- 证据状态；
- 下一自动动作。

高级详情**折叠**展示（UI_STATE_ACCEPTANCE_MATRIX.md 引用）：

- message_id；
- workflow_id；
- job_id；
- policy_decision_id；
- adapter / session ID；
- hashes（content_hash / artifact sha256）；
- raw schema；
- audit chain。
