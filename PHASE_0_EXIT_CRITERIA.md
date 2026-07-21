```text
Document Status: APPROVED
Normative Status: 已批准，当前为 Phase 0 Source of Truth
Approval Target Commit: b8ef21978544e1261081389ecf736e72b80e49d7
Product Baseline: AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md
Product Baseline Commit: c9cc522b4ab01008fed390c282d6bd5a816ee779
Governance Baseline Commit: cf8e66b431b36103fb7c049048521c15df2e3701
Governance Approval Record: GOVERNANCE_APPROVAL_0002.md
Current Phase: Phase 0
Phase 1: NOT AUTHORIZED
Code Status: NO PRODUCT CODE
Overall Status: NOT READY
Phase 0: OPEN
```

# PHASE_0_EXIT_CRITERIA.md（Phase 0 退出门禁）

> 本文件定义 Phase 0 关闭所必须满足的退出条件、证据要求、状态与批准程序。
> **创建本文件不代表 Phase 0 已通过**；在全部条件满足并经独立审核与 owner 批准前，
> `Overall Status` 不得标记为就绪，`Phase 0` 不得关闭，`Phase 1` 不得授权。
> 本文件（作为退出门禁契约本身）已通过 Reviewer-2 对 target commit `b8ef21978544e1261081389ecf736e72b80e49d7` 的独立审核，并已由 `GOVERNANCE_APPROVAL_0002.md` 批准，当前为 **Phase 0 Source of Truth**；批准表示门禁契约生效，**不表示** Phase 0 已满足退出条件——Phase 0 仍为 `OPEN` / `NOT READY`，Phase 1 仍未授权。其自身状态与各条件的演进见第 1、5、6 节。

---

## 1. 状态枚举

### 1.1 单项退出条件状态（per-criterion）

每项退出条件 MUST 使用以下状态之一：

- `NOT_STARTED`：尚未开始。
- `DRAFT`：草稿/提案已提出，未审核。
- `REVIEW_FAILED`：独立审核未通过。
- `REVIEW_PASSED`：独立审核通过（Reviewer-2 PASS）。
- `OWNER_APPROVED`：owner 或有效 governance_approver 批准生效。
- `BLOCKED`：被未解决项阻塞。
- `NOT_APPLICABLE_WITH_REASON`：经裁定不适用，并给出原因。

### 1.2 总体 Phase 0 状态（overall）

Phase 0 总体状态 MUST 使用以下三值之一：

- `NOT_READY`：存在未满足的实质性条件；不得关闭。
- `READY_FOR_CLOSURE`：所有实质性条件已满足，等待治理关闭事件。
- `CLOSED`：已通过 `PHASE_0_CLOSURE_APPROVAL_0001.md` 治理事件正式关闭。

逻辑：

1. 所有实质性条件满足后，整体才能进入 `READY_FOR_CLOSURE`；
2. `PHASE_0_CLOSURE_APPROVAL_0001.md` 是把状态从 `READY_FOR_CLOSURE` 转为 `CLOSED` 的治理事件；
3. 关闭记录**不是**"进入 `READY_FOR_CLOSURE` 前已经存在"的前置条件；
4. 当前状态仍为 `NOT_READY / OPEN`；
5. Phase 1 授权仍必须通过独立的 `PHASE_1_AUTHORIZATION_0001.md`。

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
- **status**：第 1.1 节状态枚举之一。
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

> 第一包已批准生效，但 Phase 0 整体仍未关闭（见第 1.2、5、6 节）。

---

## 4. 第二包状态（Phase 0 Source of Truth package #2：实施契约、安全与退出门禁）

以下四份新增规范性文档的第二包提交链（已由 Reviewer-2 独立审核 `PASS`，并由 owner 经 `GOVERNANCE_APPROVAL_0002.md` 批准）：

* Initial Draft Commit：`98412a1970fdc78ef771918068908ee691fca460`
* Round-1 Revision Commit：`225b39948e3d2fa18402d8789013282b51feb8f8`
* Mechanical Correction Commit：`154638bdba30675d67739228d9cf48be2ef9c8a4`
* Final Review Target：`b8ef21978544e1261081389ecf736e72b80e49d7`
* Reviewer Decision：`PASS`
* Owner Approval Record：`GOVERNANCE_APPROVAL_0002.md`

说明：

* 上述四份文档已由 Reviewer-2 对 Final Review Target `b8ef219…` 完成独立审核并给出 `PASS`；
* owner 已通过 `GOVERNANCE_APPROVAL_0002.md` 批准其成为 Phase 0 当前规范性 Source of Truth；
* 批准表示设计契约生效，**不表示**产品代码、安全控制或测试已实现；Phase 0 仍未关闭，Phase 1 仍未授权。

> 以下表格记录第二包各文档的**历史初始状态（`DRAFT`）**，仅用于追溯；其当前状态已晋级为 `OWNER_APPROVED`（见第 5 节主矩阵 `P2-ARCH` / `P2-PROTO` / `P2-SEC` / `P2-EXIT`），不代表当前状态。

| Criterion ID | 文档 | 历史初始状态 |
|---|---|---|
| P2-ARCH | `ARCHITECTURE.md` | `DRAFT`（历史初始） |
| P2-PROTO | `PROTOCOL.md` | `DRAFT`（历史初始） |
| P2-SEC | `SECURITY.md` | `DRAFT`（历史初始） |
| P2-EXIT | `PHASE_0_EXIT_CRITERIA.md` | `DRAFT`（历史初始） |

> 第二包 `SOURCE_OF_TRUTH.md` §5.1 治理记录分类增量已由 Reviewer-2 审核通过（见第 5 节 `P0-P2-SOT`），并随第二包一并由 `GOVERNANCE_APPROVAL_0002.md` 批准生效。

---

## 第三包状态（Phase 0 Source of Truth package #3：UX、UI 与交互契约）

以下五份设计契约文档的第三包提交链（已由 Reviewer-2 对 Final Review Target `b74e4076ce59e567de52d67916133a5d9ca26596` 独立审核 `PASS`，并由 owner 经 `GOVERNANCE_APPROVAL_0003.md` 批准）：

* Initial Draft Commit：`aca2fd972fea958919b320c346262615c605e7b7`
* Round-1 Revision Commit：`919e6697a5570f4a48c3f153724c867fa76378a5`
* Round-2 Mechanical Cleanup Commit：`1a231f424949ae0728edf1c3feaba991a26d305f`
* Final Review Target：`b74e4076ce59e567de52d67916133a5d9ca26596`
* Reviewer：Reviewer-2
* Review Decision：PASS
* Owner Approval Record：`GOVERNANCE_APPROVAL_0003.md`
* Status：OWNER_APPROVED

明确：

* 批准的是设计契约；
* 当前没有实现；
* 不关闭 Phase 0；
* 不授权 Phase 1。

---

## 第四包状态（视觉设计基线与视觉资产治理：Phase 1 UI 实现入口门禁 / Phase 1 并行准备轨道）

> 本包**仅为方向登记**，当前**未开始**；本轮不得创建其正文、图片、资产清单或批准记录。经 `ADR-0002`（视觉基线作为 Phase 1 UI 实现入口门禁）登记，**本包不是 Phase 0 关闭前置条件**，而是 **Phase 1 UI 实现入口门禁 / Phase 1 并行准备轨道**。其条件登记于本节（§4）、ROADMAP 与 ADR-0002（视觉基线作为 Phase 1 UI 实现入口门禁），不在 Phase 0 主矩阵中，状态均为 `NOT_STARTED`，并在 Phase 1 授权前/准备阶段进行，不阻塞 Phase 0 关闭。本包登记三项设计产物与两项治理证据（仅登记预期名称，不创建文件）：

**三项设计产物（design artifacts，非治理记录）**：
- `VISUAL_DESIGN_BRIEF.md`（精确的视觉设计任务书）；
- Control Room 高保真基准资产（owner 认可的 Control Room 高保真基准）；
- `VISUAL_ASSET_MANIFEST.md`（视觉资产清单：尺寸 / 文件 hash / 版本 / 契约映射）。

**两项治理证据（governance evidence，独立审核与批准记录，非设计产物）**：
- `VISUAL_PACKAGE_REVIEW_0001.md`（独立审核记录，预期由独立于 Reviewer-2 的 visual design lead / Phase 0 design lead 完成设计后，由 Reviewer-2 独立审核）；
- `GOVERNANCE_APPROVAL_0005.md`（owner 批准记录，预期由 owner 最终批准，专用于第四包视觉设计产物；与批准本次治理一致性修订 / ADR-0001 / ADR-0002 的 `GOVERNANCE_APPROVAL_0004.md` 区分）。Phase 0 仍为 `OPEN` / `NOT READY`、Phase 1 仍 `NOT AUTHORIZED` 是因为其他未满足条件（机器可读契约、capability token 契约等），而非视觉包。

---

## 5. 证据矩阵（主矩阵）

每一行 MUST 含：Criterion ID | Requirement | Normative Source | Artifact | Exact Commit | Reviewer | Review Decision | Approval Record | Status | Blocker | Notes。

第二包相关行已由 Reviewer-2 对 target commit `b8ef21978544e1261081389ecf736e72b80e49d7` 审核 `PASS`，并由 owner 经 `GOVERNANCE_APPROVAL_0002.md` 批准，Status 晋级为 `OWNER_APPROVED`；本次晋级**不改变** Phase 0 总体状态（仍 `NOT READY` / `OPEN`），亦**不表示**任何产品代码或安全控制已实现。

| Criterion ID | Requirement | Normative Source | Artifact | Exact Commit | Reviewer | Review Decision | Approval Record | Status | Blocker | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| P0-V11 | V1.1 产品设计批准 | V1.1 全文（已批准 SoT） | AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md | c9cc522b4ab01008fed390c282d6bd5a816ee779 | Reviewer-2 | PASS | REVIEWER_2_FINAL_APPROVAL.md；governance promotion commit f24993148a5754e12502eb38dbf566dcbb54c2c1 | OWNER_APPROVED | — | 已批准生效 |
| P0-PKG1 | 第一包（治理与权威体系）批准 | SOURCE_OF_TRUTH.md / 第一包全部文件 | 第一包 7 文件 | cf8e66b431b36103fb7c049048521c15df2e3701 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0001.md | OWNER_APPROVED | — | effective 93dbdbbfbe630740eff5a39850576dd993e6450e |
| P0-P2-SOT | 第二包 SOURCE_OF_TRUTH.md §5.1 治理记录分类增量 | SOURCE_OF_TRUTH.md §5.1 | SOURCE_OF_TRUTH.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 内容最初落在 98412a1…，当前第二包批准锚点统一为 b8ef219… |
| P2-ARCH | ARCHITECTURE.md 实施架构契约批准 | ARCHITECTURE.md 全文 | ARCHITECTURE.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 第二包，Reviewer-2 PASS + owner 批准生效 |
| P2-PROTO | PROTOCOL.md 规范性协议批准 | PROTOCOL.md 全文 | PROTOCOL.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 第二包，Reviewer-2 PASS + owner 批准生效 |
| P2-SEC | SECURITY.md 安全与威胁模型批准 | SECURITY.md 全文 | SECURITY.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 第二包，Reviewer-2 PASS + owner 批准生效；控制均未实现，Validation Status=UNVALIDATED |
| P2-EXIT | PHASE_0_EXIT_CRITERIA.md 退出门禁批准 | 本文件全文 | PHASE_0_EXIT_CRITERIA.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 门禁契约批准生效；Phase 0 仍 OPEN / NOT READY |
| P2-THREAT | 威胁模型批准 | SECURITY.md §5（STRIDE） | SECURITY.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 威胁模型契约批准生效；控制均未实现，Validation Status=UNVALIDATED |
| P2-SDK | SDK contract 批准 | PROTOCOL.md §1/§6/§7/§8/§9/§10；ARCHITECTURE.md §3/§4（注：ARCHITECTURE.md §6 为 workflow 状态机，非完整 SDK contract） | PROTOCOL.md + ARCHITECTURE.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 已引用上述全部章节，契约批准生效 |
| P2-SIGN | signing / trust-root 机制批准 | SECURITY.md §8（Phase 1 签名基线） | SECURITY.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 签名基线契约批准生效；机制尚未实现或测试 |
| P0-LICENSE | license policy 批准 | LICENSE.md | LICENSE.md | cf8e66b431b36103fb7c049048521c15df2e3701 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0001.md | OWNER_APPROVED | — | 第一包 |
| P0-TPL | third-party license policy 批准 | THIRD_PARTY_LICENSES.md | THIRD_PARTY_LICENSES.md | cf8e66b431b36103fb7c049048521c15df2e3701 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0001.md | OWNER_APPROVED | — | 第一包 |
| P0-SBOM | SBOM policy 批准 | SBOM_POLICY.md | SBOM_POLICY.md | cf8e66b431b36103fb7c049048521c15df2e3701 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0001.md | OWNER_APPROVED | — | 第一包 |
| P2-BACKUP | backup / recovery contract 批准 | ARCHITECTURE.md §8 | ARCHITECTURE.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 第二包，契约批准生效 |
| P0-NOCONFLICT | 无未解决 P0 设计冲突 | V1.1 设计冲突全部关闭 | V1.1_CHANGESET_FINAL.md / REVIEWER_2_ADJUDICATION.md | c9cc522b4ab01008fed390c282d6bd5a816ee779 | Reviewer-2 | PASS | V1.1_CHANGESET_FINAL.md / REVIEWER_2_ADJUDICATION.md / REVIEWER_2_FINAL_APPROVAL.md；governance promotion commit f24993148a5754e12502eb38dbf566dcbb54c2c1 | OWNER_APPROVED | — | V1.1 冲突已裁决关闭 |
| P0-NOCODE | 授权前无产品代码 | Code Status: NO PRODUCT CODE | 仓库根（无 src/ 产品代码） | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | 当前无产品代码；批准不授权编码 |
| P2-REVIEW | 第二包独立审核 | Reviewer-2 对第二包全部文件 PASS | 本矩阵 + Reviewer-2 审核记录 | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | Reviewer-2 对 b8ef219 完成独立审核 PASS |
| P2-OWNER | 第二包 owner 批准 | owner 或有效 governance_approver 批准第二包 | GOVERNANCE_APPROVAL_0002.md | b8ef21978544e1261081389ecf736e72b80e49d7 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0002.md | OWNER_APPROVED | — | owner 批准的内容目标是 b8ef219；批准记录从包含 GOVERNANCE_APPROVAL_0002.md 的治理晋级提交推送至 main 后生效 |
| P3-UX | 整体 UX、角色、无人值守原则、Interaction Contract、完成与异常语义已冻结 | V1.1 / ARCHITECTURE / PROTOCOL / SECURITY | `UX_INTERACTION.md` | b74e4076ce59e567de52d67916133a5d9ca26596 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0003.md | OWNER_APPROVED | — | DESIGN ONLY；未实现 |
| P3-IA | 导航、页面、低保真结构和 Action Traceability 已冻结 | V1.1 / ARCHITECTURE / PROTOCOL / SECURITY | `UI_INFORMATION_ARCHITECTURE.md` | b74e4076ce59e567de52d67916133a5d9ca26596 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0003.md | OWNER_APPROVED | — | DESIGN ONLY；未实现 |
| P3-DESIGN-SYSTEM | 视觉语义、设计 token、组件状态和 Accessibility Contract 已冻结 | V1.1 / ARCHITECTURE / PROTOCOL / SECURITY | `UI_DESIGN_SYSTEM.md` | b74e4076ce59e567de52d67916133a5d9ca26596 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0003.md | OWNER_APPROVED | — | 候选技术不是已批准依赖；DESIGN ONLY；未实现 |
| P3-STATE-MATRIX | UI 状态、防伪完成、权限和 mock 验收矩阵已冻结 | V1.1 / ARCHITECTURE / PROTOCOL / SECURITY | `UI_STATE_ACCEPTANCE_MATRIX.md` | b74e4076ce59e567de52d67916133a5d9ca26596 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0003.md | OWNER_APPROVED | — | 34 状态行；尚未执行实现验收；DESIGN ONLY；未实现 |
| P3-UX-SKILLS | 供应商无关的 UX Agent skills 工作规范已冻结 | V1.1 / ARCHITECTURE / PROTOCOL / SECURITY | `UX_AGENT_SKILLS_SPEC.md` | b74e4076ce59e567de52d67916133a5d9ca26596 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0003.md | OWNER_APPROVED | — | 只是规范；未创建真实 skill；DESIGN ONLY；未实现 |
| P3-REVIEW | 第三包通过独立 Reviewer-2 审核 | V1.1 / ARCHITECTURE / PROTOCOL / SECURITY | 第三包五份文档及 `GOVERNANCE_APPROVAL_0003.md` | b74e4076ce59e567de52d67916133a5d9ca26596 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0003.md | OWNER_APPROVED | — | 最终审核目标为 b74e407 |
| P3-OWNER | owner 对第三包作出明确批准 | V1.1 / ARCHITECTURE / PROTOCOL / SECURITY | `GOVERNANCE_APPROVAL_0003.md` | b74e4076ce59e567de52d67916133a5d9ca26596 | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0003.md | OWNER_APPROVED | — | 批准记录随原子 governance-only commit 推送 main 后生效 |
| P0-GOV-CONSISTENCY | Phase 0 治理一致性修订（§2/§4 文档分类与优先级 / §10.2 独立审核证据规则 / ROADMAP 关闭-授权分离门禁与第四包改登记为 Phase 1 UI 入口门禁 / 本矩阵门禁登记 / ADR-0001 治理批准真实性 / ADR-0002 视觉基线作为 Phase 1 UI 实现入口门禁 / SECURITY.md §8–§15 与 T-05 一致性 / SBOM_POLICY §8 签名基线一致性 / V1.1_TRACEABILITY_MATRIX §5 表述修正 / PRODUCT_PROPOSAL 优先级链修正 / README 当前状态描述） | README.md、SOURCE_OF_TRUTH.md（§2 / §4 / §5.1 / §10.2）、PRODUCT_PROPOSAL.md、ROADMAP.md、V1.1_TRACEABILITY_MATRIX.md、SBOM_POLICY.md、SECURITY.md（§8 / §15 / T-05）、PHASE_0_EXIT_CRITERIA.md、DECISIONS/README.md、DECISIONS/ADR-0001、DECISIONS/ADR-0002（已重命名为 ...UI-IMPLEMENTATION-GATE.md） | README.md、SOURCE_OF_TRUTH.md、PRODUCT_PROPOSAL.md、ROADMAP.md、V1.1_TRACEABILITY_MATRIX.md、SBOM_POLICY.md、SECURITY.md、PHASE_0_EXIT_CRITERIA.md、DECISIONS/README.md、ADR-0001、ADR-0002（本轮提案；target commit `fd2574c28843ee2885460cb191c735dd0d257d1a`；PROTOCOL.md 经核查无重复 §5 标题，本轮未改） | fd2574c28843ee2885460cb191c735dd0d257d1a | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0004.md | OWNER_APPROVED | PENDING_EFFECTIVE_COMMIT_BACKFILL | 批准随 `GOVERNANCE_APPROVAL_0004` 的 Commit A/B/C 经晋级 PR 合并并可从 main 到达后生效；Phase 0 仍 OPEN / NOT_READY；Phase 1 仍 NOT AUTHORIZED；NO PRODUCT CODE |
| P0-APPROVAL-AUTHENTICITY-ADR | 治理批准真实性 ADR（人工批准证据构成 / 不得虚假声称加密签名 / 机器授权签名边界） | SOURCE_OF_TRUTH.md §5.1；SECURITY.md §15；SECURITY.md §8（referenced signing baseline）；DECISIONS/README.md §3 / §5 / §7 | DECISIONS/ADR-0001-GOVERNANCE-APPROVAL-AUTHENTICITY.md | fd2574c28843ee2885460cb191c735dd0d257d1a | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0004.md | OWNER_APPROVED | PENDING_EFFECTIVE_COMMIT_BACKFILL | ADR-0001 已通过独立 Reviewer-2 审核（review_run_id `29fe1f22-6b59-4f69-9c5a-82a6fa8deceb`，评论 `5035425502`）并经 owner 批准；批准随晋级 PR 合并后可从 main 到达生效 |
| P0-VISUAL-GATE-ADR | 将视觉设计基线与视觉资产治理包登记为 Phase 1 UI 实现入口门禁 / Phase 1 并行准备轨道（非 Phase 0 关闭前置）的阶段边界决策 | SOURCE_OF_TRUTH.md §9 / §10；DECISIONS/README.md §5 / §7 / §8；ROADMAP.md；本文件 | DECISIONS/ADR-0002-VISUAL-BASELINE-AS-UI-IMPLEMENTATION-GATE.md | fd2574c28843ee2885460cb191c735dd0d257d1a | Reviewer-2 | PASS | GOVERNANCE_APPROVAL_0004.md | OWNER_APPROVED | PENDING_EFFECTIVE_COMMIT_BACKFILL | ADR-0002 已批准，仅表示“视觉包是 Phase 1 UI 实现入口门禁”决策生效，不表示视觉产物已完成或获 `GOVERNANCE_APPROVAL_0005` 批准；第四包视觉产物另由未来 `GOVERNANCE_APPROVAL_0005.md` 批准；视觉包不阻塞 Phase 0 关闭 |
| P0-CAPABILITY-TOKEN-SIGNING-CONTRACT | capability token 完整签名契约（规范化 payload / issuer / audience / 最小权限 scope / issued_at / expires_at / token ID 或 nonce / 防重放 / 撤销 / 签名算法引用 / 签名预映像 / 独立 key_purpose / 校验失败 fail-closed） | V1.1 §4.2.1；SECURITY.md §8 / §15；PROTOCOL.md §4.3；ADR-0001 | （待后续批准的 PROTOCOL.md / SECURITY.md 契约修订或专门规范文件） | — | — | — | — | NOT_STARTED | capability token 完整签名与防重放契约尚未形成；Phase 0 不得进入 READY_FOR_CLOSURE | 本轮只登记缺口，不修改 PROTOCOL 或实现 token |
| P0-MACHINE-READABLE-CONTRACTS | 机器可读契约（ARP envelope / message types、adapter capability / lifecycle、health_check、policy bundle、signature manifest、capability token Schema）形成规范性文件 | V1.1 §4.2.1 / §10 / §11 / §20.1；PROTOCOL.md §1–§12；SECURITY.md §8 / §15；ADR-0001 | （待后续批准的 PROTOCOL.md / SECURITY.md 契约修订或专门规范文件） | — | — | — | — | NOT_STARTED | 机器可读契约（含 capability token Schema、policy bundle、signature manifest 等）尚未形成规范性文件；Phase 0 不得进入 READY_FOR_CLOSURE | 本轮只登记缺口，不创建 Schema 或实现 |

---

## 6. Phase 0 关闭与 Phase 1 授权分离

- **Phase 0 关闭** MUST 通过独立记录：`PHASE_0_CLOSURE_APPROVAL_0001.md`。
- **Phase 1 进入** MUST 通过另一份独立记录：`PHASE_1_AUTHORIZATION_0001.md`。
- Phase 0 关闭 **不自动授权 Phase 1**；两者为独立治理事件。
- `PHASE_0_CLOSURE_APPROVAL_0001.md` 是把总体状态从 `READY_FOR_CLOSURE` 转为 `CLOSED` 的治理事件（见 §1.2）；它**不是**进入 `READY_FOR_CLOSURE` 的前置条件。
- Phase 1 授权 MUST 引用已关闭的 Phase 0 commit（即 `PHASE_0_CLOSURE_APPROVAL_0001.md` 的 effective commit）。
- 两份记录均须 owner 或有效 governance_approver 批准（见 SOURCE_OF_TRUTH.md §10）。
- **当前两份记录均不存在**，因此 Phase 0 仍为 `OPEN` / `NOT_READY`，Phase 1 仍为 `NOT AUTHORIZED`。

---

## 7. Fail-closed

任一必需证据缺失、commit 不匹配、审核失败或批准记录缺失时：

- Phase 0 **不得关闭**；
- Phase 1 **不得授权**；
- **不得开始编码**。

> 本文件仅为 Phase 0 退出条件的记录与追踪工具；它本身不构成对任何阶段的批准或授权。
