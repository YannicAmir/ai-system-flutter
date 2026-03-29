---
name: qa enforcement pattern
description: Shared retry loop logic and output format for all QA Enforcer agents. Defines attempt tracking, stop conditions, violation report format, and delegation rules. Referenced by all domain Enforcer instructions files.
---

# QA Enforcement Pattern

## Purpose
Every Enforcer agent checks changed files against a domain guidance file, reports violations, and drives a correction loop via the domain Reporter and Corrector. This file defines the shared rules for that loop.

---

## Attempt Tracking

The attempt number is passed to the Enforcer by its caller:
- **Attempt = 1**: first check, called by a Builder, Updater, Corrector, or RunDepOps after implementation is complete
- **Attempt = N (N > 1)**: a re-check after a Reporter→Corrector correction cycle

The Enforcer does not track attempt state internally — it reads the number from context.

---

## Enforcement Steps

1. **Read the changed files** passed by the caller
2. **Read the domain guidance checklist** (referenced in the Enforcer's own instructions file)
3. **Compare line by line** — for each checklist item, determine whether the changed code satisfies it
4. **Collect all violations** found across all changed files

---

## Output Format — Violations Found

Report violations as a numbered list. Each item must include:

```
1. File: <path>
   Rule violated: <exact checklist item from the guidance>
   Location: <line reference or method/class name>
   Detail: <one sentence explaining what was found vs what is required>
```

---

## Pass Outcome

If no violations are found:
- Report to the user: "QA check passed. All [domain] changes comply with the [domain] guidance."
- Include: number of files checked and number of items verified
- Do not call the Reporter

---

## Fail Outcome — Retry Logic

### Attempt <= 3
Delegate to the domain Reporter immediately. Pass:
- The full violations list (structured as above)
- The list of changed files
- The current attempt number

Do not wait for user confirmation before delegating.

### Attempt > 3 (loop limit reached)
Stop the loop. Do NOT delegate to the Reporter. Report to the user:
- "QA enforcement loop limit reached (3 correction attempts). The following violations remain unresolved:"
- The full violations list
- A note that manual review is required

---

## Checklist
- [ ] Changed files read before checking
- [ ] Every item in the domain guidance checklist verified
- [ ] Violations reported with file, rule, location, and detail
- [ ] On pass: success reported to user, Reporter NOT called
- [ ] On fail + attempt <= 3: Reporter delegated to immediately with violations + files + attempt
- [ ] On fail + attempt > 3: loop stopped, user given full violations summary
