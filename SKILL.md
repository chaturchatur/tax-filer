---
name: tax-filer
description: >
  Guide users through filing federal and state income taxes step by step.
  Determine required forms, collect source document values, compute form entries,
  verify totals, and provide filing guidance. Use this skill when the user says
  "help me file taxes", "do my taxes", "tax return", "I need to file",
  "what forms do I need", "help with 1040", or anything related to preparing
  or filing income tax returns. Also trigger when the user mentions W-2s,
  1099s, tax deductions, tax credits, or tax refunds in the context of filing.
allowed-tools: Read, Grep, Glob, Write, Edit, WebSearch, WebFetch, Agent, Bash
user-invocable: true
---

# Tax Filer

Walks users through filing federal and state income taxes. Determines required forms, collects source document data, computes form entries, verifies everything, and suggests how to file for free.

**Read this entire skill before doing anything.**

---

## Phase 0: Security and Disclaimer

Before starting, always:

1. Tell the user: "I can help you prepare your return, but I'm not a licensed tax professional. For complex situations, consult a CPA."
2. Check if `projects/taxes/` is in `.gitignore`. If not, add it.
3. These rules apply throughout the entire process:
   - **Never store full SSNs.** Last 4 digits only (`XXX-XX-1234`).
   - **Never store bank account or routing numbers.**
   - If the user pastes a full SSN, warn them immediately and only record the last 4.
   - All sensitive data lives in `projects/taxes/YYYY/`, never inside the skill folder.

---

## Phase 1: Intake and Form Determination

Goal: figure out what forms the user needs.

### Step 0: Check for existing progress

Before starting fresh, check if `projects/taxes/YYYY/` already exists for the requested tax year. If it does, read `profile.json` and scan for existing data:

- **profile.json exists with `required_forms`**: Intake is done. Skip to the earliest incomplete phase.
- **source-documents/ has files**: Some documents already collected. Skip those in Phase 3.
- **forms/ has files**: Some forms already computed. Skip those in Phase 4.
- **verification-log.md exists**: Verification was run. Check if it passed or had unresolved issues.

Tell the user what you found:

> "I found existing progress for tax year YYYY: profile complete, 2 W-2s collected, Schedule C and SE filled out. Want to pick up where you left off, or start over?"

If resuming, jump to the first incomplete phase. If starting over, archive the existing directory to `projects/taxes/YYYY-backup-[date]/` and begin fresh.

### Step 1: Set up the tax year directory

```
projects/taxes/YYYY/
├── README.md
├── profile.json
├── source-documents/
├── forms/
├── state/
└── verification-log.md
```

Create the directory structure and an empty `profile.json`.

### Step 2: Walk through the intake questionnaire

Read `templates/intake-questionnaire.md` for the full question flow. Ask questions **conversationally** -- do not dump all questions at once. Group them into stages:

1. **Filing basics**: tax year, filing status, dependents, state of residence
2. **Income types**: W-2 employment, self-employment, interest, dividends, capital gains, rental, unemployment, Social Security, other
3. **Adjustments and deductions**: IRA contributions, student loan interest, HSA, standard vs itemized
4. **Credits**: education (1098-T), child tax credit, childcare, marketplace insurance, EITC
5. **Other situations**: estimated payments, extensions, prior year balances

After each stage, update `profile.json` with the answers and trigger flags.

### Step 3: Determine required forms

Read `references/federal/form-index.json`. Match the user's trigger flags against the form triggers to build the required forms list. Present the list to the user:

> "Based on your answers, you'll need: Form 1040, Schedule B, Form 8863. Does that sound right?"

Save the required forms list to `profile.json`.

---

## Phase 2: State Tax Research

Triggered when the user specifies their state in Phase 1.

### No-income-tax states

Skip this phase entirely for: AK, FL, NV, NH, SD, TN, TX, WA, WY. Note it in `profile.json` and move on.

### For all other states

Dispatch the state researcher agent (`agents/state-researcher.md`) with:
- The user's state
- Tax year
- Filing status and income types from `profile.json`

The agent performs web research and writes results to `projects/taxes/YYYY/state/research.md`. Then condense the research into `state/instructions.md` -- a line-by-line guide for the state return.

---

## Phase 3: Source Document Collection

For each source document type the user has:

### Step 1: Read the field guide (bootstrap if needed)

Check if a field guide exists in `references/source-docs/` for this document type. Look up the `source_doc_guides` section in `form-index.json` to find the file path and check `cached`.

If the guide exists, read it. If not, bootstrap it the same way as form instructions:

> "I need a field guide for [Document Type] to know which boxes to collect. Three options:
> 1. Drop the IRS/issuer instructions into `references/source-docs/raw/`
> 2. Give me a URL describing the form fields
> 3. I'll search for it myself"

Parse the source and write a structured field guide to `references/source-docs/<type>.md`:

```markdown
---
document_type: "[type, e.g. 1099-K]"
tax_year: [year]
source: "[where content came from]"
---

# [Document Type]: [Title]

## Overview
[What this document reports and who receives it]

## Box-by-Box Guide

| Box | Name | Description | Flows To |
|-----|------|-------------|----------|
| 1   | ...  | ...         | ...      |

## JSON Schema
[Schema for storing extracted values]

## Sanity Checks
- [check 1]
```

Set `cached` to `true` in `form-index.json` after writing. Pre-built guides exist for W-2, 1099 variants, and 1098 variants.

### Step 2: Collect values

Two modes:
- **Manual entry**: Ask for values box by box using the field guide
- **Image/PDF extraction**: If the user provides an image or PDF, read it with the Read tool, extract values, and present them for confirmation

Important: remind the user to redact their SSN before sharing any images.

### Step 3: Save and sanity check

Save each document to `projects/taxes/YYYY/source-documents/<type>-<identifier>.json`.

Quick checks:
- W-2 Box 1 should be reasonable for the stated employment
- W-2 Box 2 should be less than Box 1
- 1099-NEC amounts should match what the user reported as self-employment income
- Sum of all income sources should roughly match what the user described in intake

Flag anything that looks off.

---

## Phase 4: Form Completion

Process forms in dependency order (schedules and supporting forms before 1040, state last).

### Dependency order

1. Schedule C (business income) -- needs 1099-NEC data
2. Schedule SE (self-employment tax) -- needs Schedule C net profit
3. Schedule D / Form 8949 (capital gains) -- needs brokerage data
4. Schedule E (rental income) -- needs rental docs
5. Schedule A (itemized deductions) -- needs 1098s, receipts
6. Form 8889 (HSA) -- needs HSA statements
7. Form 8863 (education credits) -- needs 1098-T
8. Schedule 1 (additional income and adjustments) -- aggregates from above
9. Schedule 2 (additional taxes) -- needs SE tax, etc.
10. Schedule 3 (additional credits) -- needs education credits, etc.
11. **Form 1040** -- final aggregation of everything
12. **State return** -- needs federal AGI

### Instruction Bootstrap

Before processing any form, check if its instruction file exists (look up the path in `form-index.json` and check `instructions_cached`). If the file doesn't exist yet, you need to build it from an authoritative source before you can compute form values.

Tell the user:

> "I need the IRS instructions for [Form Name] before I can fill it out. Three options:
> 1. Drop the instruction PDF into `references/federal/raw/` (download from irs.gov)
> 2. Give me the URL (e.g. https://www.irs.gov/instructions/i1040)
> 3. I'll search for it myself"

Then based on their choice:

- **PDF**: Read the PDF from `references/federal/raw/`, extract the line-by-line content
- **URL**: WebFetch the page, parse the relevant sections
- **Search**: WebSearch for `"Instructions for [Form Name] [tax year]" site:irs.gov`, then WebFetch the top result

Parse the raw content into a structured instruction file at `references/federal/<form>.md`:

```markdown
---
form: "[form number]"
tax_year: [year]
source: "[PDF filename, URL, or search result URL]"
---

# Form [Number]: [Title]

## Overview
[What this form is for and who files it]

## Line-by-Line Instructions

### [Section Name]

#### Line [N]: [Description]
- **Source**: [what document/box provides this value]
- **Computation**: [formula or lookup]
- **Notes**: [gotchas, edge cases]

## Flows To
- **[Target form] Line [N]**: [what value flows there]

## Common Mistakes
1. [mistake]
```

After writing the file, set `instructions_cached` to `true` in `form-index.json` and confirm with the user before proceeding.

Some forms already ship with pre-built instruction files (marked `instructions_cached: true` in `form-index.json`). The bootstrap only runs for forms that don't have one yet. Over time, the reference library grows as the user files returns with different forms. If the tax year changes, the user can re-bootstrap any form to get updated instructions.

### Dynamic Form Registration

If the user needs a form that isn't in `form-index.json` at all (e.g., Form 4868 for extensions, Form 1040-X for amendments, Form 8829 for home office, or any other IRS form), register it on the fly:

1. Ask the user what the form is for and what triggers it
2. Add a new entry to `form-index.json`:
   ```json
   "4868": {
     "name": "Application for Automatic Extension of Time to File",
     "required": false,
     "instructions_file": "references/federal/4868.md",
     "instructions_cached": false,
     "depends_on": [],
     "triggers": {},
     "trigger_logic": "any",
     "notes": "Dynamically registered"
   }
   ```
3. Add the form to `processing_order` in the appropriate position (before forms that depend on it, after forms it depends on)
4. Add any new trigger flags to `profile.json` if needed
5. Proceed with the Instruction Bootstrap to build the instruction file

This means the skill can handle any IRS form, not just the ones pre-defined in the index. The index grows organically as users encounter new tax situations.

### Per-form process

For each required form:

1. Check `instructions_cached` in `form-index.json`. If `false`, run the Instruction Bootstrap above.
2. Read the form's instruction file from `references/federal/<form>.md`
3. Read all relevant source document JSON files from `projects/taxes/YYYY/source-documents/`
4. Compute each line using the instructions (Source, Computation, Notes fields)
5. Present computed values to the user for confirmation -- **never save silently**
6. Save to `projects/taxes/YYYY/forms/<form>.json`

Each line in the JSON stores:
```json
{
  "value": 15000,
  "source": "w2-employer1.json Box 1",
  "verified": false
}
```

---

## Phase 5: Verification

After all forms are complete, dispatch the verifier agent (`agents/verifier.md`).

### Checks performed

1. **Income reconciliation**: sum of all W-2 Box 1 = 1040 Line 1
2. **AGI arithmetic**: income - adjustments = AGI
3. **Deduction logic**: if itemizing, verify total > standard deduction
4. **Tax computation**: taxable income x bracket rates = tax
5. **Credit phase-outs**: credits not claimed above income thresholds
6. **Payment reconciliation**: W-2 Box 2 (all) + estimated payments = total payments
7. **Refund/balance due**: total payments - total tax
8. **State consistency**: federal AGI flows to state correctly
9. **Common errors**: >4 years AOTC, missing SE tax, forgotten income sources

Write results to `projects/taxes/YYYY/verification-log.md`. Present any discrepancies to the user and resolve them before proceeding.

---

## Phase 6: Filing Guidance

### Step 1: Determine filing options

Read `references/filing-options.json`. Filter by the user's AGI and required forms.

### Step 2: Present options (free first)

Ranking:
1. **IRS Direct File** (if eligible -- limited states and form types)
2. **IRS Free File** (AGI under threshold)
3. **Free File Fillable Forms** (any income, no guidance but this skill provides it)
4. **VITA** (in-person, if eligible)
5. **Paper filing by mail** (always free, slowest)

For state: use the free e-file portal found during state research.

Only mention paid options if the user specifically asks.

### Step 3: Generate summary

Use `templates/tax-return-summary.md` to create a final summary in `projects/taxes/YYYY/README.md` that includes:
- Filing status and key figures (AGI, taxable income, total tax, refund/owed)
- All forms completed
- Recommended filing method with step-by-step instructions
- Important deadlines

---

## Form Data Schemas

### profile.json

```json
{
  "tax_year": 2025,
  "filing_status": "single",
  "is_dependent": false,
  "dependents": [],
  "state": "MD",
  "state_residency": "full_year",
  "ssn_last4": "1234",
  "triggers": {
    "has_w2": true,
    "w2_count": 1,
    "has_self_employment_income": false,
    "has_1099_nec": false,
    "has_interest_income": false,
    "interest_over_1500": false,
    "has_dividend_income": false,
    "has_capital_gains": false,
    "has_rental_income": false,
    "has_unemployment": false,
    "has_social_security": false,
    "has_traditional_ira_contribution": false,
    "has_student_loan_interest": false,
    "has_hsa": false,
    "itemize_deductions": false,
    "is_student": false,
    "has_1098_t": false,
    "has_education_expenses": false,
    "has_children_under_17": false,
    "has_childcare_expenses": false,
    "has_marketplace_insurance": false,
    "has_estimated_payments": false
  },
  "required_forms": [],
  "notes": ""
}
```

### Source document JSON (e.g., w2-employer1.json)

```json
{
  "document_type": "W-2",
  "identifier": "employer1",
  "employer_name": "Acme Corp",
  "employer_ein": "XX-XXXXXXX",
  "employee_ssn_last4": "1234",
  "boxes": {
    "1_wages": 50000,
    "2_federal_tax_withheld": 5000,
    "3_social_security_wages": 50000,
    "4_social_security_tax": 3100,
    "5_medicare_wages": 50000,
    "6_medicare_tax": 725,
    "12a_code": null,
    "12a_amount": null,
    "13_statutory_employee": false,
    "13_retirement_plan": false,
    "15_state": "MD",
    "16_state_wages": 50000,
    "17_state_tax_withheld": 2500
  }
}
```

### Form values JSON (e.g., 1040.json)

```json
{
  "form": "1040",
  "tax_year": 2025,
  "status": "draft",
  "lines": {
    "1": {"value": 50000, "source": "w2-employer1.json Box 1", "verified": false},
    "9": {"value": 50000, "source": "sum of income lines", "verified": false},
    "11": {"value": 50000, "source": "line 9 - line 10", "verified": false},
    "12": {"value": 15700, "source": "standard deduction (single)", "verified": false},
    "15": {"value": 34300, "source": "line 11 - line 14", "verified": false}
  },
  "computed_values": {
    "total_income": 50000,
    "agi": 50000,
    "taxable_income": 34300,
    "total_tax": 3878,
    "total_payments": 5000,
    "refund": 1122
  }
}
```

---

## Tax Year Rollover

When the user says "start new tax year", "new tax year", or begins filing for a different year than what's currently in `form-index.json`:

1. **Update form-index.json**: Set `instructions_cached` to `false` on ALL forms and source doc guides. IRS updates forms annually so last year's instructions may be wrong.

2. **Update tax data**: Set `tax_year` in form-index.json to the new year. Clear `standard_deduction_YYYY` and `tax_brackets_YYYY` -- these will be re-fetched from IRS.gov during form completion via WebSearch.

3. **Scaffold the new directory**:
   ```
   projects/taxes/[NEW_YEAR]/
   ├── README.md
   ├── profile.json
   ├── source-documents/
   ├── forms/
   ├── state/
   └── verification-log.md
   ```

4. **Preserve old data**: Do NOT delete the previous year's directory. It stays at `projects/taxes/[OLD_YEAR]/` for reference.

5. **Delete old instruction files**: Remove all `references/federal/*.md` files (except `filing-status.md` which is structural) and all `references/source-docs/*.md` files. They'll be re-bootstrapped from fresh IRS sources when needed.

6. Tell the user: "Ready for tax year [YYYY]. All instruction files have been cleared -- I'll rebuild them from current IRS sources as we go."

---

## Rules

- Process forms in dependency order. Schedules before 1040. State last.
- Never skip the verification phase.
- Always show computed values to the user before saving. No silent writes.
- If the situation is complex (AMT, foreign income, trust, business with employees, audit), recommend a CPA.
- Keep monetary values as numbers (not strings) in JSON, rounded to nearest dollar.
- Use the tax year (2025) for directory naming, not the filing year (2026).
- Always prefer free filing options. Only mention paid options if the user asks.
- When computing tax, use the official IRS tax brackets for the relevant year. If unsure, look them up via WebSearch.
