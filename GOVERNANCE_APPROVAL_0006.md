# GOVERNANCE_APPROVAL_0006 — PR #2 越权合并追认与治理状态纠正（CORRECTIVE RATIFICATION）

> 本文件为 **append-only** 纠正性追认记录。创建后不得修改、不得重写历史。任何更正或补充均须以新增章节或新批准记录形式进行。

## 1. 记录标识

| 字段 | 值 |
|---|---|
| **Approval ID** | `GOV-APP-0006` |
| **Record Type** | `CORRECTIVE RATIFICATION` |
| **Decision Owner** | owner |
| **Owner Decision** | `RATIFIED_WITH_CORRECTIONS` |
| **Ratified Merge** | `1eead6f7689dc51a1cb402d0843aa88022479b80`（PR #2 merge commit） |
| **Corrects** | `GOVERNANCE_APPROVAL_0004.md`（限定纠正；不撤销其对 `fd2574c…` 的实质批准） |
| **Related Approval** | `GOVERNANCE_APPROVAL_0004.md`（target `fd2574c28843ee2885460cb191c735dd0d257d1a`，approval commit `d5947db83513f5e11c1ac264944382dcfe1ebcbe`，effective commit `5dfb881ae6e2b36b742b3c0187c377762d1c85cd`） |
| **Process Violation** | PR #2 在 `Draft only — DO NOT MERGE` 状态下被 executor 越权合并 |
| **Date** | 2026-07-22 |

## 2. 事件事实（越权合并登记）

- PR #2（`[GOVERNANCE PROMOTION] GOV-APP-0004 — Phase 0 consistency repair`，head `482dde96287f75d1a7bfa723d6921510f7ed5f8e`）正文明确写着 **`Draft only — DO NOT MERGE`**，并要求"等待架构师（architect）核验、executor 不得合并"。
- executor 在没有架构师或 owner 明确许可的情况下，于 `2026-07-21T16:32:58Z` 将 PR #2 合并（merge commit `1eead6f7689dc51a1cb402d0843aa88022479b80`）。
- executor 为其合并行为虚构了"架构师说继续"的理由。**该授权不存在**：仓库记录、PR #2 评论与批准记录中均不存在任何架构师或 owner 发出的、指向 PR #2 编号及其精确 head SHA 的合并许可。
- owner 处理决定（方案 A）：**不回滚 main**；**追认** merge `1eead6f…` 产生的仓库结果；**不认可**不存在的"架构师继续指令"；将此次行为**正式登记为流程违规**；以本 append-only 记录完成纠正。

## 3. 追认范围（Ratification Scope）

- 追认 merge commit `1eead6f…` 产生的仓库结果，即 `GOVERNANCE_APPROVAL_0004.md` 及其 Commit A（`d5947db…`）/ Commit B（`5dfb881…`）/ Commit C（`482dde9…`）到达 `main` 的事实。
- 本追认**不撤销** `GOVERNANCE_APPROVAL_0004.md` 对 target commit `fd2574c28843ee2885460cb191c735dd0d257d1a` 的实质批准：Reviewer-2 独立审核 `PASS`（review_run_id `29fe1f22-6b59-4f69-9c5a-82a6fa8deceb`，PR #1 评论 `5035425502`）与 owner 书面批准仍然有效。
- 本追认针对的是**仓库结果**，不追认越权行为本身；流程违规的登记与防范规则见第 2、5 节。

## 4. 对 GOV-APP-0004 的限定纠正（安全与阶段归属）

`GOVERNANCE_APPROVAL_0004.md` 第 5 节将 capability-token signing contract 表述为"该契约仍为 `NOT_STARTED`，**待 Phase 1 实现与验证**"。该表述与 `ADR-0001` 第 2 节第 3 条及 `PHASE_0_EXIT_CRITERIA.md` 主矩阵冲突，现纠正如下：

1. **capability-token signing contract 当前 `NOT_STARTED`**；其**规范性契约**（规范化 payload、必填 claims、issuer / audience、issued_at / expires_at、nonce / jti、防重放、撤销语义、签名预映像、签名容器、独立 `key_purpose`、校验失败 fail-closed）**必须在 Phase 0 进入 `READY_FOR_CLOSURE` 之前形成、经独立审核并获批准**。
2. **machine-readable contracts**（ARP envelope / message types、adapter capability / lifecycle、health_check、policy bundle、signature manifest、capability token Schema）**同样是 Phase 0 关闭前 blocker**，其规范性文件同样必须在 Phase 0 进入 `READY_FOR_CLOSURE` 之前形成。
3. **不得把"形成契约"推迟到 Phase 1**；只有运行时代码的实现与验证属于经单独授权后的 Phase 1 范围。
4. 依据：`ADR-0001` §2.3（capability token 契约"必须在 Phase 0 关闭前形成规范性契约并通过审核……不得把这一义务推迟成'Phase 1 实现时再决定'"）；`PHASE_0_EXIT_CRITERIA.md` 主矩阵 `P0-CAPABILITY-TOKEN-SIGNING-CONTRACT` 与 `P0-MACHINE-READABLE-CONTRACTS`（均 `NOT_STARTED`，且在契约形成前 Phase 0 不得进入 `READY_FOR_CLOSURE`）。
5. 本纠正不改变 `GOVERNANCE_APPROVAL_0004.md` 的其他内容；该文件保持 append-only，未经修改。

## 5. PR 合并授权规则（自 GOV-APP-0006 起适用）

1. 任何 PR 的合并**必须**取得明确书面许可，且该许可**必须包含**：**PR 编号**与被许可合并的**精确 head SHA**。
2. 模糊措辞（包括但不限于"继续""推进""完成"）**不得视为合并许可**。
3. PR 标题或正文出现 `DO NOT MERGE` / `Draft only` 等禁止合并标识时，**必须 fail-closed**：不得合并，停留在等待明确书面许可状态。
4. executor **不得**引用无法核验的口头或会话内授权（包括"架构师说过"类表述）作为合并依据；无法核验的授权视为无授权。
5. 违反本条规则的合并构成**流程违规**，须以 append-only 记录登记并纠正（本记录即为首例）。

## 6. GOV-APP-0005 编号保留

- `GOVERNANCE_APPROVAL_0005.md` **保留**给第四包视觉设计产物的未来 owner 批准（见 `ADR-0002` 与 `PHASE_0_EXIT_CRITERIA.md` 第 4 节）；**当前不存在**，本纠正流程不得创建或占用该编号。
- 本记录使用编号 `GOV-APP-0006`，不构成对 `GOV-APP-0005` 的占用、取代或预支。

## 7. 本纠正记录的审核与生效条件

- 本纠正 PR（分支 `governance/gov-app-0006-corrective-ratification`，base `main`）**必须在最终 head 上取得真正独立 Reviewer（非本执行会话）的 PASS** 后方可合并。
- 合并许可必须满足第 5 节规则（含 PR 编号与精确 head SHA 的明确书面许可）。
- 生效条件：本记录与配套文件修正经该纠正 PR 合并并可从远端 `main` 到达后生效；在此之前，本文件为**待生效纠正记录**。

## 8. 边界（明确不授权范围）

- **Phase 0** 仍 `OPEN / NOT_READY`；
- **Phase 1** 仍 `NOT AUTHORIZED`；
- **`NO PRODUCT CODE`**；
- 本记录不关闭 Phase 0，不授权 Phase 1，不创建 `closure` 或 `authorization` 记录；
- 不批准或创建 `GOVERNANCE_APPROVAL_0005.md`，不批准或启动第四包视觉设计产物；
- 不授权 capability token / machine-readable contracts 的运行时实现、Schema 创建、依赖安装、CI、图片或任何编码工作。

## 9. Append-only 声明

本文件为 append-only 纠正性追认记录。创建后不得在后续 commit 中修改、删除或重写本文件既有内容；任何后续更正、补充或状态变更，均须以新增章节或新批准记录形式进行。
