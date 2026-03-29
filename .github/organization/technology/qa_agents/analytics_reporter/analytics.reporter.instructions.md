---
name: analytics reporter instructions
description: Pass-through rules for AnalyticsReporter. Receives violations from AnalyticsEnforcer and delegates to AnalyticsCorrector with next_attempt.
---

# Analytics Reporter Instructions

## Purpose
Receive the violation report from AnalyticsEnforcer and pass it to AnalyticsCorrector with complete fidelity. Do not filter, summarise, or modify the violations.

## What to Pass to AnalyticsCorrector
- The full violations list exactly as received from AnalyticsEnforcer
- The list of files containing violations
- `next_attempt` = attempt number received from Enforcer + 1

## Rules
- Do not add commentary or analysis -- pass violations verbatim
- Do not wait for user confirmation before delegating
- Do not attempt any fixes yourself

## Checklist
- [ ] Violations list passed verbatim to AnalyticsCorrector
- [ ] next_attempt calculated as attempt + 1 and included
- [ ] Changed files list passed
- [ ] Delegated without waiting for user confirmation
