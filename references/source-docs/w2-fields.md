---
document: "W-2"
source: "IRS Form W-2 Wage and Tax Statement"
---

# W-2: Wage and Tax Statement

Issued by employers to employees. One per employer. Collect all W-2s before starting.

## Box-by-Box Guide

When collecting values, ask for each box in order. Remind user to redact SSN (Box a) -- we only store last 4.

### Employee/Employer Info

| Box | Name | What it is | Store as |
|-----|------|------------|----------|
| a | Employee SSN | Social Security Number | **Last 4 only**: `ssn_last4` |
| b | Employer EIN | Employer Identification Number | `employer_ein` (can store full) |
| c | Employer name/address | Employer identity | `employer_name` |
| e | Employee name | Your name | Verify matches, don't store |

### Income Boxes

| Box | Name | What it is | Flows to |
|-----|------|------------|----------|
| 1 | Wages, tips, other compensation | Total taxable wages | 1040 Line 1a |
| 2 | Federal income tax withheld | How much employer sent to IRS for you | 1040 Line 25a |
| 3 | Social Security wages | Wages subject to SS tax | Usually = Box 1 (capped at $176,100 for 2025) |
| 4 | Social Security tax withheld | 6.2% of Box 3 | Informational |
| 5 | Medicare wages and tips | Wages subject to Medicare tax | Usually = Box 1 (no cap) |
| 6 | Medicare tax withheld | 1.45% of Box 5 | Informational |
| 7 | Social Security tips | Tips subject to SS tax | Rare |
| 8 | Allocated tips | Tips allocated by employer | Rare, only food/beverage |

### Box 10-14

| Box | Name | What it is | Flows to |
|-----|------|------------|----------|
| 10 | Dependent care benefits | Employer-paid dependent care | Form 2441 if > $5,000 |
| 11 | Nonqualified plans | Distributions from nonqualified deferred comp | Included in Box 1 |
| 12 | Codes (a-d) | Various items -- see code table below | Varies by code |
| 13 | Checkboxes | Statutory employee, Retirement plan, Third-party sick pay | Affects other forms |
| 14 | Other | State/local info, union dues, etc. | Varies |

### Box 12 Codes (common ones)

| Code | Meaning | Tax Impact |
|------|---------|------------|
| C | Group-term life insurance > $50k | Included in Box 1 |
| D | 401(k) contributions | Pre-tax, already excluded from Box 1 |
| DD | Employer-sponsored health coverage cost | Informational only, not taxable |
| E | 403(b) contributions | Pre-tax, already excluded from Box 1 |
| G | 457(b) contributions | Pre-tax |
| W | HSA employer contributions | Form 8889, already excluded from Box 1 |
| AA | Roth 401(k) contributions | After-tax, included in Box 1 |
| BB | Roth 403(b) contributions | After-tax, included in Box 1 |

### Box 13 Checkboxes

| Checkbox | Meaning | Impact |
|----------|---------|--------|
| Statutory employee | Special category (drivers, salespeople) | File Schedule C instead of reporting as wages |
| Retirement plan | Employer offers retirement plan | May limit traditional IRA deduction |
| Third-party sick pay | Sick pay from a third party | Informational |

### State/Local Section (Boxes 15-20)

| Box | Name | What it is | Flows to |
|-----|------|------------|----------|
| 15 | State / Employer state ID | State and state tax ID | State return |
| 16 | State wages | Wages subject to state tax | State return |
| 17 | State income tax withheld | State tax withheld by employer | State return |
| 18 | Local wages | Wages subject to local tax | Local return (if applicable) |
| 19 | Local income tax withheld | Local tax withheld | Local return |
| 20 | Locality name | Name of local jurisdiction | Local return |

## Sanity Checks

After collecting all values, verify:
- Box 1 should be reasonable for the stated employment (full-time vs part-time)
- Box 2 should be less than Box 1 (usually 10-25% of Box 1 for most people)
- Box 3 should equal Box 1 (unless Box 1 > SS wage base $176,100)
- Box 4 should be approximately Box 3 x 0.062
- Box 5 should equal Box 1 (or close)
- Box 6 should be approximately Box 5 x 0.0145
- Box 16 (state wages) often equals Box 1 but may differ
- If Box 12 Code D/E/G exists: Box 1 already excludes those contributions

## JSON Schema

Save to `projects/taxes/YYYY/source-documents/w2-{identifier}.json`:

```json
{
  "document_type": "W-2",
  "identifier": "employer1",
  "employer_name": "",
  "employer_ein": "",
  "employee_ssn_last4": "",
  "boxes": {
    "1_wages": 0,
    "2_federal_tax_withheld": 0,
    "3_social_security_wages": 0,
    "4_social_security_tax": 0,
    "5_medicare_wages": 0,
    "6_medicare_tax": 0,
    "7_social_security_tips": 0,
    "8_allocated_tips": 0,
    "10_dependent_care": 0,
    "12a_code": null,
    "12a_amount": null,
    "12b_code": null,
    "12b_amount": null,
    "12c_code": null,
    "12c_amount": null,
    "12d_code": null,
    "12d_amount": null,
    "13_statutory_employee": false,
    "13_retirement_plan": false,
    "13_third_party_sick_pay": false,
    "14_other": null,
    "15_state": "",
    "16_state_wages": 0,
    "17_state_tax_withheld": 0,
    "18_local_wages": 0,
    "19_local_tax_withheld": 0,
    "20_locality": ""
  }
}
```
