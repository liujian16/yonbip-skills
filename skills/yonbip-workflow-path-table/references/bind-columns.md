# 列绑定

## 步骤 3.1：绑定条件列

对于每个条件列：

1. 搜索字段:
   ```bash
   bip-cli workflow business-object --busi-obj-code <busiObjCode值> --org-id <路径表的组织ID> --search <字段关键词>
   ```

2. 如果字段是参照类型（xxx-ref）:
   ```bash
   bip-cli workflow ref-data-list --type <实体URI> --keywords <搜索关键词>
   ```

3. 执行绑定:
   ```bash
   bip-cli workflow condition-bind \
     --excel-id <excelId> --source <source> \
     --col <列索引> \
     --field <属性编码> --field-name <属性名称> \
     --form-id <formId值>
   ```

   可选参数 `--field-type` 覆盖路径表中已存储的字段类型。

**列索引从 0 开始**。

## 步骤 3.2：绑定环节列

对于每个环节列（审批角色）：

1. 搜索模板:
   ```bash
   bip-cli workflow segment-list --code <单据类型编码> --org-id <路径表的组织ID> --search <关键词>
   ```

2. 如果没有合适模板，创建新模板:
   ```bash
   bip-cli workflow segment-save \
     --code <code值> --source <source> --name <角色名称> \
     --org-id <路径表的组织ID> \
     --biz-domain <领域code> --domain-cloud <领域云code> \
     --classify-name <分类全路径>
   ```

   可选参数：
   - `--segment-type`：环节类型（默认 NORMAL），可选 SHARE_SEGMENT / SHARE_PROCESS / VIDEO / MSG
   - `--model-id`：流程模型ID，来自路径表的 modelId 字段，必须传递以确保模板使用正确的 processModel

3. 添加审批人:
   ```bash
   bip-cli workflow segment-design-save \
     --template-id <模板ID> --source <source> --code <code值> \
     --name <角色名称> --candidate-ids <用户ID> --org-id <路径表的组织ID>
   ```

   可选参数：
   - `--assign-method`：参与人指定方式（默认 selectMulti），可选 selectSingle
   - `--multi-instance`：多实例模式（默认 Grab），可选 CounterSign
   - `--existing-model-id`：使用已有的流程模型ID，而不是创建新的

4. 启用模板:
   ```bash
   bip-cli workflow segment-enable --id <模板ID> --status 1
   ```

5. 绑定到列:
   ```bash
   bip-cli workflow segment-bind \
     --excel-id <excelId> --source <source> --col <列索引> \
     --template-id <模板ID> --template-name <角色名称>
   ```

**关键**: 所有 segment 命令都必须传 `--org-id`（路径表的组织ID）。

**环节列索引 = 条件列数 + 序号**。

## 步骤 3.3：保存生效

```bash
bip-cli workflow path-save --excel-id <excelId> --source <source>
```

**必须**: condition-bind 和 segment-bind 只验证不持久化，必须调用此命令。
