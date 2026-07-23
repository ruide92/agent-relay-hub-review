# CONTRACTS — 机器可读契约 Schema 索引（GOV-APP-0007 已批准）

> 本目录收录 Agent Relay Hub（ARH）的机器可读契约 JSON Schema 设计文件。
> **全部 Schema 为经 `GOVERNANCE_APPROVAL_0007.md` 批准的 Phase 0 设计契约，状态 APPROVED，当前为 Source of Truth；仍未实现。**
> Reviewer-2 对 target `19fdff1beaa5208fec23653f83b47046fe8c3427` 独立审核 `PASS`；批准不等于运行时实现或 Crypto/Runtime Validation 通过。
> 语义服从已批准 V1.1、`PROTOCOL.md`、`SECURITY.md` 与 `MACHINE_READABLE_CONTRACTS.md`，不得与它们冲突。

## Schema dialect

- JSON Schema Draft 2020-12（`$schema: "https://json-schema.org/draft/2020-12/schema"`）。
- `$id` 为稳定标识符，**不是网络位置**；校验方 MUST NOT 对 `$id` 发起网络请求，MUST 仅使用本地 Schema bundle 解析。
- `$ref` 仅允许相对路径引用本目录内 Schema；禁止引用外部 URL。

## Schema 索引

| # | 文件 | 契约 | 对应规范性 prose | 状态 |
|---|---|---|---|---|
| 0 | `common-defs.schema.json` | 共享原子定义 | `MACHINE_READABLE_CONTRACTS.md` §3 | APPROVED |
| 1 | `arp-envelope.schema.json` | ARP 消息信封 | `PROTOCOL.md` §1–§4；V1.1 §11 | APPROVED |
| 2 | `arp-message.schema.json` | ARP 消息类型与载荷基线 | `PROTOCOL.md` §5、§8、§11 | APPROVED |
| 3 | `adapter-capability.schema.json` | 适配器能力上报 | `PROTOCOL.md` §6；V1.1 §10.2 | APPROVED |
| 4 | `adapter-lifecycle-event.schema.json` | 适配器生命周期事件 | `PROTOCOL.md` §7；V1.1 §20.1 | APPROVED |
| 5 | `health-check.schema.json` | 健康检查 | `PROTOCOL.md` §9；V1.1 §20.1 | APPROVED |
| 6 | `policy-bundle.schema.json` | 政策包（content only） | `SECURITY.md` §6–§8；V1.1 §4.2、§15.5 | APPROVED |
| 7 | `signature-manifest.schema.json` | 签名清单（detached sidecar） | `SECURITY.md` §8 | APPROVED |
| 8 | `capability-token-claims.schema.json` | capability token claims (decoded view) | `DECISIONS/ADR-0003-…`（ACCEPTED）；V1.1 §4.2.1 | APPROVED |
| 9 | `capability-token-protected-header.schema.json` | capability token JWS protected header (decoded view) | `DECISIONS/ADR-0003-…`（ACCEPTED） | APPROVED |
| 10 | `capability-token.schema.json` | capability token wire format (JWS compact string) | `DECISIONS/ADR-0003-…`（ACCEPTED） | APPROVED |
| 11 | `capability-token-revocation.schema.json` | capability token 撤销清单（content only） | `DECISIONS/ADR-0003-…`（ACCEPTED）；`SECURITY.md` §8 | APPROVED |

## conformance/ 目录

`conformance/` 下的 JSON fixtures 为非执行型设计验收向量，用于表达"该校验路径必须接受/拒绝"。当前 160 个 fixtures（40 valid + 120 invalid），详见 `MACHINE_READABLE_CONTRACTS.md` §10。

## 当前状态

- Phase 0：`OPEN / READY_FOR_CLOSURE`（尚未关闭）
- Phase 1：`NOT AUTHORIZED`
- Code Status：`NO PRODUCT CODE`
- Schema count：**12** 个 JSON Schema 文件
- Fixture count：**160** 个 conformance fixtures（40 valid + 120 invalid）

### 分层 Validation Status

| 层 | 状态 | 说明 |
|---|---|---|
| Static Precheck | `PASS` | JSON 语法（172 文件 = 12 schema + 160 fixture）、$id 唯一性（12 ID）、$ref 可解析性（182 ref）、duplicate-key 检测（172 文件，0 重复键）、atomic preflight（UUIDv4/hash/base64url/required） |
| Schema Meta-Validation | `PASS` | 12/12 Schema 通过 Ajv 8.20.0 Draft 2020-12 **strict mode** 编译（strict=true, strictTypes=true, strictSchema=true, strictRequired=true） |
| Fixture Execution | `PASS` | 40/40 valid fixtures 通过；120/120 invalid fixtures 被正确处理（105 schema-layer、13 business-layer、2 decoded-header） |
| Business Semantic Validation | `PASS` | 13 个确定性 business vectors：default TTL、replay retention、FAILED session reuse、两类 replay、expiry、revocation、target hash、detached-preimage metadata substitution、`nbf > iat`、`iat == exp`、tier scope→registry membership、rule scope→registry membership；已由 Schema 表达的 tier/budget/A5/loop/routing/ladder 不再冒充 business checks |
| Crypto/Runtime Validation | `UNVALIDATED` | 设计契约，未实现；签名/验签/replay/revocation 均未运行 |

> **诚实声明**：Schema Meta-Validation 与 Fixture Execution 已在隔离的仓库外验证环境（Ajv 8.20.0 + ajv-formats 3.0.1, Draft 2020-12）中运行，**未在仓库安装任何依赖**。全部 12 个 Schema 已在 Ajv **strict mode**（strict=true, strictTypes=true, strictSchema=true, strictRequired=true）下编译通过；无需关闭任何 strict 检查。以上验证结果为 executor self-check，不等于独立审核。Crypto/Runtime Validation 仍 `UNVALIDATED`。
> 两个 Phase 0 契约条件（`P0-CAPABILITY-TOKEN-SIGNING-CONTRACT`、`P0-MACHINE-READABLE-CONTRACTS`）已由 `GOVERNANCE_APPROVAL_0007.md` 批准为 **OWNER_APPROVED**。这使总体门禁进入 `READY_FOR_CLOSURE`，但 Phase 0 仍 `OPEN`，未创建 closure；Phase 1 仍 `NOT AUTHORIZED`，ADR-0003 为 `ACCEPTED`。

- 批准记录：`GOVERNANCE_APPROVAL_0007.md`；Approval commit `f5e6fabec00b40b8e7d8d7041cc03ec1dd0b1238`；Effective commit `3019d2e25405f275b9d9e0c56af86b3b65c0bbb2`
- `GOVERNANCE_APPROVAL_0005.md` 保留给第四包视觉产物，当前不存在。
