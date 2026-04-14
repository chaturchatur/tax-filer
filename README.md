# tax-filer

A Claude Code skill that walks you through filing federal and state income taxes step by step. It determines what forms you need, collects your document values, computes every line with full source tracing, verifies everything, and tells you how to file for free.

Built for use with [Claude Code](https://claude.ai/code).

---

## What It Does

Instead of paying TurboTax or guessing at forms, you have a conversation with Claude. It asks the right questions, figures out which IRS forms apply to your situation, reads your W-2s and 1099s, fills out every line with references back to the source, double-checks the math, and points you to free filing options.

The skill handles:
- Single and multi-income W-2 filers
- Self-employment (Schedule C, SE tax)
- Capital gains and losses (Schedule D, Form 8949)
- Rental income (Schedule E)
- Education credits (Form 8863)
- HSA contributions and distributions (Form 8889)
- Itemized vs standard deduction comparison
- State income taxes (all 50 states)
- Any IRS form via dynamic registration

---

## Quick Start

### Prerequisites

- [Claude Code](https://claude.ai/code) installed
- A Claude Code project/workspace where you want to file taxes

### Installation

Clone this repo into your Claude Code skills directory:

```bash
git clone https://github.com/chaturchatur/tax-filer.git .claude/skills/tax-filer
```

Or copy the entire `tax-filer/` folder into `.claude/skills/` in any Claude Code project.

### Usage

Start a conversation with Claude Code and say any of:

- "help me file taxes"
- "do my taxes"
- "I need to file my tax return"
- "what forms do I need"
- "help with 1040"

Claude will take it from there.

---

## How It Works

The skill runs in 7 phases. You can stop at any point and pick up later -- it saves progress and resumes where you left off.

### Phase 0: Security and Disclaimer

Before anything else:
- Reminds you it's not a licensed tax professional
- Adds your tax data directory to `.gitignore`
- Enforces SSN redaction (last 4 digits only, always)
- Never stores bank account or routing numbers

### Phase 1: Intake and Form Determination

Claude walks you through a conversational questionnaire in 5 stages:

1. **Filing basics** -- tax year, filing status, dependents, state
2. **Income types** -- W-2s, self-employment, interest, dividends, capital gains, rental, unemployment
3. **Adjustments and deductions** -- IRA, student loans, HSA, standard vs itemized
4. **Credits** -- education, child tax, childcare, marketplace insurance
5. **Other situations** -- estimated payments, extensions, prior year balances

Each answer sets trigger flags. After the questionnaire, the skill matches your flags against its form routing table (`form-index.json`) and tells you exactly which forms you need:

> "Based on your answers, you'll need: Form 1040, Schedule B, Form 8863. Does that sound right?"

### Phase 2: State Tax Research

If you live in a state with income tax, Claude dispatches a research agent that:
- Runs 5-8 targeted web searches for your state's current-year tax rules
- Fetches official .gov sources
- Compiles brackets, deductions, credits, required forms, and free filing portals
- Flags complexity (local taxes, part-year residency, etc.)

States with no income tax (AK, FL, NV, NH, SD, TN, TX, WA, WY) skip this entirely.

### Phase 3: Source Document Collection

For each document type (W-2, 1099-NEC, 1099-INT, etc.), Claude:

1. Loads the field guide that maps every box to where it flows on your tax forms
2. Collects values either by asking you box by box, or by reading an image/PDF you provide
3. Saves each document as structured JSON with sanity checks:
   - W-2 Box 2 (tax withheld) should be less than Box 1 (wages)
   - 1099-NEC amounts should match what you described as self-employment income
   - Income totals should roughly match what you said in intake

### Phase 4: Form Completion

Forms are processed in strict dependency order -- schedules and supporting forms before 1040, state return last:

```
Schedule C -> Schedule SE -> Form 8949 -> Schedule D -> Schedule E ->
Schedule A -> Form 8889 -> Form 8863 -> Schedule 1 -> Schedule 2 ->
Schedule 3 -> Form 1040 -> State Return
```

For each form, Claude:
1. Reads the line-by-line instruction file
2. Pulls values from your source documents
3. Computes each line (with formulas from IRS instructions)
4. Shows you the computed values with sources before saving
5. Saves to JSON with full traceability

Every line stores what it computed, where the value came from, and whether you've confirmed it:

```json
{
  "value": 15000,
  "source": "w2-employer1.json Box 1",
  "verified": false
}
```

#### Instruction Bootstrap

If Claude doesn't have the IRS instructions for a form yet, it offers three ways to get them:

1. **Drop a PDF** -- download the instructions from irs.gov and put them in `references/federal/raw/`
2. **Give a URL** -- paste the IRS instructions page URL
3. **Auto-search** -- Claude searches irs.gov and fetches the instructions itself

It parses the raw content into a structured line-by-line reference file and caches it for future use.

#### Dynamic Form Registration

Need a form that isn't pre-configured (Form 4868 for extensions, Form 1040-X for amendments, Form 8829 for home office, etc.)? Claude registers it on the fly -- adds it to the routing table, bootstraps the instructions, and processes it in the right order.

### Phase 5: Verification

A dedicated verification agent cross-checks the entire return:

| Check | What It Verifies |
|-------|-----------------|
| Income reconciliation | W-2 totals match 1040 Line 1, all source docs accounted for |
| AGI arithmetic | Income - adjustments = AGI, step by step |
| Deduction logic | If itemizing, total exceeds standard deduction |
| Tax computation | Taxable income x bracket rates = correct tax |
| Credit phase-outs | Credits not claimed above income thresholds |
| Payment reconciliation | Withholding + estimated payments = total payments |
| Refund/balance due | Total payments - total tax |
| State consistency | Federal AGI flows to state correctly |
| Common errors | >4 years AOTC, missing SE tax, forgotten income |

Results are written to a verification log. Any discrepancies are presented to you with expected vs actual values before you proceed.

### Phase 6: Filing Guidance

Claude recommends how to file, always free options first:

1. **IRS Direct File** -- if eligible (limited states/forms)
2. **IRS Free File** -- if AGI under threshold
3. **Free File Fillable Forms** -- any income, no guidance needed (this skill is the guidance)
4. **VITA** -- free in-person help if income under $67k
5. **Paper filing** -- always free, slowest

For state returns, it uses the free e-file portal found during state research.

A final summary is generated with all key figures, forms completed, filing instructions, and deadlines.

---

## Resuming a Previous Session

If you come back to file taxes and there's existing progress, Claude detects it automatically:

> "I found existing progress for tax year 2025: profile complete, 2 W-2s collected, Schedule C and SE filled out. Want to pick up where you left off, or start over?"

It scans for:
- `profile.json` -- intake complete
- `source-documents/` -- which docs are already collected
- `forms/` -- which forms are already computed
- `verification-log.md` -- whether verification ran and passed

---

## Directory Structure

### Skill files (this repo)

```
tax-filer/
├── SKILL.md                          # Main orchestration (all 7 phases)
├── agents/
│   ├── state-researcher.md           # State tax research agent
│   └── verifier.md                   # Return verification agent
├── references/
│   ├── federal/
│   │   ├── form-index.json           # Master routing table
│   │   ├── filing-status.md          # Filing status decision tree
│   │   ├── 1040.md                   # Form 1040 line-by-line
│   │   ├── 1040-schedule-a.md        # Itemized Deductions
│   │   ├── 1040-schedule-b.md        # Interest and Dividends
│   │   ├── 1040-schedule-c.md        # Business Income
│   │   ├── 1040-schedule-d.md        # Capital Gains
│   │   ├── 1040-schedule-se.md       # Self-Employment Tax
│   │   ├── 8863.md                   # Education Credits
│   │   └── 8889.md                   # HSA
│   ├── source-docs/
│   │   ├── w2-fields.md              # W-2 box-by-box guide
│   │   ├── 1099-types.md             # All 1099 variants
│   │   └── 1098-types.md             # Mortgage, tuition, student loans
│   └── filing-options.json           # Free and paid filing methods
└── templates/
    ├── intake-questionnaire.md       # Conversational question flow
    ├── form-worksheet.md             # Per-form value presentation
    ├── tax-return-summary.md         # Final summary template
    └── state-research-output.md      # State research results format
```

### User data (created during filing, NOT in this repo)

```
projects/taxes/2025/
├── README.md                         # Final summary with key figures
├── profile.json                      # Filing status, triggers, required forms
├── source-documents/
│   ├── w2-employer1.json             # Parsed W-2 (SSN last 4 only)
│   ├── 1099-nec-client1.json         # Parsed 1099-NEC
│   └── ...
├── forms/
│   ├── 1040.json                     # Computed values with source tracing
│   ├── schedule-c.json
│   └── ...
├── state/
│   ├── research.md                   # Raw state research
│   ├── instructions.md               # Condensed state filing guide
│   └── state-return.json
└── verification-log.md               # Cross-check results
```

---

## Pre-Built Knowledge Base

The skill ships with line-by-line instruction files for 9 federal forms and field guides for 3 source document families. These cover the most common filing situations out of the box.

### Federal Forms (ready to use)
- Form 1040, Schedule A, Schedule B, Schedule C, Schedule D, Schedule SE, Form 8863, Form 8889, Filing Status guide

### Source Document Guides (ready to use)
- W-2 (all boxes)
- 1099 family (NEC, INT, DIV, B, G, MISC, R, SA)
- 1098 family (mortgage, tuition, student loan) + 1095-A

### Forms That Bootstrap on Demand
Schedule 1, Schedule 2, Schedule 3, Schedule E, Form 8949, Form 8812, Form 2441, Form 8962 -- and any other IRS form via dynamic registration.

---

## Tax Year Rollover

When a new tax year starts, tell Claude "start new tax year" and it will:

1. Reset all `instructions_cached` flags (IRS updates forms annually)
2. Clear tax brackets and standard deduction data for re-fetching
3. Scaffold the new year's directory
4. Preserve the old year's data for reference
5. Delete old instruction files so they get re-bootstrapped from current IRS sources

---

## Security

- Full SSNs are never stored. Last 4 digits only (`XXX-XX-1234`).
- Bank account and routing numbers are never stored.
- If you paste a full SSN, Claude warns you immediately and only records the last 4.
- All tax data lives in `projects/taxes/YYYY/`, which is added to `.gitignore`.
- Tax data is never stored inside the skill directory.

---

## Limitations

- This is not tax advice. Complex situations (AMT, foreign income, trusts, business with employees, audits) should go to a CPA.
- The skill computes values based on IRS instructions but cannot e-file for you. It tells you where and how to file.
- State tax instructions are researched via web search and may not cover every edge case.
- Form instruction files may need re-bootstrapping if the IRS changes form layouts mid-season.

---

## License

MIT
