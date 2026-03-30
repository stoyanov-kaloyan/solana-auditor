# Finding Validation

Every finding passes four sequential gates. Fail any gate -> **rejected** or **demoted** to lead. Later gates are not evaluated for failed findings.

## Gate 1 - Refutation

Construct the strongest argument that the finding is wrong. Find the concrete guard, constraint, or invariant that blocks the exact attack step.

- Concrete refutation (specific guard blocks claimed step) -> **REJECTED** (or **DEMOTE** if code smell remains)
- Speculative refutation ("probably safe") -> **clears**, continue

## Gate 2 - Reachability

Prove the vulnerable state exists in realistic deployment conditions.

- Structurally impossible due to enforced invariant/constraints -> **REJECTED**
- Requires privileged out-of-band setup outside normal operation -> **DEMOTE**
- Reachable via normal instruction flow or common account states -> **clears**, continue

## Gate 3 - Trigger

Prove an unprivileged actor can execute the attack path.

- Only trusted/admin signer can trigger -> **DEMOTE**
- Trigger cost or constraints exceed plausible impact -> **REJECTED**
- Unprivileged actor can trigger with plausible preconditions -> **clears**, continue

## Gate 4 - Impact

Prove material harm to an identifiable victim (users, treasury, protocol accounting, availability).

- Self-harm only -> **REJECTED**
- Dust-level/non-compounding impact -> **DEMOTE**
- Material value loss, privilege bypass, or protocol-locking impact -> **CONFIRMED**

## Confidence

Start at **100**, deduct:

- partial exploit trace: **-20**
- bounded non-compounding impact: **-15**
- requires specific but achievable state: **-10**

Confidence >= 80 gets description + fix. Below 80 gets description only.

## Safe patterns (do not flag by default)

- Anchor `Account<'info, T>` enforcing ownership + discriminator checks where appropriate
- Anchor `Signer<'info>` or explicit signer constraints in privileged paths
- Canonical PDA derivation via Anchor `seeds` + `bump` or deterministic `find_program_address`
- Explicit CPI target validation (`Program<'info, X>` or fixed program ID checks)
- `#[account(close = destination)]` for account closure
- Explicit `checked_*` arithmetic and safe conversion patterns (`try_from`) with handled errors
- Explicit `reload()` after mutable CPI before reading account state

## Lead promotion

Before finalizing leads, promote where warranted:

- **Cross-program echo:** same root cause confirmed as FINDING in one program and repeated elsewhere -> promote repeats.
- **Multi-agent convergence:** 2+ agents flagged same area and issue was demoted (not rejected) -> promote to FINDING at confidence 75.
- **Partial-path completion:** path is reachable and unguarded but trace is incomplete -> promote to FINDING at confidence 75, description only.

## Leads

High-signal trails for manual investigation. No confidence score and no fix.

## Do Not Report

Lints/style/naming/docs, pure compute micro-optimizations, non-exploitable centralization statements ("admin can change params" without exploit path), missing events, and hypothetical attacks with implausible preconditions unsupported by code.
