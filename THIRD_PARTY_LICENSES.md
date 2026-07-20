# THIRD PARTY LICENSES（第三方许可证受控登记表）

> 依据：已批准 V1.1 许可政策（§23.3）与 `LICENSE.md`。
> **当前 Phase 0 尚无产品代码和正式依赖，本登记表为空的受控登记表。**

---

## 登记规则

- **不允许用 fair-code、source-available 等类别标签代替正式许可证名称**（License Name 必须为正式许可证名称，如 `MIT`、`Apache-2.0`、`BSD-3-Clause`、`GPL-3.0-only`、`AGPL-3.0-only`、`Sustainable Use License` 等）。
- 必须按**实际使用版本、具体文件/包路径和对应许可证文本**逐项审查。
- **未审查组件不得进入产品依赖。**
- 每个组件必须填写 License Text Source（许可证文本的实际来源，如仓库路径、发行包内 LICENSE 文件、官方 SPDX 链接）与 Evidence（审查证据）。
- **当前无产品代码和正式依赖，登记表不得伪造任何组件记录。** 空表即为当前真实状态。
- Status 取值建议：`PENDING_REVIEW` / `APPROVED` / `REJECTED` / `BLOCKED`。
- 受限许可（GPL / AGPL / SSPL / BUSL / Sustainable Use License 等）默认 `BLOCKED`，须经逐项法律兼容审查与授权方可改为 `APPROVED`。

---

## 登记表

| Component | Version | File/Package Path | License Name | License Text Source | Intended Use | Distribution Mode | Compatibility Decision | Reviewer | Evidence | Status |
| --------- | ------- | ----------------- | ------------ | ------------------- | ------------ | ----------------- | ---------------------- | -------- | -------- | ------ |
| _(无记录 — 当前 Phase 0 无产品代码与正式依赖)_ | | | | | | | | | | |

---

> 当 Phase 1 开始出现代码与依赖时，每引入一个组件必须先在本表完成逐项审查并置为 `APPROVED`，方可进入产品依赖。相关软件物料清单生成政策见 `SBOM_POLICY.md`。
