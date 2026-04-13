# 上传路径表

上传并导入`流程路径表`到YonBIP系统前，需要确认其名称、归属组织、对应单据类型等信息。

## 前置条件

- 已通过功能 1 生成 Excel 文件（`<output.xlsx>` 路径）
- Chrome 浏览器已运行且已登录 YonBIP 系统

## 输出

上传成功后获得以下信息，功能 3（列绑定）和功能 4（验证发布）需要使用：

- `excelId`：路径表 ID
- `orgId`：组织 ID
- `source`：单据来源标识
- 单据类型字段：`code`, `formId`, `busiObjCode`, `domainCloud`, `bizDomain`, `classifyName`

---

## 步骤 2.1：确认路径表名称

默认使用 Excel 文件名。如果文件名过于通用（如 `流程路径表.xlsx`），提醒用户使用有业务含义的名称。

**命名建议**：
- 推荐格式：`{业务场景}_{版本号}`，如 `个人借款审批_v1`
- 名称应能体现审批流程的业务含义，避免使用纯数字或默认名称
- 建议用 `workflow path-list --search <名称>` 检查是否与已有路径表重名

名称需经用户确认。

## 步骤 2.2：选择归属组织

1. 询问用户流程路径表的归属组织
2. 搜索组织：
   ```bash
   bip-cli org ref-list --search <关键词>
   ```
3. 处理搜索结果：
   - **无结果** → 提示用户更换关键词或缩短关键词重试
   - **单个结果** → 直接确认
   - **多个结果** → 使用 AskUserQuestion 让用户选择，选项格式：`"组织名称 (编码)"`
4. 确认并记录组织 ID

**⚠️ 重要提醒**：
- 组织上传后不可修改
- 路径表应归属于**普通组织（法人单位，orgtype=1）**，不是部门型组织（orgtype=2）
- 如果用户不确定组织名称，可以不带 `--search` 直接列出全部组织，或使用 `bip-cli org list` 查看组织列表

## 步骤 2.3：选择单据类型

1. 根据用户输入搜索单据类型：
   ```bash
   bip-cli workflow billtype-list --search <关键词>
   ```

2. 让用户选定单据类型，使用 AskUserQuestion 交互确认

3. 获取选定单据类型的详细信息（需要 busiObjCode 等字段）：
   ```bash
   bip-cli workflow billtype-list --detail
   ```

4. 记录以下字段（功能 3 列绑定需要）：

| 字段 | 用途 |
|-----|------|
| `code` | path-import `--bill-no` |
| `formId` | condition-bind `--form-id` |
| `busiObjCode` | business-object `--busiObjCode` |
| `source` | 多个命令的 `--source` 参数 |
| `domainCloud` | segment-save `--domain-cloud` |
| `bizDomain` | segment-save `--biz-domain` |
| `classifyName` | segment-save `--classify-name` |

## 步骤 2.4：确认并上传

展示确认信息（路径表名称、组织、单据类型、Excel 文件路径），经用户确认后执行上传。

```bash
bip-cli workflow path-import \
  --file <output.xlsx> \
  --org-id <组织ID> --org-name <组织名称> \
  --bill-no <code值> --busi-obj-code <busiObjCode值> --source <source值> \
  --bill-name <单据类型名称>
```

**上传成功后记录以下信息**：

| 返回值 | 说明 | 后续用途 |
|--------|------|---------|
| `excelId` | 路径表唯一标识 | 功能 3 列绑定、功能 4 验证发布 |
| `orgId` | 组织 ID | 功能 3 列绑定 |

上传完成后，询问用户是否继续执行功能 3（列绑定）。

---

## 常见问题

| 问题 | 原因 | 解决方案 |
|-----|------|---------|
| 上传失败："Excel格式不正确" | 缺少"条件"/"环节"说明行 | 检查 Excel 是否由功能 1 生成 |
| 上传失败："无权限" | 当前用户无该组织操作权限 | 切换组织或联系管理员 |
| 单据类型搜索无结果 | 关键词不匹配 | 尝试更短或不同的关键词 |
| 组织搜索无结果 | 关键词不匹配 | 尝试更短的关键词，或不带 `--search` 列出全部 |
