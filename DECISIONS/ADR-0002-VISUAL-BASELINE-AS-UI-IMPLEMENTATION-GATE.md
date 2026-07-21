# ADR-0002 — 视觉设计基线作为 Phase 1 UI 实现入口门禁（Visual Baseline as Phase 1 UI Implementation Gate）

Status: PROPOSED
ADR Number: ADR-0002
Title: 视觉设计基线作为 Phase 1 UI 实现入口门禁（第四包阶段边界治理）
Date: 2026-07-21
Proposed By: Phase 0 design lead / external product architect
Implemented By: executor（WorkBuddy）

> 本 ADR 当前为 `PROPOSED`；没有有效 `Approval Record` 前不得标记 `ACCEPTED`（见 `SOURCE_OF_TRUTH.md` 第 10 节与 `DECISIONS/README.md` 第 3 节）。

---

## 1. 背景（Context）

1. V1.1 与第三包（UX / UI / 交互契约）已经定义 Control Room、交互、状态语义与设计系统，但没有把"高保真视觉基线"明确列为任何阶段的强制门禁。
2. 视觉设计基线（精确视觉任务书、Control Room 高保真基准、视觉资产清单）是 Phase 1 UI 实现前的稳定输入；把它作为 Phase 1 UI 实现入口门禁（entry gate）属于阶段边界细化，必须经过独立 ADR 审核与 owner 批准，不能静默生效。
3. 视觉包**不是 Phase 0 关闭的前置条件**：Phase 0 关闭由其他未满足条件（机器可读契约、capability token 契约等）决定，不因视觉包未完成而阻塞。
4. 当前所有第四包条件（`P4-VISUAL-BRIEF` / `P4-CONTROL-ROOM-BASELINE` / `P4-VISUAL-ASSET-MANIFEST` / `P4-REVIEW` / `P4-OWNER`）均处于 `NOT_STARTED`，作为 Phase 1 UI 并行准备轨道登记，尚未开始。

## 2. 决策（Decision）

1. 将"视觉设计基线与视觉资产治理包"登记为 **Phase 1 UI 实现入口门禁（Phase 1 UI implementation entry gate）**，同时作为 Phase 1 的**并行准备轨道**（parallel preparation track），**不是 Phase 0 关闭前置条件**。
2. 第四包包含三项设计产物（design artifacts，非治理记录）：
   - `VISUAL_DESIGN_BRIEF.md`（精确的视觉设计任务书）；
   - Control Room 高保真基准资产（owner 认可的 Control Room 高保真基准）；
   - `VISUAL_ASSET_MANIFEST.md`（视觉资产清单：尺寸 / 文件 hash / 版本 / 契约映射）。
3. 第四包包含两项治理证据（governance evidence，独立审核与批准记录，非设计产物）：
   - `VISUAL_PACKAGE_REVIEW_0001.md`（独立审核记录）；
   - `GOVERNANCE_APPROVAL_0005.md`（owner 批准记录；注意：本编号专用于第四包视觉设计产物的 owner 批准，与用于批准本次治理一致性修订 / ADR-0001 / ADR-0002 的 `GOVERNANCE_APPROVAL_0004.md` 区分）。
4. 视觉设计必须由独立于 Reviewer-2 的 design lead（如 visual design lead / Phase 0 design lead）产出；Reviewer-2 只做独立审核；owner 只做最终批准。
5. 本 ADR 未获批准前，第四包只能是 proposed gate，不得声称已经生效。
6. 即使本 ADR 获批，视觉包仍作为 Phase 1 入口门禁与并行准备轨道；Phase 0 关闭由其他条件决定，Phase 1 授权仍须独立 `PHASE_1_AUTHORIZATION_*` 记录，且授权前须形成 `PHASE_1_IMPLEMENTATION_PLAN.md`（见 `ROADMAP.md`）。

## 3. 替代方案（Alternatives）

- **把视觉基线作为 Phase 0 关闭前置条件**：拒绝（本轮修订结论）。会造成 Phase 0 因视觉包而阻塞，与"Phase 0 关闭由机器可读契约等条件决定"不一致；且视觉基线更宜作为 Phase 1 UI 实现的稳定输入。
- **只把视觉方案作为非阻塞参考**：拒绝。无法形成可审核、可追溯的实现基准，Phase 1 UI 实现易漂移。

## 4. 后果（Consequences）

- **正面**：把视觉设计基线确立为可审核、可追溯的 Phase 1 UI 实现入口门禁，避免 Phase 1 UI 实现阶段返工与基准漂移，同时不阻塞 Phase 0 关闭。
- **负面 / 风险**：增加 Phase 1 UI 实现前置条件，依赖独立 design lead 资源。
- **后续义务**：第四包三项设计产物与两项治理证据须在 Phase 1 授权前/准备阶段真实完成，并经独立审核与 owner 批准（预期 `GOVERNANCE_APPROVAL_0005.md`）。

## 5. 证据（Evidence）

- V1.1 全文（已批准 SoT）与第三包（UX / UI / 交互契约）。
- `ROADMAP.md`（第四包登记为 Phase 1 UI 入口门禁 / 并行准备轨道）。
- `PHASE_0_EXIT_CRITERIA.md` 第 4、5 节（第四包状态与主矩阵 `P4-*` 作为 Phase 1 并行准备轨道）。
- 本 ADR 提出前尚未通过独立审核与 owner 批准，故无生效证据。

## 6. 批准字段（待审批后填写）

- **批准 commit（Approval commit）**：—
- **Decision Owner**：owner
- **Reviewer（独立审核者）**：待独立审核（不得填写当前设计负责人）
- **Governance Approver（治理批准者）**：owner（待批准）
- **Approval Record**：—（本 ADR 与本次治理一致性修订未来由 `GOVERNANCE_APPROVAL_0004.md` 批准；第四包视觉设计产物另由 `GOVERNANCE_APPROVAL_0005.md` 批准）
- **Effective Commit（生效 commit）**：—
- **Affected Normative Documents**：`ROADMAP.md`、`PHASE_0_EXIT_CRITERIA.md`
- **Supersedes / Superseded By**：无 / 无

## 7. 状态规则声明

没有有效 `Approval Record` 时，本 ADR 状态只能为 `PROPOSED`，不得标记 `ACCEPTED`（见 `SOURCE_OF_TRUTH.md` 第 10 节与 `DECISIONS/README.md` 第 3 节）。本 ADR 不得修改 V1.1 正文，不得声称已生效。
