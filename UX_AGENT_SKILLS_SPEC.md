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
Last Revised: 2026-07-21 (Reviewer-2 Round 1)
```

# UX_AGENT_SKILLS_SPEC.md（UX 设计 Skills 规范）

> 本文件定义 ARH **项目自己的 UX 设计 skills 规范**，**不安装任何第三方 skill**，不创建可执行 skill 文件。
> 当前为 `PROPOSED`，未经 Reviewer-2 审核与 owner 批准前**不属于 Source of Truth**；
> 不得描述为已批准、已实现或已测试；不得因完成规范就关闭 Phase 0。
> 这些 skill 是**供应商无关的工作规范**，未来可实现为任意 Agent 平台的 skill（见 §5）。

---

## 1. arh-ux-architect（UX 架构师）

**职责**：

- 用户任务建模（owner / governance approver / operator / reviewer / observer，UX_INTERACTION.md §2）；
- 信息架构（UI_INFORMATION_ARCHITECTURE.md）；
- 页面结构（导航、页面目录、低保真）；
- 自动化边界（哪些由系统自动处理，哪些形成用户待办，UX_INTERACTION.md §3）；
- 减少用户介入（默认无人值守，仅在 A5 / 安全 / 治理时介入）。

**输入**：

- V1.1（产品基线）；
- ARCHITECTURE（实施架构契约）；
- PROTOCOL（消息/事件契约）；
- SECURITY（安全边界）；
- UX 当前文档（本包五份）。

**输出**：

- page map（页面地图）；
- user journeys（用户旅程）；
- interruption policy（介入策略）；
- wireframes（低保真，仅 ASCII / Mermaid）；
- unresolved UX risks（未决 UX 风险清单）。

**禁止**：

- 更改产品权限（A0–A5 由 SECURITY.md §6 / V1.1 §4.2 定义）；
- 新增 workflow 终态（ARCHITECTURE.md §6.2 为权威）；
- 把聊天设为默认产品形态（UX_INTERACTION.md §1）；
- 绕过安全边界（SECURITY.md 全域）。

---

## 2. arh-interaction-contract（交互契约）

**职责**：把每个交互转换为可验证链路：

```text
user intent
→ precondition
→ command
→ policy decision
→ pending state
→ event/evidence
→ success/failure
→ terminal presentation
```

**必须检查**（每个 UI 命令，见 UX_INTERACTION.md §4 表）：

- 按钮是否有**明确后端 command**（Core Command 列非空）；
- 是否**等待证据**（Pending State → Required Evidence → Success Event）；
- 是否存在**失败出口**（Failure/Safe State 列）；
- 是否可能**无限 loading**（须有终态或安全停止）；
- 是否**错误显示成功**（对照 UI_STATE_ACCEPTANCE_MATRIX.md §3 防伪完成）。

---

## 3. arh-ui-system（UI 系统）

**职责**：

- design tokens（UI_DESIGN_SYSTEM.md §3）；
- component states（§4、§5）；
- typography（§1、§3）；
- accessibility（§7）；
- visual consistency（§5 状态视觉规则）；
- Storybook-style state inventory（各组件状态清单，**仅规范，不生成真实 Storybook**）。

**禁止**：自行改变领域状态语义（权威状态来自 ARCHITECTURE.md §6 / §7、PROTOCOL.md §5/§7；UI 仅为派生视图）。

---

## 4. arh-ux-reviewer（UX 审查员，独立）

必须独立审查（不与 arh-ux-architect 同一上下文，避免自我通过）：

- 是否把**系统能自动做**的事情交给用户（违反 UX_INTERACTION.md §3.1）；
- 是否将 **ACK / worker 声明**当完成（违反 §6、UI_STATE_ACCEPTANCE_MATRIX.md §3）；
- 是否遗漏**异常、恢复和 safe mode**（UX_INTERACTION.md §7、UI_DESIGN_SYSTEM.md §5）；
- 是否存在**无后端语义的按钮**（违反 §4、arh-interaction-contract）；
- 是否存在**只有正常状态的页面**（违反 UI_STATE_ACCEPTANCE_MATRIX.md §2 覆盖要求）；
- 是否有 **A5 绕过**（SECURITY.md §6、UI_STATE_ACCEPTANCE_MATRIX.md §4）；
- 是否有**未经证据支持的绿色成功状态**（违反防伪完成）；
- 是否让 **reviewer 直接确认 evidence / 终态**（违反职责分离：reviewer→`review.finding`，evidence_verifier→`evidence`，Policy Engine→release gate，UX_INTERACTION.md §2、§5）；
- 是否将**未来 Phase 3/4 功能宣称为 Phase 1 已实现**（违反 UI_INFORMATION_ARCHITECTURE.md §1）。

> 审查结论 MUST 以 `review.finding` 形式提交（PROTOCOL.md §11），关联 finding_id / fingerprint，不自行改写权威状态。

---

## 5. Skill 可移植性

明确：

- 这些是**供应商无关的工作规范**，不绑定具体 Agent 平台；
- **不绑定** WorkBuddy、Qoder、ChatGPT 或 Canva；
- 未来可以实现为**任意 Agent 平台**的 skill（如以 SKILL.md 形式落地）；
- **当前只是规范**，不创建可执行 skill 文件（不写 SKILL.md、不装依赖）；
- **不安装 GitHub 上的随机热门 skill**；
- 外部 skill MUST 先经过：权限审查、提示注入审查、许可证审查、供应链审查（SECURITY.md §11、THIRD_PARTY_LICENSES.md、SBOM_POLICY.md）方可引入。
