# THIRD PARTY LICENSES（第三方许可证受控登记表）

> 依据：已批准 V1.1 许可政策（§23.3）与 `LICENSE.md`。
> **当前 Phase 0 尚无产品代码和正式依赖，本登记表为空的受控登记表。**

---

## 登记规则

- **不允许用 fair-code、source-available 等类别标签代替正式许可证名称**（License Name 必须为正式许可证名称，如 `MIT`、`Apache-2.0`、`BSD-3-Clause`、`GPL-3.0-only`、`AGPL-3.0-only`、`Sustainable Use License` 等）。
- 必须按**实际使用版本、具体文件/包路径和对应许可证文本**逐项审查。
- **未审查组件不得进入产品依赖。**
- **Status 仅允许以下固定取值，不得使用"建议"等模糊表述**：
  - `PENDING_REVIEW`：已登记，待审查；
  - `APPROVED`：已审查并批准（须有精确证据）；
  - `REJECTED`：已审查并否决；
  - `BLOCKED`：默认拦截（受限许可或待澄清），不得进入产品依赖。
- **受限许可**（GPL / AGPL / SSPL / BUSL / Sustainable Use License 等）默认 `BLOCKED`，须经逐项法律兼容审查与授权方可改为 `APPROVED`。
- **许可证证据来源**：主要证据须来自**实际使用版本**的发行包、源码仓库或制品中的 `LICENSE` / `NOTICE` / `COPYING` 文件；通用 SPDX 页面（如 spdx.org/licenses）**仅可作为标准化标识或辅助参考**，不得替代上述实际来源。
- 每个组件必须填写以下证据字段（见登记表）：`Source Artifact / Commit Hash`、`License File Path`、`License Text Source`、`License Text Hash`、`Reviewer`、`Review Commit`、`Approval Record`、`Evidence`。
- **没有精确证据（具体版本来源 + License File Path + License Text Hash + Review Commit + Approval Record）不得标记 `APPROVED`。**
- **当前无产品代码和正式依赖，登记表不得伪造任何组件记录。** 空表即为当前真实状态。

---

## 登记表

| Component | Version | File/Package Path | License Name | Source Artifact / Commit Hash | License File Path | License Text Source | License Text Hash | Intended Use | Distribution Mode | Compatibility Decision | Reviewer | Review Commit | Approval Record | Evidence | Status |
| --------- | ------- | ----------------- | ------------ | ----------------------------- | ---------------- | ------------------- | ----------------- | ------------ | ----------------- | ---------------------- | -------- | ------------- | --------------- | -------- | ------ |
| _(无记录 — 当前 Phase 0 无产品代码与正式依赖)_ | | | | | | | | | | | | | | | |

---

> 当 Phase 1 开始出现代码与依赖时，每引入一个组件必须先在本表完成逐项审查并置为 `APPROVED`（须具备上述精确证据），方可进入产品依赖。相关软件物料清单生成政策见 `SBOM_POLICY.md`。
