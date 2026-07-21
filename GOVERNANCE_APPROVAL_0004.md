# GOVERNANCE_APPROVAL_0004 — Phase 0 治理一致性修订批准

> 本文件为 **append-only** 批准记录。创建后不得修改、不得重写历史。任何更正或补充均须以新增章节或新批准记录（如 `GOVERNANCE_APPROVAL_0005`）形式进行。

## 1. 批准标识

| 字段 | 值 |
|---|---|
| **Approval ID** | `GOV-APP-0004` |
| **Decision Owner** | owner |
| **Owner Decision** | `APPROVED` |
| **Target Commit** | `fd2574c28843ee2885460cb191c735dd0d257d1a` |
| **Base Commit** | `3657518c19ca0858e12b1d03509a901fb316f13c` |
| **Proposal PR** | #1 — `[PROPOSAL] Phase 0 governance consistency repair #0004` |
| **Reviewer** | Reviewer-2（产品身份 WorkBuddy，模型版本 `UNKNOWN`） |
| **Reviewer Decision** | `PASS` |
| **review_run_id** | `29fe1f22-6b59-4f69-9c5a-82a6fa8deceb` |
| **正式审核产物（PR 评论）** | https://github.com/ruide92/agent-relay-hub-review/pull/1#issuecomment-5035425502 |
| **GitHub comment ID** | `5035425502` |

- **Independent review context identifier**：平台原生 session / task ID 不可得；已按 `DECISIONS/README.md` 第 8 节 fallback 规则提供完整 review context evidence（含 review_run_id、UTC ISO-8601 开始时间、独立干净工作区标识、repository URL、base/target SHA、reviewer/product/model identity、未参与声明、直接读取仓库/git 对象声明、完整审核产物与明确 verdict、PR 评论 URL）。该 fallback 仅解决可追溯性，**未**使执行者或原执行会话获得独立 Reviewer 资格。

## 2. Owner 书面批准原文

> 我以项目 owner 身份，批准治理提案最终审核目标 commit `fd2574c28843ee2885460cb191c735dd0d257d1a`，认可 Reviewer-2 在 PR #1 评论 `5035425502` 中给出的独立审核 PASS。
>
> 授权按 `GOVERNANCE_APPROVAL_0004` 流程进行治理晋级。
>
> 本批准不关闭 Phase 0，不授权 Phase 1，不授权编写产品代码、创建 Schema、安装依赖、生成图片或启动视觉设计，也不批准尚未形成的 capability-token signing contract 与 machine-readable contracts。

## 3. 批准范围

本批准针对 Target Commit `fd2574c…` 中的完整治理一致性修订，即 PR #1 合并范围内的 **11 个变更文件**，以及 `ADR-0001`（治理批准真实性）与 `ADR-0002`（视觉设计基线作为 Phase 1 UI 实现入口门禁）两项决策与跨文档治理一致性修订：

1. `README.md`
2. `SOURCE_OF_TRUTH.md`
3. `PRODUCT_PROPOSAL.md`
4. `V1.1_TRACEABILITY_MATRIX.md`
5. `SBOM_POLICY.md`
6. `SECURITY.md`
7. `DECISIONS/README.md`
8. `DECISIONS/ADR-0001-GOVERNANCE-APPROVAL-AUTHENTICITY.md`
9. `PHASE_0_EXIT_CRITERIA.md`
10. `ROADMAP.md`
11. `DECISIONS/ADR-0002-VISUAL-BASELINE-AS-UI-IMPLEMENTATION-GATE.md`（由 `ADR-0002-VISUAL-BASELINE-AS-PHASE-0-GATE.md` 重命名）

本批准不修改 `V1.1` / `V1.0` 正文，不修改 `GOV-APP-0001` / `GOV-APP-0002` / `GOV-APP-0003`。

## 4. 与 GOV-APP-0001 / 0002 / 0003 的关系

- 本记录**不撤销、不取代** `GOVERNANCE_APPROVAL_0001.md`（第一包治理与权威）、`GOVERNANCE_APPROVAL_0002.md`（第二包实施契约/安全/退出门禁，含 `SECURITY.md` §8 签名基线）、`GOVERNANCE_APPROVAL_0003.md`（第三包 UX/UI）。
- 本记录为在上述三包已批准基础上，对 Phase 0 治理一致性修订与 ADR-0001/ADR-0002 的决策批准。
- **Supersedes**: `None`

## 5. 批准边界（明确不授权范围）

本批准**仅**批准上述 Target Commit 中的治理文档修订与 ADR 决策；明确**不包含**以下任何一项：

- **Phase 0** 仍 `OPEN / NOT_READY`；
- **Phase 1** 仍 `NOT AUTHORIZED`；
- **`NO PRODUCT CODE`**（不授权编写任何产品代码）；
- 不批准 **`GOVERNANCE_APPROVAL_0005`**（第四包视觉产物批准保留给未来独立流程）；
- 不批准或启动**第四包视觉设计产物**（视觉资产、基准、清单、视觉包审核均不启动）；
- 不批准 **`capability-token signing contract`**（该契约仍为 `NOT_STARTED`，待 Phase 1 实现与验证）；
- 不批准 **`machine-readable contracts`**（ARP envelope / message types、adapter capability/lifecycle、health_check、policy bundle、signature manifest、capability token Schema 均为 `NOT_STARTED`）；
- 不创建 **`closure`** 或 **`authorization`** 记录；
- 不授权代码、Schema、依赖、CI、图片或任何实现工作。

## 6. 生效条件（Effective Condition）

本批准记录在其 **Commit A（本文件）、Commit B（机械状态晋级）、Commit C（精确 SHA 回填）** 全部经晋级 PR（`governance/gov-app-0004-promotion`）合并，并可从远端 `main` 到达之后方为**生效**；在此之前，本文件仅为**待生效批准记录（pending）**，相关 ADR 状态不得视为最终生效。

## 7. Append-only 声明

本文件为 append-only 批准记录。任何后续更正、补充或状态变更，均须以新增章节或新批准记录（如 `GOVERNANCE_APPROVAL_0005` 等）形式进行，不得修改、删除或重写本文件既有内容。
