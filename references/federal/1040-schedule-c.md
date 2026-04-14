---
form: "Schedule C"
tax_year: 2025
source: "IRS Instructions for Schedule C"
---

# Schedule C: Profit or Loss from Business (Sole Proprietorship)

## Overview

Report income and expenses from self-employment, freelancing, gig work, or sole proprietorship. One Schedule C per business/activity.

## Who Files This

- Received 1099-NEC for freelance/contract work
- Self-employment income of any kind
- Sole proprietor of a business
- Statutory employees (W-2 Box 13 checked)

## Line-by-Line Instructions

### Income Section

#### Line 1: Gross receipts or sales
- **Source**: Sum of all 1099-NEC Box 1 for this business + any unreported cash income
- **Computation**: Total business revenue
- **Notes**: Must report ALL income, even if no 1099 received. IRS gets copies of 1099s.

#### Line 2: Returns and allowances
- **Source**: User-provided
- **Computation**: Refunds given to clients

#### Line 4: Cost of goods sold
- **Source**: Computed from Part III (if applicable)
- **Notes**: Only if selling physical products

#### Line 5: Gross profit
- **Computation**: Line 1 - Line 2 - Line 4

#### Line 6: Other income
- **Source**: User-provided
- **Notes**: Interest on business accounts, recovered bad debts, etc.

#### Line 7: Gross income
- **Computation**: Line 5 + Line 6

### Expense Section (Lines 8-27)

Ask the user about each category. Common ones for freelancers:

| Line | Expense | Examples |
|------|---------|----------|
| 8 | Advertising | Website, business cards, ads |
| 10 | Car and truck | Business miles x IRS rate (70 cents/mile for 2025, verify) |
| 11 | Commissions/fees | Platform fees, agent commissions |
| 13 | Depreciation | Large equipment, computers (or use Section 179) |
| 15 | Insurance | Business liability, professional insurance |
| 17 | Legal/professional | Accounting, legal fees |
| 18 | Office expense | Supplies, software subscriptions |
| 22 | Supplies | Materials used in business |
| 24a | Travel | Business travel (not commuting) |
| 24b | Meals | Business meals (50% deductible) |
| 25 | Utilities | If home office, proportional share |
| 27a | Other expenses | Anything not in above categories |

#### Line 28: Total expenses
- **Computation**: Sum of lines 8-27

#### Line 29: Tentative profit
- **Computation**: Line 7 - Line 28

#### Line 30: Home office deduction (Form 8829 or simplified method)
- **Source**: Form 8829 or simplified calculation
- **Computation**: Simplified method: $5/sq ft, max 300 sq ft = max $1,500
- **Notes**: Must use space regularly and exclusively for business

#### Line 31: Net profit or loss
- **Computation**: Line 29 - Line 30
- **Notes**: This flows to Schedule 1 Line 3. If >= $400, must file Schedule SE.
- **Triggers**: Schedule SE if >= $400

## Flows To

- **Schedule 1 Line 3**: Net profit (or loss)
- **Schedule SE**: If net profit >= $400
- **1040 Line 15** (indirectly): Through Schedule 1 -> AGI

## Common Mistakes

1. Not reporting income paid in cash (IRS can find discrepancies)
2. Claiming personal expenses as business (meals must be business-related)
3. Forgetting to track mileage throughout the year
4. Not understanding home office requirements (regular and exclusive use)
5. Deducting startup costs incorrectly (first $5,000 deductible, rest amortized over 15 years)

## JSON Schema

```json
{
  "form": "schedule-c",
  "tax_year": 2025,
  "status": "draft",
  "business_name": "",
  "business_code": "",
  "accounting_method": "cash",
  "lines": {
    "1": {"value": 0, "source": "", "verified": false},
    "2": {"value": 0, "source": "", "verified": false},
    "5": {"value": 0, "source": "line 1 - line 2", "verified": false},
    "7": {"value": 0, "source": "line 5 + line 6", "verified": false},
    "28": {"value": 0, "source": "sum of expenses", "verified": false},
    "29": {"value": 0, "source": "line 7 - line 28", "verified": false},
    "30": {"value": 0, "source": "home office deduction", "verified": false},
    "31": {"value": 0, "source": "line 29 - line 30", "verified": false}
  },
  "expenses": {
    "8_advertising": 0,
    "10_car_truck": 0,
    "11_commissions": 0,
    "13_depreciation": 0,
    "15_insurance": 0,
    "17_legal_professional": 0,
    "18_office": 0,
    "22_supplies": 0,
    "24a_travel": 0,
    "24b_meals": 0,
    "25_utilities": 0,
    "27a_other": 0
  }
}
```
