# GOVERNANCE_APPROVAL_0003.md（治理批准记录 #3）

> 本文件属于 **B 类解释性治理记录**（与 `SOURCE_OF_TRUTH.md` 第 5 节同类），append-only。
> 后续不得静默修改；撤销或替代 MUST 通过新增治理记录并在新记录中写明 `Supersedes`。

---

## 元数据

| 字段 | 值 |
| --- | --- |
| Approval ID | `GOV-APP-0003` |
| Decision Owner | owner |
| Target Commit | `b74e4076ce59e567de52d67916133a5d9ca26596` |
| Reviewer | Reviewer-2 |
| Reviewer Decision | PASS |
| Owner Decision | APPROVED |
| Scope | Phase 0 Source of Truth package #3（UX、UI 与交互契约） |
| Initial Draft Commit | `aca2fd972fea958919b320c346262615c605e7b7` |
| Round-1 Revision Commit | `919e6697a5570f4a48c3f153724c867fa76378a5` |
| Round-2 Mechanical Cleanup Commit | `1a231f424949ae0728edf1c3feaba991a26d305f` |
| Final Review Target | `b74e4076ce59e567de52d67916133a5d9ca26596` |
| Governance Baseline Effective Commit | `427f0135eb60253a977a6bb846314874de76135c` |
| Phase 0 | OPEN / NOT_READY |
| Phase 1 | NOT AUTHORIZED |
| Code Status | NO PRODUCT CODE |
| Implementation Status | DESIGN ONLY |
| Effective Condition | 本批准记录和机械状态晋级作为同一个 governance-only commit 推送至远端 main 后生效 |
| Approval Source | owner 的本次明确书面批准指令 |
| Supersedes | None |
| Relation | 不撤销或取代 `GOVERNANCE_APPROVAL_0001.md`、`GOVERNANCE_APPROVAL_0002.md` |

---

## 批准范围（逐项列出）

以下五份文档在 target commit `b74e4076ce59e567de52d67916133a5d9ca26596` 中的精确内容，经 Reviewer-2 独立审核 `PASS` 并由 owner `APPROVED`，成为 Phase 0 Source of Truth：

1. `UX_INTERACTION.md`
2. `UI_INFORMATION_ARCHITECTURE.md`
3. `UI_DESIGN_SYSTEM.md`
4. `UI_STATE_ACCEPTANCE_MATRIX.md`
5. `UX_AGENT_SKILLS_SPEC.md`

---

## 批准边界

本批准：

- 不关闭 Phase 0；
- 不授权 Phase 1；
- 不授权创建前端项目；
- 不授权写产品代码；
- 不授权安装 npm、NuGet 或其他依赖；
- 不授权接入 Fluent UI、XState、Storybook 或 React Flow；
- 不授权创建真实 Agent skill；
- 不授权调用真实 Git、Qoder、ChatGPT、Playwright、UIA 或外部 API；
- 不表示设计已经通过可用性测试；
- 不表示 WCAG、键盘操作或屏幕阅读器支持已经实现；
- 不表示 mock 状态验收已经执行；
- 不修改 V1.1、V1.0 或已批准架构、安全和协议正文。

---

## 非阻塞解释（控制性）

`UX_INTERACTION.md §7` 异常恢复表中的"提交治理"属于用户处理方向描述，不自动创建按钮、Interaction ID 或 Core Command。

可执行 UI 必须以以下内容为准：

1. `UI_INFORMATION_ARCHITECTURE.md` 的 Action Traceability；
2. `UI_STATE_ACCEPTANCE_MATRIX.md` 的 Allowed Actions；
3. `UX_INTERACTION.md` §4 的 Interaction Contract。

因此在 `BLOCKED_NEEDS_POLICY_DECISION` 等状态中，"提交治理"在当前版本只能实现为：

> 导航至 Policies & Authorization（NAV）

不得实现为未定义的提交按钮。未来只有新增并经治理批准的 Interaction ID 才能形成新的 command。

该解释不修改第三包正文，不新增 Interaction ID，也不改变领域状态机。
