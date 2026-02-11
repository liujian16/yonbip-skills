# YonBIP Workflow Path Table Format Specification

## Table Structure

### Standard Columns (in order)

| Column | Description | Format |
|--------|-------------|--------|
| 费用类型 | Expense type category | Text: 差旅费用, 通讯费, 办公费, etc. |
| 金额 | Amount condition/range | See amount syntax below |
| 业务主管 | Business supervisor | Sequence number or n/a |
| 业务经理 | Business manager | Sequence number or n/a |
| 业务总经理 | Business GM | Sequence number or n/a |
| 财务专员 | Finance specialist | Sequence number or n/a |
| 财务经理 | Finance manager | Sequence number or n/a |
| 总裁 | President/CEO | Sequence number or n/a |

## Amount Syntax

### Basic Comparisons

| Format | Meaning | Example |
|--------|---------|---------|
| `< X` | Less than X | `< 5000` |
| `<= X` | Less than or equal to X | `<= 10000` |
| `> X` | Greater than X | `> 200` |
| `>= X` | Greater than or equal to X | `>= 10000` |
| `全部` | All amounts | `全部` |

### Ranges

| Format | Meaning | Example |
|--------|---------|---------|
| `[X, Y)` | X ≤ amount < Y (half-open) | `[5000, 10000)` |
| `(X, Y]` | X < amount ≤ Y | `(5000, 10000]` |
| `[X, Y]` | X ≤ amount ≤ Y (closed) | `[5000, 10000]` |

**Note:** YonBIP standard prefers half-open intervals `[X, Y)` for ranges.

## Approval Node Numbering

### Sequential Numbers

Each participating role gets a sequential number indicating approval order:
- `1` = First approver
- `2` = Second approver
- `3` = Third approver
- etc.

### Non-Participation

- Use `n/a` when a role does not participate in the approval flow

### Conditional Approval

Format: `N:[condition]`

Where:
- `N` = sequence number
- `condition` = business rule that must be satisfied

Example: `2:[金额>=500]` means this is the 2nd approval step AND requires amount ≥ 500

## Natural Language Mapping

### Common Chinese Phrases to Format

| Phrase | Format |
|--------|--------|
| X元以下 | `< X` |
| X元以内 | `<= X` |
| X-Y元 | `[X, Y)` |
| X元以上 | `>= X` |
| 大于X元 | `> X` |
| 大于等于X元 | `>= X` |
| 小于X元 | `< X` |
| 所有金额/全部 | `全部` |

### Role Name Normalization

| Input Variants | Standard |
|----------------|----------|
| 主管, 业务主管 | 业务主管 |
| 经理, 业务经理 | 业务经理 |
| 总经理, 业务总经理 | 业务总经理 |
| 财务, 财务专员 | 财务专员 |
| 财务经理 | 财务经理 |
| 总裁, 董事长 | 总裁 |

### Approval Chain Phrases

| Phrase | Interpretation |
|--------|----------------|
| 走A和B | A then B (sequential) |
| 增加C | Add C to existing chain |
| 全审/全部 | All roles participate |
| 同时/并列 | Parallel approval (use same number) |
| C环节需满足X | C has condition X |

## Examples

### Example 1: Simple Three-Tier Approval

| 费用类型 | 金额 | 业务主管 | 业务经理 | 业务总经理 | 财务专员 | 财务经理 | 总裁 |
|---------|------|---------|---------|-----------|---------|---------|------|
| 差旅费用 | < 5000 | 1 | n/a | n/a | 2 | n/a | n/a |
| 差旅费用 | [5000, 20000) | 1 | 2 | n/a | 3 | n/a | n/a |
| 差旅费用 | >= 20000 | 1 | 2 | 3 | 4 | 5 | 6 |

### Example 2: Conditional Approval

| 费用类型 | 金额 | 业务主管 | 业务经理 | 业务总经理 | 财务专员 | 财务经理 | 总裁 |
|---------|------|---------|---------|-----------|---------|---------|------|
| 采购费 | 全部 | 1 | 2 | n/a | 3 | n/a | n/a |
| 采购费 | >= 50000 | 1 | 2 | 3:[需总经理审批] | 4 | 5 | n/a |

### Example 3: Multiple Conditions

| 费用类型 | 金额 | 业务主管 | 业务经理 | 业务总经理 | 财务专员 | 财务经理 | 总裁 |
|---------|------|---------|---------|-----------|---------|---------|------|
| 招待费 | < 1000 | 1 | n/a | n/a | 2 | n/a | n/a |
| 招待费 | [1000, 5000) | 1 | 2 | n/a | 3 | 4 | n/a |
| 招待费 | >= 5000 | 1 | 2 | 3 | 4 | 5 | 6 |
