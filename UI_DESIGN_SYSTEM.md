```text
Document Status: PROPOSED
Normative Status: 未批准，当前不是 Source of Truth
Product Baseline: AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md
Product Baseline Commit: c9cc522b4ab01008fed390c282d6bd5a816ee779
Architecture Baseline Approval Target: b8ef21978544e1261081389ecf736e72b80e49d7
Governance Approval Record: GOVERNANCE_APPROVAL_0002.md
Current Phase: Phase 0
Phase 1: NOT AUTHORIZED
Code Status: NO PRODUCT CODE
Implementation Status: DESIGN ONLY
Last Revised: 2026-07-21 (Reviewer-2 Round 1)
```

# UI_DESIGN_SYSTEM.md（设计系统）

> 本文档定义 ARH 视觉与组件语言，**不选择或安装真实依赖**。
> 当前为 `PROPOSED`，未经 Reviewer-2 审核与 owner 批准前**不属于 Source of Truth**；
> 不得描述为已批准、已实现或已测试；不得因完成 UI 设计就关闭 Phase 0。
> 所有内容必须服从已批准基线；技术候选库仅为候选，非已批准依赖。

---

## 1. 设计方向

关键词（整体像现代控制中心，而非聊天机器人或加密货币大屏）：

- calm（冷静）
- precise（精确）
- trustworthy（可信）
- operational（运维导向）
- evidence-first（证据优先）
- **not chat-first**（不以聊天为主）
- **not cyberpunk**
- **not gaming dashboard**
- **not overly decorative**（不过度装饰）

---

## 2. 技术候选边界（不得安装）

可评估但**不得安装**的候选库：

- React
- WebView2（仅作为 ARH 自身管理界面宿主，ARCHITECTURE.md §9）
- Fluent UI React v9
- Storybook
- XState
- React Flow

规则：

- 均为**候选**，不是已批准依赖；
- 正式引入 MUST 经第三方许可证登记（THIRD_PARTY_LICENSES.md）、SBOM（SBOM_POLICY.md）和供应链审查（SECURITY.md §11）；
- **XState 只允许管理临时前端交互状态**；业务事实仍来自 Event Ledger（ARCHITECTURE.md §2）；
- **React Flow Phase 1 只允许只读或不采用**；正式编辑器至少 Phase 3；
- 不得把任一库写成**核心领域依赖**（ARCHITECTURE.md §2：内核与 UI 解耦）。

---

## 3. Design Tokens（候选，未硬编码品牌色）

至少定义候选 token（语义化，未定最终品牌色）：

- typography（字阶 / 字重）
- spacing（间距尺度）
- radius（圆角）
- elevation（阴影 / 层级）
- motion（动效时长 / 缓动）
- density（密度）
- focus ring（焦点环）
- border（边框）
- surface（表面层级）
- semantic status（语义状态色）

语义 token（强调）：

- `neutral`
- `info`
- `success.evidence_verified`
- `success.terminal_completed`
- `warning`
- `danger`
- `security`
- `reconciliation`
- `blocked`
- `disabled`

强调规则：

- **`success` 必须拆分为两个明确语义，不得混用**：
  - **`success.evidence_verified`**：仅表示**某项 evidence 已验证**，不代表整个 workflow 完成；
  - **`success.terminal_completed`**：仅用于**有终态 + 所需 evidence 的完成**（如 `COMPLETED_APPROVED` / `COMPLETED_WITH_DEFERRED_LOW_RISK`）；
  - 两者 MUST 有**不同文字与 accessible label**，不得仅靠相同绿色表达；
- **ACK / result received / verification pending / release gate pending 均不得使用任何 success token**（UX_INTERACTION.md §6、UI_STATE_ACCEPTANCE_MATRIX.md §3）；
- **waiting evidence 使用中性或信息态**（非 success）；
- **safe mode 必须与普通 error 清楚区分**（用 `security` 语义，非 `danger`）。

---

## 4. 核心组件

每个组件定义 purpose / required data / variants / forbidden usage / accessibility / phase availability。

| 组件 | Purpose | Required Data | Variants | Forbidden Usage | Accessibility Contract | Phase |
|---|---|---|---|---|---|---|
| App Shell | 导航框架 | 角色/权限 | default | 承载状态权威 | name="ARH 导航框架"；announcement 导航切换播报；keyboard Tab/方向键聚焦导航项；focus 当前页入口有焦点环；live region 否；non-color 当前页下划线+图标+文本 | 1 |
| Global Health Banner | 全局健康 | 健康聚合 | OK/DEGRADED/SAFE MODE | 显示 secret | name="全局健康状态"；announcement 状态变更(OK/DEGRADED/SAFE MODE) 经 aria-live；keyboard 非交互；focus 整条可聚焦显示文本；live region 是(assertive for SAFE MODE)；non-color 图标+文本+横幅形状 | 1 |
| Status Badge | 非终态状态 | state enum | pending/running/stalled… | 仅靠颜色 | name=状态名(如"运行中")；announcement 状态文本；keyboard 静态可选中；focus 可选中复制；live region 是(状态变更)；non-color 图标+文本+虚/实线边框 | 1 |
| Terminal State Badge | 终态 | terminal enum | completed/blocked… | 伪造成功 | name=终态名；announcement "工作流已[完成/阻断]"；keyboard 静态；focus 可选中；live region 是；non-color 图标(✔/⛔)+文本+形状 | 1 |
| Workflow Card | 工作流摘要 | workflow projection | list/detail | 写事实 | name="工作流 WF-xxx 摘要"；announcement 状态+进度；keyboard Enter 打开详情；focus 卡片可聚焦；live region 否(详情页播报)；non-color 状态 badge | 1 |
| Evidence Badge | 证据状态 | evidence status | pending/verified | 标 success 当 pending | name="证据 ev-xx 状态"；announcement pending/verified；keyboard 静态；focus 可选中；live region 是；non-color 图标+文本(verified 用 `success.evidence_verified` 语义) | 1 |
| Finding Card | finding 摘要 | finding + severity | HIGH/CRITICAL | 隐藏严重度 | name="finding xx 严重度"；announcement "HIGH/CRITICAL finding"；keyboard Enter 打开；focus 卡片；live region 否；non-color 严重度图标+文本+边框 | 1 |
| Timeline | 证据化时间线 | 节点数组 | horizontal/vertical | 改事实 | name="证据化时间线"；announcement 节点状态变更；keyboard 节点可聚焦+方向键遍历；focus 节点可聚焦；live region 否；non-color 节点图标+文本+连接线形状 | 1 |
| Step Detail Drawer | 节点详情 | node data | collapsed/expanded | —— | name="节点 xx 详情"；announcement 打开/关闭；keyboard Esc 关闭, Tab 陷阱；focus 打开移入, 关闭回触发点；live region 否；non-color 标题+文本 | 1 |
| Adapter Health Card | 适配器健康 | lifecycle + caps | loading/ready/busy/degraded/failed | 重命名 FAILED | name="适配器 mock-xx 健康"；announcement lifecycle 变更；keyboard Enter 打开；focus 卡片；live region 否；non-color lifecycle 图标+文本 | 1 |
| Budget Meter | 预算 | used/limit | normal/warn/exhausted | 超上限仍绿 | name="预算 已用/上限"；announcement 警告/耗尽；keyboard 静态；focus 可选中；live region 是(耗尽)；non-color 数值文本+进度条+警戒图标 | 1 |
| Policy Summary | 政策摘要 | policy ref | loaded/safe-mode | 显示密钥值 | name="政策 policy-xx"；announcement 加载/safe-mode；keyboard Enter 打开；focus 可选中；live region 否；non-color 状态文本+图标(非密钥值) | 1 |
| Authorization Scope Card | A5 范围 | scope + status | waiting/approved/rejected | 默认批准 | name="A5 范围: ..."；announcement waiting/approved/rejected；keyboard 提交；focus 范围可读；live region 是；non-color 状态图标+文本(默认非批准) | 1 |
| Reconciliation Case Card | 对账案例 | RECONCILIATION_REQUIRED | pending/resolved | 盲重试按钮 | name="对账 RC-xx"；announcement required/resolved；keyboard 查询；focus 卡片；live region 是；non-color 状态图标+文本(无盲重试按钮) | 1 |
| Audit Event Row | 审计行 | hash-chain event | normal/rejected | 显 secret | name="审计事件 hash"；announcement 正常/拒绝；keyboard 静态；focus 可选中；live region 否；non-color 状态图标+文本(非 secret) | 1 |
| Safe Mode Panel | 安全模式 | safe mode state | active/inactive（含重载批准 bundle 入口） | 暴露 A1–A5 控件；编辑/签署/修复 policy；自动生成 trust root | name="安全模式面板"；announcement 进入/退出 safe mode(assertive)；keyboard 导出/重载按钮可聚焦；focus 控件可操作；live region 是(aria-live=assertive)；non-color 锁图标+文本+警示形状(区别于 danger) | 1 |
| Notification Center | 通知 | 五级通知 | silent/in-app/summary/attention/critical | 含敏感 payload | name="通知中心"；announcement 新通知(count+级别)；keyboard 打开/标记已读；focus 列表；live region 是；non-color 级别图标+文本 | 1 |
| Empty State | 空态 | 上下文 | list/detail | —— | name="空态: ..."；announcement 空态文本；keyboard 创建按钮可聚焦；focus 创建按钮；live region 否；non-color 图标+文本 | 1 |
| Error State | 错误态 | error model | retry/safe | 伪装成功 | name="错误: ..."；announcement 错误描述；keyboard 重试/安全；focus 按钮；live region 是；non-color 图标+文本(不伪装成功) | 1 |
| Skeleton | 加载占位 | 结构 | —— | 假进度 | name="加载中"；announcement "内容加载中"(aria-busy)；keyboard 不捕获；focus 不捕获；live region 是(aria-busy)；non-color 占位形状(非假进度) | 1 |
| Confirmation Dialog | 确认 | 精确范围 | normal/destructive | "是否继续？" | name="确认: [精确范围]"；announcement 对话框打开+范围(role=dialog aria-modal)；keyboard Tab 陷阱, Esc 取消, Enter 确认；focus 打开移入默认按钮, 关闭回触发点；live region 是；non-color 标题+范围文本+按钮文本(非"是否继续?") | 1 |

通用 forbidden：不得仅以颜色区分状态（见 §5）；不得绕过 Policy Engine；不得直接写 SQLite/projection/policy bundle/终态。

---

## 5. 状态视觉规则

必须区分以下状态，**禁止仅靠颜色**，必须同时有 icon + text + shape/border + accessible label：

- pending
- running
- stalled
- degraded
- acknowledged
- verification pending
- verified
- completed
- blocked
- quarantined
- reconciliation required
- cancelled
- safe mode

示例：

- `verification pending` → 中性图标 + "待验证" 文本 + 虚线边框；
- `verified` → 勾选图标 + "已验证（evidence）" + `success.evidence_verified` 语义（仅表示 evidence 已验证，不代表 workflow 完成）；
- `completed` → 完成图标 + "已完成（终态）" + `success.terminal_completed` 语义（须有终态 + 所需 evidence，UI_STATE_ACCEPTANCE_MATRIX.md §3）；
- `safe mode` → 锁图标 + "安全模式" + `security` 语义（区别于 `danger` 的 error）；
- `acknowledged` → 收件图标 + "已确认" + 中性（**非 success**）。

---

## 6. 动效

只允许表达：

- 状态变化；
- 数据加载；
- 进度；
- 页面关系；
- 注意力引导。

禁止：

- 虚假进度（假装执行中）；
- 无限炫酷动画；
- 用动效**伪装任务仍在执行**；
- 终态未确认前播放**成功动画**（UX_INTERACTION.md §6）。

---

## 7. Accessibility

关键组件逐项可验收行为见 §4 Accessibility Contract 列。全局通用要求至少包含：

- **WCAG AA** 目标对比度；
- **keyboard navigation**（全组件可聚焦 / 可操作）；
- **focus management**（对话框/抽屉陷阱焦点）；
- **screen reader labels**（状态均有 accessible label，见 §5）；
- **reduced motion**（尊重 `prefers-reduced-motion`）；
- **200% scaling**（布局不破损）；
- **不依赖颜色**（状态有文本/形状，见 §5）；
- 表格和时间线的**替代阅读顺序**（DOM 顺序或 aria 顺序可理解）。
