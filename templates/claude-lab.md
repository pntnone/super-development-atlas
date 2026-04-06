# Feature Lab: {{LAB_NAME}}

> **SCOPE RESTRICTION:** You are working ONLY on the "{{LAB_NAME}}" feature. Do NOT read, reference, or modify any code outside this directory. Everything you need is here.

## Feature Description

{{FEATURE_DESCRIPTION}}

## Requirements

See `.atlas-lab/requirements.md` for full functional and non-functional requirements.

## Contracts to Satisfy

This feature MUST implement the following contracts from the main pipeline:

{{CONTRACTS_LIST}}

Read each contract in `contracts/` carefully. Your implementation must satisfy every interface, behavior, and constraint defined in these contracts.

## Coding Rules

All code MUST follow:

1. **Universal rules** — Business logic and UI strictly separated, SOLID principles, loose coupling
2. **Custom rules** — {{CUSTOM_RULES_SUMMARY}}

### Mandatory Architecture

```
src/
├── business/        ← Business logic ONLY. No UI imports. No framework dependencies.
│   ├── models/      ← Domain models and entities
│   ├── services/    ← Business services and use cases
│   └── interfaces/  ← Interfaces this feature exposes
├── ui/              ← UI components ONLY. No business logic.
│   ├── components/  ← Presentational components
│   └── containers/  ← Components that connect UI to business logic
├── data/            ← Data access, API calls, persistence
└── tests/           ← Tests for business logic and contract compliance
```

## Contract Compliance Testing

Every contract listed above must have a corresponding test that verifies:
- The interface is correctly implemented
- The expected behavior is met
- Integration points work as specified

## For Claude

- You can ONLY see and modify files in this directory
- Check `contracts/` before writing any integration code
- If you need something from the main pipeline that isn't in contracts, STOP and ask — a new contract may be needed
- When the feature is complete, it will be merged via `/atlas lab merge {{LAB_NAME}}`
- Focus on doing ONE thing well — this feature, nothing else
