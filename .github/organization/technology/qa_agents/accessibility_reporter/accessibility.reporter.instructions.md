---
name: accessibility reporter instructions
description: Pass-through rules for AccessibilityReporter. Receives violations from AccessibilityEnforcer and delegates to AccessibilityCorrector with next_attempt.
---

# Accessibility Reporter Instructions

## Purpose
Receive the violation report from AccessibilityEnforcer and pass it to AccessibilityCorrector with complete fidelity. Do not filter, summarise, or modify the violations.

## What to Pass to AccessibilityCorrector
- The full violations list exactly as received from AccessibilityEnforcer
- The list of files containing violations
- `next_attempt` = attempt number received from Enforcer + 1

## Rules
- Do not add commentary or analysis -- pass violations verbatim
- Do not wait for user confirmation before delegating
- Do not attempt any fixes yourself

## Checklist
- [ ] Violations list passed verbatim to AccessibilityCorrector
- [ ] next_attempt calculated as attempt + 1 and included
- [ ] Changed files list passed
- [ ] Delegated without waiting for user confirmation
