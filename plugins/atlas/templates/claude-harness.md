# Preview Harness: {{PROJECT_NAME}}

> **SCOPE:** This is the shared UI preview harness for all feature labs. It provides the minimal app shell, providers, and contract mocks needed to run any lab's UI in isolation.

## Purpose

This harness allows testing and previewing UI from any feature lab or sub-lab WITHOUT duplicating app scaffolding code. Each lab mounts into this harness via its `preview.config.json`.

## Structure

```
.atlas-harness/
├── config.json          ← Harness configuration (tech stack, providers)
├── CLAUDE.md            ← This file
├── shell/               ← Minimal app entry point
│   └── {{ENTRY_FILE}}   ← App shell that mounts lab UI
├── providers/           ← Shared providers (theme, navigation, state)
│   ├── theme.{{EXT}}
│   ├── navigation.{{EXT}}
│   └── state.{{EXT}}
├── mocks/               ← Auto-generated contract mocks
│   └── {{CONTRACT_NAME}}.mock.{{EXT}}
└── scripts/             ← Preview runner scripts
    └── mount.{{EXT}}    ← Lab mounting logic
```

## Rules

1. **No business logic here** — only app shell, providers, and mocks
2. **No lab-specific code** — the harness is generic, labs plug into it
3. **Mocks are contract-driven** — every mock implements a contract interface exactly
4. **Keep it minimal** — only what's needed to render a lab's UI
5. **Never copy lab code here** — the harness reads from the lab's directory

## How It Works

1. A lab has a `preview.config.json` declaring its entry component and routes
2. The harness shell reads this config and mounts the lab's UI component
3. Contract dependencies are satisfied by auto-generated mocks in `mocks/`
4. Labs can override specific mocks via `preview.config.json` → `dependencies.mockOverrides`

## For Claude

- When modifying the harness, ensure ALL existing labs can still mount
- When adding a new provider, update the shell to include it
- When a contract changes, regenerate its mock
- Do NOT add lab-specific code — if a lab needs something custom, it goes in mockOverrides
