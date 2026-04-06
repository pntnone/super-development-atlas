---
name: atlas-lab-list
description: List all active feature labs with status, contracts, and tree view
argument-hint: "[--all]"
level: 2
---

<Purpose>
Display all feature labs for the current project, including their status, assigned contracts, developers, and sub-lab hierarchy. Provides a clear overview of all ongoing isolated development work.
</Purpose>

<Use_When>
- User says "/atlas lab list", "list labs", "show labs", "what labs exist"
- User wants an overview of all feature development in progress
- Before merging, to check which labs are ready
</Use_When>

<Pre_Flight>
1. Verify `.atlas/config.json` exists — if not, tell user to run `/atlas init`
2. Read `.atlas/registry.json`
3. Read `.atlas/config.json` for labs path
</Pre_Flight>

<Steps>

## Step 1: Read Registry

Read `.atlas/registry.json` and collect all lab entries.

## Step 2: Enrich Data

For each lab in the registry:
1. Check if the lab directory still exists at its registered path
2. If it exists, read `.atlas-lab/manifest.json` for current status and details
3. If it doesn't exist, mark as "missing"

## Step 3: Display

### Default view (active labs only):
```
Atlas Labs for "{project-name}"
Labs repo: {labs-path}

├── retouching-face [active] — AI face retouching
│   Contracts: editing-history, filter-pipeline, ui-panel
│   ├── face-detection [merged] — Detect faces in image
│   ├── skin-smoothing [active] — Smooth skin regions
│   │   Developer: @alice
│   ├── blemish-removal [active] — Remove blemishes
│   │   Developer: @bob
│   └── eye-enhancement [pending] — Enhance eyes
│
└── background-blur [active] — Background blur effect
    Contracts: editing-history, filter-pipeline, ui-panel
    Developer: @charlie

Active: 4 | Merged: 1 | Pending: 1
```

### With `--all` flag (include merged/archived):
Show all labs including those with status "merged" or "archived", dimmed or with strikethrough indication.

## Step 4: Warnings

Flag any issues:
- Labs with stale contracts (parent contracts changed since lab creation)
- Labs marked active but directory is missing
- Labs with no developer assigned
- Sub-labs blocking their parent from merging

</Steps>

<Tool_Usage>
- Use `Read` for registry and manifest files
- Use `Bash` to check directory existence
- Output formatted tree directly — no tool needed
</Tool_Usage>
