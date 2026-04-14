---
form: "Schedule A"
tax_year: 2025
source: "IRS Instructions for Schedule A"
---

# Schedule A: Itemized Deductions

## Overview

Use this instead of the standard deduction if your total itemized deductions exceed the standard deduction amount for your filing status.

## When to Itemize

Itemize if total exceeds:
- Single: $15,700
- MFJ: $31,400
- HOH: $23,500

If unsure, compute both and recommend the larger deduction.

## Line-by-Line Instructions

### Medical and Dental Expenses (Lines 1-4)

#### Line 1: Medical and dental expenses
- **Source**: User-provided (receipts, statements)
- **Notes**: Includes insurance premiums (not pre-tax), copays, prescriptions, dental, vision, medical equipment, mileage to medical appointments

#### Line 2: AGI from 1040 Line 11
- **Source**: 1040 Line 11

#### Line 3: Multiply Line 2 by 7.5%
- **Computation**: Line 2 x 0.075

#### Line 4: Deductible medical expenses
- **Computation**: max(0, Line 1 - Line 3)
- **Notes**: Only the amount OVER 7.5% of AGI is deductible. For most people, this is $0.

### Taxes You Paid (Lines 5-7)

#### Line 5a: State and local income taxes OR sales taxes
- **Source**: W-2 Box 17 (state tax withheld) + state estimated payments + state tax paid with prior year return
- **Notes**: Can choose state income tax OR state sales tax -- whichever is higher. Most people use income tax.

#### Line 5b: State and local personal property taxes
- **Source**: Vehicle registration tax (the ad valorem portion), personal property tax bills

#### Line 5c: State and local real estate taxes
- **Source**: Property tax bills, 1098 Box 10

#### Line 5d: Total of 5a-5c
- **Computation**: Sum

#### Line 5e: SALT cap
- **Computation**: min(Line 5d, $10,000)
- **Notes**: The SALT deduction is capped at $10,000 ($5,000 if MFS). This is a major limitation for high-tax states.

### Interest You Paid (Lines 8-10)

#### Line 8a: Home mortgage interest from 1098
- **Source**: 1098 Box 1
- **Notes**: Deductible on up to $750,000 of mortgage debt (post Dec 2017)

#### Line 8b: Home mortgage interest not reported on 1098
- **Source**: User-provided (private mortgages)

#### Line 8c: Points not reported on 1098
- **Source**: HUD-1 settlement statement

#### Line 9: Investment interest
- **Source**: Form 4952
- **Notes**: Limited to net investment income

### Gifts to Charity (Lines 11-14)

#### Line 11: Cash contributions
- **Source**: Receipts, bank statements
- **Notes**: Need written acknowledgment for gifts of $250+. Generally limited to 60% of AGI for cash to public charities.

#### Line 12: Noncash contributions
- **Source**: Form 8283 if over $500
- **Notes**: Fair market value. Need appraisal for items over $5,000.

#### Line 13: Carryover from prior year
- **Source**: Prior year return

#### Line 14: Total charitable
- **Computation**: Lines 11 + 12 + 13

### Other Deductions (Line 16)

- **Source**: User-provided
- **Notes**: Limited items: gambling losses (up to winnings), casualty/theft losses from federally declared disaster, certain unrecovered pension investments

### Total Itemized Deductions (Line 17)

- **Computation**: Line 4 + Line 5e + Line 8a + Line 8b + Line 8c + Line 9 + Line 14 + Line 16

## Flows To

- **1040 Line 12**: Total itemized deductions (Line 17)

## Decision Helper

Calculate both and compare:
```
Standard deduction: Look up for filing status
Itemized total: Schedule A Line 17

If itemized > standard: use Schedule A
If standard >= itemized: skip Schedule A, use standard deduction
```

## Common Mistakes

1. Forgetting the $10,000 SALT cap
2. Including pre-tax health insurance premiums (already excluded from W-2 Box 1)
3. Not having documentation for charitable contributions
4. Including commuting costs as medical mileage
5. Double-counting state taxes (already paid vs estimated)
