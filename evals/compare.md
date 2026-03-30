# Eval Compare

Compare an audit report against benchmark ground truth.

You are given two files:

1. **Ground truth** benchmark file with known findings
2. **Report** audit output (`final-report.md` or `full-output.txt`)

## Steps

1. Parse each `FINDING` line and matching `description:` from ground truth.
2. Parse report sections:
   - **Findings**: between `## Findings` and `## Leads`
   - **Leads**: from `## Leads` to end
3. For each ground-truth finding, classify status:
   - **FOUND**: same root cause in Findings section (program + instruction context match semantically)
   - **LEAD**: appears only in Leads section
   - **MISSED**: absent from both sections

## Output

Write `summary.md` in run directory with this format:

```
## Eval Results

| Metric | Value |
|--------|-------|
| Recall (findings) | {found} / {total} ({pct}%) |
| In leads only | {leads} |
| Missed | {missed} |
| High | {high_found} / {high_total} |
| Medium | {med_found} / {med_total} |
| Reported findings | {count from report} |

### Per-finding breakdown

| Status | Severity | ID | Program.Instruction | Bug Class |
|--------|----------|----|---------------------|-----------|
| FOUND | High | H-1 | Program.instruction | bug-class |
| LEAD | Medium | M-2 | Program.instruction | bug-class |
| MISSED | Medium | M-3 | Program.instruction | bug-class |
```

## Rules

- Match semantically, not by strict keyword matching.
- Leads do not count toward findings recall.
- If same root cause is identified in nearby instruction context within same program, count as FOUND.
- If one reported finding covers multiple benchmark findings, count each covered benchmark finding as FOUND.
