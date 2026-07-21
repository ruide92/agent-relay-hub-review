```text
Document Status: PROPOSED
Normative Status: 未批准——本文件为 Phase 0 治理提案 #0007 的设计契约草案，经独立审核与 owner 批准（预期 GOVERNANCE_APPROVAL_0007）后方成为 Source of Truth
Product Baseline: AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md
Product Baseline Commit: c9cc522b4ab01008fed390c282d6bd5a816ee779
Proposal PR: PENDING_PROPOSAL
Approval Record: 无（未创建；预期未来 GOVERNANCE_APPROVAL_0007，本轮未创建）
Current Phase: Phase 0
Phase 1: NOT AUTHORIZED
Code Status: NO PRODUCT CODE
Schema Static Check Status: PASS（JSON 语法、$id 唯一性、$ref 目标存在）
Schema Meta-Validation Status: SCHEMA_META_VALIDATION_NOT_RUN（无预装 Draft 2020-12 validator）
Crypto/Runtime Validation Status: UNVALIDATED（设计契约，未实现）
```

# MACHINE_READABLE_CONTRACTS.md（机器可读契约规范性总说明，提案 #0007）

> 本文件是 Agent Relay Hub（ARH）全部机器可读契约（JSON Schema bundle）的**规范性总说明**。
> 本文件与其登记的 `CONTRACTS/` Schema 均为 **Phase 0 设计契约（提案 #0007）**：**未实现、未运行验证、未获批准**。
> 本提案不授权 Phase 1，不授权编写产品代码；只有经独立审核与 owner 批准（预期 `GOVERNANCE_APPROVAL_0007`，本轮未创建）后，本文件才成为 Source of Truth。
> 语义服从已批准 V1.1（`AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md`）、`PROTOCOL.md` 与 `SECURITY.md`，不得与它们冲突。

---

## 1. 范围与契约清单

本契约包覆盖以下机器可读契约（Schema 位于 `CONTRACTS/`，索引见 `CONTRACTS/README.md`）：

| 契约 | Schema | 对应规范性 prose |
|---|---|---|
| ARP 消息信封 | `CONTRACTS/arp-envelope.schema.json` | `PROTOCOL.md` §1–§4；V1.1 §11 |
| ARP 消息类型与载荷基线 | `CONTRACTS/arp-message.schema.json` | `PROTOCOL.md` §5、§8、§11；V1.1 §11 |
| 适配器能力上报 | `CONTRACTS/adapter-capability.schema.json` | `PROTOCOL.md` §6；V1.1 §10.2 |
| 适配器生命周期事件 | `CONTRACTS/adapter-lifecycle-event.schema.json` | `PROTOCOL.md` §7；V1.1 §20.1 |
| 健康检查 | `CONTRACTS/health-check.schema.json` | `PROTOCOL.md` §9；V1.1 §20.1 |
| 政策包 | `CONTRACTS/policy-bundle.schema.json` | `SECURITY.md` §6–§8；V1.1 §4.2、§15.5 |
| 签名清单（detached） | `CONTRACTS/signature-manifest.schema.json` | `SECURITY.md` §8 |
| capability token claims | `CONTRACTS/capability-token-claims.schema.json` | `DECISIONS/ADR-0003-…`（PROPOSED）；V1.1 §4.2.1 |
| capability token wire format | `CONTRACTS/capability-token.schema.json` | `DECISIONS/ADR-0003-…`（PROPOSED） |
| capability token 撤销清单 | `CONTRACTS/capability-token-revocation.schema.json` | `DECISIONS/ADR-0003-…`（PROPOSED）；`SECURITY.md` §8 |
| 共享定义 | `CONTRACTS/common-defs.schema.json` | 本文件 §3 |

明确不包含：任何产品代码、运行时配置、真实密钥、真实 token、真实签名、数据库、CI、依赖清单。

---

## 2. JSON Schema dialect、版本规则与执行模型

### 2.1 Schema 文件层（meta-validation）

加载 Schema bundle 时执行以下校验，失败则拒绝加载整个 bundle：

1. **dialect**：每个 Schema 文件 MUST 自身声明 `$schema: "https://json-schema.org/draft/2020-12/schema"`。缺失、不可识别或非 Draft 2020-12 时拒绝。
2. **`$id`**：每个 Schema 全局唯一（见 §3）。
3. **`$ref`**：所有 `$ref` 目标 MUST 在本地 bundle 中可解析（不允许外部 URL）。
4. **format-assertion**：validator MUST 启用 Draft 2020-12 format-assertion；若 validator 实现不支持 format-assertion，则安全关键字段（时间戳、UUID、哈希）MUST 额外使用显式 `pattern` 以保证 fail-closed。
5. **Schema 文件自身不包含 `$schema` 以外的实例字段**：`$schema` 是 Schema 文件自身的 dialect 声明，**不是**业务实例应携带的字段。

### 2.2 实例层（instance validation）

验证业务实例时执行以下校验：

1. **实例不携带 `$schema`**：业务实例 MUST NOT 包含 `$schema` 字段；实例的 dialect 由校验方选定的 Schema 决定，不由实例自行声明。
2. **schema_version**：实例 MUST 携带 `schema_version`；v1 Schema 中 `schema_version` 为 `const: "1.0.0"`（不接受其他值）。未来 v2 将使用不同 const 值。版本不支持时 MUST 拒绝（fail-closed）。
3. **protocol_version**：v1 中为 `const: 1`。
4. **兼容性规则**：
   - 同 `MAJOR` 内：仅允许新增**可选**字段、扩展枚举须同步注册表（见 §5）与 prose；
   - 删除字段、收窄类型或修改语义为 breaking，MUST 升 `MAJOR`（`/v2/`），经治理提案批准；
   - 实例 `schema_version` 不在支持集合内时 MUST 拒绝，不得按最近版本猜测解析。

---

## 3. `$id`、`$schema` 与共享定义

1. **`$id` 规则**：每个 Schema 具有稳定、全局唯一的 `$id`，形如 `https://agent-relay-hub.dev/contracts/v1/<name>.schema.json`。`$id` 是**稳定标识符而非网络位置**：校验方 MUST NOT 对 `$id` 发起网络请求；解析 MUST 仅使用本地 Schema bundle。
2. **共享定义**：跨 Schema 复用的原子定义集中于 `CONTRACTS/common-defs.schema.json`（标识符、RFC3339 时间戳、SHA-256 哈希字符串、`key_id`、`jti`、scope 格式、namespaced extensions 容器等）。其他 Schema MUST 以 `$ref` 引用共享定义，MUST NOT 复制定义造成漂移。
3. **`$ref` 规则**：仅允许相对路径引用本仓库 `CONTRACTS/` 内的 Schema 文件；MUST NOT 引用外部 URL；`$ref` 目标 MUST 存在且 `$id` 全 bundle 唯一。

---

## 4. 兼容性与未知字段处理

1. **Envelope 层（ARP 消息信封）**：与 `PROTOCOL.md` §3 一致——接收方 MUST 保留未知字段用于透传/审计，MUST NOT 因未知字段拒绝合法消息。因此 `arp-envelope.schema.json` 在信封顶层允许额外字段；**所有实际存在的字段（含未知字段）MUST 纳入 canonical hash**（见 §6 与 `PROTOCOL.md` §4.3），防止未签名字段被篡改。
2. **安全关键对象**（capability token claims、signature manifest、policy bundle、revocation list、adapter capability）采用 **fail-closed**：`additionalProperties: false`，未命名字段 MUST 被拒绝。
3. **扩展机制**：安全关键对象的合法扩展 MUST 放入明确的 **namespaced `extensions` 容器**（对象键 MUST 为 dotted namespace，如 `arh.experimental`、`vendor.example`）；MUST NOT 在安全关键对象顶层新增未命名模糊权限字段。
4. **安全关键枚举 MUST 使用封闭枚举**，MUST NOT 用自由文本替代（如 `key_purpose`、`alg`、`delivery_semantics`、生命周期状态、撤销原因）。

---

## 5. 注册表（prose 层封闭集合）

以下集合在本契约包中以封闭枚举或 prose 注册表固定；新增取值 MUST 经治理提案并同步 Schema：

- **audience 注册表**（capability token `aud` 允许值）：`arh:core`、`arh:adapter-host`。不在注册表内的 audience MUST 被拒绝。
- **scope 注册表**（capability token `scope` 允许动作）：`workflow:read`、`workflow:execute`、`artifact:read`、`artifact:write`、`message:send`、`message:receive`、`review:request`、`plan:generate`、`git:commit`、`git:push`、`pr:create`、`deploy:staging`、`adapter:health_check`。scope 格式 MUST 匹配 `<domain>:<action>`；未登记动作 MUST 被拒绝。
- **授权档位与 scope 映射**（V1.1 §4.2）：`A0`→读取类；`A1`→`review:request`、`plan:generate`；`A2`→`workflow:execute`；`A3`→`git:commit`、`git:push`、`pr:create`；`A4`→`deploy:staging`；`A5` **MUST NOT** 由 capability token 授予（V1.1 §4.2：A5 默认禁止）。token 的 `max_authorization_tier` 与 `scope` 必须一致；超权 MUST 被拒绝。
- **issuer 注册表**：`arh:policy-engine`（capability token 唯一合法签发者）。

---

## 6. Canonical JSON 与 hashing 规则

1. **Canonical JSON**：所有需要哈希或签名的 JSON 值 MUST 先规范化为 **RFC 8785（JSON Canonicalization Scheme, JCS）** 形式：UTF-8、对象键按码点排序、无冗余空白、数字最短精确形式。
2. **哈希算法**：SHA-256；哈希字符串统一 `sha256:<64 位小写 hex>` 格式（共享定义 `common-defs`）。
3. **Envelope 哈希预映像**：与 `PROTOCOL.md` §4.3 一致——`hash_preimage = canonical_json(完整消息排除 integrity.content_hash 与 integrity.signature)`；`hash_algorithm` 与实际存在的其他 integrity 元数据 MUST 纳入预映像；`content_hash` MUST NOT 参与其自身计算。
4. **Detached 对象哈希**：policy bundle / SBOM / revocation list 的 `target_hash` = 其 canonical JSON 字节的 SHA-256；一致性校验 MUST 重算比对，不一致 MUST 拒绝。
5. **JWS 签名预映像**：capability token 的签名输入为 `ASCII(BASE64URL(protected header)) + "." + ASCII(BASE64URL(payload))`（RFC 7515）；payload 由签发方按 JCS canonical 序列化，验证方对原始字节验证、MUST NOT 重新序列化。token 层与 envelope 层的 canonicalization 各自独立、不得混用（见 ADR-0003）。

---

## 7. 跨 Schema 不变量

以下不变量跨越多个 Schema，属于业务校验层（JSON Schema 结构校验之外），校验方 MUST 强制执行；任一违反 MUST 拒绝：

1. **签名四元组共现**：`integrity.signature_algorithm`、`integrity.key_id`、`integrity.key_purpose`、`integrity.signature` 要么全部存在要么全部省略（`PROTOCOL.md` §4.3）；不得伪造空签名。
2. **key_purpose / target_type 匹配**：detached signature manifest 的每个签名条目 `key_purpose` MUST 与 `target_type` 一致（`policy_bundle`↔`policy_bundle`、`sbom`↔`sbom`、`capability_token_revocation_list`↔`capability_token_revocation`）；capability token MUST NOT 使用 detached manifest 作为签名容器（其容器为 JWS，见 ADR-0003），MUST NOT 与 policy bundle / SBOM 共用模糊 key purpose。
3. **`alg=none` 禁止**：任何签名容器不得声明 `alg=none` 或无算法；capability token 的 `alg` 仅允许 `ES256`。
4. **时间一致性**：token `nbf ≤ iat < exp`；`exp − iat` MUST ≤ 最大 TTL（候选默认 15 分钟，政策可收紧）；`expires_at`（envelope）晚于 `created_at`；clock skew 候选容忍 60 秒（ADR-0003）。
5. **生命周期合法迁移**：迁移对 MUST 在 `PROTOCOL.md` §7 白名单内；`FAILED` / `STOPPED` 为（实例）终态；`FAILED` 实例的 `session_id` MUST NOT 复用，恢复 MUST 产生新 `session_id` 并从 `LOADING` 开始。
6. **撤销优先**：`jti` 出现在有效撤销清单中的 token MUST 被拒绝，即使签名与时间有效；撤销清单 `list_version` MUST 单调递增，低于已见版本的清单 MUST 被拒绝（防回滚）。
7. **防重放**：`jti` 在 replay cache（缓存至 `exp + clock skew`）中已见的 token MUST 被拒绝。
8. **三层授权一致性**：token `scope`、`max_authorization_tier`、绑定对象（`workflow_id` / `job_id` / `adapter_id` / `session_id` / `policy_bundle_hash` / `policy_decision_id`）与实际调用上下文 MUST 一致；最终授权上限 = min(自报能力, 政策上限, 令牌权限)（V1.1 §4.2.1）。
9. **禁止字段**：适配器能力上报 MUST NOT 含 `supports_exactly_once_claim`（V1.1 §10.2）；任何对象 MUST NOT 含明文密钥、真实 token 值（`PROTOCOL.md` §12）。
10. **capability token 泄漏防护**：完整 token MUST NOT 出现在 ARP envelope payload、Event Ledger、日志或截图中；引用 MUST 仅含 `jti` 与 token 哈希（见 `PROTOCOL.md` §13 与 `SECURITY.md` §8.1）。

---

## 8. Fail-closed 校验顺序

校验方 MUST 按以下顺序校验，**任一失败即整体拒绝并按 `PROTOCOL.md` §12.1 记录脱敏审计事件**，MUST NOT 跳过、猜测或部分接受：

1. **语法层**：UTF-8 JSON 解析；失败 → 拒绝（`validation`）。
2. **dialect 层**：`$schema` 存在且为 Draft 2020-12；失败 → 拒绝。
3. **版本层**：`schema_version` 在支持集合内；`protocol_version` 协商区间重叠（PROTOCOL §1）；失败 → 拒绝（`protocol_incompatible`）。
4. **结构层**：Schema 校验（必填、类型、封闭枚举、条件必填、`additionalProperties`）；失败 → 拒绝（`validation`）。
5. **完整性层**：canonical hash 重算比对（envelope `content_hash`、detached `target_hash`）；失败 → 拒绝。
6. **签名层**：签名容器形式检查（JWS 三段 / detached manifest）、`alg` 白名单、签名验证、`kid` 在可信钥集且未撤销；失败 → 拒绝（`authentication`）。
7. **时间与注册表层**：`iat/nbf/exp/TTL`、clock skew、issuer/audience/scope 注册表；失败 → 拒绝（`authorization`）。
8. **状态层**：撤销清单、replay cache、生命周期迁移白名单、session 复用禁止；失败 → 拒绝。
9. **授权上下文层**：绑定对象与政策裁决一致性（`policy_bundle_hash` / `policy_decision_id` / 三层授权上限）；失败 → 拒绝（`authorization` / `policy_denied`）。

---

## 9. 规范文本与 Schema 冲突裁决

冲突裁决顺序（高 → 低）：

1. 已批准 V1.1（`AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md`）
2. 已批准规范性 prose：`PROTOCOL.md`、`SECURITY.md`、`ARCHITECTURE.md`
3. 本文件（`MACHINE_READABLE_CONTRACTS.md`，经批准后）
4. `CONTRACTS/` 内各 JSON Schema
5. `CONTRACTS/conformance/` fixtures（仅设计验收向量，不构成规范）

- 任何下层与上层冲突时 MUST 以上层为准并修正下层；冲突涉及权限、安全、协议、阶段边界时 MUST 走独立 ADR 审核（`SOURCE_OF_TRUTH.md` §9、§10）。
- 实现/校验方 MUST NOT 自行选择有利解释；发现冲突 MUST 停止并按治理流程上报。

---

## 10. Fixture / conformance 证据规则

1. `CONTRACTS/conformance/` 下的 fixtures 为**非执行型设计验收向量**：仅用于表达"该校验路径必须接受/拒绝"，供未来实现期 conformance 测试引用。
2. 每个 fixture 为单个 JSON 文件，外壳字段：`fixture_id`、`target_schema`、`expected`（`valid` / `invalid`）、`expected_violation`（invalid 时必填：`layer` = `schema` / `business`，`rule` 描述命中的规范条款）、`note`、`instance`（被校验对象）。
3. fixtures **MUST NOT** 包含真实密钥、测试私钥、真实 token 或真实有效签名；哈希与签名字段为结构示例（如 `sha256:` + 模式化 hex），`note` MUST 标注“结构示例，非密码学有效证据”。
4. `expected=invalid` 的 fixture MUST 能被第 8 节校验顺序中的**恰好一层**确定性捕获；fixture 的 `expected_violation.layer` 标明该层（`schema` = 第 4 层结构校验；`business` = 第 5–9 层不变量校验）。
5. fixtures 不替代独立审核与运行时测试；当前全部**未执行**（无实现），仅作设计验收登记。

---

## 11. 当前状态

- 本契约包为 **PROPOSED** 设计契约；未实现、未运行验证（`Validation Status: UNVALIDATED`）。
- 本契约包的批准预期走 `GOVERNANCE_APPROVAL_0007`（**本轮未创建**）；`GOVERNANCE_APPROVAL_0005.md` 保留给第四包视觉产物，当前不存在。
- Phase 0 仍 `OPEN / NOT_READY`；Phase 1 仍 `NOT AUTHORIZED`；`NO PRODUCT CODE`。
