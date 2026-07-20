# SBOM POLICY（软件物料清单政策）

> 本文件**只定义政策，不生成任何 SBOM**。
> **当前 Phase 0 无产品代码与依赖，本轮只建立政策，不生成空壳 SBOM 冒充真实产物。**
> 依据：已批准 V1.1（`AGENT_RELAY_HUB_PROJECT_PROPOSAL_v1.1.md`）与 `LICENSE.md` / `THIRD_PARTY_LICENSES.md`。

---

## 1. 标准

- SBOM 采用 **CycloneDX** 标准格式。
- **CycloneDX schema / spec 版本必须在 CI 配置中固定**，并记录可升级追踪（升级须经治理提交并说明依据）。

---

## 2. 覆盖范围与组件类型

SBOM 必须覆盖以下全部组件类型，不得遗漏：

- **直接依赖**（direct dependencies）；
- **传递依赖**（transitive dependencies）；
- **runtime / build / dev / test 依赖**（按使用场景分别标注）；
- **插件**（plugins）；
- **容器基础镜像**（container base images）及其内层包；
- **纳入产品或构建环境的系统包**（system packages）。

---

## 3. 生成时机

- **Phase 1 开始出现代码和依赖后**才生成 SBOM；Phase 0 不生成。
- 每次以下事件均须（重新）生成并归档 SBOM：
  - 依赖新增、升级或移除；
  - 每个构建产物（build artifact）产出时；
  - 每个发布候选（release candidate）生成时。

---

## 4. 存放位置（规范性模式）

- SBOM 产物**必须**存放于仓库受控目录 `sbom/`（或制品仓库的受控路径），并纳入版本追溯。
- 文件名须包含生成时的 commit SHA 与产物标识，便于与构建产物、发布候选一一绑定。

---

## 5. 绑定关系

- 每份 SBOM 必须与以下绑定并可回溯：
  - 生成时的 **commit SHA**；
  - 对应的**构建产物**（含产物哈希）；
  - 对应的**发布候选**标识。

---

## 6. 对账与完整性

- 每份 SBOM **必须与 lockfile、包管理器解析图（resolution graph）和实际构建输入三方对账**。
- **缺失已声明组件，或出现未声明组件（幽灵依赖），CI 门禁失败**（fail-closed），不得跳过或降级。

---

## 7. CI 门禁

- CI 流水线必须在构建阶段生成 SBOM 并执行**依赖许可证校验**（对照 `THIRD_PARTY_LICENSES.md`：未审查或 `BLOCKED` 许可证的组件出现即失败）。
- **校验失败时阻止发布**（fail-closed），不得跳过或降级。

---

## 8. 完整性与签名

- 每份 SBOM 必须计算**哈希**并进行**签名**，以保证完整性与来源可验证。
- 篡改或哈希不匹配视为门禁失败。
- **SBOM 签名密钥、信任根（trust root）、密钥轮换与撤销机制由后续 `SECURITY.md` 固定**（见第 9 节）。在 `SECURITY.md` 未定义该机制前，Phase 0 不得关闭。

---

## 9. 历史保留

- SBOM **历史版本须长期保留**，与对应 commit / 产物 / 发布候选一一对应，支持事后审计。

---

## 10. 当前状态

- **Phase 0：无产品代码与依赖，暂不生成 SBOM。** 本文件仅确立上述政策；任何空壳或占位 SBOM 均视为伪造真实产物，禁止。

---

> 本政策的变更（尤其涉及 CI 门禁、许可证校验、签名要求、覆盖范围）须走独立 ADR 审核（见 `DECISIONS/README.md`）。
