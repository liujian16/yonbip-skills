# 验证发布

## 步骤 4.1：CLI验证

```bash
yonyoucloud-cli workflow path-table-detail --excel-id <excelId> --source <source>
```

检查列元信息、绑定状态、表格数据。

## 步骤 4.2：浏览器验证

```bash
yonyoucloud-cli workflow --env <环境> path-open --excel-id <excelId> --source <source>
```

在浏览器中检查：
1. 基本信息（名称、组织、单据类型）
2. 条件列绑定
3. 环节列绑定
4. 审批流程预览

## 步骤 4.3：手工发布

在页面中点击"发布"按钮使路径表正式生效。
