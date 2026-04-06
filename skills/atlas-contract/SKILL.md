---
name: atlas-contract
description: Manage contracts — add, list, edit, or remove contracts in the main pipeline
argument-hint: "<add|list|edit|remove> [contract-name]"
level: 3
---

<Purpose>
Manage the contract system in the main pipeline. Contracts are the core mechanism that enables clean merging of feature labs. This skill allows creating new contracts, listing existing ones, editing contracts, and removing obsolete ones. When contracts change, it identifies affected labs that need syncing.
</Purpose>

<Use_When>
- User says "/atlas contract add {name}", "add contract", "new contract"
- User says "/atlas contract list", "show contracts", "what contracts exist"
- User says "/atlas contract edit {name}", "update contract"
- User says "/atlas contract remove {name}", "delete contract"
- During `/atlas init` when proposing initial contracts
- When a new cross-cutting concern is identified
</Use_When>

<Pre_Flight>
1. Verify `.atlas/config.json` exists
2. Read `.atlas/contracts/_index.json` for current contract list
</Pre_Flight>

<Steps>

## Command: `add`

### Step 1: Define Contract
Ask the user OR Claude proposes (hybrid approach):

If user-initiated:
1. Ask: "What does this contract define?" (purpose)
2. Ask: "Which features must implement it?" (all features, or specific ones)
3. Ask: "What's the interface?" (methods, properties, events)

If Claude-proposed (during init or analysis):
1. Claude describes the contract based on codebase analysis
2. User reviews and approves/modifies

### Step 2: Generate Contract File
Using the contract template, create `.atlas/contracts/{name}.contract.md` with:
- Purpose
- Consumers (all features or specific list)
- Interface definition
- Expected behavior
- Integration points
- Constraints
- Validation checklist

### Step 3: Update Index
Add to `.atlas/contracts/_index.json`:
```json
{
  "contracts": {
    "{name}": {
      "file": "{name}.contract.md",
      "consumers": "all" | ["{feature-1}", "{feature-2}"],
      "createdAt": "{timestamp}",
      "version": 1
    }
  }
}
```

### Step 4: Notify Affected Labs
Check `.atlas/registry.json` for active labs:
- If contract is "all features": ALL active labs need sync
- If contract targets specific features: only matching labs

```
New contract "{name}" affects these active labs:
- retouching-face
- background-blur

Run /atlas lab sync {lab-name} to update each lab.
```

## Command: `list`

Display all contracts:
```
Atlas Contracts for "{project-name}":

1. editing-history (v2) — All features
   Every feature must integrate with undo/redo history

2. filter-pipeline (v1) — All features
   Image processing features register as pipeline filters

3. ui-panel (v1) — All features
   Features expose a UI panel with standard props

4. face-regions (v1) — retouching-face internal
   Face detection output consumed by processing sub-features

Total: 4 contracts (3 global, 1 internal)
```

## Command: `edit`

1. Read the current contract
2. Show it to the user
3. User or Claude proposes changes
4. Update the contract file
5. Increment version in `_index.json`
6. Identify affected labs and recommend sync

## Command: `remove`

1. Verify no active labs depend on this contract
2. If active labs depend on it:
   ```
   Cannot remove "{name}" — these active labs depend on it:
   - retouching-face
   - background-blur

   Merge or archive these labs first, or force remove with --force.
   ```
3. If no dependencies or `--force`: remove the contract file and update `_index.json`

</Steps>

<Tool_Usage>
- Use `AskUserQuestion` for contract definition when user-initiated
- Use `Write` to create contract files
- Use `Read` to read existing contracts and registry
- Use `Edit` to update index and existing contracts
</Tool_Usage>

<Validation>
After any contract operation:
- [ ] Contract file exists/removed in `.atlas/contracts/`
- [ ] `_index.json` is up to date
- [ ] Affected labs are identified and user is notified about sync needs
</Validation>
