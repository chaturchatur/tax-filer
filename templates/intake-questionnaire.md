# Intake Questionnaire

Ask these questions conversationally, one stage at a time. After each stage, update profile.json with answers and trigger flags.

---

## Stage 1: Filing Basics

1. **Tax year**: What tax year are you filing for? (e.g., 2025)
2. **Filing status**: What's your filing status?
   - Single
   - Married Filing Jointly
   - Married Filing Separately
   - Head of Household (must have a qualifying dependent and paid >50% of household costs)
   - Qualifying Surviving Spouse (spouse died in prior 1-2 years, have dependent child)
   - If unsure, read `references/federal/filing-status.md` and help them determine it
3. **Dependent status**: Can anyone claim you as a dependent on their return? (common for students whose parents still claim them)
4. **Dependents**: Do you have any dependents?
   - If yes: name, relationship, last 4 of SSN, date of birth, months lived with you
5. **State**: What state did you live in during the tax year?
6. **State residency**: Did you live in that state all year, or did you move?
   - If moved: what states, and roughly when did you move?

After Stage 1: save filing basics to profile.json. Trigger state research (Phase 2) based on state answer.

---

## Stage 2: Income Types

These set the form trigger flags. Ask conversationally.

7. **W-2 employment**: Did you work as a W-2 employee? How many W-2s will you have?
   - Sets: `has_w2`, `w2_count`
8. **Self-employment**: Did you earn any self-employment or freelance income?
   - Sets: `has_self_employment_income` -> triggers Schedule C, possibly Schedule SE
9. **1099-NEC**: Did you receive any 1099-NEC forms (for freelance/contract work)?
   - Sets: `has_1099_nec` -> triggers Schedule C
10. **Interest**: Did you earn any interest income (bank accounts, CDs, bonds)?
    - Sets: `has_interest_income`
    - Follow-up: was total interest over $1,500? Sets `interest_over_1500` -> triggers Schedule B
11. **Dividends**: Did you receive any dividend income?
    - Sets: `has_dividend_income` -> if over $1,500, triggers Schedule B
12. **Capital gains**: Did you sell any stocks, crypto, or other investments?
    - Sets: `has_capital_gains` -> triggers Schedule D, Form 8949
13. **Rental income**: Did you receive any rental income?
    - Sets: `has_rental_income` -> triggers Schedule E
14. **Unemployment**: Did you receive unemployment compensation?
    - Sets: `has_unemployment` -> Schedule 1
15. **Social Security**: Did you receive Social Security benefits?
    - Sets: `has_social_security` -> may be partially taxable
16. **Other income**: Any other income? (gambling, jury duty, alimony received before 2019, etc.)

After Stage 2: update profile.json triggers.

---

## Stage 3: Adjustments and Deductions

17. **Traditional IRA**: Did you contribute to a traditional IRA?
    - Sets: `has_traditional_ira_contribution` -> Schedule 1 adjustment
18. **Student loan interest**: Did you pay student loan interest? (1098-E)
    - Sets: `has_student_loan_interest` -> Schedule 1 adjustment
19. **HSA**: Did you contribute to a Health Savings Account?
    - Sets: `has_hsa` -> Form 8889
20. **Deduction preference**: Do you want to itemize deductions or take the standard deduction?
    - If unsure: "We can compute both and see which saves you more."
    - Sets: `itemize_deductions`
    - If itemizing, ask about: mortgage interest (1098), state/local taxes paid, charitable contributions, medical expenses over 7.5% of AGI

After Stage 3: update profile.json triggers.

---

## Stage 4: Credits

21. **Education**: Did you pay college tuition or receive a 1098-T?
    - Sets: `has_1098_t`, `is_student`, `has_education_expenses` -> Form 8863
22. **Child Tax Credit**: Do you have children under 17?
    - Sets: `has_children_under_17` -> Form 8812
23. **Childcare**: Did you pay for childcare so you could work?
    - Sets: `has_childcare_expenses` -> Form 2441
24. **Marketplace insurance**: Did you get health insurance through the marketplace (healthcare.gov)?
    - Sets: `has_marketplace_insurance` -> Form 8962, needs 1095-A
25. **EITC**: Earned Income Tax Credit is computed automatically based on income and dependents. No question needed -- just note it for computation.

After Stage 4: update profile.json triggers.

---

## Stage 5: Other Situations

26. **Estimated payments**: Did you make estimated tax payments during the year?
    - Sets: `has_estimated_payments`
    - If yes: collect dates and amounts
27. **Extension**: Did you file for an extension?
28. **Prior year balance**: Do you owe anything from a prior year?
29. **Disaster**: Were you affected by a federally declared disaster? (may extend deadlines)

After Stage 5: finalize profile.json. Run form determination (Phase 1, Step 3).
