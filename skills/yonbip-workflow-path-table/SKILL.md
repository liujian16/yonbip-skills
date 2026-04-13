---
name: yonbip-workflow-path-table
description: 将自然语言审批流程描述转换为YonBIP标准Excel路径表并导入系统。
  当用户提到审批规则、审批流程、路径表、审批流、流程导入、流程上传、条件绑定、
  YonBIP工作流配置、费用审批、合同审批、报销流程时使用。
  即使用户只说"帮我配置审批"、"做个审批表"、"审批流程怎么配"，
  也应使用此skill。
---

# YonBIP 审批流程路径表生成器

将自然语言描述的审批流程转换为YonBIP标准的Excel路径表。

## 功能概览

| 功能 | 标识 | 输入 | 输出 | 主要功能 |
|-----|------|------|------|---------|
| 1 | `生成路径表` | 自然语言规则 | Markdown → Excel + 结构信息 | 解析规则，生成表格，生成Excel |
| 2 | `上传路径表` | Excel文件 | excelId + orgId + source + billtype信息 | 配置参数并上传到YonBIP |
| 3 | `列绑定` | excelId + orgId + 结构信息 | 绑定完成 + 已保存 | 绑定条件列和环节列 |
| 4 | `验证发布` | excelId + source | 发布生效 | 验证配置并发布 |

## 前置条件

- **功能 1**：无特殊依赖
- **功能 2-4**：Chrome 浏览器已运行且已登录 YonBIP 系统

## 功能自动选择

当用户未指定 `--function` 时，根据输入自动判断执行功能：

| 用户输入特征 | 推断功能 |
|-------------|---------|
| 描述审批规则（如"1000元以下走主管审批"） | 生成路径表|
| 提供已存在的 Excel 文件路径 + 提到"导入"、"上传" | 上传路径表 |
| 提供 excelId + 提到"绑定" | 列绑定 |
| 提供 excelId + 提到"验证"或"发布" | 验证发布 |

除了发布功能，完成用户的指定功能后，询问用户是否要执行后续功能。

## 调用方式

### 完整流程（默认）

```
/yonbip-workflow-path-table <审批规则描述>
```

### 按功能调用

```
# 仅生成 Excel 文件
/yonbip-workflow-path-table --function 生成路径表 <审批规则描述>

# 仅上传 Excel 到 YonBIP
/yonbip-workflow-path-table --function 上传路径表 --input <Excel文件路径>

# 仅绑定条件列和环节列
/yonbip-workflow-path-table --function 列绑定 --excel-id <excelId> --org-id <orgId> --source <source>

# 仅验证并发布
/yonbip-workflow-path-table --function 验证发布 --excel-id <excelId> --source <source>
```

### 试运行模式

```
/yonbip-workflow-path-table --dryrun <审批规则描述>
```

仅生成 Markdown 表格，跳过澄清和 Excel 生成。

## 详细参考文档

执行各功能时，查阅对应详细说明：

| 功能 | 详细说明 | CLI 参考 |
|-----|---------|---------|
| 生成路径表 | `${SKILL_DIR}/references/generate-path-table.md` | - |
| 上传路径表 | `${SKILL_DIR}/references/upload-path-table.md` | `${SKILL_DIR}/references/cli-reference.md` |
| 列绑定 | `${SKILL_DIR}/references/bind-columns.md` | `${SKILL_DIR}/references/cli-reference.md` |
| 验证发布 | `${SKILL_DIR}/references/verify-publish.md` | `${SKILL_DIR}/references/cli-reference.md` |

## 功能概要

### 功能 1：生成路径表

解析自然语言审批规则，提取条件维度和审批角色，生成互斥的 Markdown 表格并转换为 Excel。详见 `${SKILL_DIR}/references/generate-path-table.md`。

### 功能 2：上传路径表

配置路径表名称、归属组织和单据类型，将 Excel 上传到 YonBIP 系统获取 excelId。详见 `${SKILL_DIR}/references/upload-path-table.md`。

### 功能 3：列绑定

将条件列绑定到业务对象字段，将环节列绑定到审批模板。详见 `${SKILL_DIR}/references/bind-columns.md`。

### 功能 4：验证发布

通过 CLI 和浏览器验证路径表配置，确认无误后手工发布生效。详见 `${SKILL_DIR}/references/verify-publish.md`。
