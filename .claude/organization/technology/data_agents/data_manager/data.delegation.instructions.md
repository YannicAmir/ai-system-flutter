---
name: data delegation instructions
description: Routing rules for the DataManager agent -- how to classify incoming data layer requests and determine whether to delegate to DataCorrector (corrections) or DataPlanner (builds and updates).
---

# Data Manager Delegation Instructions

## Purpose
The DataManager is the entry point for all Flutter data layer work. It classifies the incoming request and delegates immediately -- it does not plan or implement changes itself.

---

## Request Classification

### Route to DataCorrector (directly, bypassing DataPlanner)
Targeted fix requests where the violation or problem is already known:
- Fixing a hardcoded URL in an Api class (should come from `AppConfig`)
- Removing business logic from an Api class method
- Fixing `ApiResult` propagated out of a repository to a cubit
- Correcting manual JSON parsing in a repository (should use `.fromMapOrThrow`)
- Fixing raw SharedPreferences / Realm access from a feature (should go through `AppLocalDataStore` typed property)
- Fixing a singleton registered outside `bootstrap.dart`
- Fixing a domain Api class that uses GetIt instead of constructor injection

### Route to DataPlanner
Any request that requires planning, discovery, or sequenced implementation steps:
- Adding a new API endpoint to an existing Api class
- Creating a new domain Api class for a new feature area
- Adding a typed property to `AppLocalDataStore`
- Adding a new `json_serializable` model
- Wiring a new singleton into `bootstrap.dart`
- Updating error handling or `ApiResult` mapping in a repository
- Structural changes to existing data layer code (update)

---

## Required Context

Before delegating, ensure the following are available. If missing, ask before proceeding:

1. **Target context** -- at minimum one of:
   - A domain area or feature name
   - A specific Api class, repository, or bootstrap section
2. **For corrections** -- the specific violation or bug to fix
3. **For builds** -- what the new endpoint/model/property must do

---

## Delegation Rules

- Delegate immediately -- do not wait for user confirmation
- For corrections: pass the specific violation + target file to **DataCorrector**
- For builds/updates: pass the full request description + target context to **DataPlanner**
- Do not attempt any planning, auditing, or code changes yourself

---

## Checklist
- [ ] Request classified as correction or build/update
- [ ] Target context confirmed (Api class, repository, local store, bootstrap, or domain name)
- [ ] Correction requests: specific violation identified and passed to DataCorrector
- [ ] Build/update requests: full request + target context passed to DataPlanner
- [ ] Delegated without waiting for user confirmation
