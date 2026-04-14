---
document: "1099 Series"
source: "IRS 1099 Form Instructions"
---

# 1099 Forms: Information Returns

1099s report various types of non-wage income. There are many variants. Here are the common ones.

## 1099-NEC: Nonemployee Compensation

Issued when you earned $600+ as a freelancer/contractor.

| Box | Name | Flows to |
|-----|------|----------|
| 1 | Nonemployee compensation | Schedule C (gross receipts) |
| 4 | Federal tax withheld | 1040 Line 25b |

**JSON schema** (`1099-nec-{identifier}.json`):
```json
{
  "document_type": "1099-NEC",
  "identifier": "client1",
  "payer_name": "",
  "payer_tin": "",
  "recipient_ssn_last4": "",
  "boxes": {
    "1_nonemployee_compensation": 0,
    "4_federal_tax_withheld": 0
  }
}
```

---

## 1099-INT: Interest Income

Issued by banks, credit unions, brokerages for interest earned.

| Box | Name | Flows to |
|-----|------|----------|
| 1 | Interest income | 1040 Line 2b (taxable interest) |
| 2 | Early withdrawal penalty | Schedule 1 (adjustment) |
| 3 | Interest on US savings bonds | May be excluded if used for education |
| 4 | Federal tax withheld | 1040 Line 25b |
| 8 | Tax-exempt interest | 1040 Line 2a (informational) |

**JSON schema** (`1099-int-{identifier}.json`):
```json
{
  "document_type": "1099-INT",
  "identifier": "bank1",
  "payer_name": "",
  "boxes": {
    "1_interest_income": 0,
    "2_early_withdrawal_penalty": 0,
    "3_us_savings_bond_interest": 0,
    "4_federal_tax_withheld": 0,
    "8_tax_exempt_interest": 0
  }
}
```

---

## 1099-DIV: Dividends and Distributions

Issued by brokerages, mutual funds, companies paying dividends.

| Box | Name | Flows to |
|-----|------|----------|
| 1a | Total ordinary dividends | 1040 Line 3b |
| 1b | Qualified dividends | 1040 Line 3a (taxed at lower capital gains rates) |
| 2a | Total capital gain distributions | Schedule D |
| 3 | Nondividend distributions | Return of capital (reduces basis, not taxable) |
| 4 | Federal tax withheld | 1040 Line 25b |

**JSON schema** (`1099-div-{identifier}.json`):
```json
{
  "document_type": "1099-DIV",
  "identifier": "brokerage1",
  "payer_name": "",
  "boxes": {
    "1a_ordinary_dividends": 0,
    "1b_qualified_dividends": 0,
    "2a_capital_gain_distributions": 0,
    "3_nondividend_distributions": 0,
    "4_federal_tax_withheld": 0
  }
}
```

---

## 1099-B: Proceeds from Broker Transactions

Issued by brokerages for stock/crypto sales. Often comes as a consolidated statement.

| Box | Name | Flows to |
|-----|------|----------|
| 1d | Proceeds | Form 8949 |
| 1e | Cost basis | Form 8949 |
| 1f | Accrued market discount | Form 8949 |
| 2 | Short-term/long-term | Determines which part of Schedule D |
| 4 | Federal tax withheld | 1040 Line 25b |

**Notes**: Can have many transactions. May be easier to summarize totals by category (short-term covered, long-term covered, etc.) rather than entering each trade.

**JSON schema** (`1099-b-{identifier}.json`):
```json
{
  "document_type": "1099-B",
  "identifier": "brokerage1",
  "payer_name": "",
  "summary": {
    "short_term_covered": {"proceeds": 0, "cost_basis": 0, "gain_loss": 0},
    "short_term_uncovered": {"proceeds": 0, "cost_basis": 0, "gain_loss": 0},
    "long_term_covered": {"proceeds": 0, "cost_basis": 0, "gain_loss": 0},
    "long_term_uncovered": {"proceeds": 0, "cost_basis": 0, "gain_loss": 0}
  },
  "federal_tax_withheld": 0
}
```

---

## 1099-G: Government Payments

Covers unemployment compensation, state tax refunds.

| Box | Name | Flows to |
|-----|------|----------|
| 1 | Unemployment compensation | Schedule 1 Line 7 |
| 2 | State/local tax refund | Schedule 1 Line 1 (only if itemized last year) |
| 4 | Federal tax withheld | 1040 Line 25b |

**JSON schema** (`1099-g-{identifier}.json`):
```json
{
  "document_type": "1099-G",
  "identifier": "state1",
  "payer_name": "",
  "boxes": {
    "1_unemployment": 0,
    "2_state_tax_refund": 0,
    "4_federal_tax_withheld": 0
  }
}
```

---

## 1099-MISC: Miscellaneous Income

Less common after 1099-NEC was introduced. Now mainly for rents, royalties, prizes.

| Box | Name | Flows to |
|-----|------|----------|
| 1 | Rents | Schedule E |
| 2 | Royalties | Schedule E |
| 3 | Other income | Schedule 1 |
| 4 | Federal tax withheld | 1040 Line 25b |

---

## 1099-R: Retirement Distributions

Covers IRA, 401(k), pension distributions.

| Box | Name | Flows to |
|-----|------|----------|
| 1 | Gross distribution | 1040 Line 4a or 5a |
| 2a | Taxable amount | 1040 Line 4b or 5b |
| 4 | Federal tax withheld | 1040 Line 25a |
| 7 | Distribution code | Determines tax treatment |

---

## 1099-SA: HSA/MSA Distributions

| Box | Name | Flows to |
|-----|------|----------|
| 1 | Gross distribution | Form 8889 |
| 2 | Earnings on excess contributions | Form 8889 |
| 3 | Distribution code | 1 = normal, 2 = excess contribution |
