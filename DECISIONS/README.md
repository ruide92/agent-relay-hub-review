# DECISIONS — 架构决策记录（ADR）索引与规则

> 本目录收录 Agent Relay Hub（ARH）的架构决策记录（Architecture Decision Records, ADR）。
> 本目录包含 ADR 规则、索引及具体 ADR；具体 ADR 在获得有效 `Approval Record` 前只能是 `PROPOSED`，ADR 不得因进入索引就被视为已批准。
> 依据与优先级见 `SOURCE_OF_TRUTH.md`。

---

## 1. ADR 编号格式

- 格式：`ADR-NNNN-<短横线标题>.md`，`NNNN` 为四位零填充递增序号（如 `ADR-0001-event-ledger-as-source-of-truth.md`）。
- 编号一经分配不复用；被否决或废弃的 ADR 保留其编号与文件。

## 2. 状态

每份 ADR 必须标注状态之一：

| 状态 | 含义 |
|---|---|
| `PROPOSED` | 已提出，待审核 |
| `ACCEPTED` | 已批准并生效 |
| `SUPERSEDED` | 已被后续 ADR 取代（须注明取代它的 ADR 编号） |
| `REJECTED` | 已否决（保留记录，不得静默删除） |

## 3. 每份 ADR 必须记录的内容

1. **背景（Context）**：问题、约束与触发原因。
2. **决策（Decision）**：明确采纳的方案。
3. **替代方案（Alternatives）**：考虑过但未采纳的方案及原因。
4. **后果（Consequences）**：正负面影响、风险与后续义务。
5. **证据（Evidence）**：支撑决策的验证、探针结果或引用。
6. **批准 commit（Approval commit）**：批准该 ADR 时的 commit SHA。
7. **Decision Owner（决策所有者）**：对应治理角色 `owner` 或其有效委派；对决策负最终责任。
8. **Reviewer（独立审核者）**：提出通过/否决意见的角色；不得为 proposer/executor 本人。
9. **Governance Approver（治理批准者）**：`owner` 或有效委派的 `governance_approver`；其批准使决策生效。
10. **Approval Record（批准记录）**：仓库内可追溯的批准记录引用（如 `REVIEWER_2_FINAL_APPROVAL.md` 或本仓库批准记录）。
11. **Effective Commit（生效 commit）**：该决策实际生效的 commit SHA。
12. **Affected Normative Documents（受影响的规范性文档）**：列出被本 ADR 影响的 A 类文档（见 `SOURCE_OF_TRUTH.md` 第 4 节）。
13. **Supersedes / Superseded By（取代 / 被取代）**：彼此引用的 ADR 编号。

**状态规则**：没有有效 `Approval Record` 时，ADR 状态只能为 `PROPOSED`，不得标记 `ACCEPTED`（见 `SOURCE_OF_TRUTH.md` 第 10 节）。

## 4. 与已批准 V1.1 的关系

- **已批准 V1.1 不能被普通 ADR 静默覆盖。** 任何与 V1.1 正文冲突的决策必须显式声明冲突点，并走 Source of Truth 变更审批流程（`SOURCE_OF_TRUTH.md` 第 9 节），不得以 ADR 形式绕过。
- ADR 不得复活已被冻结 changeset 或 Reviewer-2 否决的设计。

## 5. 需独立审核的 ADR

以下类别的 ADR **必须独立审核**（不得随普通改动合入）：

- 权限（authorization / capability token / 三层授权）
- 安全（凭据、隔离、安全模式、威胁模型）
- 协议（SDK 契约、消息/事件协议、投递语义）
- 阶段边界（Phase 门禁、进入/退出条件）
- 许可（许可政策、第三方组件、分发模式）

## 6. ADR 索引

| ADR | 标题 | 状态 | 批准 commit |
|-----|------|------|-------------|
| ADR-0001 | 治理批准真实性 | PROPOSED | — |
| ADR-0002 | 视觉设计基线作为 Phase 0 退出门禁 | PROPOSED | — |

---

> 新增 ADR 时：在本目录创建 `ADR-NNNN-*.md`，并在上表登记编号、标题、状态与批准 commit。

---

## 7. 治理角色与批准约束

治理角色（`owner` / `governance_approver` / `reviewer` / `executor`）与批准规则（精确目标 commit、独立 reviewer 结论、owner 或有效委派 governance_approver 批准、仓库内可追溯批准记录、无批准记录则状态只能为 `PROPOSED`）的权威定义见 `SOURCE_OF_TRUTH.md` 第 10 节。

约束：

- **AI reviewer、worker、messenger 或 coordinator 不得自行批准自己提出或实施的变更。**
- 权限、安全、协议、阶段边界、许可类 ADR（见第 5 节）除满足第 3 节必填字段外，还须满足 `SOURCE_OF_TRUTH.md` 第 10.2 节的批准四要素，并走独立审核。
