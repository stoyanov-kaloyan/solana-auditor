# Solana Attack Vectors (Helius-derived)

This catalog is derived from Helius' article:
**A Hitchhiker's Guide to Solana Program Security**
(`https://www.helius.dev/blog/a-hitchhikers-guide-to-solana-program-security`).

Use these vectors as a checklist during audits. IDs are stable and intentionally compact for vector-scan classification.

## Vector Index

| ID  | Vector |
| --- | ------ |
| V01 | Account Data Matching |
| V02 | Account Data Reallocation |
| V03 | Account Reloading |
| V04 | Arbitrary CPI |
| V05 | Authority Transfer Functionality |
| V06 | Bump Seed Canonicalization |
| V07 | Closing Accounts |
| V08 | Duplicate Mutable Accounts |
| V09 | Frontrunning / Expected vs Actual Mismatch |
| V10 | Insecure Initialization |
| V11 | Loss of Precision (Multiply After Divide) |
| V12 | Saturating Arithmetic Misuse |
| V13 | Rounding Direction Errors |
| V14 | Missing Ownership Check (including read-only/sysvars) |
| V15 | Missing Signer Check |
| V16 | Overflow and Underflow |
| V17 | Unsafe Integer Casting |
| V18 | PDA Sharing Across Roles |
| V19 | Unvalidated `remaining_accounts` |
| V20 | Seed Collisions |
| V21 | Type Cosplay / Missing Discriminator Checks |
| V22 | Rust Unsafe/Panic Footguns |

## V01 - Account Data Matching

**Root cause:** instruction logic trusts account data without verifying expected relationships (for example, stored admin equals signer key).

**Exploit sketch:** attacker passes a validly shaped account but with attacker-controlled fields and bypasses authorization checks.

**Mitigation checklist:**
- Enforce key equality checks for authority relationships.
- Prefer Anchor constraints: `has_one`, `constraint = ...`.
- Validate token-account owner/mint relationships when business logic depends on them.

**Helius reference:** Account Data Matching.

## V02 - Account Data Reallocation

**Root cause:** incorrect use of `realloc(new_len, zero_init)` leaks stale bytes or wastes compute.

**Exploit sketch:** shrink and regrow account data in one transaction with `zero_init = false`, then read stale bytes to influence logic.

**Mitigation checklist:**
- If account was shrunk earlier in same instruction flow, regrow with `zero_init = true`.
- Avoid unnecessary repeated realloc calls in one path.
- Consider design alternatives that avoid frequent resizing.

**Helius reference:** Account Data Reallocation.

## V03 - Account Reloading

**Root cause:** stale deserialized Anchor account used after CPI mutates that account.

**Exploit sketch:** CPI updates stake/reward/accounting, but caller continues using stale in-memory value for authorization or payout.

**Mitigation checklist:**
- Call `account.reload()?` after CPI when subsequent logic depends on updated state.
- Avoid decisions using values read before mutable CPI calls.

**Helius reference:** Account Reloading.

## V04 - Arbitrary CPI

**Root cause:** CPI target program account is user-controlled and not validated.

**Exploit sketch:** attacker supplies malicious program ID that accepts expected interface but executes arbitrary behavior.

**Mitigation checklist:**
- Enforce exact CPI target IDs (`Program<'info, X>` or explicit key check).
- Validate expected executable account and owner constraints.

**Helius reference:** Arbitrary CPI.

## V05 - Authority Transfer Functionality

**Root cause:** admin/authority cannot be rotated safely, or rotation is one-step and brittle.

**Exploit sketch:** compromised authority remains permanent; accidental transfer to wrong key is irreversible.

**Mitigation checklist:**
- Use two-step transfer (`nominate -> accept`).
- Gate both steps with signer checks and explicit pending-authority validation.

**Helius reference:** Authority Transfer Functionality.

## V06 - Bump Seed Canonicalization

**Root cause:** user-provided or non-canonical bump accepted for PDA derivation.

**Exploit sketch:** multiple valid PDAs for same semantic entity enable duplicate state or bypass assumptions.

**Mitigation checklist:**
- Use canonical bump (`find_program_address` or Anchor `seeds` + `bump`).
- Store and validate bump in account state if reused later.

**Helius reference:** Bump Seed Canonicalization.

## V07 - Closing Accounts

**Root cause:** account lamports drained without full close semantics and reuse protection.

**Exploit sketch:** account sits in limbo, can be revived/funded, and reused unexpectedly.

**Mitigation checklist:**
- Prefer Anchor `#[account(close = destination)]`.
- If manual close is required, zero data and mark closed discriminator safely.
- Ensure closed accounts cannot be reused in sensitive paths.

**Helius reference:** Closing Accounts.

## V08 - Duplicate Mutable Accounts

**Root cause:** same account can be passed into two mutable parameters expected to be distinct.

**Exploit sketch:** reward and bonus accounts alias to same key, doubling credit or corrupting state.

**Mitigation checklist:**
- Enforce inequality checks (`a.key() != b.key()`).
- Add Anchor constraints for distinct mutable accounts.

**Helius reference:** Duplicate Mutable Accounts.

## V09 - Frontrunning / Expected vs Actual Mismatch

**Root cause:** user-side transaction assumes stale quote/price/value without explicit expected-value checks.

**Exploit sketch:** seller/admin updates sale price before purchase instruction executes; buyer overpays.

**Mitigation checklist:**
- Include `expected_*` parameters and assert bounds on-chain.
- Use deadlines or slot/price validity checks where applicable.

**Helius reference:** Frontrunning.

## V10 - Insecure Initialization

**Root cause:** initialize instruction can be called by arbitrary signer before intended authority.

**Exploit sketch:** attacker front-runs first initialize and permanently sets authority/account graph.

**Mitigation checklist:**
- Restrict init caller to known authority (for upgradeable programs: upgrade authority pattern).
- Enforce one-time init invariants.

**Helius reference:** Insecure Initialization.

## V11 - Loss of Precision (Multiply After Divide)

**Root cause:** integer division before multiplication truncates intermediate value.

**Exploit sketch:** attacker chooses values where truncation systematically favors extraction.

**Mitigation checklist:**
- Prefer multiply-before-divide where safe from overflow.
- Use precise fixed-point helpers and bounded arithmetic.

**Helius reference:** Loss of Precision -> Multiplication After Division.

## V12 - Saturating Arithmetic Misuse

**Root cause:** `saturating_*` silently caps values; business logic assumes exact arithmetic.

**Exploit sketch:** large operands saturate to max/min and underpay/overpay users or bypass accounting.

**Mitigation checklist:**
- Use `checked_*` for critical accounting paths.
- Treat saturation events as explicit error cases when exactness is required.

**Helius reference:** Loss of Precision -> saturating_* Arithmetic Functions.

## V13 - Rounding Direction Errors

**Root cause:** rounding mode benefits user in value-moving formulas.

**Exploit sketch:** repeated favorable rounding extracts value via arbitrage loops.

**Mitigation checklist:**
- Align rounding direction with protocol safety policy.
- Use floor/ceil intentionally and consistently.
- Add tests for boundary and repeated-action accumulation.

**Helius reference:** Loss of Precision -> Rounding Errors.

## V14 - Missing Ownership Check (including read-only/sysvars)

**Root cause:** account owner is not validated against expected program.

**Exploit sketch:** attacker supplies forged config/state account with attacker-controlled fields.

**Mitigation checklist:**
- Validate account owner explicitly or via Anchor `Account<'info, T>`.
- Validate read-only accounts too (address + owner), especially sysvars not fetched via `get()`.

**Helius reference:** Missing Ownership Check + Read-Only Accounts.

## V15 - Missing Signer Check

**Root cause:** privileged account key is compared but signature requirement is not enforced.

**Exploit sketch:** attacker passes correct pubkey account metadata without owning signing key.

**Mitigation checklist:**
- Require `Signer<'info>` for privileged actors or check `is_signer`.
- Pair signer checks with ownership/type checks.

**Helius reference:** Missing Signer Check.

## V16 - Overflow and Underflow

**Root cause:** unchecked arithmetic in release-mode build paths.

**Exploit sketch:** user-provided amount underflows/overflows and wraps balances/counters.

**Mitigation checklist:**
- Use `checked_*` arithmetic for critical state updates.
- Consider `overflow-checks = true` when compute budget permits.

**Helius reference:** Overflow and Underflow.

## V17 - Unsafe Integer Casting

**Root cause:** lossy `as` casts truncate/extend unexpectedly.

**Exploit sketch:** large value cast to smaller integer wraps/truncates and bypasses limits.

**Mitigation checklist:**
- Use `try_from` for fallible conversions.
- Use `from` for guaranteed lossless widening conversions.

**Helius reference:** Overflow and Underflow -> Casting.

## V18 - PDA Sharing Across Roles

**Root cause:** same PDA namespace/signing authority reused across distinct roles/domains.

**Exploit sketch:** actor authorized for one flow reuses PDA authority in another flow.

**Mitigation checklist:**
- Derive role-specific PDAs with unique seed prefixes.
- Scope PDA authority to smallest domain possible.

**Helius reference:** PDA Sharing.

## V19 - Unvalidated `remaining_accounts`

**Root cause:** dynamic accounts are consumed without manual validation.

**Exploit sketch:** attacker injects forged user/state accounts into loop-based processing.

**Mitigation checklist:**
- Validate owner, address derivation, discriminator, and semantic relation for each remaining account.
- Reject unknown/unexpected extras in sensitive flows.

**Helius reference:** Remaining Accounts.

## V20 - Seed Collisions

**Root cause:** seed scheme allows semantically distinct objects to collide or overlap.

**Exploit sketch:** crafted identifiers collide with existing namespace and break availability/authorization.

**Mitigation checklist:**
- Use domain-separated seed prefixes.
- Include stable unique identifiers (for example user key + nonce).
- Validate generated PDA uniqueness assumptions in logic.

**Helius reference:** Seed Collisions.

## V21 - Type Cosplay / Missing Discriminator Checks

**Root cause:** account is deserialized as type T without enforcing discriminator/type identity.

**Exploit sketch:** attacker passes different account layout that partially deserializes into expected fields.

**Mitigation checklist:**
- Prefer Anchor `Account<'info, T>` wrappers for discriminator checks.
- For manual deserialization, verify discriminator/tag before use.

**Helius reference:** Type Cosplay.

## V22 - Rust Unsafe/Panic Footguns

**Root cause:** unsafe memory operations or panic-prone operations (`unwrap`, OOB indexing, divide by zero) in critical paths.

**Exploit sketch:** adversarial inputs trigger panic/abort behavior, causing DoS or brittle behavior.

**Mitigation checklist:**
- Minimize and isolate `unsafe` blocks.
- Replace panic-prone code with explicit error handling (`Result`, checked access, checked math).
- Add tests for adversarial boundaries.

**Helius reference:** Rust-Specific Errors.
