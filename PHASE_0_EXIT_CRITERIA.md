```text
Overall Status: NOT READY
Phase 0: OPEN
Phase 1: NOT AUTHORIZED
```

# PHASE_0_EXIT_CRITERIA.md（Phase 0 退出门禁）

> 本文件定义 Phase 0 关闭所必须满足的退出条件、证据要求、状态与批准程序。
> **创建本文件不代表 Phase 0 已通过**；在全部条件满足并经独立审核与 owner 批准前，
> `Overall Status` 不得标记为就绪，`Phase 0` 不得关闭，`Phase 1` 不得授权。
> 本文件当前为草案；其自身状态的演进见第 3、4 节。

---

## 1. 状态枚举

每项退出条件 MUST 使用以下状态之一：

- `NOT_STARTED`：尚未开始。
- `DRAFT`：草稿/提案已提出，未审核。
- `REVIEW_FAILED`：独立审核未通过。
- `REVIEW_PASSED`：独立审核通过（Reviewer-2 PASS）。
- `OWNER_APPROVED`：owner 或有效 governance_approver 批准生效。
- `BLOCKED`：被未解决项阻塞。
- `NOT_APPLICABLE_WITH_REASON`：经裁定不适用，并给出原因。

---

## 2. 证据要求

每一项 MUST 记录以下证据字段：

- **Criterion ID**：条件唯一标识。
- **requirement**：该条件的具体要求。
- **normative source**：权威依据（如 V1.1 章节、ARCHITECTURE.md 等规范性文档）。
- **artifact**：满足条件的产物（文件、commit、测试报告）。
- **exact commit**：证据所对应的精确 commit SHA。
- **reviewer**：独立审核者（如 Reviewer-2）。
- **review decision**：审核结论（PASS / FAIL）。
- **approval record**：可追溯的批准记录引用。
- **status**：第 1 节状态枚举之一。
- **blocker**：阻塞项（如有）。
- **notes**：补充说明。

---

## 3. 第一包状态（Phase 0 Source of Truth package #1：治理与权威体系）

| 字段 | 值 |
|---|---|
| 第一包 target commit | `cf8e66b431b36103fb7c049048521c15df2e3701` |
| Reviewer-2 | PASS |
| Owner | APPROVED |
| Approval Record | `GOVERNANCE_APPROVAL_0001.md` |
| Effective Commit | `93dbdbbfbe630740eff5a39850576dd993e6450e` |
| Status | `OWNER_APPROVED` |

> 第一包已批准生效，但 Phase 0 整体仍未关闭（见第 5、6 节）。

---

## 4. 第二包状态（Phase 0 Source of Truth package #2：实施契约、安全与退出门禁）

以下四份新增规范性文档的初始状态 MUST 为 `DRAFT`；**不得提前写 `REVIEW_PASSED` 或 `OWNER_APPROVED`**（须先经 Reviewer-2 审核与 owner 批准）。

| Criterion ID | 文档 | 初始状态 |
|---|---|---|
| P2-ARCH | `ARCHITECTURE.md` | `DRAFT` |
| P2-PROTO | `PROTOCOL.md` | `DRAFT` |
| P2-SEC | `SECURITY.md` | `DRAFT` |
| P2-EXIT | `PHASE_0_EXIT_CRITERIA.md` | `DRAFT` |

---

## 5. Phase 0 必须关闭的项目

Phase 0 关闭前，以下每一项 MUST 达到 `OWNER_APPROVED`（或 `NOT_APPLICABLE_WITH_REASON` 且理由成立）：

- V1.1 approved（commit `c9cc522…`）；
- 第一包 approved（见第 3 节）；
- `ARCHITECTURE.md` approved；
- `PROTOCOL.md` approved；
- `SECURITY.md` approved；
- threat model approved（SECURITY.md §5）；
- SDK contract approved（PROTOCOL.md §7 / ARCHITECTURE.md §6）；
- licensing policy approved（`LICENSE.md`）；
- SBOM policy approved（`SBOM_POLICY.md`）；
- signing / trust-root mechanism approved（SECURITY.md §8，须先经 SECURITY.md 批准）；
- backup / recovery contract approved（ARCHITECTURE.md §8）；
- no unresolved P0 design conflict（V1.1 设计冲突全部关闭）；
- no product code created before authorization（授权前无产品代码）；
- independent review passed（Reviewer-2 对全部包 PASS）；
- owner closure decision recorded（见第 6 节 `PHASE_0_CLOSURE_APPROVAL_0001.md`）。

---

## 6. Phase 0 关闭与 Phase 1 授权分离

- **Phase 0 关闭** MUST 通过独立记录：`PHASE_0_CLOSURE_APPROVAL_0001.md`。
- **Phase 1 进入** MUST 通过另一份独立记录：`PHASE_1_AUTHORIZATION_0001.md`。
- Phase 0 关闭 **不自动授权 Phase 1**；两者为独立治理事件。
- Phase 1 授权 MUST 引用已关闭的 Phase 0 commit（即 `PHASE_0_CLOSURE_APPROVAL_0001.md` 的 effective commit）。
- 两份记录均须 owner 或有效 governance_approver 批准（见 SOURCE_OF_TRUTH.md §10）。
- **当前两份记录均不存在**，因此 Phase 0 仍为 `OPEN`，Phase 1 仍为 `NOT AUTHORIZED`。

---

## 7. Fail-closed

任一必需证据缺失、commit 不匹配、审核失败或批准记录缺失时：

- Phase 0 **不得关闭**；
- Phase 1 **不得授权**；
- **不得开始编码**。

> 本文件仅为 Phase 0 退出条件的记录与追踪工具；它本身不构成对任何阶段的批准或授权。
