---
name: atlas-rules
description: Manage coding rules — view, add custom rules, or regenerate Claude-suggested rules
argument-hint: "[show|add|suggest|reset]"
level: 2
---

<Purpose>
Manage the three-layer coding rules system: universal (built-in defaults), custom (developer-defined), and suggested (Claude-analyzed). Rules are enforced in every feature lab and the main pipeline through CLAUDE.md files.
</Purpose>

<Use_When>
- User says "/atlas rules", "show rules", "add rule", "update rules"
- User wants to add project-specific coding standards
- User wants Claude to re-analyze and suggest new rules
- User wants to see what rules are currently active
</Use_When>

<Pre_Flight>
1. Verify `.atlas/config.json` exists
2. Read `.atlas/rules/universal.md`, `custom.md`, and `suggested.md`
</Pre_Flight>

<Steps>

## Command: `show` (default)

Display all active rules grouped by type:

```
Atlas Coding Rules for "{project-name}":

═══ Universal (built-in, always active) ═══
1. Clean Code — meaningful names, small functions, no magic numbers
2. Separation of Concerns — business/UI/data strictly separated
3. SOLID Principles — SRP, OCP, LSP, ISP, DIP
4. Loose Coupling — interfaces, no circular deps, DI
5. DRY with Judgment — extract at 3+, don't over-abstract
6. Error Handling — handle at right level, fail fast, no silent failures
7. Testing — isolated business logic tests, test behavior not impl
8. Documentation — self-documenting code, comments explain why
9. Security — validate inputs, no secrets in code
10. Performance — don't optimize prematurely, right data structures

═══ Custom (developer-defined) ═══
1. {custom rule 1}
2. {custom rule 2}
(empty if none defined)

═══ Suggested (Claude-analyzed) ═══
1. {suggested rule 1 based on project patterns}
2. {suggested rule 2}
```

## Command: `add`

1. Ask: "What rule do you want to add?"
2. Ask: "Why is this rule important for your project?" (context helps enforce it correctly)
3. Append to `.atlas/rules/custom.md`
4. Confirm: "Rule added. It will apply to all new feature labs. Existing labs need manual CLAUDE.md update."

## Command: `suggest`

1. Analyze the current project codebase using an `explore` agent
2. Look for:
   - Patterns used consistently (should be codified as rules)
   - Anti-patterns found (should be rules to prevent them)
   - Stack-specific best practices not yet captured
   - Patterns in existing labs that worked well
3. Present suggestions to user
4. User approves/rejects each suggestion
5. Write approved suggestions to `.atlas/rules/suggested.md`

## Command: `reset`

Reset rules to defaults:
- `universal.md` → restore from Atlas template
- `custom.md` → clear (with user confirmation)
- `suggested.md` → clear and re-generate via `suggest`

</Steps>

<Tool_Usage>
- Use `Read` to display current rules
- Use `Edit` or `Write` to update rule files
- Use `explore` agent to analyze codebase for `suggest` command
- Use `AskUserQuestion` for rule additions and suggestion approvals
</Tool_Usage>

<Validation>
- [ ] Rule files exist and are well-formatted
- [ ] Custom rules are appended, not overwritten
- [ ] Suggested rules reflect actual project patterns
</Validation>
