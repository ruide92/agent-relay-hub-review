# Reviewer-2 独立 adjudication（第二轮独立核验）

> 核验对象：commit `1db8f421ccadd0828ecde00df7db265dd2e1fe56`
> 被审查基线：`AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.0.md`（Git blob SHA `31b7287764e4eaf008b0f87852596e33bcfb4af3`）
> 关联文档：`PROJECT_PROPOSAL_REVIEW.md`、`V1.1_REQUIRED_CHANGES.md`、`IMPLEMENTATION_READINESS_REPORT.md`（均为 Reviewer-1 第一轮意见，非最终裁决、非 Source of Truth）
> 本轮性质：**仅文档 adjudication 与最终 changeset，不修改 V1.0、不编写 V1.1 正文、不编写代码、不实现任何真实适配器**

---

## 0. 验收结论

- commit `1db8f421ccadd0828ecde00df7db265dd2e1fe56` **验收通过**。
- 仓库中 `AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.0.md` 与原始上传文件的 Git blob SHA 完全一致，均为 `31b7287764e4eaf008b0f87852596e33bcfb4af3`。
- 确认：V1.0 正文未修改、未写代码、三份报告的 Reviewer-1 状态说明与 README 均符合要求。

---

## 1. 总体可行性裁决

ARH 在架构上具备可行性，但必须遵守以下通道优先级与边界：

- **API / 官方 SDK / CLI 优先**；
- **浏览器 DOM / Playwright 次之**；
- **Windows UI Automation 再次**；
- **屏幕视觉、鼠标坐标和键盘模拟只允许作为最后降级手段**；
- **GUI 通道不得成为无人值守工作流的唯一完成路径**。

当前 Qoder 已有官方 CLI、headless 自动化和 Agent SDK，因此 **Qoder 的首个真实适配器应优先使用 CLI/SDK，不应优先控制 Qoder 桌面窗口**。

---

## 2. 对 Reviewer-1 C1–C21 的裁决

### C1 ｜ 许可证 — 修正后接受
Phase 0–4 暂定：
- ARH 产品源码保持私有、专有；
- 暂不承诺 Open Core；
- 核心依赖优先 MIT、Apache-2.0、BSD；
- GPL、AGPL、SSPL、BUSL、Sustainable Use License 等组件在完成逐项法律兼容审查前，不得随产品捆绑分发；
- Phase 7 产品化前重新评估开源、Open Core 或全专有商业模式。

不得直接采用 Reviewer-1 推荐的"核心 Apache-2.0 + 商业版专有"作为既定结论。

### C2 ｜ MVP 收窄 — 接受收窄方向，但修正阶段定义
区分两个概念：
- `Core Prototype`：内核原型；
- `Product MVP`：具备真实产品价值的首个闭环。

Phase 1 只允许：状态机；SQLite 事件账本；政策引擎；模拟适配器；模拟工作流；防重、预算、循环限制、崩溃恢复和终态测试。
**Phase 1 不包含**真实 Git、GitHub、浏览器、UIA、Qoder、ChatGPT 或 WorkBuddy 适配器。

Phase 2 只做可行性探针：本地 Git 探针；Qoder CLI/SDK 探针；Playwright 浏览器探针；Windows UIA 探针；锁屏、UAC、焦点和会话状态探针。
Phase 3 才实现一个真实代码开发闭环。
Phase 4 才加入多审核、证据验证、仲裁和夜间无人值守。

### C3 ｜ 策略完整性 — 接受，但补充可信根
政策文件仅保存普通哈希不够，攻击者可以连政策和哈希一起修改。必须规定：
- 政策包使用签名或由操作系统保护的可信哈希锚点；
- 信任根不得与可修改政策文件放在同一权限域；
- 政策签名或可信锚点校验失败后进入 `POLICY_INTEGRITY_SAFE_MODE`；
- safe mode 仅允许本地 A0：查看本地状态、查看审计记录、导出脱敏诊断、由人工修复或重新签署政策；
- 默认禁止所有外部网络请求、模型调用、CLI 命令执行、Git 写入、浏览器自动化和桌面自动化；
- 不自动切换备用适配器；
- 不自动恢复 A1–A5；
- 只有可信政策重新加载并通过校验后才能退出 safe mode。

### C4 ｜ 持久化计数器 — 接受
所有 message_id、SHA、尝试次数、修复轮数、仲裁次数、备用切换次数、预算消耗均必须持久化，重启后不得归零。

### C5 ｜ 授权天花板 — 修正后接受
不得把插件自报的 `authorization_tier_ceiling` 当成安全边界。应分为：
- 插件上报：技术能力；
- 政策引擎配置：允许权限上限；
- 能力令牌：本次调用的实际最小权限。

最终授权上限由 ARH 政策引擎保存和裁决，插件、协调器和模型均无权自行提高。

### C6 ｜ 政策缺口终态 — 修正后接受
新增 `BLOCKED_NEEDS_POLICY_DECISION` 前，必须先执行默认决策阶梯：
1. 可安全跳过则跳过；
2. 有保守默认值则采用；
3. 有等价备用工具则切换；
4. 可降级完成则降级完成；
5. 以上全部不成立，才进入 `BLOCKED_NEEDS_POLICY_DECISION`。

系统必须同时生成建议新增的政策规则，不能只是把选择题延迟到第二天重新交给用户。

### C7 ｜ 非代码证据 — 接受推荐方案 A
MVP 仅做代码开发闭环。文档、研究、营销和数据分析工作流改为未来候选，等各自证据类型定义后再进入实现。

### C8 ｜ 备份/审计 — 拆分处理
Phase 1 必须完成：SQLite 一致性备份；WAL checkpoint 策略；产物备份；恢复演练；事件重放一致性校验。外部不可变审计锚定属于 Team/Enterprise，不能阻塞 Phase 1。

### C9–C15 ｜ 原则接受，但修正
- 最大执行时间按工作流模板设置，必须有默认值和绝对上限；
- 严重度无法裁决时：若同一 finding 的可信严重度候选中包含 `CRITICAL` 或 `HIGH`，且证据验证无法排除该高风险候选，则发布门禁阻断；只有全部可信候选严重度都低于发布门禁，才允许进入 `DEFERRED`；证据不足不能自动降级严重度；`DEFERRED` 必须记录候选严重度、未决原因和重新评估条件；
- SDK 生命周期、错误码、取消、恢复和健康检查必须在 Phase 0 文档化。

### C16–C21
- C16、C17 应纳入 V1.1；
- C18、C19、C21 可留路线图；
- C20"子代理权限继承"由 P2 提升为 P0：子代理不得获得高于父代理的权限、预算、路径、域名或凭据范围。

---

## 3. 第二审核员新增发现（R2-01 ~ R2-09）

### R2-01 技术栈升级（P0）
V1.0 中 `.NET 8` 改为 `.NET 10 LTS`。
原因：当前时间为 2026 年 7 月，.NET 8 将于 2026 年 11 月结束支持，而 .NET 10 LTS 支持到 2028 年 11 月。

### R2-02 适配器优先级（P0）
统一规定：
`Official API/SDK > CLI > Webhook > Playwright DOM > UIA > Visual/Coordinate Automation`
不得因为桌面界面"看得见"就优先采用 UIA。

### R2-03 Qoder 实现路线（P1）
Phase 2 以本地 Qoder CLI 和稳定 Agent SDK 为基线，优先验证：Qoder CLI；Qoder headless；Qoder Agent SDK；结构化输出；流式事件；取消和恢复；工具权限；进程退出码；会话关联。任何标记为 experimental / unstable 的云端能力不得成为正式闭环的唯一依赖。桌面 UIA 只作为 Qoder CLI/SDK 无法覆盖功能的补充通道。

### R2-04 取消 exactly-once 承诺（P0）
删除或废弃：`supports_exactly_once_claim`
改为：`supports_idempotency_key`、`supports_status_query`、`supports_external_correlation_id`、`supports_reconciliation`、`delivery_semantics`

ARH 对任意外部系统只能保证："至少一次尝试、幂等优先、状态对账、无法确认时安全停止。"不得宣传跨任意第三方系统的 exactly-once。

### R2-05 事务型 Outbox（P0）
Outbox 保证内部状态变更与"待投递意图"在同一数据库事务中提交；外部动作记录至少区分：`PREPARED`、`DISPATCHING`、`ACKNOWLEDGED`、`VERIFIED`、`RECONCILIATION_REQUIRED`。崩溃恢复后不得盲目重发。若适配器支持 idempotency key、external correlation ID 或 status query，必须先查询并对账；若无法确定动作是否已经执行，进入 `RECONCILIATION_REQUIRED` 或对应安全终态。不得宣称 Outbox 能对任意第三方外部副作用实现 exactly-once。

### R2-06 浏览器凭据（P0）
Playwright storage state、Cookie、localStorage、浏览器配置目录和会话令牌全部视同凭据：
- 不得进入 Git；
- 不得进入普通产物仓库；
- 不得出现在 prompt、日志和截图；
- 使用独立受保护目录；
- 支持撤销、过期和清理。

### R2-07 WebView2 边界（P1）
WebView2 只负责 ARH 的管理界面和仪表盘。第三方网页自动化必须由独立 Playwright Browser Runner 执行，使用专用浏览器上下文，不与 ARH UI WebView2 或用户个人浏览器混用。

### R2-08 桌面自动化门禁（P1）
UIA 探针必须验证：锁屏；UAC；权限级别不同；窗口失焦；RDP 最小化或断开；应用更新；多窗口同名；输入法与剪贴板污染；目标进程崩溃；UI 元素变化。任何一项可能导致错误窗口写入时，立即停止桌面通道，不允许盲输。

### R2-09 状态权威与政策裁决权威（P0）
- 事件账本是工作流事实、外部动作意图、投递结果和审计事件的唯一 Source of Truth；
- 当前状态视图、仪表盘和查询表均为可从事件账本重建的派生 projection，不得反向覆盖事件事实；
- 已签名政策包是授权边界与默认决策规则的 Source of Truth；
- 政策引擎是权限、预算、工具路由和发布门禁的唯一裁决者，但不保存或伪造工作流事实；
- 外部 worker、reviewer、messenger、repository 和模型只能提交声明、结果与证据，不能直接改写权威状态或政策。

---

## 4. 与 Reviewer-1 的矛盾裁定

Reviewer-1 的 `IMPLEMENTATION_READINESS_REPORT.md` 存在一处直接矛盾：§6 既要求 Phase 1"包含 1 个真实 Git 适配器"，又在同一节"明确禁止真实 GitHub 适配器"。

本 adjudication 裁定：原始 V1.0 将 Git 探针归入 **Phase 2**，Phase 1 仅为内核 + 模拟适配器 + 假工作流；因此 **Phase 1 不含任何真实适配器**（含 Git / GitHub）。Reviewer-1 的上述矛盾表述以本裁决为准，不作为执行指令。最终可执行清单见 `V1.1_CHANGESET_FINAL.md`。
