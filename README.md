# Agent Relay Hub（ARH）

> 本仓库是 Agent Relay Hub 项目的**设计文档仓库**（Phase 0 Source of Truth 文档集）。
> **本仓库不包含任何产品代码、密钥、账户、交易数据或 STS 交易系统的实现。**

## 仓库性质与边界
- 文档中出现的 **STS、OKX** 仅作为「明确禁止接入与项目隔离」的说明性引用；**不存在 STS 源码、交易策略参数、API 密钥、账户数据、服务器地址或资金信息**。
- 本仓库为公开仓库，对所有人可见；请勿在公开评论或 Issue 中粘贴任何凭据或敏感数据。

## 当前状态
- **产品设计基线**：`AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md`（V1.1）已批准，为当前 Source of Truth。批准 commit `c9cc522b4ab01008fed390c282d6bd5a816ee779`；治理晋级 commit `f24993148a5754e12502eb38dbf566dcbb54c2c1`。
- **Phase 0 Source of Truth 三包已批准**：
  - 第一包（治理与权威体系）：`GOVERNANCE_APPROVAL_0001.md`
  - 第二包（实施契约、安全与退出门禁）：`GOVERNANCE_APPROVAL_0002.md`
  - 第三包（UX、UI 与交互契约）：`GOVERNANCE_APPROVAL_0003.md`
- **Phase 0（产品设计冻结）：进行中，尚未关闭（`OPEN` / `NOT READY`）。**
- **Phase 1（Core Prototype）：尚未授权（`NOT AUTHORIZED`）；当前禁止编写任何产品代码（`NO PRODUCT CODE`）。**

## 待审批治理提案
- 当前存在一项未决治理提案：**Phase 0 治理一致性修订（#0004）**，位于提案分支 `governance/phase0-consistency-repair-0004-proposal`。
- 提案范围（本轮）：`SOURCE_OF_TRUTH.md` 文档分类与审批规则、`ROADMAP.md` 关闭-授权分离门禁、`PHASE_0_EXIT_CRITERIA.md` 门禁登记、`ADR-0001`（治理批准真实性）、`ADR-0002`（视觉基线作为 Phase 1 UI 实现入口门禁）等。
- **状态：`PROPOSAL ONLY / NOT INDEPENDENTLY REVIEWED / NOT APPROVED`。** 须经独立审核与 owner 批准（预期 `GOVERNANCE_APPROVAL_0004.md`）后方可生效；在批准前不视为已批准、不关闭 Phase 0、不授权 Phase 1。

## 权威来源与文档清单
- 权威定义、文档优先级与分类见 `SOURCE_OF_TRUTH.md`。
- 阶段路线图见 `ROADMAP.md`。
- 架构决策记录（ADR）见 `DECISIONS/README.md`。

## 范围声明
纯文档仓库，**未编写任何产品代码、未实现 ARH 系统**。所有设计契约仅表示设计落位，不代表代码已实现。
