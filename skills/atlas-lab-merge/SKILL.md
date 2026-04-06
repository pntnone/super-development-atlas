---
name: atlas-lab-merge
description: Claude-assisted merge of feature lab back into parent with contract validation
argument-hint: "<lab-name>"
level: 3
---

<Purpose>
Merge a completed feature lab back into its parent (main pipeline or parent lab). Claude validates contract compliance, reviews code quality, generates an integration plan, executes the merge, and runs post-merge validation. This is the critical step where isolated work rejoins the main codebase.
</Purpose>

<Use_When>
- User says "/atlas lab merge {name}", "merge lab", "integrate feature"
- A feature lab is complete and ready to rejoin the parent
- All sub-labs have been merged into the parent lab and the parent is ready to merge into main
</Use_When>

<Do_Not_Use_When>
- Lab doesn't exist or has already been merged
- Lab has active sub-labs that haven't been merged yet — merge sub-labs first
- Lab is in "active" development and not ready — inform user
</Do_Not_Use_When>

<Pre_Flight>
1. Read `.atlas/registry.json` to verify the lab exists
2. Read the lab's `.atlas-lab/manifest.json` to get contracts and parent info
3. Check if the lab has sub-labs — if any are still active, block merge and tell user to merge sub-labs first
4. Read the lab's `.atlas-lab/origin.json` to locate the parent
5. Verify the parent project/lab still exists at the expected path
</Pre_Flight>

<Steps>

## Phase 1: Pre-Merge Checks

### Step 1: Sub-Lab Status Check
If the lab has sub-labs in its manifest:
- Check each sub-lab's status
- If ANY sub-lab is still "active": **BLOCK merge**
  ```
  Cannot merge "{lab-name}" — these sub-labs are still active:
  - {sub-lab-1}: active
  - {sub-lab-2}: active

  Merge sub-labs first with: /atlas lab merge {sub-lab-name}
  ```
- If all sub-labs are "merged": proceed

### Step 2: Contract Compliance Validation
For each contract in `manifest.contracts.mustSatisfy`:

1. Read the contract definition from `contracts/`
2. Read the lab's source code
3. Verify EACH requirement in the contract:
   - **Interface:** Does the implementation match the defined interface?
   - **Behavior:** Does the code implement the expected behaviors?
   - **Integration points:** Are registration, invocation, and data flow correct?
   - **Constraints:** Are non-functional requirements met?
   - **Validation checklist:** Are all items satisfied?

4. Generate a compliance report:
   ```
   Contract Compliance Report:

   ✅ editing-history — PASS
      - Implements HistoryAction interface ✓
      - Pushes to undo stack on every edit ✓
      - Supports redo ✓

   ❌ filter-pipeline — FAIL
      - Registers as pipeline filter ✓
      - Process function signature matches ✓
      - Does NOT handle async processing ✗ ← Required by contract

   Fix required before merge can proceed.
   ```

5. If ANY contract fails: **BLOCK merge**, show what needs fixing, stop
6. If ALL contracts pass: proceed to Phase 2

### Step 3: Code Quality Review
Review the lab's code against:
- Universal coding rules (`.atlas/rules/universal.md`)
- Custom rules (`.atlas/rules/custom.md`)
- Suggested rules (`.atlas/rules/suggested.md`)

Check specifically:
- Business logic and UI are separated
- No tight coupling to external modules
- Tests exist for business logic
- Contract compliance tests exist
- No dead code, no magic numbers
- Error handling follows rules

Generate quality report:
```
Code Quality Review:

✅ Separation of concerns — business/ and ui/ properly separated
✅ Loose coupling — dependencies through interfaces
⚠️  Warning: 2 functions in services/face-detect.ts exceed 30 lines
✅ Tests cover all business logic
✅ Contract compliance tests present
```

Warnings don't block merge. Only critical issues (rule violations) block.

## Phase 2: Integration Planning

### Step 4: Analyze Integration Points
Read the parent (main pipeline or parent lab) to determine:
1. Where should the lab's files go in the parent structure?
2. What existing files need modification?
3. What new imports/registrations are needed?
4. Are there any conflicts with changes made to the parent since lab creation?

### Step 5: Check for Stale Contracts
Compare the lab's contract copies with the current contracts in the parent:
- If contracts have changed since the lab was created: **warn the user**
  ```
  ⚠️ Contract "editing-history" has changed since this lab was created.
  Changes: {diff}

  Options:
  A) Sync contracts and fix implementation first (/atlas lab sync {name})
  B) Proceed anyway (if changes don't affect this feature)
  ```

### Step 6: Generate Integration Plan
Create a detailed plan:
```
Integration Plan for "{lab-name}" → {parent}:

New files to add:
  lab/src/business/services/retouching.ts → src/features/retouching/services/retouching.ts
  lab/src/business/models/face-region.ts → src/features/retouching/models/face-region.ts
  lab/src/ui/components/RetouchPanel.tsx → src/components/panels/RetouchPanel.tsx
  lab/src/tests/*.test.ts → src/features/retouching/__tests__/

Files to modify:
  src/pipeline/registry.ts — Add retouching to filter registry
  src/components/Sidebar.tsx — Add RetouchPanel to sidebar menu
  src/types/index.ts — Export new types

No conflicts detected.
```

Present to user for approval.

## Phase 3: Execute Merge

### Step 7: Execute File Operations
On user approval:

1. **Copy new files** from lab to parent at planned locations
2. **Apply modifications** to existing files
3. **Update imports** and references
4. **Register the feature** in relevant registries/configs in the parent

### Step 8: Post-Merge Validation
Run validation in the parent project:

1. **Type check** — if applicable (TypeScript, etc.)
2. **Lint** — run the project's linter
3. **Tests** — run existing test suite + the newly added tests
4. **Build** — attempt a build to catch compilation issues

Report results:
```
Post-Merge Validation:
✅ Type check — passed
✅ Lint — passed
✅ Tests — 142 passed, 0 failed (12 new)
✅ Build — successful

Merge complete!
```

If validation fails:
```
Post-Merge Validation:
✅ Type check — passed
❌ Tests — 140 passed, 2 failed
   FAIL: src/pipeline/registry.test.ts — "registers all filters"
   FAIL: src/components/Sidebar.test.ts — "renders all panels"

These tests likely need updating to include the new feature.
Want me to fix them?
```

## Phase 4: Cleanup

### Step 9: Update Registry
Update `.atlas/registry.json`:
- Set lab status to "merged"
- Add `mergedAt` timestamp
- Record where files were integrated

### Step 10: Update Lab Manifest
Update the lab's `.atlas-lab/manifest.json`:
- Set status to "merged"
- Add merge metadata (when, where, by whom)

### Step 11: Archive or Keep
Ask the user:
```
Lab "{lab-name}" has been successfully merged.

What should I do with the lab directory?
A) Archive it (keep for reference, mark as inactive)
B) Delete it (remove the directory entirely)
C) Keep it active (in case more work is needed)
```

### Step 12: Summary
```
✅ Feature "{lab-name}" merged into {parent}!

Files added: {N}
Files modified: {N}
Tests added: {N}
All validations passed.

The feature is now live in the main pipeline.
```

</Steps>

<Tool_Usage>
- Use `explore` agent or `Read` to analyze lab source code for contract compliance
- Use `Agent(subagent_type="oh-my-claudecode:code-reviewer")` for code quality review
- Use `AskUserQuestion` for integration plan approval and archive decision
- Use `Bash` for running tests, builds, linting, type checks
- Use `Read` and `Write` for file copy and modification operations
- Use `Edit` for modifying existing parent files (imports, registrations)
- Never overwrite existing parent code without showing the diff first
</Tool_Usage>

<Validation>
After merge completes, verify:
- [ ] All contract compliance checks passed before merge
- [ ] Code quality review completed
- [ ] Integration plan was approved by user
- [ ] All new files exist in correct parent locations
- [ ] All modified files are correctly updated
- [ ] Post-merge tests pass
- [ ] Post-merge build succeeds
- [ ] Registry is updated with "merged" status
- [ ] Lab manifest is updated with merge metadata
</Validation>
