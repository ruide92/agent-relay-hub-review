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
- `success`
- `warning`
- `danger`
- `security`
- `reconciliation`
- `blocked`
- `disabled`

强调规则：

- **`success` 只能用于可验证的成功**（evidence verified / terminal completed）；
- **ACK 不得使用 success**（adapter ACK 不是完成，UX_INTERACTION.md §6）；
- **waiting evidence 使用中性或信息态**（非 success）；
- **safe mode 必须与普通 error 清楚区分**（用 `security` 语义，非 `danger`）。

---

## 4. 核心组件

每个组件定义 purpose / required data / variants / forbidden usage / accessibility / phase availability。

| 组件 | Purpose | Required Data | Variants | Forbidden Usage | Phase |
|---|---|---|---|---|---|
| App Shell | 导航框架 | 角色/权限 | default | 承载状态权威 | 1 |
| Global Health Banner | 全局健康 | 健康聚合 | OK/DEGRADED/SAFE MODE | 显示 secret | 1 |
| Status Badge | 非终态状态 | state enum | pending/running/stalled… | 仅靠颜色 | 1 |
| Terminal State Badge | 终态 | terminal enum | completed/blocked… | 伪造成功 | 1 |
| Workflow Card | 工作流摘要 | workflow projection | list/detail | 写事实 | 1 |
| Evidence Badge | 证据状态 | evidence status | pending/verified | 标 success 当 pending | 1 |
| Finding Card | finding 摘要 | finding + severity | HIGH/CRITICAL | 隐藏严重度 | 1 |
| Timeline | 证据化时间线 | 节点数组 | horizontal/vertical | 改事实 | 1 |
| Step Detail Drawer | 节点详情 | node data | collapsed/expanded | —— | 1 |
| Adapter Health Card | 适配器健康 | lifecycle + caps | loading/ready/busy/degraded/failed | 重命名 FAILED | 1 |
| Budget Meter | 预算 | used/limit | normal/warn/exhausted | 超上限仍绿 | 1 |
| Policy Summary | 政策摘要 | policy ref | loaded/safe-mode | 显示密钥值 | 1 |
| Authorization Scope Card | A5 范围 | scope + status | waiting/approved/rejected | 默认批准 | 1 |
| Reconciliation Case Card | 对账案例 | RECONCILIATION_REQUIRED | pending/resolved | 盲重试按钮 | 1 |
| Audit Event Row | 审计行 | hash-chain event | normal/rejected | 显 secret | 1 |
| Safe Mode Panel | 安全模式 | safe mode state | active/inactive | 暴露 A1–A5 控件 | 1 |
| Notification Center | 通知 | 五级通知 | silent/in-app/summary/attention/critical | 含敏感 payload | 1 |
| Empty State | 空态 | 上下文 | list/detail | —— | 1 |
| Error State | 错误态 | error model | retry/safe | 伪装成功 | 1 |
| Skeleton | 加载占位 | 结构 | —— | 假进度 | 1 |
| Confirmation Dialog | 确认 | 精确范围 | normal/destructive | "是否继续？" | 1 |

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
- `verified` → 勾选图标 + "已验证" + `success` 语义（仅此处可用 success）；
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

至少包含：

- **WCAG AA** 目标对比度；
- **keyboard navigation**（全组件可聚焦 / 可操作）；
- **focus management**（对话框/抽屉陷阱焦点）；
- **screen reader labels**（状态均有 accessible label，见 §5）；
- **reduced motion**（尊重 `prefers-reduced-motion`）；
- **200% scaling**（布局不破损）；
- **不依赖颜色**（状态有文本/形状，见 §5）；
- 表格和时间线的**替代阅读顺序**（DOM 顺序或 aria 顺序可理解）。
