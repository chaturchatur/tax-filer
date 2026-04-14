---
document: "1098 Series"
source: "IRS 1098 Form Instructions"
---

# 1098 Forms: Deduction-Related Information Returns

1098s report payments that may be deductible or qualify for credits.

## 1098: Mortgage Interest Statement

Issued by mortgage lenders.

| Box | Name | Flows to |
|-----|------|----------|
| 1 | Mortgage interest received | Schedule A Line 8a |
| 2 | Outstanding mortgage principal | Used for interest deduction limit |
| 5 | Mortgage insurance premiums | Schedule A Line 8d (if applicable) |
| 6 | Points paid | Schedule A Line 8c (deductible in year paid or amortized) |

**Notes**:
- Mortgage interest deduction only applies if itemizing
- Interest deductible on up to $750,000 of mortgage debt (acquired after Dec 15, 2017)
- For older mortgages: up to $1,000,000

**JSON schema** (`1098-mortgage-{identifier}.json`):
```json
{
  "document_type": "1098",
  "identifier": "lender1",
  "lender_name": "",
  "boxes": {
    "1_mortgage_interest": 0,
    "2_outstanding_principal": 0,
    "5_mortgage_insurance": 0,
    "6_points_paid": 0
  }
}
```

---

## 1098-T: Tuition Statement

Issued by educational institutions. Key form for education credits.

| Box | Name | Flows to |
|-----|------|----------|
| 1 | Payments received for qualified tuition | Form 8863 |
| 4 | Adjustments for prior year | May reduce credit |
| 5 | Scholarships or grants | Reduces qualified expenses |
| 7 | Checkbox: amounts for academic period beginning Jan-Mar of next year | Affects which tax year to claim |

**Qualified expenses for credit**: Box 1 - Box 5 = net qualified expenses (use on Form 8863)

**Notes**:
- American Opportunity Credit: up to $2,500 for first 4 years of undergrad. 40% refundable ($1,000 max).
- Lifetime Learning Credit: up to $2,000, no limit on years. Nonrefundable.
- Cannot claim both for the same student in the same year.
- MAGI phase-outs apply (check current year thresholds).

**JSON schema** (`1098-t-{identifier}.json`):
```json
{
  "document_type": "1098-T",
  "identifier": "university1",
  "school_name": "",
  "boxes": {
    "1_payments_received": 0,
    "4_adjustments_prior_year": 0,
    "5_scholarships_grants": 0,
    "7_next_year_indicator": false
  },
  "computed": {
    "net_qualified_expenses": 0
  }
}
```

---

## 1098-E: Student Loan Interest Statement

Issued by student loan servicers.

| Box | Name | Flows to |
|-----|------|----------|
| 1 | Student loan interest received | Schedule 1 Line 21 (adjustment) |

**Notes**:
- Deduction up to $2,500 of student loan interest paid
- Above-the-line deduction (don't need to itemize)
- MAGI phase-out: Single $80,000-$95,000, MFJ $165,000-$195,000 (2025, verify)
- Cannot claim if filing MFS

**JSON schema** (`1098-e-{identifier}.json`):
```json
{
  "document_type": "1098-E",
  "identifier": "servicer1",
  "servicer_name": "",
  "boxes": {
    "1_student_loan_interest": 0
  }
}
```

---

## 1098-C: Contributions of Motor Vehicles, Boats, and Airplanes

Rare. Only needed if donating a vehicle worth over $500 and claiming as charitable deduction on Schedule A.

---

## 1095-A: Health Insurance Marketplace Statement

Not technically a 1098, but collected with source docs. Required for Form 8962 (Premium Tax Credit).

| Column | Name | Flows to |
|--------|------|----------|
| A | Monthly enrollment premiums | Form 8962 |
| B | Monthly SLCSP premium | Form 8962 |
| C | Monthly advance payment of PTC | Form 8962 |

**JSON schema** (`1095-a-{identifier}.json`):
```json
{
  "document_type": "1095-A",
  "identifier": "marketplace1",
  "monthly_data": [
    {"month": "January", "enrollment_premium": 0, "slcsp_premium": 0, "advance_ptc": 0},
    {"month": "February", "enrollment_premium": 0, "slcsp_premium": 0, "advance_ptc": 0}
  ],
  "annual_totals": {
    "enrollment_premium": 0,
    "slcsp_premium": 0,
    "advance_ptc": 0
  }
}
```
