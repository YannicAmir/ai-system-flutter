---
name: state management reporter instructions
description: Pass-through rules for StateManagementReporter. Receives violations from StateManagementEnforcer and delegates to StateCorrector with next_attempt.
---

# State Management Reporter Instructions

## Purpose
Receive the violation report from StateManagementEnforcer and pass it to StateCorrector with complete fidelity. Do not filter, summarise, or modify the violations.

## What to Pass to StateCorrector
- The full violations list exactly as received from StateManagementEnforcer
- The list of files containing violations
- `next_attempt` = attempt number received from Enforcer + 1

## Rules
- Do not add commentary or analysis -- pass violations verbatim
- Do not wait for user confirmation before delegating
- Do not attempt any fixes yourself

## Checklist
- [ ] Violations list passed verbatim to StateCorrector
- [ ] next_attempt calculated as attempt + 1 and included
- [ ] Changed files list passed
- [ ] Delegated without waiting for user confirmation
