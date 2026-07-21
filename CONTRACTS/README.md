# CONTRACTS — 机器可读契约 Schema 索引（提案 #0007）

> 本目录收录 Agent Relay Hub（ARH）的机器可读契约 JSON Schema 设计文件。
> **全部 Schema 为 Phase 0 设计契约（提案 #0007），状态 PROPOSED，未实现、未运行验证、未获批准。**
> 经独立审核与 owner 批准（预期 `GOVERNANCE_APPROVAL_0007`，本轮未创建）后方可成为 Source of Truth。
> 语义服从已批准 V1.1、`PROTOCOL.md`、`SECURITY.md` 与 `MACHINE_READABLE_CONTRACTS.md`，不得与它们冲突。

## Schema dialect

- JSON Schema Draft 2020-12（`$schema: "https://json-schema.org/draft/2020-12/schema"`）。
- `$id` 为稳定标识符，**不是网络位置**；校验方 MUST NOT 对 `$id` 发起网络请求，MUST 仅使用本地 Schema bundle 解析。
- `$ref` 仅允许相对路径引用本目录内 Schema；禁止引用外部 URL。

## Schema 索引

| # | 文件 | 契约 | 对应规范性 prose | 状态 |
|---|---|---|---|---|
| 0 | `common-defs.schema.json` | 共享原子定义 | `MACHINE_READABLE_CONTRACTS.md` §3 | PROPOSED |
| 1 | `arp-envelope.schema.json` | ARP 消息信封 | `PROTOCOL.md` §1–§4；V1.1 §11 | PROPOSED |
| 2 | `arp-message.schema.json` | ARP 消息类型与载荷基线 | `PROTOCOL.md` §5、§8、§11 | PROPOSED |
| 3 | `adapter-capability.schema.json` | 适配器能力上报 | `PROTOCOL.md` §6；V1.1 §10.2 | PROPOSED |
| 4 | `adapter-lifecycle-event.schema.json` | 适配器生命周期事件 | `PROTOCOL.md` §7；V1.1 §20.1 | PROPOSED |
| 5 | `health-check.schema.json` | 健康检查 | `PROTOCOL.md` §9；V1.1 §20.1 | PROPOSED |
| 6 | `policy-bundle.schema.json` | 政策包 | `SECURITY.md` §6–§8；V1.1 §4.2、§15.5 | PROPOSED |
| 7 | `signature-manifest.schema.json` | 签名清单（detached） | `SECURITY.md` §8 | PROPOSED |
| 8 | `capability-token-claims.schema.json` | capability token claims | `DECISIONS/ADR-0003-…`（PROPOSED）；V1.1 §4.2.1 | PROPOSED |
| 9 | `capability-token.schema.json` | capability token wire format (JWS) | `DECISIONS/ADR-0003-…`（PROPOSED） | PROPOSED |
| 10 | `capability-token-revocation.schema.json` | capability token 撤销清单 | `DECISIONS/ADR-0003-…`（PROPOSED）；`SECURITY.md` §8 | PROPOSED |

## conformance/ 目录

`conformance/` 下的 JSON fixtures 为非执行型设计验收向量，用于表达"该校验路径必须接受/拒绝"。当前全部未执行（无实现）。详见 `MACHINE_READABLE_CONTRACTS.md` §10。

## 当前状态

- Phase 0：`OPEN / NOT_READY`
- Phase 1：`NOT AUTHORIZED`
- Code Status：`NO PRODUCT CODE`
- Validation Status：`UNVALIDATED`
- 预期批准编号：`GOVERNANCE_APPROVAL_0007`（**本轮未创建**）
- `GOVERNANCE_APPROVAL_0005.md` 保留给第四包视觉产物，当前不存在。