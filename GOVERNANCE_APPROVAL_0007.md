# GOVERNANCE_APPROVAL_0007 — Phase 0 机器可读契约与 Capability Token 签名契约批准

> 本文件为 **append-only** 批准记录。创建后不得修改、不得重写历史。任何更正或补充均须以新增批准记录形式进行。

## 1. 批准标识

| 字段 | 值 |
|---|---|
| **Approval ID** | `GOV-APP-0007` |
| **Decision Owner** | owner |
| **Owner Decision** | `APPROVED` |
| **Target Commit** | `19fdff1beaa5208fec23653f83b47046fe8c3427` |
| **Base Commit** | `0ebd537771735aa1bb18143bd821621eb2cea536` |
| **Proposal PR** | #4 — `[PROPOSAL] Phase 0 machine-readable contracts and capability-token signing #0007` |
| **Reviewer** | Reviewer-2；Codex in ChatGPT Work Mode；精确模型/版本 `UNKNOWN` |
| **Reviewer Decision** | `PASS` |
| **Independent review context identifier** | `/root/reviewer2_pr4` |
| **review_run_id** | `4ae06323-1db1-4754-85fa-4f43638f7fab` |
| **正式审核产物（PR 评论）** | https://github.com/ruide92/agent-relay-hub-review/pull/4#issuecomment-5051429528 |
| **GitHub comment ID** | `5051429528` |

独立审核直接绑定上述 Base / Target SHA，并记录了完整 fallback provenance、UTC 起止时间、独立只读工作区、直接读取不可变 GitHub 对象的声明、完整 findings 与明确 verdict。该审核未参与被审提案的修改或指导。

## 2. Owner 书面批准原文

> 我以项目 owner 身份，批准 PR #4 治理提案最终审核目标 commit `19fdff1beaa5208fec23653f83b47046fe8c3427`，认可 Reviewer-2 在 PR #4 评论 `5051429528`、review_run_id `4ae06323-1db1-4754-85fa-4f43638f7fab` 中给出的独立审核 PASS。
>
> 授权按 `GOVERNANCE_APPROVAL_0007` 流程创建批准记录并完成治理晋级提案。
>
> 本批准不直接关闭 Phase 0，不授权 Phase 1，不授权产品代码、安装依赖、生成图片或启动视觉设计；在最终晋级 PR 的精确 head 未再次获得 owner 合并许可前，不得合并 main。

## 3. 批准范围

本批准绑定 Target Commit `19fdff1…` 的完整 PR #4 提案范围（相对 Base Commit 共 **183 files changed, 16491 insertions, 5 deletions**），批准其作为 Phase 0 设计契约与规范性证据进入治理晋级流程：

1. `MACHINE_READABLE_CONTRACTS.md` 规范性总说明；
2. `CONTRACTS/README.md`、12 个 Draft 2020-12 JSON Schema 与 160 个 conformance fixtures；
3. `DECISIONS/ADR-0003-CAPABILITY-TOKEN-WIRE-FORMAT-AND-SIGNING.md`；
4. `PROTOCOL.md`、`SECURITY.md`、`ARCHITECTURE.md` 中提案 #0007 的设计契约增补；
5. `PHASE_0_EXIT_CRITERIA.md`、`SOURCE_OF_TRUTH.md`、`ROADMAP.md`、`V1.1_TRACEABILITY_MATRIX.md`、`DECISIONS/README.md` 的治理、状态与追踪登记。

本批准确认：

- capability token 采用 JWS Compact Serialization / ES256，并使用独立 `key_purpose=capability_token`；
- 机器可读契约包包含 12 个 Schema 与 160 个 fixtures；
- Reviewer-2 对 exact target 的独立验证结论为 `PASS`；
- Schema / fixture / business semantic 的设计级验证证据可作为 Phase 0 契约证据；
- Crypto / Runtime Validation 仍为 `UNVALIDATED`，本批准不将设计验证描述为运行时实现或密码学验证。

## 4. 与既有批准记录的关系

- 本记录不撤销、不取代 `GOVERNANCE_APPROVAL_0001.md`、`0002.md`、`0003.md`、`0004.md` 或纠正性记录 `0006.md`。
- `GOVERNANCE_APPROVAL_0005.md` 继续保留给第四包视觉设计产物，当前不存在；本记录不占用、不批准该视觉包。
- 本记录落实 `GOVERNANCE_APPROVAL_0006.md` 第 4 节所登记的 Phase 0 关闭前契约义务。
- **Supersedes**: `None`

## 5. 批准边界（明确不授权范围）

本批准只批准设计契约、Schema、conformance fixtures、ADR-0003 与相关规范性文档修订；明确不包含：

- 不直接关闭 Phase 0，不创建 `PHASE_0_CLOSURE_APPROVAL_*`；
- 不授权 Phase 1，不创建 `PHASE_1_AUTHORIZATION_*`；
- 不授权产品代码、运行时代码、Schema 运行时实现、依赖安装、CI 或部署；
- 不授权真实密钥、真实 token、真实签名或密码学运行时验证；
- 不生成图片，不批准或启动第四包视觉设计；
- 不创建或批准 `GOVERNANCE_APPROVAL_0005.md`；
- 不得把 Schema / fixture 的设计验证 `PASS` 描述为 Crypto / Runtime Validation `PASS`；该层仍为 `UNVALIDATED`；
- Phase 1 仍为 `NOT AUTHORIZED`，Code Status 仍为 `NO PRODUCT CODE`。

## 6. 晋级与生效条件

1. 本记录为晋级流程的 **Commit A**，创建后不得在后续 commit 中修改。
2. **Commit B** 仅执行与本批准一致的机械状态晋级与审核/批准证据回填。
3. **Commit C** 仅将 Commit B 的真实 SHA 回填为 Effective Commit，不得扩大批准范围。
4. Commit A / B / C 必须置于独立 Draft 晋级 PR 中；该 PR 的最终精确 head 必须重新取得独立 Reviewer `PASS`。
5. 依据 `GOVERNANCE_APPROVAL_0006.md` 第 5 节，合并许可必须明确包含晋级 PR 编号与其精确 head SHA；任何“继续”“推进”等模糊措辞均不构成合并许可。
6. 在 owner 对最终晋级 PR 精确 head 再次给出书面合并许可、且该 PR 以普通 merge 进入远端 `main` 并可到达之前，本记录与相关状态晋级均为 **pending effective**。

## 7. Phase 状态声明

- 本批准**不等于 Phase 0 closure**。Phase 0 保持 `OPEN`；是否达到 `READY_FOR_CLOSURE` 必须由 `PHASE_0_EXIT_CRITERIA.md` 的完整矩阵另行计算并经独立治理流程确认。
- Phase 1 保持 `NOT AUTHORIZED`。
- Code Status 保持 `NO PRODUCT CODE`。
- 本记录不创建 closure 或 authorization 记录。

## 8. Append-only 声明

本文件为 append-only 批准记录。任何后续更正、补充或状态变化均须通过新的治理记录完成，不得修改、删除或重写本文件既有内容。
