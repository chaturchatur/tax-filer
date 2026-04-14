# Form Worksheet: {FORM_NAME}

Use this template when presenting form values to the user for confirmation.

---

## {Form ID}: {Form Title}

Tax Year: {YYYY}
Status: Draft

### Computed Values

| Line | Description | Value | Source |
|------|-------------|-------|--------|
| {N}  | {desc}      | ${X}  | {source document and field} |

### Summary

- **Key figure**: ${amount}
- **Flows to**: {parent form, line number}

### Notes

- {Any flags, warnings, or things the user should double-check}

---

After user confirms, save to `projects/taxes/YYYY/forms/{form-id}.json` and mark all lines as `"verified": true`.
