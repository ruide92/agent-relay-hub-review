# DECISIONS — 架构决策记录（ADR）索引与规则

> 本目录收录 Agent Relay Hub（ARH）的架构决策记录（Architecture Decision Records, ADR）。
> **本轮只创建 ADR 规则与索引，不创建任何具体 ADR。**
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
| _(暂无 — 本轮只建立规则与索引，未创建具体 ADR)_ | | | |

---

> 新增 ADR 时：在本目录创建 `ADR-NNNN-*.md`，并在上表登记编号、标题、状态与批准 commit。
