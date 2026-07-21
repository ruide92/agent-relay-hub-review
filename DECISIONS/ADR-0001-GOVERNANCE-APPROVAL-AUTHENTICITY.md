# ADR-0001 — 治理批准真实性（Governance Approval Authenticity）

Status: PROPOSED
ADR Number: ADR-0001
Title: 治理批准真实性（Phase 0 人工治理批准证据构成与签名边界）
Date: 2026-07-21
Proposed By: Phase 0 design lead / external product architect
Implemented By: executor（WorkBuddy）

> 本 ADR 当前为 `PROPOSED`；没有有效 `Approval Record` 前不得标记 `ACCEPTED`（见 `SOURCE_OF_TRUTH.md` 第 10 节与 `DECISIONS/README.md` 第 3 节）。

---

## 1. 背景（Context）

Phase 0 的治理批准目前由 owner 的明确书面指令、精确 target commit、内容寻址 commit（git SHA）与 append-only 治理记录（`GOVERNANCE_APPROVAL_*.md`）构成。随着 Phase 0 文档集扩大（第一/二/三包已批准生效），需要明确两类边界：

1. **人工治理批准记录的真实性边界**：当前 Markdown 批准记录可能被误读为具有加密签名，或被视为运行时权限令牌。
2. **人工批准与未来机器可执行授权的关系**：Phase 1 将引入 policy bundle、capability token 等机器可验证授权，其签名要求已在 `SECURITY.md` 规定，但人工批准记录与机器授权之间的分界尚不明晰。

## 2. 决策（Decision）

1. Phase 0 当前人类治理批准证据由"owner 明确书面指令 + 精确 target commit + 内容寻址 commit + append-only 治理记录"构成。
2. 当前 Markdown 治理批准记录**不得虚假声称已经具有加密签名**。
3. `SECURITY.md` §8 已批准的签名设计基线包括 ECDSA P-256、SHA-256，以及适用于 policy bundle / SBOM 的 detached signature manifest 与两者不同的 `key_purpose`。该基线**未定义** capability token 的 wire format、签名容器或 purpose schema。capability token 属机器可执行授权，**必须**加密签名，且不得与 policy bundle 或 SBOM 共用含糊的 `key_purpose`；但**本 ADR 不规定 capability token 必须使用 detached signature manifest，也不负责定义其 wire format**。capability token 的规范化 payload、签名预映像（signing preimage）、签名容器、必填 claims、issuer、audience、issued_at / expires_at、nonce / jti、防重放（replay protection）、撤销（revocation）语义及独立 `key_purpose` **必须在 Phase 0 关闭前**形成规范性契约并通过审核；在该契约完成前，Phase 0 不得进入 `READY_FOR_CLOSURE`；不得把这一义务推迟成“Phase 1 实现时再决定”。
4. Markdown 治理记录是**解释性治理证据**，不直接作为运行时权限令牌。
5. 是否对未来人类治理记录附加加密签名，可以后续升级，但**不能削弱**机器授权签名要求。
6. 本 ADR 在独立审核和 owner 批准前**不生效**。

## 3. 替代方案（Alternatives）

- **给当前所有 `GOVERNANCE_APPROVAL_*.md` 补加密签名**：被否决。会虚假声称已具备签名能力，与"当前记录不得虚假声称已签名"原则直接冲突；签名能力应作为未来可升级项明确规划，而非回溯伪装。
- **将 Markdown 治理记录直接作为运行时权限令牌**：被否决。人工批准记录缺乏机器可验证的密码学绑定，且违反 `SECURITY.md` 的机器授权签名基线。

## 4. 后果（Consequences）

- **正面**：明确人工批准记录与机器授权的边界，避免误用；为未来签名升级留出清晰接口而不削弱安全要求。
- **负面 / 风险**：当前人工批准记录无加密签名，依赖 owner 书面指令与仓库 append-only 记录的可追溯性；若仓库历史被篡改则真实性受损（缓解：依赖 git 内容寻址与远端 `main` 一致性）。
- **后续义务**：若未来引入签名，须新增独立 ADR 并经审批，不得削弱 `SECURITY.md` 规定的机器授权签名要求。

## 5. 证据（Evidence）

- `SOURCE_OF_TRUTH.md` §5.1（治理批准与授权记录分类，append-only）。
- `GOVERNANCE_APPROVAL_0001.md` / `_0002.md` / `_0003.md`（现有 append-only 治理记录）。
- `SECURITY.md` §8（Phase 1 签名基线：ECDSA P-256 / SHA-256）。
- 本 ADR 提出前尚未通过独立审核与 owner 批准，故无生效证据。

## 6. 批准字段（待审批后填写）

- **批准 commit（Approval commit）**：—
- **Decision Owner**：owner
- **Reviewer（独立审核者）**：Reviewer-2（待独立审核）
- **Governance Approver（治理批准者）**：owner（待批准）
- **Approval Record**：—
- **Effective Commit（生效 commit）**：—
- **Affected Normative Documents**：`SOURCE_OF_TRUTH.md` §5.1、`SECURITY.md` §15（治理批准记录真实性）。`SECURITY.md` §8 为签名算法依据（referenced baseline；仅定义 policy bundle / SBOM 用途分离，未定义 capability token 的 purpose schema）。
- **Supersedes / Superseded By**：无 / 无

## 7. 状态规则声明

没有有效 `Approval Record` 时，本 ADR 状态只能为 `PROPOSED`，不得标记 `ACCEPTED`（见 `SOURCE_OF_TRUTH.md` 第 10 节与 `DECISIONS/README.md` 第 3 节）。
