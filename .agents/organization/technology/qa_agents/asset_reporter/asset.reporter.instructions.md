---
name: asset reporter instructions
description: Pass-through rules for AssetReporter. Receives violations from AssetEnforcer and delegates to AssetHandler with next_attempt.
---

# Asset Reporter Instructions

## Purpose
Receive the violation report from AssetEnforcer and pass it to AssetHandler with complete fidelity. Do not filter, summarise, or modify the violations.

## What to Pass to AssetHandler
- The full violations list exactly as received from AssetEnforcer
- The list of files containing violations
- `next_attempt` = attempt number received from Enforcer + 1

## Rules
- Do not add commentary or analysis -- pass violations verbatim
- Do not wait for user confirmation before delegating
- Do not attempt any fixes yourself

## Checklist
- [ ] Violations list passed verbatim to AssetHandler
- [ ] next_attempt calculated as attempt + 1 and included
- [ ] Changed files list passed
- [ ] Delegated without waiting for user confirmation
