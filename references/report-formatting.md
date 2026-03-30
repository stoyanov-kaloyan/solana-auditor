# Report Formatting

## Report Path

Save the report to `assets/findings/{project-name}-solana-ai-audit-report-{timestamp}.md` where `{project-name}` is the repo root basename and `{timestamp}` is `YYYYMMDD-HHMMSS` at scan time.

## Output Format

````
# Security Review - <ProgramName or repo name>

---

## Scope

|                                  |                                                                 |
| -------------------------------- | --------------------------------------------------------------- |
| **Mode**                         | ALL / default / filename                                        |
| **Files reviewed**               | `file1.rs` · `file2.rs`<br>`file3.rs` · `file4.rs`             |
| **Programs reviewed**            | `program_a` · `program_b`                                       |
| **Confidence threshold (1-100)** | N                                                               |

---

## Findings

[95] **1. <Title>**

`Program.instruction` - Confidence: 95 - `group_key: Program | instruction | bug-class`

**Description**
<One short sentence describing the exploitable bug pattern>

**Exploit Path**
<One short line with attacker input -> vulnerable step -> impact>

**Fix**

```diff
- vulnerable line(s)
+ fixed line(s)
```
---

[82] **2. <Title>**

`Program.instruction` - Confidence: 82 - `group_key: Program | instruction | bug-class`

**Description**
<One short sentence describing the exploitable bug pattern>

**Exploit Path**
<One short line with attacker input -> vulnerable step -> impact>

**Fix**

```diff
- vulnerable line(s)
+ fixed line(s)
```
---

< ... all above-threshold findings >

---

[75] **3. <Title>**

`Program.instruction` - Confidence: 75 - `group_key: Program | instruction | bug-class`

**Description**
<One short sentence describing the exploitable bug pattern>

**Exploit Path**
<One short line with attacker input -> vulnerable step -> impact>

---

< ... all below-threshold findings (description + exploit path only, no Fix block) >

---

Findings List

| # | Confidence | Program.Instruction | Title |
|---|---|---|---|
| 1 | [95] | Program.instruction | <title> |
| 2 | [82] | Program.instruction | <title> |
| 3 | [75] | Program.instruction | <title> |

---

## Leads

_Security trails with concrete code smells where the full exploit path could not be completed in one pass. Not scored._

- **<Title>** - `Program.instruction` - Code smells: <missing signer, unchecked owner, stale reload, etc.> - <1-2 sentence summary of what remains unverified>
- **<Title>** - `Program.instruction` - Code smells: <...> - <1-2 sentence summary>

---

> WARNING: This review was performed by an AI assistant. AI analysis cannot guarantee the absence of vulnerabilities. Manual review, competitive audit, and on-chain monitoring are strongly recommended.

````

**Rules:** Follow the template exactly. Sort findings by confidence (highest first). Findings below threshold get no **Fix** block. Draft final findings directly in report format.
