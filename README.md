# Solana Auditor

A security agent with a simple mission - findings in minutes, not weeks.

Built for:

- **Solana/Anchor developers** who want a security check before every commit
- **Security researchers** looking for fast, high-signal review passes
- **Protocol teams** that want repeatable guardrail checks in daily development

Not a substitute for a formal audit - but the check you should not skip.

## Usage

```
Install https://github.com/stoyanov-kaloyan/solana-auditor and run solana auditor on the codebase
```

```
run solana auditor on *specified files*
```

```
update skill to latest version
```

## Install with npx skills

From the repository root:

```
npx skills add . --skill solana-auditor
```

Optional (install to all supported agents):

```
npx skills add . --skill solana-auditor --all -y
```

## Coverage focus

- Account validation and authority bugs (`owner`, `signer`, type/discriminator checks)
- PDA safety (canonical bumps, seed collisions, PDA sharing)
- CPI safety (arbitrary CPI target, stale account reload after CPI)
- State lifecycle hazards (realloc, close/reinit, duplicate mutable accounts)
- Arithmetic and precision issues (overflow/underflow, rounding, saturation)

## Source basis

The vulnerability taxonomy and examples are based on Helius' article:
`A Hitchhiker's Guide to Solana Program Security`.
