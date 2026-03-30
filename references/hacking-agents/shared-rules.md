# Shared Scan Rules

## Reading

Your bundle has two sections:

1. **Core source** (inline) - read in parallel chunks (offset + limit), compute offsets from the line count in your prompt.
2. **Peripheral file manifest** - file paths under `# Peripheral Files (read on demand)`. Read only relevant files for your specialty.

When matching handlers, check both direct instruction functions and internal helper variants (`instruction`, `process_instruction`, prefixed helpers like `_instruction`).

## Cross-program pattern replay

When you find a bug in one program, weaponize that pattern across all programs and instructions in the bundle. Missing repeats is an audit failure.

After scanning, escalate each finding to the worst realistic exploit variant, then revisit neighboring branches and helper paths.

## Do not report

Admin-only operations doing explicitly admin things, style issues, pure refactors, and speculative concerns without a concrete exploit path.

## Output

Return structured blocks only - no preamble, no narration. Exception: vector scan agent outputs classification block first.

FINDINGs require concrete exploitability. LEADs are for real code smells with incomplete exploit traces. Prefer LEAD over dropping.

**Every FINDING must include `proof:` with concrete values, state transitions, or call sequence from the code.** No proof = LEAD.

**One vulnerability per item.** Same root cause = one item. Different fixes = separate items.

```
FINDING | program: Name | instruction: ix_name | bug_class: kebab-tag | group_key: Program | instruction | bug-class
path: caller -> instruction -> state change -> impact
proof: concrete values/trace demonstrating exploitability
description: one sentence
fix: one-sentence suggestion

LEAD | program: Name | instruction: ix_name | bug_class: kebab-tag | group_key: Program | instruction | bug-class
code_smells: concrete suspicious patterns
description: one sentence explaining trail and what remains unverified
```

The `group_key` enables deduplication: `Program | instruction | bug_class`.
