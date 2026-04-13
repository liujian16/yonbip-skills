# 列绑定

## 步骤 3.1：绑定条件列

对于每个条件列：

1. 搜索字段:
   ```bash
   yonyoucloud-cli workflow business-object --busiObjCode <busiObjCode值> --org-id <路径表的组织ID> --search <字段关键词>
   ```

2. 如果字段是参照类型（xxx-ref）:
   ```bash
   bip ref-data-list --ref-code <参照编码> --busi-obj <busiObjCode值>
   ```

3. 执行绑定:
   ```bash
   yonyoucloud-cli workflow condition-bind \
     --excel-id <excelId> --col <列索引> \
     --field <属性编码> --field-name <属性名称> \
     --form-id <formId值> --source <source值>
   ```

**列索引从 0 开始**。

## 步骤 3.2：绑定环节列

对于每个环节列（审批角色）：

1. 搜索模板:
   ```bash
   yonyoucloud-cli workflow segment-list --code <单据类型编码> --org-id <路径表的组织ID> --search <关键词>
   ```

2. 如果没有合适模板，创建新模板:
   ```bash
   yonyoucloud-cli workflow segment-save \
     --code <code值> --source <source> --name <角色名称> \
     --org-id <路径表的组织ID> \
     --biz-domain <领域code> --domain-cloud <领域云code> \
     --classify-name <分类全路径>
   ```

3. 添加审批人:
   ```bash
   yonyoucloud-cli workflow segment-design-save \
     --template-id <模板ID> --source <source> --code <code值> \
     --name <角色名称> --candidate-ids <用户ID> --org-id <路径表的组织ID>
   ```

4. 启用模板:
   ```bash
   yonyoucloud-cli workflow segment-enable --id <模板ID>
   ```

5. 绑定到列:
   ```bash
   yonyoucloud-cli workflow segment-bind \
     --excel-id <excelId> --source <source> --col <列索引> \
     --template-id <模板ID> --template-name <角色名称>
   ```

**⚠️ 关键**: 所有 segment 命令都必须传 `--org-id`（路径表的组织ID）。

**环节列索引 = 条件列数 + 序号**。

## 步骤 3.3：保存生效

```bash
yonyoucloud-cli workflow path-save --excel-id <excelId> --source <source>
```

**⚠️ 必须**: condition-bind 和 segment-bind 只验证不持久化，必须调用此命令。
