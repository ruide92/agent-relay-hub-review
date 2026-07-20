# SOURCE OF TRUTH（权威来源定义）

> 本文件定义 Agent Relay Hub（ARH）项目"什么是事实、以哪个文件为准、冲突如何裁决"。
> 本文件本身属于**规范性文档**，修改必须独立 commit 并经审批（见第 9 节）。

---

## 1. 当前批准的产品设计基线

**`AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md` 是当前批准的产品设计基线（Source of Truth）。**

- 批准 commit（最终批准基线）：`c9cc522b4ab01008fed390c282d6bd5a816ee779`
- 治理晋级 commit（V1.1 标记为 SoT）：`f24993148a5754e12502eb38dbf566dcbb54c2c1`
- V1.0 基线 blob：`31b7287764e4eaf008b0f87852596e33bcfb4af3`
- 冻结 changeset commit：`3e9a0598aba2e8963c8a7ea8ac9b3dbfe271ff76`（对应 `V1.1_CHANGESET_FINAL.md`）
- Reviewer-2 最终一致性审核：**通过**（审核对象 commit `c9cc522…`）

---

## 2. 文档优先级

当出现任何内容差异时，按以下从高到低顺序裁决：

```
已批准 V1.1  >  冻结 changeset  >  Reviewer-2 裁决  >  历史 Reviewer-1 文件
```

具体文件优先级：

1. `AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md`（已批准 V1.1，最高权威）
2. `V1.1_CHANGESET_FINAL.md`（冻结 changeset）
3. `REVIEWER_2_FINAL_APPROVAL.md` / `REVIEWER_2_ADJUDICATION.md`（Reviewer-2 裁决）
4. `PROJECT_PROPOSAL_REVIEW.md` / `V1.1_REQUIRED_CHANGES.md` / `IMPLEMENTATION_READINESS_REPORT.md`（历史 Reviewer-1 文件，仅供追溯，不具当前约束力）

---

## 3. 文档冲突处理

- 任何文档与已批准 V1.1 冲突时，**以 V1.1 为准**，冲突文档必须修正或标注为过期。
- 派生/入口文档（如 `PRODUCT_PROPOSAL.md`）**不得**形成与 V1.1 独立的第二份产品正文；只能指针引用。
- 冲突若涉及权限、安全、协议、阶段边界或许可，必须经独立审核裁决（见第 9 节与 `DECISIONS/README.md`）。
- **禁止从聊天记录、模型记忆或 Reviewer-1 历史意见恢复已被否决的设计。** 已被冻结 changeset 或 Reviewer-2 否决的内容不得复活。

---

## 4. 规范性文档（Normative — 具当前约束力）

- `AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md`
- `V1.1_TRACEABILITY_MATRIX.md`
- `SOURCE_OF_TRUTH.md`（本文件）
- `PRODUCT_PROPOSAL.md`（稳定入口/指针）
- `ROADMAP.md`
- `LICENSE.md`
- `THIRD_PARTY_LICENSES.md`
- `SBOM_POLICY.md`
- `DECISIONS/README.md` 及经批准的 ADR

---

## 5. 历史审查记录（Historical — 仅供追溯，不构成当前设计）

- `PROJECT_PROPOSAL_REVIEW.md`
- `V1.1_REQUIRED_CHANGES.md`
- `IMPLEMENTATION_READINESS_REPORT.md`
- `REVIEWER_2_ADJUDICATION.md`
- `REVIEWER_2_FINAL_APPROVAL.md`
- `V1.1_CHANGESET_FINAL.md`（冻结记录，既是裁决依据也是历史冻结点）
- `AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.0.md`（V1.0 基线，只读历史）

> 历史文件**不得被修改**以改变已发生的审查事实；如需更正只能新增记录。

---

## 6. 路线图（Roadmap — 未来意图，非当前承诺）

- `ROADMAP.md` 描述 Phase 0–7 的目标与门禁，属**未来意图**。
- 可追溯矩阵中标记 `ROADMAP_ONLY` 的条目（P2×3）仅为后续路线图，**不得描述为已实现能力**。
- 任何"闭环""多审核""夜间无人值守""NAS 控制中心""商业化"均为目标能力，当前 Phase 0 未实现。

---

## 7. 必须独立 commit 的文件

以下文件修改时**必须单独 commit**（不得与其他改动混提），以保证审查可追溯：

- `AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md`（已冻结，仅治理/勘误级修改且需审批）
- `V1.1_TRACEABILITY_MATRIX.md`
- `SOURCE_OF_TRUTH.md`
- `LICENSE.md`
- `THIRD_PARTY_LICENSES.md`
- `SBOM_POLICY.md`
- 任何涉及权限、安全、协议、阶段边界、许可的 ADR

---

## 8. 当前阶段状态

- **Phase 0（产品设计冻结）：进行中，尚未关闭。**
- **Phase 1（Core Prototype）：尚未授权，禁止编写任何产品代码。**
- V1.1 已成为批准的产品设计基线；后续任务为补齐 Phase 0 Source of Truth 文档集，而非编码。

---

## 9. Source of Truth 变更审批流程

1. 任何对 V1.1 正文的实质性变更，必须以新的 changeset 提案发起，经独立一致性审核（对照 V1.0 与既有 V1.1），批准后方可形成新基线版本（如 V1.2），并更新本文件的批准 commit。
2. 规范性文档（第 4 节）的变更须独立 commit，并在提交信息中说明依据来源与优先级。
3. 涉及权限、安全、协议、阶段边界、许可的变更，必须走独立 ADR 审核（见 `DECISIONS/README.md`），不得由普通改动静默覆盖已批准 V1.1。
4. 变更审批的证据（审核 commit、裁决记录）须可在仓库中回溯。
