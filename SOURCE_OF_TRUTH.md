# SOURCE OF TRUTH（权威来源定义）

> 本文件定义 Agent Relay Hub（ARH）项目"什么是事实、以哪个文件为准、冲突如何裁决"。
> 本文件本身属于**规范性文档**，修改必须治理提交（governance-only commit）并经审批（见第 7、9、10 节）。

---

## 1. 当前批准的产品设计基线

**`AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md` 是当前批准的产品设计基线（Source of Truth）。**

- 批准 commit（最终批准基线）：`c9cc522b4ab01008fed390c282d6bd5a816ee779`
- 治理晋级 commit（V1.1 标记为 SoT）：`f24993148a5754e12502eb38dbf566dcbb54c2c1`
- V1.0 基线 blob：`31b7287764e4eaf008b0f87852596e33bcfb4af3`
- 冻结 changeset commit：`3e9a0598aba2e8963c8a7ea8ac9b3dbfe271ff76`（对应 `V1.1_CHANGESET_FINAL.md`）
- Reviewer-2 最终一致性审核：**通过**（审核对象 commit `c9cc522…`）

---

## 2. 文档优先级与裁决顺序

当**规范性文档（第 4 节 A 类）**之间出现内容冲突时，按以下顺序裁决：

1. `AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md`（已批准 V1.1，最高权威）
2. 其他 A 类规范性文档（一致性派生或细化文档）须与 V1.1 一致；冲突以 V1.1 为准。

解释性批准记录（第 5 节 B 类）与历史审查记录（第 6 节 C 类）**不参与上述优先级链**：

- B 类文件用于解释"为何批准 V1.1"，在链中位于 V1.1 之下，其作用为解释批准依据，而非提供高于 V1.1 的当前约束力，也**绝非"仅历史、无当前作用"的废弃文件**。
- C 类文件仅供追溯，不得作为当前设计依据。

> 原 V1.0 审查流程曾以"已批准 V1.1 > 冻结 changeset > Reviewer-2 裁决 > 历史 Reviewer-1 文件"描述来源优先级；本办法将其归一为：V1.1 为规范性 apex，changeset 与 Reviewer-2 文件归入 B 类（解释性批准记录），不再同时具有"优先级源"与"无作用历史"的双重矛盾描述。

---

## 3. 文档冲突处理

- 任何文档与已批准 V1.1 冲突时，**以 V1.1 为准**，冲突文档必须修正或标注为过期。
- 派生/入口文档（如 `PRODUCT_PROPOSAL.md`）**不得**形成与 V1.1 独立的第二份产品正文；只能指针引用。
- 冲突若涉及权限、安全、协议、阶段边界或许可，必须经独立审核裁决（见第 7、9、10 节与 `DECISIONS/README.md`）。
- **禁止从聊天记录、模型记忆或 Reviewer-1 历史意见恢复已被否决的设计。** 已被冻结 changeset 或 Reviewer-2 否决的内容不得复活。

---

## 4. A. 规范性文档（Normative — 具当前约束力）

以下文件为当前具约束力的设计、治理、路线图、许可与契约文档：

- `AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md`（已批准 V1.1，最高权威）
- `V1.1_TRACEABILITY_MATRIX.md`
- `SOURCE_OF_TRUTH.md`（本文件）
- `PRODUCT_PROPOSAL.md`（稳定入口/指针）
- `ROADMAP.md`
- `LICENSE.md`
- `THIRD_PARTY_LICENSES.md`
- `SBOM_POLICY.md`
- `DECISIONS/README.md` 及经批准的 ADR

以下文件**已预先登记为后续规范性文档，创建并经审批后生效**（生效前不是 Source of Truth）：

- `ARCHITECTURE.md`（创建并批准后生效）
- `PROTOCOL.md`（创建并批准后生效）
- `SECURITY.md`（创建并批准后生效）
- `PHASE_0_EXIT_CRITERIA.md`（创建并批准后生效）

规范性文档约束规则：

- 上述预先登记的文档**必须服从已批准 V1.1**；只能细化实现契约，不得静默更改产品范围、权限、阶段边界或许可。
- `PHASE_1_IMPLEMENTATION_PLAN.md` 属实施计划，层级不高于上述规范性文档。
- 任何规范性文档的**未批准草稿都不是 Source of Truth**。
- `ROADMAP.md` 属 A 类规范性文档，但其描述的是 Phase 0–7 **未来意图**；其中 `ROADMAP_ONLY` 条目（P2×3）及"闭环""多审核""夜间无人值守""NAS 控制中心""商业化"等均为目标能力，**当前 Phase 0 未实现，不得描述为已实现**。

---

## 5. B. 解释性批准记录（Interpretive Approval Records）

以下文件用于解释"为何批准 V1.1"——修订原因、裁决与歧义澄清，**不构成当前产品正文，也不得覆盖已批准 V1.1**：

- `V1.1_CHANGESET_FINAL.md`（冻结 changeset，V1.1 的修订依据与冻结点）
- `REVIEWER_2_ADJUDICATION.md`（Reviewer-2 冻结验收裁决）
- `REVIEWER_2_FINAL_APPROVAL.md`（Reviewer-2 最终一致性审核通过记录）

约束：

- B 类文件**不得被普通修改**以改变已发生的批准事实；如需更正只能新增记录。
- B 类文件**不等于当前产品正文**；与 V1.1 冲突时以 V1.1 为准。
- B 类文件在优先级链中位于 V1.1 之下（见第 2 节），其作用是解释批准依据，而非提供高于 V1.1 的当前约束力。

---

## 6. C. 历史审查记录（Historical Review Records — 仅供追溯）

以下文件仅供追溯，**不构成当前设计依据**：

- `PROJECT_PROPOSAL_REVIEW.md`
- `V1.1_REQUIRED_CHANGES.md`
- `IMPLEMENTATION_READINESS_REPORT.md`
- `AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.0.md`（V1.0 基线，只读历史）
- 被取代的旧候选稿（如早期 V1.1 候选稿）与旧批准记录

> 历史文件**不得被修改**以改变已发生的审查事实；如需更正只能新增记录。

---

## 7. 规范性变更与治理提交规则

- 规范性文档（第 4 节 A 类）与 B 类批准记录的变更，**必须放入专门的 governance change set / 治理提交（governance-only commit）**，不得夹带产品代码、依赖安装或无关文档。
- 为保持原子一致性，**相互依赖的多份规范性文档可以在同一个治理提交中协调更新**（不要求每份文件各自独立 commit）。
- 涉及权限、安全、协议、阶段边界、许可的变更，仍须按第 9、10 节与 `DECISIONS/README.md` **独立审核**，不得由普通改动静默覆盖已批准 V1.1。
- 一个治理提交**必须列出**：受影响文件清单、依据来源、目标 commit、批准记录引用。

---

## 8. 当前阶段状态

- **Phase 0（产品设计冻结）：进行中，尚未关闭。**
- **Phase 1（Core Prototype）：尚未授权，禁止编写任何产品代码。**
- V1.1 已成为批准的产品设计基线；后续任务为补齐 Phase 0 Source of Truth 文档集，而非编码。

---

## 9. Source of Truth 变更审批流程

1. 任何对 V1.1 正文的实质性变更，必须以新的 changeset 提案发起，经独立一致性审核（对照 V1.0 与既有 V1.1），批准后方可形成新基线版本（如 V1.2），并更新本文件的批准 commit。
2. 规范性文档（第 4 节 A 类）的变更须治理提交（见第 7 节），并在提交信息中说明依据来源与优先级。
3. 涉及权限、安全、协议、阶段边界、许可的变更，必须走独立 ADR 审核（见 `DECISIONS/README.md`），不得由普通改动静默覆盖已批准 V1.1。
4. 变更审批的证据（审核 commit、裁决记录）须可在仓库中回溯。

---

## 10. 治理批准权（Governance Approval Authority）

### 10.1 治理角色

- **owner**：项目最终治理所有者，对 Source of Truth、权限、安全、协议、阶段边界与许可变更拥有最终批准权。
- **governance_approver**：由 owner 通过已记录的政策或批准记录明确委派的治理批准角色；其批准在委派范围内等效于 owner 批准。
- **reviewer**：负责独立审核并提出通过/否决意见，**但不能单独使决策生效**。
- **executor**：只能执行修改，**不能批准自己的修改**。

### 10.2 批准规则

- **AI reviewer、worker、messenger 或 coordinator 不得自行批准自己提出或实施的变更。**
- Source of Truth、权限、安全、协议、阶段边界和许可变更，必须同时具备：
  1. **精确目标 commit**（变更所针对的基线 commit）；
  2. **独立 reviewer 结论**（非 proposer/executor 本人）；
  3. **owner 或有效委派的 governance_approver 批准**；
  4. **仓库内可追溯的批准记录**（如 `REVIEWER_2_FINAL_APPROVAL.md`、ADR 的 Approval Record）。
- **owner 可提前委派治理批准角色**以避免每次人工介入，但委派本身必须明确：范围、期限、可撤销条件；委派记录须在仓库内可追溯。
- **没有有效批准记录时，状态只能是 `PROPOSED`，不得标记 `ACCEPTED`**（适用于 ADR 与任何决策状态）。

### 10.3 与 ADR 的衔接

ADR 的必填字段（含 Decision Owner / Reviewer / Governance Approver / Approval Record / Effective Commit / Affected Normative Documents / Supersedes-Superseded By）与批准约束见 `DECISIONS/README.md` 第 3、7 节。
