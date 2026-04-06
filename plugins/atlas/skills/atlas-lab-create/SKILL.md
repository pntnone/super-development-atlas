---
name: atlas-lab-create
description: Create an isolated feature lab with contracts and scoped CLAUDE.md
argument-hint: "<feature-name>"
level: 3
---

<Purpose>
Create a fully isolated Feature Lab for developing a complex feature. The lab is a separate directory containing only the relevant contracts from the main pipeline, a scoped CLAUDE.md that restricts Claude's context, and auto-generated requirements from a developer brief + Q&A expansion. Includes complexity analysis that suggests splitting into sub-labs when needed.
</Purpose>

<Use_When>
- User says "/atlas lab create {name}", "create lab", "new feature lab"
- User wants to develop a complex feature in isolation
- User wants to start a new feature that would bloat the main pipeline context
</Use_When>

<Do_Not_Use_When>
- Atlas is not initialized — prompt user to run `/atlas init` first
- The feature is simple enough to develop directly in the main pipeline
- A lab with the same name already exists — inform user and suggest a different name
</Do_Not_Use_When>

<Pre_Flight>
1. Verify `.atlas/config.json` exists — if not, tell user to run `/atlas init`
2. Read `.atlas/config.json` to get project name and labs path
3. Read `.atlas/registry.json` to check for name conflicts
4. Read `.atlas/contracts/_index.json` to know available contracts
</Pre_Flight>

<Steps>

## Step 1: Feature Brief

Ask the user for a brief description of the feature (1-3 sentences).

Example:
```
"What feature are you building? (brief description)"
```

## Step 2: Requirements Q&A

Conduct a short, focused Q&A to expand the brief into full requirements. Ask ONE question at a time, building on previous answers.

**Question categories to cover:**
1. **Core functionality** — What exactly should this feature do?
2. **Users/actors** — Who uses this feature?
3. **Inputs/outputs** — What data goes in and comes out?
4. **Edge cases** — What happens in unusual situations?
5. **Non-functional** — Performance, security, accessibility requirements?
6. **Dependencies** — Does this feature depend on other features or external services?

**Stop asking when** requirements are clear enough to identify contracts and estimate complexity. Usually 3-6 questions.

## Step 3: Contract Identification

Based on the requirements, identify which contracts from the main pipeline this feature must satisfy:

1. Read all contracts from `.atlas/contracts/`
2. Match requirements against contract purposes
3. **Cross-cutting contracts** (marked "All features") are automatically included
4. **Feature-specific contracts** are included based on relevance
5. Present the contract list to the user for confirmation
6. If the feature needs something not covered by existing contracts, flag it: "A new contract may be needed for X. Want to create one now?"

## Step 4: Complexity Analysis

Analyze the feature to determine if it should be split:

**Complexity signals (suggest splitting if 3+ are true):**
- Feature has 4+ distinct sub-systems or components
- Feature touches 3+ contracts
- Requirements document has 10+ distinct acceptance criteria
- Feature involves both frontend and backend work with complex interaction
- Estimated implementation would exceed 15+ files
- Feature has independent sub-components that could be developed in parallel

**If complex:**
```
"This feature is complex. I recommend splitting into sub-labs:

1. {sub-feature-1} — {description}
2. {sub-feature-2} — {description}
3. {sub-feature-3} — {description}

This allows parallel development and cleaner integration.
Want to split now, or proceed as a single lab?"
```

**If manageable:** proceed with single lab creation.

## Step 5: Create Lab Directory

1. Determine lab path: `{labs-repo-path}/{lab-name}/`
2. Create directory structure:
   ```
   {lab-name}/
   ├── CLAUDE.md
   ├── .atlas-lab/
   │   ├── manifest.json
   │   ├── origin.json
   │   └── requirements.md
   ├── contracts/
   └── src/
       ├── business/
       │   ├── models/
       │   ├── services/
       │   └── interfaces/
       ├── ui/
       │   ├── components/
       │   └── containers/
       ├── data/
       └── tests/
   ```

## Step 6: Copy Contracts

1. Copy each required contract from `.atlas/contracts/` to `{lab}/contracts/`
2. These are READ-ONLY references — the lab implements them, doesn't modify them

## Step 7: Generate Requirements Document

Write `.atlas-lab/requirements.md` with:
```markdown
# Feature: {lab-name}

## Description
{expanded description from Q&A}

## Functional Requirements
1. {FR-1}
2. {FR-2}
...

## Non-Functional Requirements
1. {NFR-1}
2. {NFR-2}
...

## Contracts to Satisfy
- {contract-1}: {what this feature must implement}
- {contract-2}: {what this feature must implement}

## Acceptance Criteria
- [ ] {AC-1}
- [ ] {AC-2}
...

## Out of Scope
- {things explicitly NOT in this feature}
```

## Step 8: Generate Manifest

Write `.atlas-lab/manifest.json`:
```json
{
  "version": "1.0.0",
  "lab": {
    "name": "{lab-name}",
    "description": "{description}",
    "createdAt": "{timestamp}",
    "status": "active"
  },
  "parent": {
    "type": "main-pipeline",
    "name": "{project-name}",
    "path": "{absolute-path-to-main-project}"
  },
  "contracts": {
    "mustSatisfy": ["{contract-1}", "{contract-2}"],
    "exposes": []
  },
  "requirements": {
    "functional": ["{FR-1}", "{FR-2}"],
    "nonFunctional": ["{NFR-1}", "{NFR-2}"]
  },
  "subLabs": [],
  "developers": []
}
```

## Step 9: Generate Origin File

Write `.atlas-lab/origin.json`:
```json
{
  "mainPipeline": "{absolute-path-to-main-project}",
  "labsRepo": "{absolute-path-to-labs-repo}",
  "projectName": "{project-name}",
  "parentLab": null
}
```

## Step 10: Generate Scoped CLAUDE.md

Generate the lab's CLAUDE.md using the `claude-lab.md` template. Fill in:
- Lab name and description
- Feature requirements summary
- List of contracts to satisfy with brief description of each
- Coding rules summary (universal + custom)
- The mandatory `src/` architecture (business/ui/data/tests separation)

**CRITICAL:** The CLAUDE.md must include the scope restriction:
> "You are working ONLY on the '{lab-name}' feature. Do NOT read, reference, or modify any code outside this directory."

## Step 11: Register Lab

Update `.atlas/registry.json` in the main pipeline:
```json
{
  "labs": {
    "{lab-name}": {
      "path": "{absolute-lab-path}",
      "status": "active",
      "createdAt": "{timestamp}",
      "contracts": ["{contract-1}", "{contract-2}"],
      "parentLab": null,
      "developers": []
    }
  }
}
```

## Step 12: Summary

Display:
```
Feature Lab "{lab-name}" created successfully!

Location: {lab-path}
Contracts: {list of contracts}
Requirements: {lab-path}/.atlas-lab/requirements.md

Next steps:
1. Open Claude in the lab: cd {lab-path} && claude
2. Start developing — Claude will only see this feature's context
3. When done: /atlas lab merge {lab-name}
```

If complexity analysis suggested splitting:
```
Recommended: Split this lab into sub-labs with /atlas lab split {lab-name}
```

</Steps>

<Tool_Usage>
- Use `AskUserQuestion` for the feature brief and Q&A questions
- Use `Read` to read existing contracts and config
- Use `Bash` for creating directories
- Use `Write` for generating all lab files
- Ask ONE question at a time during Q&A — never batch
- Explore the main pipeline codebase if needed to inform contract selection
</Tool_Usage>

<Validation>
After lab creation, verify:
- [ ] Lab directory exists at the correct path
- [ ] `CLAUDE.md` exists with scope restriction and contract list
- [ ] `.atlas-lab/manifest.json` has correct parent and contracts
- [ ] `.atlas-lab/origin.json` points to correct main pipeline
- [ ] `.atlas-lab/requirements.md` has full requirements from Q&A
- [ ] `contracts/` contains copies of all required contracts
- [ ] `src/` has the mandatory business/ui/data/tests structure
- [ ] Main pipeline `.atlas/registry.json` is updated with new lab entry
- [ ] Lab name doesn't conflict with existing labs
</Validation>
