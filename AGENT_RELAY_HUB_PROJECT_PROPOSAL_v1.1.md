# Agent Relay Hub（智能体中继台）项目书 V1.1

> **文档状态**：V1.1 审核候选稿（尚未成为 Source of Truth）
> **基线**：V1.0 blob `31b7287764e4eaf008b0f87852596e33bcfb4af3`
> **修订依据**：冻结 changeset commit `3e9a0598aba2e8963c8a7ea8ac9b3dbfe271ff76`（`V1.1_CHANGESET_FINAL.md`，经 Reviewer-2 冻结验收）
> **当前阶段**：仍处于 **Phase 0 产品设计冻结**，未进入 Phase 1
> **一致性审核**：本文档尚须经过独立一致性审核（对照 V1.0 与冻结 changeset）后才能成为 Source of Truth
> **代码状态**：未编写或授权实现任何产品代码；本文档仅为设计候选稿
> 项目代号：ARH ｜ 产品定位：供应商无关、可插拔、可审计的智能体中继与自主工作流平台

---

## 0. V1.1 修订摘要（相对 V1.0）

本版仅落实冻结 changeset 的修订，不引入任何 changeset 之外的设计变更。核心变化：

1. **阶段体系重构**：Phase 1 仅 Core Prototype（内核 + 模拟，无任何真实适配器）；Phase 2 仅一次性、隔离、不可直接复用为生产组件的"真实工具可行性探针"；Phase 3 才实现正式适配器与首个真实闭环；Phase 4 为多审核/证据/仲裁/夜间无人值守。明确区分 `Core Prototype` 与 `Product MVP`，Phase 1 内核原型不称为完整产品 MVP。
2. **技术栈升级**：全文 `.NET 8` → `.NET 10 LTS`（2026-11 EOL → 2028-11 LTS）。
3. **适配器优先级**：写入统一原则 `Official API/SDK > CLI > Webhook > Playwright DOM > UIA > Visual/Coordinate Automation`；GUI 自动化不得作为无人值守闭环唯一通道。
4. **三层授权与政策完整性**：适配器自报技术能力 ≠ 政策授权上限 ≠ 本次能力令牌；插件/模型/协调器/外部工具均无权自提权限；新增 `POLICY_INTEGRITY_SAFE_MODE`。
5. **状态权威**：事件账本是工作流事实唯一 Source of Truth，派生 projection 可重建且不得反向覆盖；签名政策包是授权/默认决策唯一 SoT，政策引擎是权限/预算/路由/发布门禁唯一裁决者，但不伪造工作流事实。
6. **投递语义**：删除 `supports_exactly_once_claim`，改为五字段能力；仅承诺"至少一次尝试、幂等优先、状态对账、无法确认时安全停止"；Outbox 同事务提交，外部动作状态含 `RECONCILIATION_REQUIRED`。
7. **浏览器/桌面安全**：浏览器凭据视同凭据并隔离；WebView2 仅用于 ARH 自身管理界面；UIA 探针覆盖全量边界测试。
8. **审核与严重度**：候选严重度含 HIGH/CRITICAL 且无法排除即阻断；`DEFERRED` 必须记录候选严重度/未决原因/重估条件；release gate 不接受纯文本"完成"。
9. **许可模型**：Phase 0–4 产品源码私有专有，暂不承诺 Open Core；受限许可组件未经逐项审查不得捆绑。
10. **P1/P2 落位**：全部 P0、P1 进入本版正文；P2 进入明确标注的后续路线图，不描述为已实现能力。

---

## 1. 执行摘要

Agent Relay Hub（以下简称 ARH）不是另一个"大模型"，也不是只服务于 STS 交易系统的一次性脚本。它是一套位于多个 AI、自动化工具、代码仓库和业务系统之间的"自主协调层"。

ARH 的任务是代替用户完成以下机械性和管理性工作：

- 发现新任务和新版本；
- 把完整上下文、附件和约束交给正确的智能体；
- 等待长时间任务完成，不因固定轮询而打断执行者；
- 组织多个独立审核角色；
- 合并重复问题并验证证据；
- 将裁决交给执行者修复；
- 检查 Git SHA、测试、CI、构建产物和审计证据；
- 在限定轮次内自动复审；
- 在工具失效时自动切换备用适配器；
- 在无法继续时进入安全终态，而不是把选择题重新抛给用户；
- 输出简明的完成报告或异常摘要。

产品的核心体验是：**用户一次授权，系统在授权边界内持续自主运行；用户不再充当传话筒、调度员、监工或重复确认者。**

所谓"全权代理"不是取消安全边界，而是将边界、默认决策、失败策略和终止条件提前写成机器可执行的政策。运行期间 ARH 不询问"要不要重试""选哪个模型""是否继续修复"等常规问题，而是依照政策自动决定。

**V1.1 阶段澄清**：V1.1 当前仅处于 Phase 0 设计冻结，叙述中的"闭环""多审核""夜间无人值守"均为目标能力（Phase 3/4 才实现），不在 Phase 1（Core Prototype）范围内。

---

## 2. 项目背景与问题定义

当前由多个独立工具组成的 AI 开发流程存在明显断点：

1. 编程智能体、审核模型、浏览器自动化和 GitHub 彼此不认识。
2. 每个工具的任务完成时间不确定，简单定时轮询可能重复发送或打断任务。
3. 用户必须在多个窗口之间复制、粘贴、等待、核对 SHA、上传附件和转发报告。
4. AI 容易自称完成，但其声明不一定与 Git、测试或 CI 一致。
5. 多轮审核容易陷入相同问题反复修补的死循环。
6. 桌面 GUI、网页、API、CLI 的控制方式不同，替换任一工具都可能导致整条流程重写。
7. 现有自动化通常要么擅长 API 工作流，要么擅长桌面 RPA，要么擅长多智能体对话，缺少统一的、可替换的闭环治理。

ARH 要解决的不是"如何再造一个 AI"，而是"如何让不同 AI 和工具在不依赖用户传话的情况下，可靠、有限、可验证地协作"。

---

## 3. 产品愿景

### 3.1 一句话愿景

让任何 AI、自动化工具和业务系统都能作为可替换角色加入一个可靠的自主工作流，而用户只负责一次授权和查看结果。

### 3.2 长期目标

- 从代码开发中继扩展到文档、研究、数据分析、营销、运维等流程（后四类需先定义各自证据类型，见 §18.2）；
- 从个人 Windows 工具扩展为 NAS 自托管控制中心；
- 从固定流程扩展为可配置模板和插件市场；
- 从单用户扩展为团队、多租户和商业部署；
- 支持不同厂商模型与执行工具自由替换，避免供应商锁定；
- 将"AI 自述成功"升级为"外部证据证明完成"。

### 3.3 非目标

首期不做以下事项：

- 不训练自有大模型；
- 不取代 Git、CI、测试框架或业务系统；
- 不在首期实现万能拖拽式平台；
- 不通过固定坐标和无保护的鼠标宏假装实现稳定自动化；
- 不自动处理密码、短信验证码、MFA、支付和身份实名认证；
- 不允许任意插件默认获得全磁盘、全网络或密钥访问权限；
- 不将 ARH 与 STS 交易系统代码混在同一仓库；
- **不在 Phase 1 实现任何真实 Git/GitHub/浏览器/UIA/Qoder/ChatGPT/WorkBuddy 适配器**（见 §19、§20）。

---

## 4. 自主代理原则

### 4.1 "不打扰用户"是产品需求，不是口号

运行期间，ARH 不得因以下常规事项询问用户：

- 选择主模型或备用模型；
- 是否继续等待仍有心跳的任务；
- 传输失败是否重试；
- 审核意见重复时如何合并；
- 普通测试失败是否退回执行者；
- 两个审核员意见冲突时交给谁；
- 在预设轮次内是否继续修复；
- 主工具不可用时是否启用备用工具；
- 是否生成报告、保存证据或发送每日摘要。

这些选择由预先配置的政策自动完成。

### 4.2 有界全权代理

系统默认采用以下授权档位：

| 档位 | 行为 | 默认状态 |
|---|---|---|
| A0 观察 | 读取状态、生成摘要 | 允许 |
| A1 研究 | 调用审核/规划工具、生成建议 | 允许 |
| A2 隔离执行 | 在沙箱或任务分支修改文件、运行测试 | 允许 |
| A3 交付候选 | commit、push 候选分支、创建/更新 PR | 允许 |
| A4 非生产部署 | 部署到 Demo、测试或临时环境 | 按工作流预授权 |
| A5 不可逆/生产 | 支付、删除重要数据、合并受保护主分支、实盘、解除安全门禁 | 默认禁止 |

用户"不介入"不意味着 A5 自动放行。A5 遇阻时系统不实时追问，而是自动选择"不执行"，保留现状并写入摘要。这样用户无需守在电脑前，系统也不会因无人应答而做出高风险决定。

#### 4.2.1 三层授权模型（V1.1 新增）

权限必须由三层显式分离，**插件、模型、协调器和外部工具均无权自行提高任何一层**：

1. **适配器自报技术能力**（capability report）：仅声明"我能做什么"（如 `execute`、`upload_artifact`），是事实描述，不是授权。
2. **政策引擎配置的允许权限上限**（policy ceiling）：由 ARH 政策包定义，是允许边界。
3. **本次调用的能力令牌**（capability token）：携带本次任务实际最小权限，由政策引擎在调度时签发。

最终授权上限 = min(自报能力, 政策上限, 令牌权限)，由 ARH 政策引擎保存并裁决。插件自报的 `authorization_tier_ceiling` **不得**被当作安全边界。

#### 4.2.2 子代理权限继承（V1.1 提升为 P0）

worker 派生的子代理（sub-agent）**不得**获得高于父代理的：

- 授权档位；
- 预算；
- 可访问路径；
- 可访问域名；
- 凭据范围；
- 工具能力。

子代理继承父档位上限，禁止任何隐性提权；越权请求由政策引擎拒绝并留痕。

### 4.3 无人值守默认决策

| 场景 | 自动决策 |
|---|---|
| 主适配器无响应 | 按顺序切换备用适配器（受切换抖动防护限制，见 §16.1） |
| 任务仍有心跳 | 继续等待，不重复发送 |
| 任务无心跳超过阈值 | 标记 STALLED，尝试一次会话恢复 |
| 相同逻辑 message 已创建 | 不重复生成新逻辑任务；允许受控 delivery_attempt 重试（相同幂等键 + attempt 序号） |
| 相同 SHA 已审查 | 复用结果，不重复计费 |
| CI 失败 | 自动退回执行者 |
| 审核意见冲突 | 进入证据验证和仲裁 |
| 修复达到最大轮数 | 进入 BLOCKED_REDESIGN_REQUIRED |
| 登录失效 | 尝试备用适配器；无备用则安全暂停该通道 |
| Windows 会话锁定 | 桌面适配器暂停；其他非桌面任务继续 |
| 费用达到预算 | 停止新增模型调用并生成预算摘要 |
| 发现不可逆操作 | 拒绝执行并记录，不询问用户 |
| 政策未覆盖且非安全情景 | 先走默认决策阶梯（见 §4.3.1），仍不成立才进入 `BLOCKED_NEEDS_POLICY_DECISION` |

#### 4.3.1 政策缺口默认决策阶梯（V1.1 新增 `BLOCKED_NEEDS_POLICY_DECISION` 前置）

新增 `BLOCKED_NEEDS_POLICY_DECISION` 终态**前**，系统必须先执行默认决策阶梯：

1. 可安全跳过则跳过；
2. 有保守默认值则采用；
3. 有等价备用工具则切换；
4. 可降级完成则降级完成；
5. 以上全部不成立，才进入 `BLOCKED_NEEDS_POLICY_DECISION`。

进入该终态时，系统**必须同时生成建议新增的政策规则**，不能只是把选择题延迟到第二天重新交给用户。

---

## 5. 市场与竞品分析

### 5.1 成熟产品提供的可借鉴能力

- **n8n**：可视化工作流、自托管、AI 节点和人工介入；适合借鉴连接器与流程画布，但其 fair-code 许可（Sustainable Use License）需要在商业嵌入前单独逐项评估。  
  参考：<https://github.com/n8n-io/n8n>
- **Node-RED**：节点化、可扩展、流程 JSON、Apache-2.0；适合借鉴插件和模板机制。  
  参考：<https://nodered.org/docs/creating-nodes/>、<https://github.com/node-red/node-red/blob/main/LICENSE>
- **Power Automate Desktop**：通过 UI 元素和选择器控制 Windows 应用，适合借鉴桌面自动化的选择器设计。  
  参考：<https://learn.microsoft.com/en-us/power-automate/desktop-flows/ui-elements>
- **UiPath**：成熟的 attended/unattended RPA、机器人治理和审计；适合借鉴运行节点、队列和恢复机制，但首期不采用其重型商业平台。
- **LangGraph**：持久状态、流式执行、人工介入和多智能体图；适合借鉴状态保存和中断恢复。  
  参考：<https://docs.langchain.com/oss/python/langgraph/overview>
- **Temporal**：可靠的长任务、重试、恢复和 durable execution；适合借鉴执行语义，MVP 不直接引入完整集群。  
  参考：<https://temporal.io/>

### 5.2 市场空缺

上述产品各自解决一部分问题，但缺少一个同时满足以下条件的轻量产品：

- 将模型、桌面 AI、浏览器 AI、CLI、人工和代码仓库统一抽象为角色；
- 支持供应商替换；
- 能在 API 与 GUI 自动化之间切换；
- 为长时间 AI 任务提供去重、恢复、仲裁、证据验证和硬性循环限制；
- 面向非技术用户提供"一次授权、自动完成"的产品体验；
- 同时适合个人自托管和后续商业化。

ARH 的差异化不是"连接更多 App"，而是"让不同智能体在有边界、有证据、有终态的情况下完成真正闭环"。

---

## 6. 角色模型

ARH 不写死产品名称，只定义逻辑角色：

| 角色 | 职责 | 可替换示例 |
|---|---|---|
| owner | 制定一次性授权政策、接收摘要 | 用户、团队管理员 |
| trigger | 发现任务或版本变化 | GitHub、邮箱、文件夹、Webhook、定时器 |
| coordinator | 分配任务、控制状态机 | ARH 内核 |
| planner | 拆解目标和计划 | ChatGPT、Claude、Gemini、本地模型 |
| reviewer | 独立审核 | 任意模型、规则引擎、人工专家 |
| evidence_verifier | 复现问题、运行确定性检查 | 测试运行器、静态扫描器、沙箱 |
| adjudicator | 对争议 finding 作裁决 | 高能力模型、规则引擎、人工策略 |
| worker | 修改代码或完成业务任务 | Qoder、Cursor、Claude Code、OpenHands |
| messenger | 传递消息和附件 | WorkBuddy、ARH 内置适配器、Webhook |
| repository | 保存版本和产物 | GitHub、GitLab、Gitea、本地 Git |
| release_gate | 根据证据决定是否晋级 | ARH 政策引擎 |
| monitor | 上线后监控和触发回滚 | 监控系统、测试探针、业务指标 |
| notifier | 输出结果摘要 | 飞书、邮件、App、Telegram |

同一个工具可以承担多个角色，但权限必须按角色拆分。例如，同一个模型既做 planner 又做 reviewer 时，也必须使用独立会话、独立上下文和独立结果记录。

> **状态权威约束（见 §12.4、§24.3）**：上述角色（含 WorkBuddy、Qoder、各模型、reviewer、worker、messenger、repository）均**不是状态权威**，仅能提交声明、结果与证据，不能改写权威状态或政策。

---

## 7. 多审核委员会设计

### 7.1 为什么不能简单投票

多个 AI 可能来自同一模型家族、共享相似训练偏差或被前一位审核员影响。三个相同错误不等于正确。因此 ARH **不采用"多数通过即通过"**，亦不使用简单多数票决定晋级。

### 7.2 标准审核拓扑

1. 协调器根据风险选择审核角色；
2. 各审核员盲审，互不可见；
3. 系统按 finding 指纹合并重复问题；
4. 证据验证器复现高风险问题；
5. 执行者可针对每项 finding 回应一次；
6. 仲裁员逐项裁决；
7. 发布门禁根据证据和政策作确定性决定。

### 7.3 可配置审核角色

- 需求一致性；
- 产品体验；
- 架构；
- 代码正确性；
- 安全；
- 红队；
- 测试质量；
- 数据质量；
- 性能；
- 运维与恢复；
- 法务与许可证；
- 领域风控；
- 自定义行业专家。

### 7.4 风险自适应审核

| 风险等级 | 默认委员会 |
|---|---|
| 低 | 正确性 + 确定性测试 |
| 中 | 正确性 + 测试 + 架构 |
| 高 | 正确性 + 安全 + 测试 + 运维 + 仲裁 |
| 极高 | 高风险委员会 + 红队 + 双证据验证 + 更严格发布门禁 |

### 7.5 Finding 标准状态

- PROPOSED
- EVIDENCE_REQUIRED
- CONFIRMED
- REJECTED
- DUPLICATE
- FIXED
- REGRESSION
- DEFERRED（**V1.1 要求**：进入 `DEFERRED` 必须保存候选严重度、未决原因、重新评估条件，见 §7.6）
- RISK_ACCEPTED（仅政策预先允许时）

### 7.6 裁决规则（V1.1 修正）

- 确定性测试失败：阻止晋级；
- 已确认 CRITICAL：阻止；
- 已确认 HIGH：默认阻止；
- **严重度无法裁决规则（V1.1 新增）**：若同一 finding 的可信严重度候选中包含 `CRITICAL` 或 `HIGH`，且证据验证无法排除该高风险候选，则发布门禁**必须阻断**；只有**全部可信候选严重度都低于发布门禁**，才允许进入 `DEFERRED`；证据不足不能自动降级严重度；`DEFERRED` 必须记录候选严重度、未决原因和重新评估条件；
- 证据不足的高风险 finding：进入一次证据验证；
- 重复 finding：合并，不增加票数；
- 意见冲突：仲裁，不做简单多数表决；
- AI 全部通过但 SHA/CI/哈希不一致：阻止；
- 审核讨论最多一轮；
- 同一 finding 最多修复两轮，之后转入重新设计终态；
- **release gate 不接受任何角色的纯文本"完成"作为晋级依据**（见 §25）；仅接受 `evidence_verifier` 确定性输出或外部可验证状态。

---

## 8. 完整自主闭环

### 8.1 主流程

```text
触发任务
→ 固化目标、仓库和精确版本
→ 能力与权限预检（三层授权，见 §4.2.1）
→ 规划
→ 隔离执行
→ 确定性测试
→ 多角色独立审核
→ 证据验证
→ 仲裁
→ 自动修复
→ 回归测试与复审
→ 发布候选
→ Demo/灰度运行
→ 运行监控
→ 自动回滚或确认稳定
→ 复盘并更新测试/规则
→ 终态归档
```

### 8.2 终态必须明确

每个任务最终必须到达以下之一：

- COMPLETED_APPROVED
- COMPLETED_WITH_DEFERRED_LOW_RISK
- BLOCKED_REDESIGN_REQUIRED
- BLOCKED_AUTH_EXPIRED
- BLOCKED_BUDGET_EXHAUSTED
- BLOCKED_NEEDS_POLICY_DECISION（V1.1 新增，须先走 §4.3.1 默认决策阶梯）
- QUARANTINED_SECURITY_RISK
- FAILED_EXTERNAL_DEPENDENCY
- CANCELLED_BY_POLICY
- POLICY_INTEGRITY_SAFE_MODE（V1.1 新增，政策可信根损坏时进入，见 §15.5）

不存在无限 RUNNING，也不存在"等用户选一个再说"的普通业务状态。

### 8.3 上线后闭环

ARH 不将"代码合并"视为完成。工作流可继续执行：

- Demo 或测试环境部署；
- 健康检查；
- 业务指标观察；
- 回归异常检测；
- 自动降级；
- 预授权回滚；
- 生成复盘；
- 将已证实问题转化为新测试和规则。

---

## 9. 防重复与防死循环

以下限制必须由内核执行，提示词无权修改；所有计数器、SHA、尝试次数、修复轮数、仲裁次数、备用切换次数、预算消耗**必须持久化于事件账本/SQLite，重启后不得归零**（V1.1 强化）。

| 项目 | 候选默认值 / 上限 |
|---|---|
| 同一逻辑 message 创建 | 1 次（message_id 唯一，逻辑消息不得重复生成） |
| 单通道投递尝试（delivery_attempt） | 默认最多 3 次，每次使用相同 idempotency key / correlation ID 并追加 attempt 序号 |
| 状态未知的外部写操作 | 0 次自动盲重试（先对账；无法确认则进入 `RECONCILIATION_REQUIRED`） |
| 重试前提 | 已确认未执行，或适配器支持有效幂等键 |
| 同一 SHA 审核 | 1 次 |
| 会话恢复 | 1 次 |
| finding 讨论 | 1 轮 |
| finding 修复 | 2 轮（候选默认，绝对上限见 §9.1） |
| 仲裁重试 | 1 次 |
| 重新设计 | 1 轮 |
| 模型费用 | 工作流预算（分工作流/每日/全局三层预算，候选默认值见 §9.1） |
| 最大执行时间 | 候选默认值 + 绝对上限（见 §9.1） |

额外规则：

- 目标 SHA 变化后才允许新一轮正式审核；
- finding 使用稳定指纹跨版本跟踪；
- 工具 BUSY 时不得发送第二个任务；
- 有心跳的长任务不得因轮询超时被打断；
- 没有心跳时先恢复状态，不直接重发；
- 达到上限后进入安全终态，不隐式重置计数；
- 任意角色不得修改循环限制、预算或发布政策。

### 9.1 默认数值与绝对上限（V1.1 新增，P1）

> 以下为 **Phase 0 候选默认值**，可由签名政策包收紧，但**不得越过绝对上限**。所有数值必须由内核强制执行，提示词无权修改；数值本身持久化于事件账本，重启后不归零。

- 预算分三层：工作流预算 / 每日预算 / 全局预算，任一超限即停止新增模型调用并生成预算摘要；
- §9 表中所有"上限"项均须有默认数值与绝对上限，禁止留空待定；
- 最大执行时间按工作流模板设置，缺省回退到全局默认值，且受绝对上限封顶。

| 参数 | 候选默认值 | 绝对上限 |
|---|---|---|
| 传输尝试次数（单通道 delivery_attempt） | 3 | 10 |
| 适配器切换次数 | 3 | 10 |
| 最大工作流执行时间 | 8 h | 24 h |
| 最大单步骤执行时间 | 30 min | 4 h |
| 每个 finding 修复轮数 | 2 | 5 |

**三层预算作用域（V1.1 明确）**——三层预算作用域彼此独立，任一超限即停止新增模型调用并生成预算摘要（不再使用含义不明的"每日模型预算"）：

| 预算层 | 作用域 | 候选默认值（token / 美元） | 绝对上限（token / 美元） |
|---|---|---|---|
| 单工作流预算 | 单个工作流实例累计 | 500,000 tokens / $2 | 2,000,000 tokens / $10 |
| 单项目每日预算 | 单个项目每日累计 | 2,000,000 tokens / $10 | 10,000,000 tokens / $50 |
| 全局每日预算 | 全部项目每日累计 | 10,000,000 tokens / $50 | 50,000,000 tokens / $200 |

**崩溃与熔断参数（V1.1 明确允许范围）**——政策只能在允许范围内调整，**不得留无界值**。注意不同参数"更严格（更保守）"的方向不同：

| 参数 | 候选默认值 | 签名政策允许范围 |
|---|---:|---:|
| 插件崩溃观察窗口 | 10 min | 1–60 min |
| 崩溃触发阈值 | 3 次 | 1–10 次 |
| 熔断冷却时间 | 15 min | 5 min–24 h |

- **更长的观察窗口更保守**（更容易累计到崩溃阈值、更早停用）；
- **更低的崩溃阈值更保守**（更少崩溃即触发停用）；
- **更长的冷却时间更保守**（停用后更久才允许恢复）。

---

## 10. 适配器与插件体系

### 10.1 适配器类型

- REST/API
- Webhook
- 浏览器 DOM/CDP
- Windows UI Automation
- CLI/进程
- Git/代码仓库
- 文件夹/对象存储
- 消息平台
- 人工收件箱
- 模拟适配器

### 10.2 能力协商

每个适配器启动时必须上报：

```text
send
receive
stream
upload_artifact
download_artifact
execute
cancel
resume
health_check
structured_output
requires_unlocked_session
requires_human_auth
supports_idempotency
supports_idempotency_key          （V1.1 新增）
supports_status_query             （V1.1 新增）
supports_external_correlation_id （V1.1 新增）
supports_reconciliation           （V1.1 新增）
delivery_semantics                （V1.1 新增，取代下方已删除项）
max_payload_size
supported_protocol_versions
```

> **V1.1 删除**：`supports_exactly_once_claim`。ARH 对任意外部系统只能承诺："至少一次尝试、幂等优先、状态对账、无法确认时安全停止。"不得宣传跨任意第三方系统的 exactly-once（详见 §16.3）。

工作流引擎先检查能力再调度，不能依靠运行时猜测。

### 10.3 插件清单

插件必须声明：

- 唯一 ID、名称、版本、厂商；
- 最低 ARH 协议版本；
- 支持的角色和能力；
- 所需文件、网络、进程和凭据权限；
- 可访问路径白名单；
- 可访问域名白名单；
- 许可证；
- 包哈希和签名；
- 健康检查方法；
- 卸载与数据清理方式。

### 10.4 插件隔离

- 默认以独立进程运行；
- 通过 JSON-RPC 2.0 over stdio 或本地受限 WebSocket 通信；
- 插件不能直接打开核心 SQLite；
- 插件不能读取其他插件的凭据；
- 单个插件崩溃不得拖垮控制中心；
- 超权限请求由政策引擎拒绝；
- 商业版本支持签名插件和可信发布者列表；
- **插件崩溃退避 + 能力降级（V1.1 新增，P1）**：同插件 10 分钟内崩溃超 3 次（候选默认值，可由签名政策收紧，见 §9.1）则停用并告警，不无限重启；运行中适配器丢失关键能力时，对在途 job 触发重路由或安全终态。

### 10.5 适配器选型优先级（V1.1 新增，P0）

统一原则，贯穿所有适配器选型：

```text
Official API/SDK > CLI > Webhook > Playwright DOM > UIA > Visual/Coordinate Automation
```

- **API / 官方 SDK / CLI 优先**；浏览器 DOM / Playwright 次之；Windows UI Automation 再次；屏幕视觉、鼠标坐标和键盘模拟只允许作为最后降级手段；
- GUI 通道**不得**成为无人值守工作流的唯一完成路径；
- 不得因为桌面界面"看得见"就优先采用 UIA；
- Qoder 首个真实适配器应优先使用 CLI/SDK，不应优先控制 Qoder 桌面窗口（见 §18.1）。

---

## 11. 统一消息协议 ARP v1

所有工具通过标准信封通信：

```json
{
  "protocol_version": 1,
  "workflow_id": "WF-001",
  "job_id": "JOB-001",
  "message_id": "MSG-001",
  "correlation_id": "CORR-001",
  "idempotency_key": "sha256:...",
  "sender_role": "reviewer.security",
  "recipient_role": "worker.primary",
  "message_type": "review.findings",
  "target": {
    "repository": "owner/repository",
    "branch": "feature/example",
    "revision": "full-sha"
  },
  "attempt": 1,
  "artifacts": [
    {"name": "diff.patch", "sha256": "...", "uri": "artifact://..."}
  ],
  "payload": {},
  "created_at": "ISO-8601",
  "expires_at": "ISO-8601"
}
```

协议要求：

- 所有输出必须绑定 request/message/job 和目标版本；
- 附件必须带哈希；
- 历史对话中的旧标记不能被识别为新结果；
- 消息不可原地修改，只能追加新版本；
- 结果解析失败不得猜测；
- 协议升级必须可迁移、可回放、可向后兼容。

---

## 12. 状态与数据架构

### 12.1 Phase 1 Core Prototype 数据库

本地 SQLite，WAL 模式，核心表建议包括：

- workflows
- workflow_versions
- jobs
- job_attempts
- messages
- deliveries（V1.1：承载 Outbox 与外部动作状态机，见 §12.2）
- adapters
- adapter_health
- role_assignments
- findings
- evidence
- adjudications
- artifacts
- policy_decisions
- budgets
- audit_events
- notifications

### 12.2 事件溯源与事务型 Outbox（V1.1 强化）

核心状态变更采用追加事件：

- 不覆盖历史；
- 每个事件带前序哈希；
- 可重放恢复当前状态；
- 外部动作记录意图、提交、响应和验证四阶段；
- 日志失败时不得宣称外部操作成功。

**事务型 Outbox（V1.1 新增，P0）**：

- Outbox 保证内部状态变更与"待投递意图"在**同一数据库事务**中提交；
- 外部动作记录至少区分以下状态：`PREPARED` / `DISPATCHING` / `ACKNOWLEDGED` / `VERIFIED` / `RECONCILIATION_REQUIRED`；
- 崩溃恢复后**不得盲目重发**；
- 若适配器支持 idempotency_key、external correlation ID 或 status query，必须先查询并对账；
- 若无法确定动作是否已经执行，进入 `RECONCILIATION_REQUIRED` 或对应安全终态；
- 不得宣称 Outbox 能对任意第三方外部副作用实现 exactly-once。

### 12.3 产物仓库

统一保存：

- prompt 和结构化消息；
- 源码快照或 bundle；
- diff/patch；
- 测试报告；
- CI 结果；
- 构建产物；
- 截图（默认脱敏并可关闭）；
- 审核结果；
- 仲裁记录；
- 每日摘要。

所有产物使用内容寻址和 SHA256，避免同名覆盖。

### 12.4 状态权威与策略裁决权威（V1.1 新增，P0）

**状态权威（事件账本）**：

- 事件账本是工作流事实、外部动作意图、投递结果和审计事件的**唯一 Source of Truth**；
- 当前状态视图、仪表盘和查询表均为可从事件账本重建的派生 projection；
- 派生状态**不得反向覆盖**事件事实；删除 projection 后可从事件账本重建相同状态。

**策略裁决权威（政策包 + 政策引擎）**：

- 已签名政策包是授权边界与默认决策规则的**唯一 Source of Truth**；
- 政策引擎是权限、预算、工具路由和发布门禁的**唯一裁决者**，但不保存或伪造工作流事实；
- 政策引擎的决策必须引用政策版本、输入事件和决策理由；
- 外部 worker、reviewer、messenger、repository 和模型只能提交声明、结果与证据，不能直接改写权威状态或政策。

### 12.5 Phase 1 备份、恢复与重放（V1.1 新增，P0，F-C8）

备份与恢复是 Phase 1 Core Prototype 必须完成并纳入自动化测试的能力（不依赖任何真实外部工具）。定义如下：

1. 暂停新写入或建立一致性快照边界；
2. 执行受控 WAL checkpoint；
3. 使用 SQLite backup API 或等价一致性备份机制；
4. 同步记录数据库快照、事件序号、产物 manifest 与哈希；
5. 产物存储与数据库快照使用同一 backup set ID；
6. 恢复时先校验哈希，再恢复数据库和产物；
7. 重放到指定事件序号；
8. 重建 projection；
9. 对账所有 `DISPATCHING` / `ACKNOWLEDGED` 未决动作；
10. 状态或哈希不一致时进入安全终态，不宣称恢复成功。

必须明确定义：

- **备份触发条件**：每次重大状态变更后、定时（如每 6 小时）、以及手动触发；
- **保留策略**：保留最近 N 个 backup set（候选默认 N=30），按时间/序号滚动删除，至少保留一个跨版本可恢复点；
- **备份失败行为**：告警并进入安全终态或降级模式，不静默继续写入；
- **恢复成功判据**：哈希校验通过 + 事件重放一致 + projection 重建一致 + 所有未决动作对账完成；
- **恢复演练频率**：每次 Phase 1 发布前至少执行一次恢复演练，并纳入自动化测试；演练须覆盖注入崩溃点后的重放与对账。

---

## 13. 系统架构

### 13.1 Phase 1 Core Prototype 单机架构

```text
ARH Desktop
├─ Tray/UI（WebView2，仅 ARH 自身管理界面，见 §13.3）
├─ Workflow Engine
├─ Policy Engine（含三层授权裁决，见 §4.2.1）
├─ Scheduler
├─ SQLite/Event Ledger（含 Outbox 状态机，见 §12.2）
├─ Artifact Store
├─ Adapter Host（Phase 1 仅加载模拟适配器）
└─ （真实 Browser/Desktop/Git Runner 在 Phase 3 才引入）
```

### 13.2 演进架构

```text
NAS Control Plane
├─ Workflow/Policy/API
├─ PostgreSQL
├─ Artifact Store
├─ Web/PWA Dashboard
└─ Scheduler

Windows Runner
├─ Desktop UI Automation
├─ Local Browser
├─ Local Git Workspace
└─ Session/Lock Guard

Cloud/API Runners
├─ Model APIs
├─ Cloud coding agents
└─ Git/CI integrations
```

### 13.3 技术建议（V1.1 修订）

- 核心与 API：**.NET 10 LTS** / ASP.NET Core；
- Windows 托盘与 Runner：**.NET 10 LTS**；
- 管理界面：Web UI，MVP 可嵌入 WebView2；
- 浏览器：Playwright，**连接专用浏览器配置/独立 Browser Runner**（见 §14.2）；
- Windows 控制：Microsoft UI Automation；
- 本地状态：SQLite；
- 后续中心状态：PostgreSQL；
- 插件协议：JSON-RPC 2.0；
- 配置：JSON/YAML + JSON Schema；
- 凭据：Windows Credential Manager，后续接 Vault；
- 日志：结构化 JSON + 审计哈希链。

> **技术栈说明（V1.1）**：V1.0 的 `.NET 8` 全部更新为 **`.NET 10 LTS`**（.NET 8 于 2026-11 EOL，.NET 10 LTS 支持至 2028-11）。核心、API、Windows Tray、Runner 及后续 Linux/NAS 控制面优先使用同一跨平台 .NET 10 技术栈。

> **WebView2 边界（V1.1 新增，P1）**：WebView2 只负责 ARH 的管理界面和仪表盘；第三方网页自动化必须由独立 Playwright Browser Runner 执行，使用专用浏览器上下文，不与 ARH UI WebView2 或用户个人浏览器混用。

不建议首期直接嵌入 n8n 或 Temporal。可借鉴其模型，在规模和需求达到门槛时再评估接入，避免 MVP 过重或商业许可证受限。

---

## 14. 桌面与浏览器运行节点

### 14.1 桌面运行节点（UIA 门禁，V1.1 强化）

- 只在已登录的指定 Windows 用户会话运行；
- 校验目标进程路径、签名、PID 和窗口；
- UIA 选择器优先，禁止仅凭绝对坐标；
- 操作前后检查前台窗口，防止把内容输入错误应用；
- 剪贴板内容使用后清除；
- 发现锁屏、UAC、登录、MFA 时进入阻塞状态；
- 不自动输入密码或验证码；
- 支持专用 Windows VM，避免个人桌面长期解锁；
- **UIA 探针门禁（V1.1 新增，P1）**：探针必须验证锁屏、UAC、权限级别不同、窗口失焦、RDP 最小化/断开、应用更新、多窗口同名、输入法与剪贴板污染、目标进程崩溃、UI 元素变化；任何一项可能导致错误窗口写入时，立即停止桌面通道，不允许盲输。

### 14.2 浏览器运行节点（V1.1 强化）

- 使用专用浏览器配置，**由独立 Playwright Browser Runner 执行，不与 WebView2 或用户个人浏览器混用**；
- 调试端口只监听 127.0.0.1；
- 对 URL、域名和会话端点做白名单；
- 登录/MFA 页面不自动提交凭据；
- DOM/可访问性定位优先；
- 上传前验证附件哈希；
- 只接受最新响应区域中的完整协议块；
- 页面结构变化后自动健康检查并停用不兼容版本；
- **浏览器凭据（V1.1 新增，P0）**：Playwright storage state、Cookie、localStorage、浏览器用户目录和会话令牌全部视同凭据——不入 Git、不入普通产物库、不进入 prompt/日志/截图；使用独立保护目录；支持过期、撤销和清理。

---

## 15. 安全模型

### 15.1 最小权限（三层授权落地，V1.1 强化）

每个角色获得一次性能力令牌，例如：

- reviewer：读指定版本和附件，不可写仓库；
- worker：只写指定 worktree/分支；
- verifier：可运行白名单测试，不可修改源码；
- adjudicator：读 finding 和证据，不可执行命令；
- release_gate：读取证据，只能改变内部晋级状态；
- notifier：只能发送脱敏摘要。

令牌权限 = min(适配器自报能力, 政策上限, 本次令牌)，由政策引擎签发（见 §4.2.1）。**插件、模型、协调器和外部工具均无权自行提高权限档位、预算、路径、域名、凭据或工具能力**；子代理继承父档位上限，禁止隐性提权（见 §4.2.2）。

### 15.2 凭据

- 不在 YAML、SQLite、日志和 prompt 中保存明文凭据；
- 使用操作系统凭据库；
- 适配器只得到所需凭据引用；
- 记录凭据使用事件但不记录值；
- 支持过期、轮换和撤销；
- 插件无法枚举全部凭据；
- 浏览器凭据（storage state/Cookie/localStorage/用户目录/会话令牌）按 §14.2 隔离处理。

### 15.3 Prompt Injection 防护（V1.1 强化 verifier 沙箱）

- 外部网页、仓库文档和评论全部视为不可信数据；
- 系统政策与任务内容分层；
- 外部文本不能修改权限、循环、预算和目标仓库；
- 高风险工具调用由政策引擎决定，不由模型文字决定；
- 附件类型、大小、路径和内容均验证；
- 结果必须符合 Schema，解析失败进入隔离；
- **verifier 沙箱出网 + 脱敏（V1.1 新增，P1）**：evidence_verifier 沙箱默认阻断出网、限制文件系统白名单；截图脱敏方法文档化（敏感字段替换 + 默认开启 + 关闭需显式策略授权并留审计）。

### 15.4 全局熔断

- 托盘紧急停止；
- 全局暂停文件/开关；
- 远程只读状态页；
- 检测异常消息风暴、重复提交或权限绕过后自动熔断；
- 熔断后不自动解锁生产级权限。

### 15.5 政策完整性安全模式（V1.1 新增，P0）

政策包使用签名或由 OS 保护的可信哈希锚点；信任根不得与可修改政策文件同权限域。

- 政策签名或可信锚点校验失败后，进入 **`POLICY_INTEGRITY_SAFE_MODE`**；
- safe mode **仅允许本地 A0**：查看本地状态、查看审计、导出脱敏诊断、由人工修复或重新签署政策；
- A1–A5 全部不可用；
- 默认禁止：外部网络请求、模型调用、CLI 命令执行、Git 写入、浏览器自动化、桌面自动化；
- 不自动切换备用适配器、不自动恢复 A1–A5；
- 只有可信政策重新加载并通过校验后才能退出 safe mode；
- 篡改政策包后，任何外部 API、模型、CLI、浏览器或 UIA 调用均被拒绝，仅本地 A0 诊断可用。

---

## 16. 可靠性与故障策略

### 16.1 健康度与自动路由

每个适配器维护：

- 可用性；
- 最近成功时间；
- 延迟；
- 失败率；
- 版本兼容；
- 登录状态；
- 费用状态；
- 当前 BUSY/IDLE；
- 能力变化。

协调器基于健康度和政策选择主/备适配器，不在运行期间询问用户。

- **无备用默认终态（V1.1 新增，P1）**：角色无任何备用适配器时，主失效直接进对应安全终态；
- **切换抖动防护（V1.1 新增，P1）**：增最大切换尝试次数（候选默认 3、绝对上限 10，见 §9.1）+ 切换退避，避免主备之间反复横跳。

### 16.2 心跳语义

- RUNNING + 心跳更新：继续等待；
- RUNNING + 无心跳：进入 STALLED；
- STALLED：尝试一次恢复或查询外部证据；
- 恢复失败：切换备用或进入终态；
- 不因固定 15/60 分钟轮询而重复提交任务。

### 16.3 崩溃恢复（V1.1 与投递语义一致）

应用重启后：

1. 重放事件恢复状态；
2. 查询外部系统确认实际状态；
3. 对提交状态未知的动作进行对账；
4. 只有确认未执行才允许重试；
5. 对可能已执行的写操作进入 `RECONCILIATION_REQUIRED`，而不是盲目重发；
6. 若适配器支持 idempotency_key / external correlation ID / status query，必须先查询并对账；
7. 无法确认外部动作是否已执行时，必须进入 `RECONCILIATION_REQUIRED` 或对应安全终态。

> ARH 对任意外部系统只能保证"至少一次尝试、幂等优先、状态对账、无法确认时安全停止"，**不得宣称跨任意第三方系统的 exactly-once**。

---

## 17. 用户体验与仪表盘

### 17.1 用户不做调度员

首页不显示大量待选择问题，而显示：

- 系统是否自主运行；
- 当前目标；
- 当前角色和工具；
- 当前阶段（明确区分 Core Prototype / 真实闭环，见 §19）；
- 执行时长与心跳；
- 审核轮次；
- 预算使用；
- 已完成和安全阻塞数量；
- 最近交付；
- 是否存在需要第二天知晓的摘要。

### 17.2 核心页面

- Overview：总览和全局暂停；
- Tools：工具、适配器、健康度和备用顺序；
- Roles：逻辑角色到具体工具的映射；
- Workflows：模板、权限和预算；
- Jobs：当前与历史任务；
- Reviews：审核委员会、finding、证据和仲裁；
- Artifacts：产物和哈希；
- Policies：自主权限、循环、费用和终态（含政策签名/锚点状态）；
- Audit：可验证、哈希链式、具备篡改检测能力的审计记录（非外部不可变存储）；
- Notifications：飞书/邮件/App 摘要策略。

> **审计能力说明（V1.1 修订）**：Personal / Core Prototype 中的审计记录指"可验证、哈希链式、具备篡改检测能力"的本地记录——**本地哈希链不等于外部不可变存储**；外部不可变审计锚定（如独立 WORM 存储、区块链式公证）属于 Team / Enterprise 后续能力，**不得在 Personal / Core Prototype 中宣称审计绝对不可变**。

### 17.3 零打扰模式

默认开启：

- 不发送常规过程通知；
- 成功后发送一次摘要；
- 阻塞后进入安全终态并发送一次摘要；
- 不发送"请选择 A/B/C"；
- 不因用户未回复而无限占用运行态；
- 每日最多一份汇总，可按项目关闭。

> **可观测性（V1.1 新增，P1）**：声明"当前仅单 job，并行为后续"；定义结构化指标/SLO/告警阈值（不止 UI 字段），见 §17.4。

### 17.4 可观测性指标与 SLO（V1.1 新增，P1，F-C17）

> 以下为 **Phase 0 候选指标与 SLO**，可在 Phase 0 一致性审核中调整具体阈值；禁止留空。完整采集与告警实现随阶段落地，但指标定义与 SLO 方向已在 V1.1 固定。

**候选指标**：

- `workflow_terminal_state_rate`（终态达成率）
- `duplicate_logical_job_rate`（重复逻辑任务率）
- `reconciliation_required_rate`（需人工/自动对账率）
- `adapter_availability`（适配器可用率）
- `adapter_error_rate`（适配器错误率）
- `recovery_success_rate`（恢复成功率）
- `policy_denial_count`（政策拒绝次数）
- `budget_exhaustion_count`（预算耗尽次数）
- `workflow_duration_p50` / `workflow_duration_p95`（工作流时长中位数/P95）
- `ledger_replay_consistency_rate`（账本重放一致率）
- `projection_rebuild_success_rate`（projection 重建成功率）
- `credential_leak_detection_count`（凭据泄漏检测次数）

**Phase 1（Core Prototype）——正确性不变量（必须满足，违反即阻断验收）**：

- 所有未被政策取消的工作流，**必须在配置的最大执行时间内到达某个明确终态**；
- `workflow_terminal_state_rate = 100%`；
- 出现无限 `RUNNING` 属于**正确性故障**，直接阻断 Phase 1 验收；
- `duplicate_logical_job_rate = 0`（逻辑任务去重）；
- `ledger_replay_consistency_rate = 100%`；
- `projection_rebuild_success_rate = 100%`。

**Phase 1（Core Prototype）——性能 SLO 与告警（性能退化，不得无限运行）**：

- SLO：99% 的模拟工作流在目标时长内到达终态；`workflow_duration_p95` 低于设定目标；`recovery_success_rate` ≥ 99%；
- 超过目标时长但未超过绝对上限时属于**性能退化**（不是正确性故障），仍不得继续无限运行，须在绝对上限内到达终态；
- 告警：reconciliation_required_rate > 0 触发待办（非告警风暴）；policy_denial_count 异常突增告警；recovery_success_rate < 99% 阻断发布。

**Phase 3（Product MVP）——正确性不变量**：

- 所有未被政策取消的工作流必须在最大执行时间内到达明确终态（到达终态是正确性不变量，不是 SLO）；`workflow_terminal_state_rate = 100%`；无限 `RUNNING` 即正确性故障。

**Phase 3（Product MVP）——性能 SLO 与告警**：

- SLO：adapter_availability ≥ 99.5%；adapter_error_rate < 1%；workflow_duration_p95 < 8 h；credential_leak_detection_count = 0（检测到即高危告警）；
- **预算语义（V1.1 修正）**：预算耗尽必须进入明确终态（`BLOCKED_BUDGET_EXHAUSTED`），**不允许超预算后继续调用**；`budget_exhaustion_count` 仅用作**容量规划指标**，不与金额阈值直接比较；
- 告警：adapter_error_rate > 2% 触发备用切换；reconciliation_required_rate > 1% 触发对账看板与人工提示；credential_leak_detection_count > 0 立即熔断并告警。

---

## 18. 配置与模板

### 18.1 工具替换（Qoder 路线，V1.1 修订）

用户未来替换工具时只修改映射：

```yaml
roles:
  planner: model_api_primary
  reviewer_security: model_api_secondary
  reviewer_correctness: local_model_api
  adjudicator: model_api_adjudicator
  worker: qoder_cli
  repository: github_api
  notifier: feishu_webhook
```

更换 Qoder 为其他执行者，只替换 `worker` 对应适配器。

> **适配器选型回退（V1.1 新增）**：以上示例均为供应商无关且符合 §10.5 优先级（官方 API/SDK > CLI > Webhook > Playwright DOM > UIA > Visual/Coordinate）。只有官方 API/SDK/CLI 无法覆盖时，才允许将某角色改为 browser/UIA 回退通道，且 UIA 仅作 CLI/SDK 无法覆盖能力的补充。

> **Qoder 实现路线（V1.1 新增，P1）**：Phase 2 以**本地 Qoder CLI 和稳定 Agent SDK 为基线**，优先验证 Qoder CLI / headless / Agent SDK / 结构化输出 / 流式事件 / 取消恢复 / 工具权限 / 进程退出码 / 会话关联；任何标记为 experimental / unstable 的云端能力**不得成为正式闭环的唯一依赖**；桌面 UIA 只作为 CLI/SDK 无法覆盖功能的补充通道。

### 18.2 工作流模板（V1.1 修订 — 非代码证据推迟）

- **首批（MVP 代码开发闭环）**：软件开发闭环、PR 多模型审核；
- **未来候选（须先定义各自证据类型后接入）**：文档起草—审核—修订、研究收集—交叉核实—报告、数据分析—复核—发布、运维告警—诊断—修复候选—验证、营销类工作流；
- **STS 产品研发闭环**：仅模板，不嵌入交易权限，且接入阶段后置于 Phase 5（见 §20 路线图）。

> **依据（F-C7）**：MVP 仅做代码开发闭环；文档/研究/营销/数据分析工作流改为未来候选，待各自证据类型（结构化 schema 校验、外部 API 状态查询、人工签名确认等）定义后再进入实现。

---

## 19. MVP 范围（V1.1 重构 — Core Prototype 与 Product MVP 区分）

### 19.1 Core Prototype（Phase 1 必须实现，不含任何真实适配器）

- 状态机；
- SQLite 事件账本（含 Outbox 状态机，§12.2）；
- 政策引擎（含三层授权裁决与 `POLICY_INTEGRITY_SAFE_MODE`，§4.2.1/§15.5）；
- **模拟适配器**（非真实 Git/GitHub/浏览器/UIA/Qoder/ChatGPT/WorkBuddy）；
- **模拟工作流**；
- 防重、预算（三层）、循环限制、崩溃恢复与终态自动化测试；
- SQLite 一致性备份、WAL checkpoint、产物备份、恢复演练、事件重放一致性校验（§12）。

> **这是内核原型，不是完整产品 MVP。** 它证明内核、政策与事件账本的正确性，不使用任何真实外部工具。

### 19.2 Product MVP（目标：Phase 3 首个真实闭环）

仅在 Phase 2 探针通过门禁后，才实现正式产品适配器（真实 Git/GitHub/浏览器/UIA/Qoder 等之一或组合）与第一个真实代码开发闭环。Product MVP 的达成依赖 Phase 3，不在 Phase 1。

### 19.3 暂不实现（含未来候选）

- 多租户；
- 插件商店；
- 云端收费；
- 手机原生 App；
- 大规模 Temporal 集群；
- 复杂低代码画布；
- 生产实盘或支付自动化；
- 任意第三方插件自动安装；
- 文档/研究/营销/数据分析工作流（未来候选，见 §18.2）；
- Phase 1 内的任何真实适配器（Git/GitHub/浏览器/UIA/Qoder/ChatGPT/WorkBuddy）。

---

## 20. 实施路线与阶段门禁（V1.1 重构）

### Phase 0：产品设计冻结（当前所处）

交付：产品书、协议、角色、权限、状态机、适配器 SDK 契约、威胁模型、验收标准。  
门禁：无代码实现；设计冲突清零；许可证路线明确；SDK 生命周期/错误码/取消/恢复/健康检查文档化（§20.1）。  
**本 V1.1 处于此阶段。**

### Phase 1：Core Prototype

交付：状态机、SQLite、事件账本、政策引擎、模拟适配器、模拟工作流、自动化测试。  
门禁：崩溃恢复、去重、循环上限、预算、终态、Outbox 对账、safe mode 全部通过自动测试。  
**不含任何真实适配器。**

### Phase 2：真实工具可行性探针

交付：**一次性、隔离、不可直接复用为生产组件**的可行性探针——本地 Git 探针、Qoder CLI/SDK 探针、Playwright 浏览器探针、Windows UIA 探针、锁屏/UAC/焦点/会话状态探针。  
门禁：不使用绝对坐标；目标窗口/页面可稳定识别；隔离仓库无意外修改；探针通过后方可进入 Phase 3。  
**不交付正式产品适配器。**

### Phase 3：正式适配器与首个真实闭环

交付：在 Phase 2 探针通过门禁后，才实现正式产品适配器和第一个真实代码开发闭环（如一个正式 worker 适配器（优先 Qoder CLI/稳定 SDK）+ 一个 reviewer 适配器（优先 API/SDK）+ repository 适配器 + notifier）。  
门禁：只在测试仓库完成端到端闭环；同一逻辑 message 只创建一次（允许受控 delivery_attempt 重试，默认最多 3 次，每次相同幂等键 + attempt 序号）；工具繁忙不打断。

### Phase 4：多审核委员会与自主夜间运行

交付：盲审、证据验证、仲裁、两轮修复、备用工具、有限自动修复、零打扰摘要。  
门禁：连续无人值守测试；登录失效、网络断开、锁屏、CI 失败等故障注入通过。

### 后续阶段（路线图，见 §23 / §20.1）

- Phase 5：STS 研发流程接入（仅模板，不接 OKX、不接实盘）；
- Phase 6：NAS 控制中心；
- Phase 7：产品化与商业版（含许可证与商业模式最终裁定）。

每个阶段必须独立审核通过后才能进入下一阶段。WorkBuddy、Qoder 或其他执行者不得自行跳阶段。

### 20.1 SDK 契约文档化（V1.1 新增，P1，Phase 0 交付物）

SDK 最小契约**必须在 Phase 0 文档化**（ARCHITECTURE / PROTOCOL 指针或骨架），作为 Phase 0 验收清单可勾选项。完整 Schema 写入 `PROTOCOL.md` / `ARCHITECTURE.md`，但以下最小语义已在 V1.1 固定：

- **生命周期状态**：`LOADING` / `NEGOTIATING` / `READY` / `BUSY` / `DEGRADED` / `STOPPING` / `STOPPED` / `FAILED`；
- **生命周期进入条件（顺序，V1.1 修正）**：

```text
初始/开始加载 → LOADING
加载成功 → NEGOTIATING
协商成功且协议兼容 → READY
接收任务 → BUSY
任务正常结束 → READY
能力部分丢失但仍可服务 → DEGRADED
收到关闭请求 → STOPPING
正常关闭完成 → STOPPED
不可恢复异常 → FAILED
```

  - `LOADING` **不是** load 成功后的状态，而是 load **正在执行**的状态；
  - `NEGOTIATING` 是**加载成功后**进行协议与能力协商的状态；
  - `READY` 才表示可以接收任务；
  - `DEGRADED` 恢复关键能力后可回到 `READY`；
  - `FAILED` **不得自动回到 `READY`**，必须重新加载或由政策决定替换；
- **统一错误分类**：含 retriable 与 non-retriable 错误，调度器据此决定是否重试或切换备用；
- **cancel 的确认语义**：cancel 请求后适配器必须确认已停止或报告进行中动作的最终状态，不得静默继续；
- **resume 所需 token / checkpoint**：恢复需携带 resume token 与最近 checkpoint，缺失则拒绝；
- **health_check 的返回 schema**：至少含状态、最近成功时间、能力集、版本，格式固定；
- **能力变化事件**：运行期能力增删必须主动上报，调度器据以重路由或降级；
- **协议版本不兼容行为**：版本区间不重叠时拒绝加载并告警，不降级静默运行；
- **超时和强制终止行为**：serve 超时不响应则进入 `DEGRADED`，连续失败转 `FAILED` 并触发替代/安全终态。

---

## 21. 测试与验收体系

### 21.1 测试层次

- 单元测试：状态、策略、协议、预算、权限；
- 属性测试：消息去重、状态不变量；
- 集成测试：适配器模拟、数据库、进程崩溃；
- 合约测试：每个适配器遵守 SDK；
- E2E：隔离仓库完整闭环；
- 故障注入：断网、超时、进程退出、登录失效、锁屏；
- 安全测试：路径逃逸、命令注入、prompt injection、密钥泄漏；
- 回放测试：历史事件恢复相同状态；
- 升级测试：旧协议/旧数据库迁移；
- 长稳测试：连续运行与内存/句柄泄漏；
- **Outbox 对账测试（V1.1 新增）**：注入外部动作执行前/执行中/执行后未写回等崩溃点，恢复后不盲目重发，能够对账的完成对账，无法确认的进入 `RECONCILIATION_REQUIRED`；
- **safe mode 测试（V1.1 新增）**：篡改政策包后，任何外部 API/模型/CLI/浏览器/UIA 调用均被拒绝，仅本地 A0 诊断可用；
- **严重度裁决测试（V1.1 新增）**：① HIGH 与 LOW 冲突且无法排除 HIGH → 阻断；② MEDIUM 与 LOW 冲突且均低于门禁 → 可 DEFERRED；③ 出现新确定性证据 → 重新计算严重度并更新 finding 状态。

### 21.2 验收标准（按阶段拆分，V1.1 修订）

> 以下四类验收分别对应 Phase 1 / 2 / 3 / 4。Phase 1 仅含内核、模拟适配器与模拟工作流，不涉及任何真实外部工具；Phase 2 只做一次性隔离探针、不交付正式适配器；Phase 3 才形成首个真实闭环；Phase 4 才补齐多审核与夜间自治。

#### Phase 1 Core Prototype 验收（仅内核、模拟适配器、模拟工作流，不涉及真实外部工具）

1. 用户启动后不需要参与测试工作流的常规选择；
2. 同一逻辑 message 只创建一次，不生成重复逻辑任务；状态未知的外部写操作不自动盲重试，优先对账、无法确认进入 `RECONCILIATION_REQUIRED`；
3. 模拟执行者 BUSY 时不会被打断；
4. 应用崩溃后可恢复状态；Outbox 对账、projection 重建、事件重放一致性校验通过；
5. 达到两轮修复上限后进入明确安全终态，不隐式重置计数；
6. 插件越权被拒绝并记录（§4.2.1）；
7. 不记录明文凭据（§15.2）；
8. 全流程可从事件和产物完整追溯；
9. 模拟工作流在隔离环境运行，无测试仓库之外文件变化；
10. 全局暂停和熔断有效；
11. 成功或阻塞后只发送一次摘要；
12. 子代理越权被拒绝并留痕（§4.2.2）；
13. 政策可信根损坏时进入 `POLICY_INTEGRITY_SAFE_MODE` 且仅本地 A0 可用（§15.5）；
14. SQLite 备份 / 恢复演练 / 事件重放一致性校验通过（§12.5）；
15. 不涉及任何真实外部工具（仅模拟适配器与模拟工作流）。

#### Phase 2 探针验收（一次性、隔离、不可复用于生产组件）

1. 真实工具探针通过：Qoder CLI/SDK、本地 Git、Playwright、UIA；
2. 锁屏 / UAC / RDP 最小化断开 / 焦点 / 权限级别 / 多窗口同名 / 输入法 / 剪贴板 / 应用更新 / 元素变化等边界探针通过（UIA 误写即停）；
3. 探针不交付正式适配器，不进入产品代码路径；
4. 隔离仓库无意外修改；Windows 锁屏时不向错误窗口输入内容。

#### Phase 3 Product MVP 验收（首个真实闭环）

1. 正式适配器接入（优先 Qoder CLI/稳定 SDK 的 worker + API/SDK 的 reviewer + repository + notifier）；
2. 测试仓库真实代码闭环完成；
3. 受控投递与对账（同一逻辑 message、幂等键、RECONCILIATION_REQUIRED）；
4. 工具 BUSY 不打断；
5. release gate 仅接受外部证据（worker 返回"完成"只生成待验证事件，不直接 `COMPLETED_APPROVED`）；
6. 不包含 Phase 4 的完整多审核与夜间自治要求。

#### Phase 4 无人值守与多审核验收

1. 双 reviewer / 多审核盲审，仲裁前互不可见；
2. evidence_verifier 复现高风险问题；
3. adjudicator 逐项裁决；
4. 有限自动修复（达到轮次上限进入安全终态）；
5. 登录失效时不要求用户立即选择，自动切换备用或安全暂停；
6. 长时间无人值守运行；
7. 登录失效、网络中断、锁屏、CI 失败等故障注入通过；
8. 零打扰最终摘要。

---

## 22. 主要风险与缓解措施

| 风险 | 影响 | 缓解 |
|---|---|---|
| GUI 更新导致选择器失效 | 桌面适配器中断 | 版本健康检查、UIA 多属性选择器、备用适配器、安全停用 |
| 多 AI 产生相同幻觉 | 虚假通过 | 盲审、模型多样性、证据验证、确定性门禁 |
| 过度通用导致 MVP 失控 | 长期无法交付 | 通用协议、窄实现；先 Core Prototype 再一个真实闭环（§19） |
| 插件窃取数据 | 严重安全事故 | 独立进程、最小权限、签名、域名/路径白名单 |
| 网页自动化违反服务变化或条款 | 通道失效 | 官方 API 优先、适配器可替换、条款审查 |
| Windows 锁屏导致 GUI 失败 | 夜间任务中断 | 专用 VM、Session Guard、备用云/API 适配器 |
| 用户不介入导致任务阻塞 | 延迟交付 | 自动备用、明确安全终态、第二天摘要 |
| 费用失控 | 经济损失 | 每任务/每日/全局三层预算、模型路由、去重、缓存 |
| n8n 等许可证影响商业化 | 法律风险 | MVP 不嵌入受限核心；优先 MIT/Apache/BSD 依赖并审计（§23.3） |
| ARH 自身被 AI 修改政策 | 安全边界失效 | 政策签名、只读加载、代码与配置分权、独立审查；可信根损坏进 safe mode（§15.5） |
| 跨第三方 exactly-once 误承诺 | 对账失败 | 删除该能力声明，仅承诺 at-least-once + 对账（§10.2/§16.3） |

---

## 23. 商业化路线

### 23.1 产品层级设想

- Personal：单机、少量适配器、本地 SQLite；
- Pro：NAS 控制中心、多个 Runner、多审核委员会、模板；
- Team：多人、角色权限、集中凭据、团队审计；
- Enterprise：私有部署、SSO、合规、插件签名、支持 SLA；
- Marketplace：适配器和工作流模板分成（前置 = Phase 7 之后 + 插件商店 + 签名 + 分账基础设施，见路线图 **F-C19**）。

### 23.2 可变现能力

- 可视化配置和易用安装；
- 稳定的 Windows/浏览器适配器；
- 多模型审核模板；
- 可靠的长任务恢复；
- 企业审计和权限；
- 行业模板；
- 私有化部署；
- 适配器开发与支持服务。

> **护城河与定价挂钩（路线图 F-C18）**：补技术/数据/网络效应壁垒思考；定价与价值度量（席位/工作流/适配器/节省人力）挂钩，反推 MVP 付费功能。

### 23.3 许可证原则（V1.1 修订 — 许可模型）

Phase 0–4：

- 产品源码保持**私有、专有**；
- **暂不承诺 Open Core**；
- 核心依赖优先 **MIT、Apache-2.0、BSD**；
- **GPL、AGPL、SSPL、BUSL、Sustainable Use License** 等组件在完成逐项法律兼容审查前，不得随产品捆绑分发；
- **Phase 7 产品化前重新评估**开源、Open Core 或全专有商业模式。

不得擅自把 ARH 改成 Apache-2.0 开源项目（不得直接采用 Reviewer-1 推荐的"核心 Apache-2.0 + 商业版专有"作为既定结论）。

- 所有第三方依赖生成 SBOM（**CycloneDX**，CI 门禁，路线图/必交付 **F-C16**）；
- **任何第三方组件必须按实际使用版本、具体文件路径和对应许可证文本逐项审查；不得使用 fair-code、source-available 等营销或类别标签代替正式许可证名称。对于适用 Sustainable Use License 或其他受限许可证的组件，未经明确兼容性审查和授权，不得随商业产品捆绑分发。**
- 每个插件独立声明许可证；
- 产品名称、图标和文案不侵犯外部厂商商标；
- 自动化第三方网页和桌面产品前审查其服务条款。

---

## 24. 项目治理

### 24.1 Source of Truth

项目根目录必须包含：

- SOURCE_OF_TRUTH.md
- PRODUCT_PROPOSAL.md（本文档在通过一致性审核前仅为候选稿）
- ARCHITECTURE.md
- PROTOCOL.md
- SECURITY.md
- ROADMAP.md
- DECISIONS/（ADR）
- LICENSE.md（产品许可证文本；本轮不要求创建该文件，但它是 Phase 0 设计冻结与进入 Phase 1 前必须存在的文档，未完成时 Phase 0 不得关闭）
- THIRD_PARTY_LICENSES.md（第三方依赖许可证汇总；同上，Phase 0 前必须存在）
- SBOM（生成与存放规则或明确指针，标准 CycloneDX，见 §23.3 F-C16；本轮不要求生成产物，但生成/存放规则须在 Phase 0 定义，未完成时 Phase 0 不得关闭）

> **许可证 Source of Truth 文件（V1.1 新增，F-C1 补充）**：`LICENSE.md`、`THIRD_PARTY_LICENSES.md` 与 SBOM 生成/存放规则是 Phase 0 设计冻结的必交付文档，也是进入 Phase 1 的前置条件。本轮仅将此项写入设计，不创建实际文件、不编写产品代码。

文档冲突时按明确优先级处理，不从聊天记录恢复产品事实。

### 24.2 变更流程

- 产品政策和权限变更必须独立 commit；
- 协议变更需要版本号和迁移说明；
- 安全边界不得由普通功能 PR 顺带修改；
- 适配器更新必须重跑合约测试；
- 每个交付固定 target SHA；
- 审核和修复不得 amend/rebase 已审计历史；
- 任何"完成"声明必须附带可复现证据；
- 政策包使用签名/可信锚点，篡改即进入 safe mode（§15.5）。

### 24.3 AI 执行者约束（V1.1 强化状态权威）

WorkBuddy、Qoder 或未来替代工具都是执行者，不是产品所有者：

- 不能自行扩大范围；
- 不能跳阶段；
- 不能修改授权政策；
- 不能以"更方便"为由降低安全；
- 不能用测试数量代替测试质量；
- 不能在未验证时宣称成功；
- 不能因为用户不在线而进行 A5 操作；
- **均不是状态权威**（见 §12.4）：只能提交声明、结果与证据，不能改写事件账本中的工作流事实，也不能改写已签名政策包中的授权/默认决策规则；发布门禁、权限、预算、路由的唯一裁决者为政策引擎。

---

## 25. 首个实际用例：软件开发闭环

> **阶段边界（V1.1 修正）**：§25.1 是 **Phase 3 Product MVP 最小闭环**（仅需一个独立 reviewer）；双 reviewer、evidence_verifier、adjudicator、盲审、仲裁与夜间无人值守属于 **Phase 4 完整参考闭环（§25.2）**，**不是 Phase 3 的进入条件，不得阻塞 Phase 3 首个真实闭环**。

### 25.1 Phase 3 Product MVP 最小闭环

初始映射（仅一个独立 reviewer）：

```text
trigger       = repository/webhook
planner       = 可选
reviewer      = 一个独立 reviewer（优先 API/SDK）
worker        = qoder_cli / 稳定 Agent SDK（本地 worker 执行通道；UIA 仅作为补充回退）
verifier      = 本地测试/CI
repository    = GitHub API 或等价 repository adapter
notifier      = webhook/API notifier
```

流程：

```text
触发
→ 固化目标 SHA
→ worker 隔离执行
→ verifier 确定性验证
→ 一个独立 reviewer 审查
→ 有界修复
→ 新 SHA 验证
→ release gate
→ 明确终态
```

> **证据门槛（V1.1）**：release_gate 仅接受 `evidence_verifier` 确定性输出或外部可验证状态；worker/任意角色返回"完成"只能生成待验证事件，**不能直接把任务改为 `COMPLETED_APPROVED`**（见 §12.4）。

### 25.2 Phase 4 完整参考闭环

以下为 **Phase 4** 能力，属于完整多审核与夜间无人值守，**不是 Phase 3 Product MVP 的进入条件，不得阻塞 Phase 3 首个真实闭环**：

初始映射（多审核）：

```text
trigger       = GitHub
planner       = 可配置大脑
reviewer_1    = 正确性审核模型
reviewer_2    = 安全审核模型
evidence_verifier = 测试运行器/静态扫描/沙箱
adjudicator   = 独立仲裁模型
worker        = qoder_cli / 稳定 Agent SDK（本地 worker 执行通道；UIA 仅作为补充回退）
verifier      = 本地测试/CI
messenger     = ARH 内置中继
notifier      = 飞书摘要
```

自动运行：

1. 发现候选 SHA；
2. 固化源码、diff、manifest 和哈希；
3. 独立发送给两位 reviewer（盲审，仲裁前互不可见）；
4. 验证结构化结果并合并 finding；
5. 高风险问题交 evidence_verifier 证据验证；
6. adjudicator 仲裁形成修复合同；
7. worker 在隔离分支修复；
8. verifier 运行测试和 CI；
9. 新 SHA 进入一次复审；
10. 通过则发布候选；未通过且达到上限则安全终止；
11. 用户只收到最终完成或阻塞摘要（夜间无人值守）。

---

## 26. 立项决策建议

### 建议立项，但必须遵守三条原则

1. **架构通用，首个实现必须窄。** 先跑通 Core Prototype（内核 + 模拟），再验证一个真实闭环，不同时支持十种模型和工具。
2. **先证明桌面/浏览器可控，再开发完整产品。** UIA 或浏览器探针（Phase 2）失败时及时止损。
3. **全权代理依靠政策，不依靠无限权限。** 预授权范围内完全自主；范围外自动拒绝并安全收尾，不把用户重新拉回调度岗位。

### 推荐的第一开发目标

先实现内核、模拟适配器和隔离测试仓库闭环（Phase 1），再经 Phase 2 探针验证一个桌面工具、一个浏览器工具和 Git，Phase 3 才形成首个真实闭环。不得直接用 STS 项目作为自动化试验场。

---

## 27. 给执行团队的启动指令

在本项目书（V1.1）完成独立一致性审核并成为 Source of Truth 前：

- 仅允许创建/更新 `E:\Agent Relay Hub` 独立仓库中的**设计文档、ADR、协议 Schema、测试计划与 V1.1 候选稿**；
- 禁止实现真实 Qoder、ChatGPT、GitHub 或 WorkBuddy 适配器；
- 禁止修改 `E:\STS\project`；
- 禁止连接 OKX 或任何交易接口；
- 禁止部署 NAS；
- 禁止创建计划任务、开机启动或无人值守权限；
- 禁止以原型成功替代产品审核；
- **禁止擅自进入 Phase 1 或编写任何产品代码**。

下一步唯一任务：对 V1.1 候选稿进行独立一致性审核（对照 V1.0 与冻结 changeset `3e9a059...`），通过后纳入 Source of Truth，再据 §20 阶段门禁推进。

---

## 附录 A：零打扰自主运行契约

ARH 必须满足：

1. 用户不承担传话、轮询、重复确认和普通冲突裁决；
2. 系统不向用户发送选择题；
3. 系统按预授权政策选择工具、备用工具和终态；
4. 任务成功后只发送一次摘要；
5. 任务无法安全继续时进入安全终态并只发送一次摘要；
6. 无用户回复不会触发危险默认值；
7. 身份验证、支付、生产实盘和不可逆删除不自动绕过；
8. 用户第二天可以查看证据，但不需要半夜介入；
9. 所有自动决策可追溯、可解释、可回放；
10. 系统宁可安全停止，也不能为了"全自动"伪造完成。

## 附录 B：成功标准

当用户可以在晚上启动一个已授权工作流，第二天只看到以下两种结果之一时，ARH 才算实现目标：

### 成功

```text
任务已完成
目标版本：...
执行者：...
审核委员会：...
测试与证据：通过
交付物：...
未执行任何越权操作
```

### 安全阻塞

```text
任务已安全停止
停止原因：...
已完成阶段：...
未完成事项：...
代码/数据保持在安全版本
未盲发重复尝试、未越权、未进入下一阶段
```

而不是：

```text
请选择 A/B/C，我才能继续。
```

---

> **路线图声明**：本文档 §20 后续阶段（Phase 5 STS、Phase 6 NAS、Phase 7 商业版）、§23.1 Marketplace 前置、§23.2 护城河定价、国际化策略（i18n）等属于 **P2 路线图项**，当前未实现、不描述为已实现能力。详见 `V1.1_TRACEABILITY_MATRIX.md`。
