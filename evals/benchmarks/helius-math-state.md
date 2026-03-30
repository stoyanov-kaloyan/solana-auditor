---
repo_url: https://github.com/coral-xyz/sealevel-attacks
repo_ref: main
contracts_dir: programs
---

# Ground Truth - Helius Math and State Lifecycle Vectors

Source basis: Helius precision, overflow, and account lifecycle guidance.

## Findings

FINDING | id: H-1 | severity: High | program: Liquidity | instruction: collateral_to_liquidity | bug_class: rounding-direction-error
description: Rounding mode can over-credit liquidity tokens versus collateral value, enabling extractable arbitrage.

FINDING | id: H-2 | severity: High | program: TokenAccounting | instruction: process_instruction | bug_class: integer-underflow
description: Balance subtraction is unchecked in release-mode path and can underflow to attacker-favorable value.

FINDING | id: M-1 | severity: Medium | program: RewardsMath | instruction: calculate_reward | bug_class: saturating-arithmetic-misuse
description: Saturating multiplication caps rewards silently and breaks expected accounting.

FINDING | id: M-2 | severity: Medium | program: AccountLifecycle | instruction: close_account | bug_class: unsafe-account-close
description: Manual close drains lamports without full close semantics, enabling limbo/reuse edge cases.

FINDING | id: M-3 | severity: Medium | program: CastingUtils | instruction: convert_amount | bug_class: unsafe-casting
description: Narrow integer cast truncates high bits and can bypass amount limits.
