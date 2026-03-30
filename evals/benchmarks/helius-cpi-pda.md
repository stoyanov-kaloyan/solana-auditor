---
repo_url: https://github.com/coral-xyz/sealevel-attacks
repo_ref: main
contracts_dir: programs
---

# Ground Truth - Helius CPI and PDA Vectors

Source basis: Helius examples for arbitrary CPI, canonical bump misuse, insecure initialization, and PDA domain issues.

## Findings

FINDING | id: H-1 | severity: High | program: RewardsDistributor | instruction: distribute_and_record | bug_class: arbitrary-cpi
description: CPI target program account is user-controlled and unvalidated, enabling execution of attacker-controlled callee program.

FINDING | id: H-2 | severity: High | program: Bootstrap | instruction: initialize | bug_class: insecure-initialization
description: Initialization can be called by arbitrary signer before intended authority, allowing attacker takeover of initial state.

FINDING | id: M-1 | severity: Medium | program: Profiles | instruction: create_profile | bug_class: non-canonical-bump
description: User-provided bump is accepted for PDA derivation, enabling duplicate semantic profiles under different bumps.

FINDING | id: M-2 | severity: Medium | program: Staking | instruction: withdraw_rewards | bug_class: pda-sharing
description: Same PDA authority namespace is reused across staking and rewards domains, weakening isolation boundaries.

FINDING | id: M-3 | severity: Medium | program: RewardsUpdater | instruction: update_rewards | bug_class: stale-account-after-cpi
description: Account state is read after mutable CPI without reload, using stale values for post-CPI logic.
