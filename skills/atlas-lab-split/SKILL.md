---
name: atlas-lab-split
description: Split a feature lab into recursive sub-labs with internal contracts
argument-hint: "<lab-name>"
level: 3
---

<Purpose>
Split an existing feature lab into smaller, focused sub-labs. Each sub-lab is its own isolated directory with its own contracts, CLAUDE.md, and requirements. The parent lab becomes a mini-pipeline that orchestrates its sub-labs. Sub-labs can be developed in parallel by different developers.
</Purpose>

<Use_When>
- User says "/atlas lab split {name}", "split lab", "break down feature"
- Complexity analysis during lab creation recommended splitting
- A feature lab has grown too large for efficient Claude work
- Multiple developers want to work on different parts of the same feature
</Use_When>

<Do_Not_Use_When>
- Lab doesn't exist — inform user
- Lab is already split and user wants to split a sub-lab deeper than max depth (default 2) — warn about complexity
- Feature is simple enough to stay as one lab
</Do_Not_Use_When>

<Pre_Flight>
1. Verify the lab exists by checking `.atlas/registry.json` or finding it in the labs directory
2. Read the lab's `.atlas-lab/manifest.json` to understand current state
3. Read the lab's `.atlas-lab/requirements.md` to understand the feature scope
4. Read the lab's contracts to understand integration boundaries
5. Check current nesting depth — warn if splitting would exceed `labs.maxDepth` from config
</Pre_Flight>

<Steps>

## Step 1: Analyze Feature for Splitting

Read the lab's requirements and any existing source code. Identify natural boundaries:

**Splitting criteria:**
- Independent sub-systems (e.g., detection, processing, rendering)
- Separate business domains within the feature
- Frontend vs backend components (if both exist)
- Components with different data dependencies
- Parts that can be developed and tested independently

## Step 2: Propose Sub-Lab Breakdown

Present the proposed split to the user:

```
Proposed split for "{lab-name}":

1. {sub-lab-1} — {description}
   Contracts: inherits {parent-contracts}, exposes {new-contract}

2. {sub-lab-2} — {description}
   Contracts: inherits {parent-contracts}, consumes {sub-lab-1-contract}

3. {sub-lab-3} — {description}
   Contracts: inherits {parent-contracts}, consumes {sub-lab-1-contract}

Dependencies:
- {sub-lab-2} and {sub-lab-3} depend on {sub-lab-1}'s output
- {sub-lab-2} and {sub-lab-3} can be developed in parallel

Want to adjust this breakdown?
```

Allow the user to:
- Approve as-is
- Add/remove sub-labs
- Change scope of sub-labs
- Adjust dependencies

## Step 3: Generate Internal Contracts

For communication between sub-labs, create contracts within the parent lab:

1. Identify what data/interfaces sub-labs need to share
2. Create contract files in the parent lab's `contracts/` directory (these are NEW contracts specific to this feature, not main pipeline contracts)
3. Each contract follows the standard contract template

**Example:** For "retouching-face" split:
- `face-regions.contract.md` — exposed by `face-detection`, consumed by `skin-smoothing`, `blemish-removal`, `eye-enhancement`

## Step 4: Create Sub-Lab Directories

For each sub-lab, create within the parent lab's `labs/` directory:

```
{parent-lab}/
├── CLAUDE.md              ← Updated to reflect parent orchestrator role
├── .atlas-lab/
│   └── manifest.json      ← Updated with subLabs list
├── contracts/
│   ├── {main-contracts}   ← From main pipeline
│   └── {internal-contracts} ← New contracts between sub-labs
├── src/                   ← Shared/orchestration code
└── labs/
    ├── {sub-lab-1}/
    │   ├── CLAUDE.md      ← Scoped to sub-lab-1 only
    │   ├── .atlas-lab/
    │   │   ├── manifest.json
    │   │   ├── origin.json    ← Points to parent lab
    │   │   └── requirements.md
    │   ├── contracts/     ← Relevant contracts only
    │   └── src/
    │       ├── business/
    │       ├── ui/
    │       ├── data/
    │       └── tests/
    ├── {sub-lab-2}/
    │   └── ...
    └── {sub-lab-3}/
        └── ...
```

## Step 5: Generate Sub-Lab Files

For EACH sub-lab:

### 5a: Requirements
Split the parent's requirements into sub-lab specific requirements. Each sub-lab gets only its portion.

### 5b: Manifest
```json
{
  "version": "1.0.0",
  "lab": {
    "name": "{sub-lab-name}",
    "description": "{sub-lab-description}",
    "createdAt": "{timestamp}",
    "status": "active"
  },
  "parent": {
    "type": "feature-lab",
    "name": "{parent-lab-name}",
    "path": "{absolute-path-to-parent-lab}"
  },
  "contracts": {
    "mustSatisfy": ["{parent-contracts}", "{internal-contracts}"],
    "exposes": ["{contracts-this-sub-lab-exposes}"]
  },
  "subLabs": [],
  "developers": []
}
```

### 5c: Origin
```json
{
  "mainPipeline": "{absolute-path-to-main-project}",
  "labsRepo": "{absolute-path-to-labs-repo}",
  "projectName": "{project-name}",
  "parentLab": "{absolute-path-to-parent-lab}"
}
```

### 5d: Scoped CLAUDE.md
Each sub-lab's CLAUDE.md:
- Scope restriction: "You are working ONLY on {sub-lab-name}"
- Lists only the contracts relevant to THIS sub-lab
- Includes the coding rules
- References the parent lab's internal contracts where applicable
- Does NOT mention sibling sub-labs' implementation details

### 5e: Copy Relevant Contracts
Only copy contracts that this specific sub-lab needs:
- Main pipeline contracts it must satisfy
- Internal contracts it must implement or consume
- Do NOT copy contracts irrelevant to this sub-lab

## Step 6: Update Parent Lab

Update the parent lab to reflect its new role as orchestrator:

### 6a: Update manifest.json
Add sub-labs to the `subLabs` array.

### 6b: Update CLAUDE.md
The parent lab's CLAUDE.md changes to orchestrator mode:
```
# Feature Lab: {parent-lab-name} (Orchestrator)

This lab has been split into sub-labs. This directory contains:
- Shared/orchestration code in src/
- Internal contracts in contracts/
- Sub-labs in labs/

Sub-labs:
- {sub-lab-1}: {description} — {status}
- {sub-lab-2}: {description} — {status}

Do NOT implement feature logic here. Feature logic lives in sub-labs.
This directory is for orchestration, integration, and shared utilities only.
```

### 6c: Add internal contracts to contracts/
The new inter-sub-lab contracts live in the parent's contracts directory.

## Step 7: Update Registry

Update the main pipeline's `.atlas/registry.json`:
- Add entries for each sub-lab
- Update parent lab entry to include `subLabs` list

## Step 8: Summary

```
Lab "{lab-name}" split into {N} sub-labs:

1. {sub-lab-1} — {description}
   Path: {path}
   Contracts: {list}

2. {sub-lab-2} — {description}
   Path: {path}
   Contracts: {list}

Internal contracts created:
- {contract-name}: {purpose}

Development order:
- Start with: {sub-lab with no dependencies}
- Then parallel: {sub-labs that can run in parallel}
- Finally: {sub-labs that depend on others}

To work on a sub-lab:
  cd {sub-lab-path} && claude
```

</Steps>

<Tool_Usage>
- Use `Read` to analyze existing lab requirements and source code
- Use `AskUserQuestion` for approving the split breakdown
- Use `Bash` for creating sub-lab directories
- Use `Write` for generating all sub-lab files
- Present the split proposal clearly with dependencies visualized
</Tool_Usage>

<Validation>
After split completes, verify:
- [ ] Each sub-lab directory exists with correct structure
- [ ] Each sub-lab has its own CLAUDE.md with scope restriction
- [ ] Each sub-lab has only its relevant contracts (not all contracts)
- [ ] Internal contracts exist in parent's contracts directory
- [ ] Parent lab's manifest lists all sub-labs
- [ ] Parent lab's CLAUDE.md reflects orchestrator role
- [ ] Main pipeline registry is updated with all sub-lab entries
- [ ] No circular dependencies between sub-labs
- [ ] Nesting depth doesn't exceed configured max
</Validation>
