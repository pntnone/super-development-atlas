# {{PROJECT_NAME}} — Main Pipeline

This project uses **Atlas** for divide & conquer development. Complex features are developed in isolated Feature Labs and merged back through contracts.

## Architecture

{{CLAUDE_GENERATED_ARCHITECTURE_SUMMARY}}

## Atlas Structure

- **Contracts:** `.atlas/contracts/` — Interface definitions that feature labs must implement
- **Rules:** `.atlas/rules/` — Coding standards (universal + custom + suggested)
- **Config:** `.atlas/config.json` — Atlas configuration
- **Registry:** `.atlas/registry.json` — Active feature labs tracker
- **Labs Repo:** `../{{PROJECT_NAME}}-labs/` — Isolated feature lab directories

## Coding Rules

All code in this project MUST follow:

1. **Universal rules** — see `.atlas/rules/universal.md`
2. **Custom rules** — see `.atlas/rules/custom.md`
3. **Suggested rules** — see `.atlas/rules/suggested.md`

Key principles:
- Business logic and UI are STRICTLY separated
- Modules communicate through contracts/interfaces (loose coupling)
- Dependencies point inward (Clean Architecture)
- Each feature is independently testable

## Contracts

All features must satisfy the contracts defined in `.atlas/contracts/`. Before modifying any feature, read its relevant contracts first.

{{CONTRACTS_LIST}}

## Working on Features

- **Small changes:** Work directly in the main pipeline
- **Complex features:** Use `/atlas lab create {name}` to create an isolated Feature Lab
- **Never bypass contracts** — All integration goes through defined interfaces

## For Claude

- When asked to add a complex feature, suggest creating a Feature Lab
- Always check contract compliance before merging
- Follow the coding rules strictly — no exceptions
- If you notice tight coupling or mixed concerns, flag it
