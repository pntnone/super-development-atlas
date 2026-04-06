---
name: atlas-init
description: Initialize Atlas in a new or existing project — sets up contracts, rules, and labs repo
argument-hint: "[--existing] [project-path]"
level: 3
---

<Purpose>
Initialize the Atlas divide & conquer framework in a project. Handles both new (empty) projects and existing codebases. For existing projects, analyzes the codebase, proposes contracts based on cross-cutting concerns, suggests refactoring where architecture doesn't meet Atlas standards, and creates the foundation for feature lab development.
</Purpose>

<Use_When>
- User says "/atlas init", "initialize atlas", "setup atlas"
- User wants to start using Atlas in their project
- User wants to retrofit Atlas into an existing codebase
</Use_When>

<Do_Not_Use_When>
- Atlas is already initialized (`.atlas/` directory exists) — inform user and suggest `/atlas rules` or `/atlas contract` instead
- User just wants to create a feature lab — use `/atlas lab create`
</Do_Not_Use_When>

<Steps>

## Pre-flight Check

1. Check if `.atlas/` already exists in the current project directory
   - If yes: inform user Atlas is already initialized, show current config, and stop
2. Detect the project root (look for `.git/`, `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, etc.)
   - If no project root found: treat as new empty project
3. Determine project name from directory name or package config

## Mode Detection

Scan the project to determine mode:
- **New project** — directory is empty or has only config files (package.json, etc.) with no source code
- **Existing project** — has source code files

## Mode: New Project

### Step 1: Project Setup
1. Ask the user: "What are you building? (brief description)"
2. Ask: "What tech stack will you use?" (or detect from existing config files)
3. Create `.atlas/` directory structure:
   ```
   .atlas/
   ├── config.json          ← Project config (name, paths, settings)
   ├── contracts/
   │   └── _index.json      ← Empty contract registry
   ├── rules/
   │   ├── universal.md     ← Copy from Atlas templates
   │   ├── custom.md        ← Empty, for developer-defined rules
   │   └── suggested.md     ← Will be generated
   └── registry.json        ← Empty labs registry
   ```

### Step 2: Generate Rules
1. Copy universal rules from Atlas templates → `.atlas/rules/universal.md`
2. Create empty `.atlas/rules/custom.md` with instructions for developer
3. Based on tech stack, generate `.atlas/rules/suggested.md` with stack-specific rules:
   - For React: component patterns, hook rules, state management
   - For Python: PEP8, type hints, async patterns
   - For Go: error handling, interface patterns
   - For any stack: infer appropriate conventions
4. Present suggested rules to user for review and approval

### Step 3: Initial Contracts
1. Based on the project description, propose cross-cutting contracts that every feature should implement
2. Common cross-cutting contracts to consider:
   - Error handling contract
   - Logging/observability contract
   - Authentication/authorization contract (if applicable)
   - State management contract (if applicable)
3. For each proposed contract, generate using the contract template
4. User reviews and approves/modifies contracts

### Step 4: Labs Repo
1. Determine labs path: `../{project-name}-labs/`
2. Create the labs directory
3. Initialize it as a git repo: `git init`
4. Create a `.gitignore` and basic README in the labs repo

### Step 5: Generate CLAUDE.md
1. Generate the main pipeline CLAUDE.md using the `claude-main.md` template
2. Fill in: project name, architecture summary, contracts list, rules summary
3. If a CLAUDE.md already exists, merge Atlas content into it (don't overwrite)

### Step 6: Summary
- Display what was created
- Show the contracts defined
- Show the rules applied
- Show the labs repo location
- Suggest next steps: "Use `/atlas lab create {name}` to start your first feature lab"

## Mode: Existing Project

### Step 1: Deep Analysis
Use an `explore` agent to analyze the codebase:
1. **Tech stack detection** — frameworks, languages, build tools
2. **Architecture analysis** — how is code organized? What patterns are used?
3. **Separation of concerns** — is business logic separated from UI? Is data access isolated?
4. **Cross-cutting concerns** — what features span the entire app? (auth, logging, error handling, state management, etc.)
5. **Coupling analysis** — how tightly coupled are the modules?
6. **Existing interfaces** — are there already well-defined interfaces/contracts?

### Step 2: Architecture Report
Present findings to the user:
```
## Current Architecture Analysis

### Strengths
- {{what's already good}}

### Issues Found
- {{tight coupling, mixed concerns, etc.}}

### Cross-Cutting Concerns Detected
- {{auth, logging, etc.}}

### Recommended Contracts
- {{contracts based on existing cross-cutting concerns}}
```

### Step 3: Refactoring Assessment
If issues are found, propose a refactoring plan:
1. Identify areas that need refactoring to align with Atlas principles
2. Group refactoring into independent units
3. **Propose each refactoring unit as a Feature Lab** — "Use Atlas to refactor into Atlas"
4. Prioritize: which refactoring is critical vs. nice-to-have?
5. User reviews and decides which to tackle

### Step 4: Create Atlas Structure
Same as New Project Steps 1-5, but:
- Rules are generated based on EXISTING code patterns (not just tech stack)
- Contracts are extracted from EXISTING cross-cutting concerns
- CLAUDE.md incorporates EXISTING architecture

### Step 5: Generate Refactoring Labs (if approved)
For each approved refactoring unit:
1. Create a lab entry in the registry (but don't create the lab directory yet)
2. Mark it as "planned" status
3. User can then `/atlas lab create {refactor-name}` when ready

### Step 6: Summary
- Display architecture analysis results
- Show contracts created
- Show refactoring labs proposed
- Show next steps

</Steps>

<Tool_Usage>
- Use `explore` agent to analyze existing codebases — never ask the user about things you can discover by reading code
- Use `AskUserQuestion` for preference questions (tech stack, project description, which refactorings to approve)
- Use `Bash` for creating directories and initializing git repos
- Use `Write` for creating config files, rules, and contracts
- Use `Read` to check for existing CLAUDE.md before overwriting
- When generating suggested rules, consider the tech stack and existing patterns — don't generate generic rules
</Tool_Usage>

<Examples>
<Good>
New project init:
```
User: /atlas init
Claude: [detects empty project]
Claude: "What are you building?"
User: "A photo editor app"
Claude: "What tech stack?"
User: "React + Python FastAPI"
Claude: [creates .atlas/ structure]
Claude: [generates rules for React + Python]
Claude: [proposes contracts: editing-history, filter-pipeline, ui-panel]
Claude: "Here's what I've set up: ..."
```
</Good>

<Good>
Existing project init:
```
User: /atlas init
Claude: [detects existing codebase with 50+ files]
Claude: [spawns explore agent to analyze]
Claude: "I've analyzed your project. Here's what I found:
         - React frontend with business logic mixed into components
         - Express backend with good route separation but no service layer
         - Auth is duplicated in 3 places
         Recommended: 3 refactoring labs to clean this up.
         Want to proceed?"
```
</Good>

<Bad>
```
User: /atlas init
Claude: "What framework are you using? Where is your auth code? How do you handle errors?"
```
Why bad: Claude should explore the codebase first, not ask the user to describe their own code.
</Bad>
</Examples>

<Validation>
After init completes, verify:
- [ ] `.atlas/config.json` exists and has correct project name and paths
- [ ] `.atlas/contracts/_index.json` exists
- [ ] `.atlas/rules/universal.md` exists with full rule set
- [ ] `.atlas/rules/custom.md` exists (even if empty)
- [ ] `.atlas/rules/suggested.md` exists with stack-appropriate rules
- [ ] `.atlas/registry.json` exists
- [ ] `../{project-name}-labs/` directory exists and is a git repo
- [ ] `CLAUDE.md` exists and includes Atlas content
- [ ] For existing projects: architecture analysis was performed and presented
</Validation>
