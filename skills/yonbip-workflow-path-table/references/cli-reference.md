# YonBIP CLI 命令参考

本文档包含所有 bip-cli workflow 命令的详细参数说明。

## 目录

1. [组织相关命令](#组织相关命令)
2. [单据类型命令](#单据类型命令)
3. [路径表命令](#路径表命令)
4. [条件列绑定命令](#条件列绑定命令)
5. [环节模板命令](#环节模板命令)
6. [参照数据命令](#参照数据命令)
7. [用户管理命令](#用户管理命令)

---

## 组织相关命令

### bip-cli org ref-list

选择路径表归属组织。

```bash
# 默认：只显示普通组织（过滤掉部门型组织 orgtype=2）
bip-cli org ref-list
bip-cli org ref-list --search <关键词>

# 如需包含部门型组织
bip-cli org ref-list --search <关键词> --include-deptorg

# 按层级过滤
bip-cli org ref-list --level 1  # 1=集团, 2=事业群, 3=部门, 4=岗位
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--search` | string | - | 搜索关键词，匹配组织名称或编码 |
| `--page` | number | 1 | 页码 |
| `--size` | number | 20 | 每页数量 |
| `--level` | number | 0 | 按层级过滤（0=全部, 1=集团, 2=事业群, 3=部门, 4=岗位） |
| `--include-deptorg` | boolean | false | 是否包含部门型组织（orgtype=2） |

**重要提醒**:
- 所属组织上传后不可修改
- 路径表应归属于普通组织（orgtype=1），不是部门型组织（orgtype=2）

### bip-cli org list

获取租户一级组织/部门列表。

```bash
bip-cli org list
bip-cli org list --search <关键词>
bip-cli org list --page 2 --size 50
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--page` | number | 1 | 页码 |
| `--size` | number | 20 | 每页数量 |
| `--search` | string | - | 搜索关键词，匹配组织名称或代码 |

### bip-cli org dept-list

获取组织部门树结构。

```bash
bip-cli org dept-list
bip-cli org dept-list --search <关键词>
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--page` | number | 1 | 页码 |
| `--size` | number | 20 | 每页数量 |
| `--search` | string | - | 搜索关键词，匹配部门名称 |

---

## 单据类型命令

### bip-cli workflow billtype-list

选择使用范围（单据类型）。

```bash
bip-cli workflow billtype-list --search <关键词>
bip-cli workflow billtype-list --source <领域>
bip-cli workflow billtype-list --detail  # 获取 busiObjCode
```

**输出字段用途映射**:

| 字段 | 用途 | 命令 |
|-----|------|-----|
| `code` | 显示编码 | path-import `--bill-no`, segment-save `--code` |
| `formId` | 表单ID | condition-bind `--form-id`（通常与 code 相同） |
| `busiObjCode` | 业务对象编码 | business-object `--busi-obj-code`, path-import `--busi-obj-code` |
| `source` | 来源标识 | path-import/condition-bind/segment-save `--source` |
| `domainCloud` | 领域云code | segment-save `--domain-cloud` |
| `bizDomain` | 领域code | segment-save `--biz-domain` |
| `classifyName` | 分类全路径 | segment-save `--classify-name` |

---

## 路径表命令

### bip-cli workflow path-import

上传 Excel 文件到 YonBIP。

```bash
bip-cli workflow path-import \
  --file <output.xlsx> \
  --org-id <组织ID> --org-name <组织名称> \
  --bill-no <code值> --busi-obj-code <busiObjCode值> --source <source值> \
  --bill-name <单据类型名称>
```

可选参数 `--save`：上传后自动保存使路径表生效。

**返回值**: `excelId`（路径表ID）和 `orgId`（组织ID）

### bip-cli workflow path-save

保存路径表使绑定生效。

```bash
bip-cli workflow path-save --excel-id <excelId> --source <source>
```

**重要**: segment-bind 和 condition-bind 只是解析/验证，不会自动持久化！必须调用此命令才能保存。

### bip-cli workflow path-list

查看路径表列表。

```bash
bip-cli workflow path-list --search <名称>
bip-cli workflow path-list --org <组织ID> --search <名称>
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--search` | string | - | 按名称模糊搜索（匹配 excelName/excelCode/orgName） |
| `--org` | string | - | 按组织ID筛选，~ 表示企业账号级 |
| `--page` | number | 1 | 页码 |
| `--size` | number | 20 | 每页数量（最大建议100） |

### bip-cli workflow path-get

获取路径表基本信息。

```bash
bip-cli workflow path-get --id <excelId> --source <source>
```

### bip-cli workflow path-table-detail

查看路径表详情（含绑定状态）。

```bash
bip-cli workflow path-table-detail --excel-id <excelId> --source <source>
```

**输出关键信息**:
- 列元信息（colMetas）：每列的 metaType、绑定状态、绑定详情
- 条件列绑定：显示绑定的字段名
- 环节列绑定：显示绑定的环节模板名称和 ID

### bip-cli workflow path-columns

查看路径表列元数据。

```bash
bip-cli workflow path-columns --excel-id <excelId>
bip-cli workflow path-columns --excel-id <excelId> --detail
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--excel-id` | string | 必填 | Excel ID 或 excelCode |
| `--source` | string | - | 来源标识（如 RBSM、HRSSP），/excel/detail 不可用时使用 |
| `--detail` | boolean | false | 显示详细信息，包括 formId、templateId 等 |

### bip-cli workflow path-table-check

校验路径表数据完整性。

```bash
bip-cli workflow path-table-check --excel-id <excelId> --source <source>
bip-cli workflow path-table-check --excel-id <excelId> --source <source> --check-model
```

### bip-cli workflow path-table-fix

诊断并修复路径表问题（如 modelId 分离）。

```bash
bip-cli workflow path-table-fix --excel-id <excelId> --source <source>
bip-cli workflow path-table-fix --excel-id <excelId> --source <source> --auto-fix
```

### bip-cli workflow path-open

在浏览器中打开路径表。

```bash
bip-cli workflow path-open --excel-id <excelId> --source <source>
```

### bip-cli workflow path-design

在浏览器中打开路径表设计页面。

```bash
bip-cli workflow path-design --id <excelId>
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--id` | string | 必填 | Excel ID |
| `--source` | string | developplatform | 来源标识 |

### bip-cli workflow path-extract

提取路径表设计数据（用于分析和调试）。

```bash
bip-cli workflow path-extract --id <excelId> --source <source>
bip-cli workflow path-extract --id <excelId> --source <source> --raw
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--id` | string | 必填 | Excel ID |
| `--source` | string | developplatform | 来源标识 |
| `--raw` | boolean | false | 显示原始 JSON |
| `--verbose` | boolean | false | 调试模式，显示所有 API 尝试 |

---

## 条件列绑定命令

### bip-cli workflow business-object

查找业务对象属性字段。

```bash
bip-cli workflow business-object --busi-obj-code <busiObjCode值> --org-id <路径表的组织ID> --search <字段关键词>
```

**重要**: `--org-id` 必须传路径表的组织ID，默认值 666666 是测试组织，会导致查不到实际字段。

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--busi-obj-code` | string | 必填 | 业务对象编码 |
| `--org-id` | string | 666666 | 组织ID |
| `--search` | string | - | 搜索关键词，匹配属性名称或编码 |
| `--verbose` | boolean | false | 调试模式，显示 API 原始返回数据 |

### bip-cli workflow condition-bind

绑定条件列到业务对象字段。

```bash
bip-cli workflow condition-bind \
  --excel-id <excelId> \
  --col <列索引> \
  --head-value <列标题文本> \
  --field <属性编码> \
  --field-name <属性名称> \
  --form-id <formId值> \
  --source <source值>
```

**列索引从 0 开始**: 第一列（条件1）为 0，第二列（条件2）为 1。

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--excel-id` | string | 必填 | Excel ID |
| `--source` | string | 必填 | 来源标识 |
| `--col` | number | 0 | 列索引（0-based） |
| `--head-value` | string | - | 列表头文本，默认从路径表数据中自动读取 |
| `--field` | string | 必填 | 属性编码 |
| `--field-name` | string | 必填 | 属性显示名称 |
| `--form-id` | string | 必填 | 单据类型的 formId |
| `--field-type` | string | - | 字段类型，覆盖路径表中已存储的类型。常用: Decimal、Integer、String、REFERENCE |

### bip-cli workflow condition-expressions

自动设置条件表达式（全部/数值区间/字符包含/等于）。

```bash
bip-cli workflow condition-expressions --excel-id <excelId> --col <列索引>
bip-cli workflow condition-expressions --excel-id <excelId> --col <列索引> --head-value <列标题文本>
```

---

## 环节模板命令

### bip-cli workflow segment-list

搜索环节模板。

```bash
bip-cli workflow segment-list --code <单据类型编码> --org-id <路径表的组织ID> --search <关键词>
bip-cli workflow segment-list --search <关键词> --detail
```

**重要**: 必须传 `--org-id`，否则创建的模板会归属错误组织。

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--code` | string | - | 单据类型编码筛选 |
| `--search` | string | - | 模糊搜索模板名称 |
| `--org-id` | string | - | 组织ID筛选 |
| `--source` | string | - | 按来源标识过滤 |
| `--detail` | boolean | false | 显示详细信息 |

### bip-cli workflow segment-types

查询可用的环节类型。

```bash
bip-cli workflow segment-types
bip-cli workflow segment-types --code <单据类型编码>
```

### bip-cli workflow segment-save

创建新的环节模板。

```bash
bip-cli workflow segment-save \
  --code <code值> --source <source> --name <角色名称> \
  --org-id <路径表的组织ID> \
  --biz-domain <领域code> --domain-cloud <领域云code> \
  --classify-name <分类全路径>
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--code` | string | 必填 | 单据类型编码 |
| `--source` | string | 必填 | 来源标识 |
| `--name` | string | 必填 | 环节模板名称 |
| `--org-id` | string | 666666 | 组织ID |
| `--segment-type` | string | NORMAL | 环节类型: NORMAL / SHARE_SEGMENT / SHARE_PROCESS / VIDEO / MSG |
| `--biz-domain` | string | - | 领域代码 |
| `--domain-cloud` | string | - | 领域云代码 |
| `--classify-name` | string | - | 分类全路径 |
| `--model-id` | string | - | 流程模型ID，来自路径表的 modelId 字段。必须传递以确保模板使用正确的 processModel |

### bip-cli workflow segment-design-save

配置模板审批人。

```bash
bip-cli workflow segment-design-save \
  --template-id <模板ID> --source <source> --code <code值> \
  --name <角色名称> --candidate-ids <用户ID> --org-id <路径表的组织ID>
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--template-id` | string | 必填 | 环节模板ID |
| `--source` | string | 必填 | 来源标识 |
| `--code` | string | 必填 | 单据类型编码 |
| `--name` | string | 必填 | 审批节点/参与人名称 |
| `--candidate-ids` | string | 必填 | 参与人用户ID，多个用逗号分隔 |
| `--org-id` | string | 666666 | 组织ID |
| `--assign-method` | string | selectMulti | 参与人指定方式: selectMulti / selectSingle |
| `--multi-instance` | string | Grab | 多实例模式: Grab(抢办) / CounterSign(会签) |
| `--existing-model-id` | string | - | 使用已有的流程模型ID。如果提供，将绑定到已有的流程模型而不是创建新的 |

### bip-cli workflow segment-enable

启用/停用模板。

```bash
bip-cli workflow segment-enable --id <模板ID> --status 1  # 启用
bip-cli workflow segment-enable --id <模板ID> --status 0  # 停用
```

**重要**: 只有在模板启用后才能绑定到列！

### bip-cli workflow segment-bind

绑定环节模板到列。

```bash
bip-cli workflow segment-bind \
  --excel-id <excelId> --source <source> --col <列索引> \
  --template-id <模板ID> --template-name <角色名称>
```

**列索引计算**: 环节列索引 = 条件列数 + 序号。例如有 2 个条件列、4 个环节列，则环节列索引分别为 2、3、4、5。

---

## 参照数据命令

### bip-cli workflow ref-data-list

查询参照类型字段的可用选项，通过名称查找实体的 id。

```bash
bip-cli workflow ref-data-list \
  --type <实体URI> \
  --keywords <搜索关键词>

# 限制返回数量
bip-cli workflow ref-data-list \
  --type pc.product.Product \
  --keywords iPhone \
  --limit 5
```

**参数说明**:
- `--type`: 实体 URI（如 `pc.product.Product`），从 business-object 命令的参照类型字段的 dataType 中获取
- `--keywords`: 搜索关键词（如"招待费"、"差旅费"）
- `--limit`: 最大返回数量（默认 20）

---

## 用户管理命令

### bip-cli user search

搜索审批人。

```bash
bip-cli user search --keyword <姓名>
bip-cli user search --keyword <手机号> --size 50
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--keyword` | string | 必填 | 搜索关键词，匹配用户名/姓名/手机号 |
| `--size` | number | 20 | 每页数量 |

### bip-cli user list

获取租户用户列表。

```bash
bip-cli user list
bip-cli user list --search <关键词>
bip-cli user list --page 2 --size 50
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--page` | number | 1 | 页码 |
| `--size` | number | 20 | 每页数量 |
| `--search` | string | - | 搜索关键词，匹配用户名/姓名/手机号 |
| `--type` | string | registerDate | 排序类型: registerDate / lastLoginTime |

### bip-cli user detail

获取用户详情。

```bash
bip-cli user detail --user-id <用户ID>
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--user-id` | string | 必填 | 用户ID，格式 uspace_xxxxx |

### bip-cli user toggle

启用或停用用户账号。

```bash
bip-cli user toggle --user-id <用户ID> --action enable
bip-cli user toggle --user-id <用户ID> --action disable --reason "离职"
```

**参数**:

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `--user-id` | string | 必填 | 用户ID，格式 uspace_xxxxx |
| `--action` | string | 必填 | 操作类型: enable / disable |
| `--reason` | string | - | 停用原因（可选） |

---

## 常见错误与恢复

| 错误场景 | 原因 | 恢复方案 |
|---------|------|---------|
| path-import 失败 | Excel 格式不正确 | 检查是否有"条件"/"环节"说明行 |
| condition-bind 找不到字段 | busiObjCode 不匹配 | 重新执行 billtype-list --detail |
| segment-bind 后不显示 | 组织ID不匹配 | 重新在正确组织下创建模板 |
| 发布后审批人不生效 | 模板未启用 | 执行 segment-enable --status 1 |
| 字段绑定无效 | 使用了猜测的子属性路径 | 只使用 business-object 显示的字段 |
