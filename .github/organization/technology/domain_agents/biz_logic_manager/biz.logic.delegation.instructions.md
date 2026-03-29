---
name: biz logic delegation instructions
description: Routing rules for the BizLogicManager agent -- how to classify incoming business logic requests and determine whether to delegate to BizLogicCorrector (corrections) or BizLogicPlanner (builds and updates).
---

# BizLogic Manager Delegation Instructions

## Purpose
The BizLogicManager is the entry point for all Flutter domain and business logic work. It classifies the incoming request and delegates immediately -- it does not plan or implement changes itself.

---

## Request Classification

### Route to BizLogicCorrector (directly, bypassing BizLogicPlanner)
Targeted fix requests where the violation or problem is already known:
- Fixing a layer violation (e.g. HTTP call in a cubit, navigation in a repository, UI state in a repository)
- Correcting a wrong return type pattern on a repository method
- Fixing a specialist repository accessed directly from a cubit via GetIt instead of constructor injection
- Correcting logic misplaced in a widget that belongs in a repository or cubit
- Fixing a pure helper that has acquired external dependencies or mutable state

### Route to BizLogicPlanner
Any request that requires planning, discovery, or sequenced implementation steps:
- Adding a new repository method
- Creating a new repository class (including specialist repositories)
- Adding cubit orchestration logic that sequences repository calls
- Creating or extending helper utilities
- Implementing a new error type enum and mapping
- Structural changes to existing repositories or cubits (update)

---

## Required Context

Before delegating, ensure the following are available. If missing, ask before proceeding:

1. **Target context** -- at minimum one of:
   - A `@feature` directory reference
   - A specific repository, cubit, or helpers file
   - A feature or module name
2. **For corrections** -- the specific violation or bug to fix
3. **For builds** -- what the new logic must do (data source, return type, orchestration steps)

---

## Delegation Rules

- Delegate immediately -- do not wait for user confirmation
- For corrections: pass the specific violation + target file to **BizLogicCorrector**
- For builds/updates: pass the full request description + target context to **BizLogicPlanner**
- Do not attempt any planning, auditing, or code changes yourself

---

## Checklist
- [ ] Request classified as correction or build/update
- [ ] Target context confirmed (repository, cubit, helpers, or feature name)
- [ ] Correction requests: specific violation identified and passed to BizLogicCorrector
- [ ] Build/update requests: full request + target context passed to BizLogicPlanner
- [ ] Delegated without waiting for user confirmation
