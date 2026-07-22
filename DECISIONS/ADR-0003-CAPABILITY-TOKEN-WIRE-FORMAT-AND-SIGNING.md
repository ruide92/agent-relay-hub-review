# ADR-0003 — Capability Token Wire Format and Signing Contract

Status: PROPOSED
ADR Number: ADR-0003
Title: Capability Token 线格式与签名契约（Phase 0 设计关闭前必须形成的规范性契约）
Date: 2026-07-22
Proposed By: Phase 0 design lead / external product architect
Implemented By: executor（WorkBuddy）

> 本 ADR 为 **PROPOSED**，未经独立审核与 owner 批准，Status 不得标记 ACCEPTED。
> 本 ADR 只定义设计；**不代表运行时已实现**。
> 本 ADR 解决 `PHASE_0_EXIT_CRITERIA.md` 中 `P0-CAPABILITY-TOKEN-SIGNING-CONTRACT` blocker。
> 预期批准编号：`GOVERNANCE_APPROVAL_0007`（**本轮未创建**）。
> `GOVERNANCE_APPROVAL_0005.md` 保留给第四包视觉产物，当前不存在。

---

## 1. 背景（Context）

V1.1 §4.2.1 定义了三层授权模型，其中第三层为能力令牌（capability token）：携带本次任务实际最小权限，由政策引擎在调度时签发。`SECURITY.md` §8 规定了 ECDSA P-256 / SHA-256 签名基线与 detached signature manifest（用于 policy bundle / SBOM），但**未定义** capability token 的 wire format、签名容器、必填 claims、签名预映像、防重放与撤销语义。

`ADR-0001` §2.3 已明确：capability token 的规范化 payload、签名预映像、签名容器、必填 claims、issuer、audience、issued_at / expires_at、nonce / jti、防重放、撤销语义及独立 `key_purpose` **必须在 Phase 0 关闭前形成规范性契约**；不得将此义务推迟到 Phase 1。`GOVERNANCE_APPROVAL_0006.md` §4 对此作出了限定纠正。

本 ADR 即为形成该规范性契约的设计提案。

## 2. 决策（Decision）

### 2.1 Wire Format 方案裁决

比较三种方案：

| 方案 | 描述 | 优点 | 缺点 |
|---|---|---|---|
| **JWS Compact Serialization** | RFC 7515 标准 JSON Web Signature 三段式：`BASE64URL(header).BASE64URL(payload).BASE64URL(signature)` | 标准化、广泛实现、库生态成熟、protected header 内嵌 alg/kid/key_purpose | 需 BASE64URL 编码；header/payload 须序列化为 JSON |
| COSE_Sign1 | RFC 8152 CBOR Object Signing and Encryption | 紧凑二进制、IoT 友好 | ARH 技术栈为 JSON/.NET；引入 CBOR 增加解析复杂度；生态较小 |
| 自定义 signed JSON | 自定义 JSON + 原始签名值 + 手工头 | 完全控制 | 非标准、无库支持、安全审计成本高、易遗漏边界 |

**裁决**：采用 **JWS Compact Serialization（RFC 7515）**，签名算法 **ES256**（ECDSA P-256 / SHA-256）。

理由：
1. ARH 技术栈为 .NET 10 LTS + JSON（`ARCHITECTURE.md` §9），JWS 有成熟 .NET 实现；
2. 与 `SECURITY.md` §8 签名基线一致（ECDSA P-256 / SHA-256）；
3. COSE_Sign1 引入 CBOR 增加复杂度且对 JSON 栈无增益；
4. 自定义方案安全审计成本高且无标准库支持。

### 2.2 Protected Header

```json
{
  "alg": "ES256",
  "typ": "arh-cap+jwt",
  "kid": "<key identifier>",
  "key_purpose": "capability_token"
}
```

- `alg` 固定为 `ES256`（ECDSA P-256 / SHA-256，`SECURITY.md` §8）。
- `alg=none` **绝对禁止**。
- `typ` 固定为 `arh-cap+jwt`（项目专用类型，非通用 JWT）。
- `key_purpose` 固定为 `capability_token`，**MUST NOT** 与 `policy_bundle` 或 `sbom` 共用（`SECURITY.md` §8 用途分离）。
- `kid` 标识验证密钥，MUST 在可信钥集中存在且未撤销（`SECURITY.md` §8 revocation list）。
- Protected header 不允许额外字段（`additionalProperties: false`，见 `capability-token-protected-header.schema.json`）。

### 2.3 Payload Claims

Token claims payload MUST 包含以下全部字段（见 `capability-token-claims.schema.json`）：

| Claim | 必填 | 描述 |
|---|---|---|
| `iss` | ✓ | Issuer，固定 `arh:policy-engine`（唯一合法签发者） |
| `aud` | ✓ | Audience，注册表 `arh:core` / `arh:adapter-host`（`MACHINE_READABLE_CONTRACTS.md` §5） |
| `sub` | ✓ | Subject，适配器或角色身份 |
| `iat` | ✓ | Issued At，RFC 7519 NumericDate（整数秒） |
| `nbf` | ✓ | Not Before，RFC 7519 NumericDate；MUST ≤ `iat` |
| `exp` | ✓ | Expiration Time，RFC 7519 NumericDate；`exp − iat` MUST ≤ 最大 TTL |
| `jti` | ✓ | Token ID，UUIDv4，全局唯一 |
| `scope` | ✓ | 最小权限 scope 数组，每项 `<domain>:<action>`，须在 scope 注册表内（§5） |
| `max_authorization_tier` | ✓ | 最高授权档位，仅允许 `A0`–`A4`；`A5` **MUST NOT** 由 token 授予（V1.1 §4.2） |
| `workflow_id` | ✓ | 绑定的工作流 ID |
| `job_id` | ✓ | 绑定的作业 ID |
| `adapter_id` | ✓ | 绑定的适配器 ID |
| `session_id` | ✓ | 绑定的会话 ID |
| `policy_bundle_hash` | ✓ | 授权此 token 的 policy bundle 的 SHA-256 哈希 |
| `policy_decision_id` | ✓ | 政策引擎决策记录 ID |
| `nonce` | 可选 | 挑战 nonce，用于请求绑定 |
| `extensions` | 可选 | Namespaced extensions 容器；**MUST NOT** 在顶层添加未命名的模糊权限字段 |

### 2.4 Payload Canonicalization

- Payload MUST 先按 **RFC 8785 JCS（JSON Canonicalization Scheme）** 规范化：UTF-8、键按码点排序、无冗余空白、数字最短精确形式。
- **签名方**按 JCS 序列化 payload 后 BASE64URL 编码。
- **验证方 MUST 对原始 payload 字节验证签名**，MUST NOT 重新序列化（防止规范化歧义攻击）。
- Token 层的 canonicalization 与 envelope 层（`PROTOCOL.md` §4.3）**各自独立**，不得混用（envelope 层使用排除 content_hash/signature 的 canonical JSON，token 层使用 JCS + JWS）。

### 2.5 Signing Preimage

```
signing_input = ASCII(BASE64URL(protected_header)) + "." + ASCII(BASE64URL(payload))
```

- 签名输入为 JWS Compact Serialization 的前两段点号连接（RFC 7515 §5.2）。
- 签名算法 ES256：ECDSA P-256 over SHA-256，签名值为 **raw R‖S** 按 JWS 规则 BASE64URL 编码（RFC 7518 §3.1）。
- `signature` MUST NOT 为空、零值或占位符。

### 2.6 TTL 约束

- 候选默认最大 TTL：**15 分钟**（`exp − iat ≤ 900 秒`）。
- Policy bundle MUST 可以收紧该限制但 MUST NOT 放宽超过 **1 小时**（绝对上限）。
- `nbf ≤ iat < exp` MUST 满足，否则 fail-closed。

### 2.7 Replay Protection（Durable Persistence）

- 已接受的 `jti` replay 状态 MUST 写入 **Event Ledger / SQLite**（持久化），**不**仅使用进程内缓存。
- 保留期至少覆盖 `exp + clock_skew`；保留期内的 `jti` MUST 在服务恢复后仍然被拒绝。
- 服务恢复时，在 replay 状态从 Event Ledger 重建完成前，验证方 **MUST fail-closed**：不接受任何 token。
- `jti` 已出现在 replay 状态中的 token MUST 被拒绝，即使签名与时间有效。
- 进程重启后重复的 token 仍 MUST 被拒绝（conformance/acceptance vector: `fixture-invalid-replay-after-restart`）。

### 2.8 Revocation

- 撤销清单格式见 `capability-token-revocation.schema.json`（content only，不内嵌 `target_hash` 或 `signature_manifest`）。
- `jti` 出现在有效撤销清单中的 token MUST 被拒绝，即使签名与时间有效。
- 撤销清单 `list_version` MUST 单调递增；低于已见版本的清单 MUST 被拒绝（防回滚攻击）。
- 撤销清单通过**独立 sidecar signature manifest** 签名（`key_purpose=capability_token_revocation`，见 `signature-manifest.schema.json`）；`target_hash` = 撤销清单内容的 JCS canonical bytes 的 SHA-256（无自引用排除规则）。
- 撤销原因枚举：`key_compromised` / `token_abused` / `policy_change` / `session_terminated` / `manual_revoke`。

### 2.9 Key Rotation

- 依据 `SECURITY.md` §8：key rotation 须有重叠验证窗口（old+new 同时可信直至窗口结束）。
- `kid` 标识具体密钥版本；验证方 MUST 只信任未撤销的 `kid`。
- 新密钥签发的 token 与旧密钥签发的 token 在重叠窗口内同时有效；窗口结束后旧密钥签发的 token 被拒绝。
- 旧密钥撤销后 MUST NOT 继续签发。

### 2.10 Clock Skew

- 候选默认 clock skew 容忍：**60 秒**。
- Token 被接受当且仅当 **`nbf - clock_skew <= now <= exp + clock_skew`**：
  - `nbf` 最多可在当前时间**之后** 60 秒（token 尚未生效但在容差范围内被接受）；
  - `exp` 最多可在当前时间**之前** 60 秒（token 刚刚过期但在容差范围内被接受）。
- Policy bundle MUST 可收紧但 MUST NOT 放宽超过 **300 秒**（绝对上限）。

### 2.11 绑定语义

- Token 绑定 `workflow_id` / `job_id` / `adapter_id` / `session_id`：实际调用的这四个值 MUST 与 token claims 中的值一致，否则 fail-closed（`authorization`）。
- Token 绑定 `policy_bundle_hash`：验证方 MUST 核对当前加载的 policy bundle 哈希与 token claim 中的 `policy_bundle_hash` 一致；不一致表示 policy bundle 已轮换但 token 未重签，MUST 拒绝。
- Token 绑定 `policy_decision_id`：audit 追溯用，MUST 存在于政策引擎决策记录中。

### 2.12 校验顺序

验证方 MUST 按以下顺序校验 token，**任一失败即整体拒绝**：

1. **形式检查**：JWS 三段式（signature 段精确 86 base64url 字符）、BASE64URL 可解码；失败 → `validation`。
2. **Protected header 检查**：解码 protected header 段，校验 `capability-token-protected-header.schema.json`：`alg=ES256`、`typ=arh-cap+jwt`、`kid` 存在、`key_purpose=capability_token`；失败 → `validation`。
3. **签名验证**：`kid` 在可信钥集且未撤销；ECDSA P-256/SHA-256 验签；失败 → `authentication`。
4. **Claims 结构校验**：解码 payload 段，执行 `capability-token-claims.schema.json` 校验（含 tier/scope 级联规则）；失败 → `validation`。
5. **时间校验**：先强制 claims 内部顺序 `nbf ≤ iat < exp`（严格禁止 `iat == exp` 的零生命周期 token），再校验 `nbf ≤ now+skew`、`now−skew ≤ exp`、`exp−iat ≤ max_TTL`；任一失败 → `authorization`（时间顺序错误 / 过期 / 未生效）。clock skew 只用于与 `now` 的比较，MUST NOT 放宽 `nbf ≤ iat < exp`。
6. **注册表校验**：`iss=arh:policy-engine`、`aud` 在 audience 注册表、`scope` 每项在 scope 注册表、`max_authorization_tier ≠ A5`；失败 → `authorization`。
7. **绑定校验**：`workflow_id/job_id/adapter_id/session_id` 与上下文一致、`policy_bundle_hash` 与当前 bundle 一致；失败 → `authorization`。
8. **撤销检查**：`jti` 不在最新有效撤销清单中；失败 → `authorization`（已撤销）。
9. **防重放检查**：`jti` 不在持久化 replay 状态（Event Ledger / SQLite）中；通过后写入持久化 replay 状态（TTL=`exp+skew`）；失败 → `authorization`（重放）。

### 2.13 失败路径

- 所有失败路径 MUST **fail-closed**：拒绝 token 并按 `PROTOCOL.md` §12.1 记录脱敏审计事件。
- 日志/审计 MUST NOT 记录完整 token 字符串或敏感 payload（`scope` 可记录，token 原文 MUST NOT）。
- 审计记录 MUST 只含 `jti`（如安全）、`kid`、拒绝原因、时间戳、`policy_decision_id`（如可得）。

### 2.14 Token 在 ARP 中的携带位置

- Capability token MUST 通过独立的安全通道或受控信令传递，**MUST NOT** 作为 ARP envelope `payload` 的普通字段。
- 在 ARP envelope 中引用 token 时，MUST 仅含 `jti` 与 token 哈希（`sha256:<hex>`），不得包含完整 token 字符串。
- **MUST NOT** 将完整 token 写入 Event Ledger、日志或截图。
- 详见 `PROTOCOL.md` §13（新增）与 `SECURITY.md` §8.1（新增）。

## 3. 替代方案（Alternatives）

- **COSE_Sign1 (RFC 8152)**：被否决。ARH 技术栈为 JSON/.NET，引入 CBOR 增加复杂度且无增益（见 §2.1）。
- **自定义 signed JSON**：被否决。非标准、无库支持、安全审计成本高、易遗漏边界（见 §2.1）。
- **将 token 签名混入 envelope integrity**：被否决。Envelope integrity 使用 inline `integrity.signature`（PROTOCOL §4.3），与 token 的独立签名容器混用会模糊用途边界。Token 需要独立的 wire format（JWS）以实现跨系统验证与标准库互操作。Detached signature manifest（sidecar）用于 policy bundle / SBOM / revocation list，不用于 envelope 或 capability token。

## 4. 后果（Consequences）

- **正面**：标准化 wire format 有成熟库支持；独立 key_purpose 与签名容器避免权限混淆；JCS canonicalization 防止签名歧义；fail-closed 校验链杜绝宽放行。
- **负面 / 风险**：JWS 要求 BASE64URL 编解码（少量开销）；replay 状态持久化增加 Event Ledger 写入负载（缓解：仅接受状态需写入，拒绝状态不需写入）。
- **后续义务**：运行时实现与验证属 Phase 1（经单独授权后）。

## 5. 证据（Evidence）

- V1.1 §4.2.1（三层授权模型、capability token）。
- `SECURITY.md` §8（ECDSA P-256 / SHA-256 签名基线、key rotation、revocation list、用途分离）。
- `ADR-0001` §2.3（capability token 契约必须在 Phase 0 关闭前形成）。
- `GOVERNANCE_APPROVAL_0006.md` §4（限定纠正：不得推迟到 Phase 1）。
- `MACHINE_READABLE_CONTRACTS.md` §5（注册表）、§6（canonicalization）、§7（不变量）、§8（校验顺序）。
- `CONTRACTS/capability-token.schema.json` / `capability-token-claims.schema.json` / `capability-token-revocation.schema.json`。

## 6. 批准字段

- **批准 commit（Approval commit）**：—（未批准）
- **Decision Owner**：owner
- **Reviewer（独立审核者）**：—（未审核；不得为本提案的 executor 本人）
- **Governance Approver（治理批准者）**：—（未批准）
- **Approval Record**：—（预期 `GOVERNANCE_APPROVAL_0007`，本轮未创建）
- **Corrective Record（纠正记录）**：—
- **Effective Commit（生效 commit）**：—
- **Affected Normative Documents**：`PROTOCOL.md`（§13 新增 token 携带位置）、`SECURITY.md`（§8.1 新增 token 签名容器）、`ARCHITECTURE.md`（Schema bundle 模块边界）、`MACHINE_READABLE_CONTRACTS.md`、`CONTRACTS/` 内 capability-token 相关 Schema
- **Supersedes / Superseded By**：无 / 无

## 7. 状态规则声明

没有有效 `Approval Record` 时，本 ADR 状态只能为 `PROPOSED`，不得标记 `ACCEPTED`（见 `SOURCE_OF_TRUTH.md` 第 10 节与 `DECISIONS/README.md` 第 3 节）。本 ADR 不得修改 V1.1 正文。本 ADR 仅为设计契约，不代表运行时已实现。
