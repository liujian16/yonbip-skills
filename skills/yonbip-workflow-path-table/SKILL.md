---
name: yonbip-workflow-path-table
description: Converts natural language business approval workflow descriptions into YonBIP-standard Excel workflow path tables to import workflow definition into YonBIP. Use when user provides approval rules in natural language and needs workflow path tables.
---

# YonBIP Workflow Path Table Generator

Converts natural language business approval workflow descriptions into YonBIP-standard Excel workflow path tables.

## Core Principle: Dynamic Column Extraction

**CRITICAL: Do NOT assume any fixed columns.** Every table structure must be derived from the user's description:

1. **Condition Columns**: Extract all condition dimensions mentioned
   - Examples: 费用类型, 金额, 供应商类型, 部门, 地区, 采购类别, etc.
2. **Role Columns**: Extract ALL roles mentioned in the approval chains
   - Examples: 部门主管, 财务经理, 总经理, 行政主管, 采购专员, 法律顾问, etc.
3. **Build table structure** based ONLY on what's actually in the description

## Workflow

**Dry Run Mode**: If user specifies `--dryrun` flag, skip user confirmation and Excel generation, only output the markdown table.

### Step 1: Extract and Structure

Analyze the natural language and identify:

**A. All Condition Dimensions (前几列)**

- What categories/dimensions determine the approval path?
- Common patterns: 费用类型, 金额区间, 供应商性质, 部门类型, etc.

**B. All Approval Roles (后几列)**

- Who are the approvers? List ALL unique roles mentioned
- Preserve the exact role names from the description
- Do NOT add roles that aren't mentioned

**C. Approval Logic (表格内容)**

- Map out each scenario's approval sequence
- Use numbers (1, 2, 3...) for approval order
- Use `n/a` for non-participating roles
- Use `N:[condition]` for conditional approvals

### Step 2: Generate Markdown Table

Build the table with extracted columns:

```
| [条件列1] | [条件列2] | ... | [角色1] | [角色2] | [角色3] | ... |
|----------|----------|-----|---------|---------|---------|-----|
| 值A      | 条件X    | ... | 1       | n/a     | 2       | ... |
```

**Present to user for confirmation.**

### Step 3: User Confirmation

Ask: "请确认以上流程路径表是否正确？如需修改请直接告诉我。"

Wait for approval or modifications.

### Step 4: Generate Excel

After confirmation, use the conversion script to generate Excel file.

write the markdown file to same folder of the Excel file.

**Script location:** `scripts/md_to_excel.py` (within this skill directory)

**Usage:**

```bash
python3 ${SKILL_DIR}/scripts/md_to_excel.py <input.md> <output.xlsx> --condition-cols <N>
```

**Parameters:**

- `--condition-cols N`: 指定前 N 列为"条件"列（如：费用类型、金额区间），其余列为"环节"列（审批角色）

The script will:

- Parse the Markdown table structure
- Add explanation row ("条件" | "环节")
- Apply YonBIP standard formatting (headers, borders, alignment)
- Convert `n/a` to empty cells
- Generate `.xlsx` file

## Examples

### Example 1: 招待费（简单场景）

**Input:**

```
招待费审批规则：1000元以下只需部门主管审批；1000-5000元需要部门主管和财务经理；5000元以上则需要部门主管、财务经理和总经理。
```

**Extracted:**

- Condition columns: 费用类型, 金额
- Role columns: 部门主管, 财务经理, 总经理

**Output:**
| 费用类型 | 金额 | 部门主管 | 财务经理 | 总经理 |
|---------|------|---------|---------|--------|
| 招待费 | < 1000 | 1 | n/a | n/a |
| 招待费 | [1000, 5000) | 1 | 2 | n/a |
| 招待费 | >= 5000 | 1 | 2 | 3 |

### Example 2: 采购审批（多条件+多角色）

**Input:**

```
行政类采购：无论金额，部门负责人(1)→行政主管(2)。超过5000元增加财务总监(3)。
生产类采购：<20000：部门负责人(1)→采购专员(2)→财务总监(3)。
≥20000：部门负责人(1)→采购经理(2)→财务总监(3)→总经理(4)。
新合作方需增加法律顾问。
```

**Extracted:**

- Condition columns: 采购类别, 金额, 供应商类型
- Role columns: 部门负责人, 行政主管, 采购专员, 采购经理, 财务总监, 总经理, 法律顾问

**Output:**
| 采购类别 | 金额 | 供应商类型 | 部门负责人 | 行政主管 | 采购专员 | 采购经理 | 财务总监 | 总经理 | 法律顾问 |
|---------|------|-----------|-----------|---------|---------|---------|---------|--------|---------|
| 行政类采购 | 全部 | 全部 | 1 | 2 | n/a | n/a | n/a | n/a | n/a |
| 行政类采购 | >= 5000 | 全部 | 1 | 2 | n/a | n/a | 3 | n/a | n/a |
| 生产类采购 | < 20000 | 全部 | 1 | n/a | 2 | n/a | 3 | n/a | n/a |
| 生产类采购 | >= 20000 | 全部 | 1 | n/a | n/a | 2 | 3 | 4 | n/a |
| 生产类采购 | >= 20000 | 新合作方 | 1 | n/a | n/a | 2 | 3 | 4 | 5 |

### Example 3: 差旅费（标准YonBIP角色）

**Input:**

```
差旅费5000元以下走主管和财务；5000-10000增加经理；10000以上全审。
```

**Extracted:**

- Condition columns: 费用类型, 金额
- Role columns: 业务主管, 财务专员, 业务经理, 业务总经理, 总裁 (注意：按出现顺序提取)

**Output:**
| 费用类型 | 金额 | 业务主管 | 财务专员 | 业务经理 | 业务总经理 | 总裁 |
|---------|------|---------|---------|---------|-----------|------|
| 差旅费用 | < 5000 | 1 | 2 | n/a | n/a | n/a |
| 差旅费用 | [5000, 10000) | 1 | 3 | 2 | n/a | n/a |
| 差旅费用 | >= 10000 | 1 | 4 | 2 | 3 | 5 |

## Value Formatting

### Condition Values

- **金额**: `< 5000`, `>= 10000`, `[5000, 10000)`, `全部`
- **类别**: 具体类别名称 (如: 差旅费用, 行政类采购)
- **通用**: `全部` 表示不限制

### Approval Nodes

- **1, 2, 3...**: Approval sequence order
- **n/a**: Role does not participate
- **N:[condition]**: Nth step with additional condition
  - Example: `2:[金额>=500]` = 2nd approver AND amount >= 500

## Common Patterns

**Threshold phrases:**

- "X元以下" → `< X`
- "X元以内" → `<= X`
- "X-Y元" → `[X, Y)`
- "X元以上" → `>= X`

**Chain phrases:**

- "走A和B" → A then B (sequential)
- "增加C" → Add C to chain
- "全审/全部" → All roles participate

**Condition phrases:**

- "如果是X且Y" → Add condition columns
- "满足XX条件" → `N:[XX]`
- "需XX审批" → Add XX role
