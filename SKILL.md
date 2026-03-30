---
name: solana-auditor
description: Security audit of Solana Rust/Anchor programs while you develop. Trigger on "audit", "check this program", "review for security". Modes - default (full repo) or specific filename(s).
---

# Solana Program Security Audit

You are the orchestrator of a parallelized Solana program security audit.

## Mode Selection

**Exclude pattern:** skip directories `.git/`, `target/`, `test/`, `tests/`, `node_modules/`, `migrations/`, `.anchor/`, `dist/`, `artifacts/`, `idl/` and files matching `*.test.rs`, `*_test.rs`, `*.spec.rs`.

- **Default** (no arguments): scan all in-scope `.rs` files using Bash `find` (not Glob).
- **`$filename ...`**: scan specified file(s) only.

**Flags:**

- `--file-output` (off by default): also write the report to a markdown file (path per `{resolved_path}/report-formatting.md`). Never write a report file unless explicitly passed.

## Orchestration

**Turn 1 - Discover.** Print the banner, then make these parallel tool calls in one message:

a. Bash `find` for in-scope `.rs` files per mode selection
b. Glob for `**/references/attack-vectors/attack-vectors.md` - extract the `references/` directory (two levels up) as `{resolved_path}`
c. ToolSearch `select:Agent`
d. Read the local `VERSION` file from the same directory as this skill
e. Bash `curl -sf https://raw.githubusercontent.com/kale/solana-auditor/main/solana-auditor/VERSION`
f. Bash `mktemp -d /tmp/solana-audit-XXXXXX` -> store as `{bundle_dir}`

If the remote VERSION fetch succeeds and differs from local, print `WARNING: You are not using the latest version. Please upgrade for best security coverage.` If it fails, skip silently.

**Turn 2 - Prepare.** In one message, make parallel tool calls: (a) Read `{resolved_path}/report-formatting.md`, (b) Read `{resolved_path}/judging.md`.

Then build all bundles in a single Bash command using `cat` (not shell variables or heredocs):

1. `{bundle_dir}/source.md` - ALL in-scope `.rs` files, each with a `### path` header and fenced code block. Also append root-level `Anchor.toml`, `Cargo.toml`, and relevant `Cargo.toml` files as peripheral context when present.
2. Agent bundles = `source.md` + agent-specific files:

| Bundle              | Appended files (relative to `{resolved_path}`) |
| ------------------- | ----------------------------------------------- |
| `agent-1-bundle.md` | `attack-vectors/attack-vectors.md` + `hacking-agents/vector-scan-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-2-bundle.md` | `hacking-agents/math-precision-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-3-bundle.md` | `hacking-agents/access-control-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-4-bundle.md` | `hacking-agents/economic-security-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-5-bundle.md` | `hacking-agents/execution-trace-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-6-bundle.md` | `hacking-agents/invariant-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-7-bundle.md` | `hacking-agents/periphery-agent.md` + `hacking-agents/shared-rules.md` |
| `agent-8-bundle.md` | `hacking-agents/first-principles-agent.md` + `hacking-agents/shared-rules.md` |

Print line counts for every bundle and `source.md`. Do NOT inline source content into agent prompts.

**Turn 3 - Spawn.** In one message, spawn all 8 agents as parallel foreground Agent calls. Prompt template (substitute real values):

```
Your bundle file is {bundle_dir}/agent-N-bundle.md (XXXX lines).
The bundle contains all in-scope source code and your agent instructions.
Read the bundle fully before producing findings.
```

**Turn 4 - Deduplicate, validate, and output.** Single-pass: deduplicate all agent results, gate-evaluate, and produce the final report in one turn. Do NOT print an intermediate dedup list.

1. **Deduplicate.** Parse every FINDING and LEAD from all 8 agents. Group by `group_key` field (`Program | instruction | bug-class`). Exact-match first, then merge synonymous `bug_class` tags sharing the same program and instruction. Keep the best version per group, number sequentially, annotate `[agents: N]`.

   Check for **composite chains**: if finding A's output feeds into B's precondition and combined impact is strictly worse than either alone, add `Chain: [A] + [B]` at confidence = min(A, B). Most audits have 0-2.

2. **Gate evaluation.** Run each deduplicated finding through the four gates in `judging.md` (do not skip or reorder). Evaluate each finding exactly once.

   **Single-pass protocol:** evaluate relevant code paths ONCE in fixed order:
   `initialize/bootstrap -> authority/admin setters -> deposit/stake/mint -> withdraw/claim/burn/close -> CPI/remaining_accounts callbacks -> migration/realloc/maintenance`.
   One-line verdict per path: `BLOCKS`, `ALLOWS`, `IRRELEVANT`, or `UNCERTAIN`. Commit after all paths; do not re-examine. `UNCERTAIN = ALLOWS`.

3. **Lead promotion and rejection guardrails.**
   - Promote LEAD -> FINDING (confidence 75) if: complete exploit chain is traced in source, OR `[agents: 2+]` demoted (not rejected) the same issue.
   - `[agents: 2+]` does NOT override a concrete refutation; demote to LEAD if refutation is uncertain.
   - No deployer-intent reasoning; evaluate what code allows, not what maintainers intend.

4. **Fix verification** (confidence >= 80 only): trace the attack with fix applied; verify no new account-locking, signer bypass, PDA mismatch, or CPI breakage is introduced; list all repeated locations when pattern recurs. If no safe fix exists, omit with a short note.

5. **Format and print** per `report-formatting.md`. Exclude rejected items. If `--file-output`, also write to file.

## Banner

Before doing anything else, print this exactly:

```
 ____   ___  _        _    _   _    _    _   _    _      _   _   _  ___ _____ ___  ____
/ ___| / _ \| |      / \  | \ | |  / \  | | | |  / \    | | | | | ||_ _|_   _/ _ \|  _ \
\___ \| | | | |     / _ \ |  \| | / _ \ | | | | / _ \   | | | | | | | |  | || | | | |_) |
 ___) | |_| | |___ / ___ \| |\  |/ ___ \| |_| |/ ___ \  | |_| |_| | | |  | || |_| |  _ <
|____/ \___/|_____/_/   \_\_| \_/_/   \_\___//_/   \_\  \___\___/ |___| |_| \___/|_| \_\
```
