---
repo_url: https://github.com/coral-xyz/sealevel-attacks
repo_ref: main
contracts_dir: programs
---

# Ground Truth - Helius Account Safety Vectors

Source basis: Helius `A Hitchhiker's Guide to Solana Program Security` examples for account validation issues.

## Findings

FINDING | id: H-1 | severity: High | program: AccountValidation | instruction: update_admin | bug_class: missing-signer-check
description: Instruction compares admin pubkey but does not require admin signer authorization, enabling unauthorized admin change.

FINDING | id: H-2 | severity: High | program: VaultConfig | instruction: admin_withdraw | bug_class: missing-ownership-check
description: Config account owner is not validated, allowing attacker-supplied forged config state to bypass admin checks.

FINDING | id: M-1 | severity: Medium | program: Rewards | instruction: distribute_rewards | bug_class: duplicate-mutable-accounts
description: Same mutable account can be supplied as both reward and bonus account, causing unintended double mutation.

FINDING | id: M-2 | severity: Medium | program: Rewards | instruction: calculate_rewards | bug_class: unvalidated-remaining-accounts
description: Accounts in remaining_accounts are processed without owner/discriminator validation, enabling forged account injection.

FINDING | id: M-3 | severity: Medium | program: AdminSettings | instruction: update_admin_settings | bug_class: type-cosplay
description: Account data is deserialized without discriminator/type checks, allowing role confusion.
