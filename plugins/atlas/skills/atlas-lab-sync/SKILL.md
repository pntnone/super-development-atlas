---
name: atlas-lab-sync
description: Sync contracts from parent to feature lab when contracts change
argument-hint: "<lab-name>"
level: 3
---

<Purpose>
Synchronize contracts between a parent (main pipeline or parent lab) and a feature lab. When contracts change in the parent after a lab was created, this skill detects the differences, updates the lab's contract copies, flags breaking changes, and helps the developer update their implementation to match.
</Purpose>

<Use_When>
- User says "/atlas lab sync {name}", "sync contracts", "update lab contracts"
- Main pipeline contracts have been updated and labs need to catch up
- Before merging, if stale contracts are detected
- A new cross-cutting contract was added that affects existing labs
</Use_When>

<Do_Not_Use_When>
- Lab doesn't exist
- No contracts have changed (inform user everything is up to date)
</Do_Not_Use_When>

<Pre_Flight>
1. Verify the lab exists
2. Read the lab's `.atlas-lab/origin.json` to locate the parent
3. Read the lab's `.atlas-lab/manifest.json` to get current contract list
4. Verify the parent exists and is accessible
</Pre_Flight>

<Steps>

## Step 1: Detect Changes

For each contract the lab must satisfy:
1. Read the lab's copy from `{lab}/contracts/{contract}.md`
2. Read the parent's current version from `.atlas/contracts/{contract}.md`
3. Compare them — generate a diff

Also check for:
- **New contracts** in the parent that apply to this lab (cross-cutting contracts marked "All features")
- **Removed contracts** in the parent that the lab still references

## Step 2: Classify Changes

Categorize each change:

- **Non-breaking** — comments updated, examples added, minor clarification
  → Auto-update, no implementation change needed

- **Additive** — new optional method/property added to interface
  → Update contract, recommend implementing the new addition

- **Breaking** — interface signature changed, behavior modified, constraint added
  → Update contract, implementation MUST be updated

- **New contract** — a new cross-cutting contract was added
  → Copy new contract, implementation MUST add compliance

- **Removed contract** — a contract no longer exists
  → Remove contract copy, related implementation can be simplified

## Step 3: Present Changes

```
Contract Sync Report for "{lab-name}":

📄 editing-history.contract.md
   Status: BREAKING CHANGE
   Changes:
   - Added: `getHistorySize(): number` to interface
   - Changed: `undo()` now returns `Promise<void>` (was `void`)
   Impact: Must update implementation to handle async undo

📄 ui-panel.contract.md
   Status: Non-breaking
   Changes:
   - Updated example code
   Impact: None — auto-updating

📄 accessibility.contract.md
   Status: NEW CONTRACT (cross-cutting)
   Changes:
   - All features must implement aria labels and keyboard navigation
   Impact: Must add accessibility support to UI components

Proceed with sync?
```

## Step 4: Execute Sync

On user approval:
1. **Copy updated contracts** from parent to lab's `contracts/` directory
2. **Copy new contracts** if any
3. **Remove deleted contracts** if any
4. **Update manifest.json** with new contract list

## Step 5: Update Lab CLAUDE.md

If new contracts were added or contracts changed significantly:
1. Update the lab's CLAUDE.md to reference new/changed contracts
2. Add notes about what changed so Claude in the lab context knows

## Step 6: Implementation Guidance

For breaking changes and new contracts:
```
Implementation needed:

1. editing-history: Update undo() to be async
   Affected files: src/business/services/history-handler.ts

2. accessibility: Add aria labels and keyboard nav
   Affected files: src/ui/components/*.tsx

Want me to help fix these now?
```

If the user agrees, assist with updating the implementation to match the new contracts.

</Steps>

<Tool_Usage>
- Use `Read` to compare contract versions
- Use `Bash` for diff operations if needed
- Use `Write` to update contract copies and manifest
- Use `Edit` to update CLAUDE.md with new contract references
- Use `AskUserQuestion` to confirm sync and ask about implementation help
</Tool_Usage>

<Validation>
After sync completes, verify:
- [ ] All contract copies in lab match parent's current versions
- [ ] New cross-cutting contracts are included
- [ ] Removed contracts are cleaned up
- [ ] Lab manifest reflects current contract list
- [ ] Lab CLAUDE.md is updated with any new contract references
- [ ] Breaking changes are clearly communicated to the developer
</Validation>
