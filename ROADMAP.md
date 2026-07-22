# ROADMAP（阶段路线图）

> 依据：已批准 V1.1（`AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md` §20），批准 commit `c9cc522b4ab01008fed390c282d6bd5a816ee779`。
> 本路线图描述**未来意图**，不得把未来能力描述为已实现。当前处于 **Phase 0，尚未关闭；Phase 1 尚未授权**。

---

## 通用规则

- **禁止跳阶段**：每个阶段必须满足上一阶段的退出门禁并获得显式授权后才能进入。
- 未通过门禁的能力不得对外声称"已实现"。
- 任一阶段的"明确不包含"项，不得以任何理由在该阶段提前实现。

---

## Phase 0：产品设计与契约冻结（当前所处）

- **目标**：冻结产品设计、协议、角色、权限、状态机、适配器 SDK 契约、威胁模型与验收标准。
- **交付物**：产品书（V1.1）、Source of Truth 文档集、协议/架构/安全文档、SDK 契约文档化、许可政策、验收标准、**机器可读契约（12 个 JSON Schema + 160 fixtures，GOV-APP-0007 APPROVED）**、**capability token 签名契约（ADR-0003 ACCEPTED）**。
- **视觉设计包（第四包方向，非 Phase 0 关闭前置）**：视觉设计基线与视觉资产治理包（精确视觉设计任务书 `VISUAL_DESIGN_BRIEF.md`、Control Room 高保真基准、视觉资产清单 `VISUAL_ASSET_MANIFEST.md`、独立审核 `VISUAL_PACKAGE_REVIEW_0001.md`、owner 批准 `GOVERNANCE_APPROVAL_0005.md`）经 `ADR-0002`（视觉基线作为 Phase 1 UI 实现入口门禁）登记为 **Phase 1 UI 实现入口门禁 / Phase 1 并行准备轨道**，**不再是 Phase 0 关闭前置条件**。因此本包不阻塞 Phase 0 关闭；`GOVERNANCE_APPROVAL_0005.md` 当前仍不存在，视觉设计未启动；Phase 0 为 `READY_FOR_CLOSURE` / `OPEN`，Phase 1 仍 `NOT AUTHORIZED`。
- **明确不包含**：任何产品代码实现；任何真实适配器。
- **进入条件**：项目启动（已满足）。
- **退出门禁（实质性条件）**：
  - V1.1 已批准（commit `c9cc522…`，Source of Truth）；
  - 治理与权威文档包（SOURCE_OF_TRUTH / PRODUCT_PROPOSAL / ROADMAP / LICENSE / THIRD_PARTY_LICENSES / SBOM_POLICY / DECISIONS）通过审核；
  - `ARCHITECTURE.md` 已创建并批准；
  - `PROTOCOL.md` 已创建并批准；
  - `SECURITY.md` 已创建并批准；
  - `PHASE_0_EXIT_CRITERIA.md` 已创建并批准；
  - SDK 契约（生命周期/错误码/取消/恢复/健康检查，V1.1 §20.1）文档化；
  - 威胁模型已建立；
  - 许可与 SBOM 政策已落地（`LICENSE.md` / `SBOM_POLICY.md`）；
  - 全部 Phase 0 设计冲突关闭；
  - machine-readable contracts 已由 `GOVERNANCE_APPROVAL_0007.md` 批准；
  - capability-token signing contract（ADR-0003）已由 `GOVERNANCE_APPROVAL_0007.md` 批准；
  - 视觉设计包（第四包）**不列为 Phase 0 退出门禁**；经 `ADR-0002` 登记为 Phase 1 UI 实现入口门禁（见上文“第四包方向”说明），其完成与批准在 Phase 1 授权前/准备阶段进行，不阻塞 Phase 0 关闭。
- **当前门禁状态**：上述实质性条件已登记满足，Phase 0 为 `READY_FOR_CLOSURE` / `OPEN`；仍须通过独立 `PHASE_0_CLOSURE_APPROVAL_*` 治理事件才能关闭。本批准与本路线图状态不授权 Phase 1。
- **Phase 0 关闭与 Phase 1 授权（两阶段、相互独立，不得合并或自动触发）**：
  1. 上述实质性条件全部满足后，Phase 0 状态进入 `READY_FOR_CLOSURE`；
  2. owner 通过**独立** `PHASE_0_CLOSURE_APPROVAL_*` 记录关闭 Phase 0；
  3. Phase 0 关闭**不自动**授权 Phase 1；
  4. Phase 0 关闭后，再通过**另一份独立** `PHASE_1_AUTHORIZATION_*` 记录授权进入 Phase 1；**授权前须先形成 `PHASE_1_IMPLEMENTATION_PLAN.md`（Phase 1 实施计划，含 scope、mock 范围、测试策略与退出判据）——该文件当前尚未创建，属未来实施计划，不得视为已存在**；
  5. 任何 closure / authorization 记录均不得提前创建（见 `SOURCE_OF_TRUTH.md` §5.1），本轮亦不创建。
- **禁止跳阶段**：Phase 0 未关闭前不得进入 Phase 1；Phase 0 关闭不等于 Phase 1 已授权。

## Phase 1：Core Prototype（仅内核与模拟）

- **目标**：实现内核原型（非完整产品 MVP）。
- **交付物**：状态机、SQLite、事件账本、政策引擎、**模拟**适配器、**模拟**工作流、自动化测试。
- **明确不包含**：任何真实适配器（Git/GitHub/浏览器/UIA/Qoder/ChatGPT/WorkBuddy）；任何真实外部工具。
- **进入条件**：Phase 0 退出门禁全部满足并获显式授权。
- **退出门禁**（全部通过自动化测试）：
  - 崩溃恢复；
  - 去重；
  - 循环上限；
  - 预算；
  - 终态（无无限 RUNNING）；
  - Outbox 对账；
  - policy safe mode；
  - SQLite 一致性备份；
  - WAL checkpoint；
  - 备份恢复演练；
  - 事件重放一致性；
  - projection 重建一致性；
  - Outbox 故障注入与对账；
  - 不接入任何真实外部工具（仅模拟）。
- **禁止跳阶段**：不得在 Phase 1 引入真实适配器或提前进入 Phase 2 生产组件。

## Phase 2：真实工具可行性探针（一次性隔离）

- **目标**：验证真实工具接入可行性，不产出可复用生产组件。
- **交付物**：一次性、隔离、不可直接复用为生产组件的探针——本地 Git 探针、Qoder CLI/SDK 探针、Playwright 浏览器探针、Windows UIA 探针、锁屏/UAC/焦点/会话状态探针。
- **明确不包含**：正式产品适配器；生产可复用组件。
- **进入条件**：Phase 1 退出门禁全部通过。
- **退出门禁**：不使用绝对坐标；目标窗口/页面可稳定识别；隔离仓库无意外修改；探针通过后方可进入 Phase 3。
- **禁止跳阶段**：探针未通过门禁不得进入 Phase 3。

## Phase 3：正式适配器与首个 Product MVP

- **目标**：实现首个真实闭环（Product MVP）。
- **交付物**：正式产品适配器与第一个真实代码开发闭环（如正式 worker 适配器（优先 Qoder CLI/稳定 SDK）+ reviewer 适配器（优先 API/SDK）+ repository 适配器 + notifier）。
- **明确不包含**：Phase 4 的完整多审核与夜间自治。
- **进入条件**：Phase 2 探针通过门禁。
- **退出门禁**：
  - 仅在测试仓库完成端到端闭环；
  - **release gate 只接受 verifier / 外部可验证证据**；
  - worker 的"完成"只能生成**待验证事件**；
  - 未确认外部副作用进入 `RECONCILIATION_REQUIRED`；
  - 同一逻辑 message 不重复创建，但允许受控 delivery attempt（默认最多 3 次）；
  - 工具繁忙不打断；
  - **Phase 3 只要求一个独立 reviewer**；双 reviewer、evidence_verifier、adjudicator 与夜间无人值守属于 Phase 4，不得作为 Phase 3 进入或退出条件。
- **禁止跳阶段**：未形成稳定首个闭环不得进入 Phase 4。

## Phase 4：多审核委员会与自主夜间运行

- **目标**：多审核与无人值守自治。
- **交付物**：盲审、证据验证、仲裁、两轮修复、备用工具、有限自动修复、零打扰摘要。
- **明确不包含**：STS 交易权限；OKX/实盘接入。
- **进入条件**：Phase 3 首个真实闭环稳定。
- **退出门禁**：连续无人值守测试；登录失效、网络断开、锁屏、CI 失败等故障注入通过。
- **禁止跳阶段**：无人值守门禁未通过不得进入 Phase 5。

## Phase 5：STS 研发流程模板（后续路线图）

- **目标**：接入 STS 产品研发流程，**仅模板**。
- **交付物**：STS 研发流程模板。
- **明确不包含**：**不接 OKX、不接实盘、不嵌入交易权限。**
- **进入条件**：Phase 4 完成。
- **退出门禁**：模板在隔离环境验证通过，无任何交易权限泄漏。
- **禁止跳阶段**：不得在此阶段引入实盘或资金权限。

## Phase 6：NAS 控制中心（后续路线图）

- **目标**：从个人 Windows 工具扩展为 NAS 自托管控制中心。
- **交付物**：NAS 控制面/控制中心。
- **明确不包含**：商业化与许可重裁（属 Phase 7）。
- **进入条件**：Phase 5 完成。
- **退出门禁**：NAS 控制面稳定运行并通过安全评估。
- **禁止跳阶段**：不得在此阶段进行商业发布。

## Phase 7：商业化与许可重裁（后续路线图）

- **目标**：产品化与商业版，含许可证与商业模式最终裁定。
- **交付物**：商业版、最终许可与商业模式裁决。
- **明确不包含**：在裁决完成前擅自采用任何开源许可证发布产品源码。
- **进入条件**：Phase 6 完成。
- **退出门禁**：完成商业模式与许可最终裁决及第三方组件法律审查。
- **禁止跳阶段**：无最终许可裁决不得对外商业发布产品源码。

---

> 备注：Phase 5/6/7 目前仅为路线图意图，对应可追溯矩阵中的方向性/`ROADMAP_ONLY` 内容，不代表已实现或已承诺交付时间。
