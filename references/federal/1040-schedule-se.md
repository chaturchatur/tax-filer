---
form: "Schedule SE"
tax_year: 2025
source: "IRS Instructions for Schedule SE"
---

# Schedule SE: Self-Employment Tax

## Overview

Computes Social Security and Medicare tax on self-employment income. Employees pay half through payroll withholding (W-2 Box 4/6), but self-employed people pay both halves.

## Who Files This

Anyone with net self-employment earnings of $400 or more (Schedule C Line 31).

## Tax Rates (2025)

- Social Security: 12.4% on first $176,100 of combined wages + SE income (verify annual cap)
- Medicare: 2.9% on all SE income (no cap)
- Additional Medicare: 0.9% on SE income over $200,000 single / $250,000 MFJ
- Total: 15.3% (up to SS wage base) + 2.9% (above SS wage base)

## Line-by-Line Instructions (Short Schedule SE)

Most filers use the short schedule. Use the long schedule only if you have church employee income, wages from an S Corp, or need to account for W-2 wages against the SS wage base.

#### Line 1a: Net farm profit or loss
- **Source**: Schedule F (if applicable)
- **Computation**: Usually 0 for most filers

#### Line 1b: Net non-farm profit or loss
- **Source**: Schedule C Line 31
- **Computation**: Net profit from self-employment

#### Line 2: Combined
- **Computation**: Line 1a + Line 1b

#### Line 3: Multiply Line 2 by 92.35%
- **Computation**: Line 2 x 0.9235
- **Notes**: This adjustment approximates the employer-equivalent portion. You only pay SE tax on 92.35% of net earnings.

#### Line 4a: Social Security tax
- **Computation**: If Line 3 <= SS wage base after W-2 wages: Line 3 x 0.124
- **Notes**: If also have W-2 wages, the SS portion only applies to the amount under the wage base minus W-2 SS wages (Box 3).

#### Line 4b: Medicare tax
- **Computation**: Line 3 x 0.029

#### Line 4c: Additional Medicare tax
- **Computation**: If applicable, 0.009 x amount over threshold

#### Line 5: Self-employment tax
- **Computation**: Line 4a + Line 4b (+ 4c if applicable)
- **Notes**: This is the total SE tax.

#### Line 6: Deduction for one-half of SE tax
- **Computation**: Line 5 x 0.5
- **Notes**: This deduction goes to Schedule 1 Line 15. It reduces AGI (above-the-line deduction).

## Flows To

- **Schedule 2 Line 4**: Self-employment tax (Line 5)
- **Schedule 1 Line 15**: Deductible half of SE tax (Line 6)

## Quick Calculation

For a simple case (no W-2 wages, below SS wage base):
```
SE tax = Schedule C net profit x 0.9235 x 0.153
Deduction = SE tax / 2
```

Example: $50,000 Schedule C profit
- $50,000 x 0.9235 = $46,175
- $46,175 x 0.153 = $7,065 (SE tax)
- $7,065 / 2 = $3,533 (deduction on Schedule 1)

## JSON Schema

```json
{
  "form": "schedule-se",
  "tax_year": 2025,
  "status": "draft",
  "lines": {
    "1b": {"value": 0, "source": "schedule-c line 31", "verified": false},
    "2": {"value": 0, "source": "line 1a + line 1b", "verified": false},
    "3": {"value": 0, "source": "line 2 x 0.9235", "verified": false},
    "5": {"value": 0, "source": "SS tax + Medicare tax", "verified": false},
    "6": {"value": 0, "source": "line 5 x 0.5", "verified": false}
  }
}
```
