# Atlas — Divide & Conquer Development Framework

Atlas solves the biggest problem when using AI coding assistants on large projects: **context bloat**. As your project grows, the AI reads everything, loses focus, produces more bugs, and wastes tokens fixing them.

Atlas fixes this with a simple principle: **divide to conquer**. Complex features are developed in fully isolated directories (Feature Labs) that communicate with the main project only through well-defined contracts. The AI sees only what it needs — nothing more.

## How It Works

```
Your Project (Main Pipeline)          Feature Labs (Isolated)
┌─────────────────────┐               ┌──────────────────┐
│  src/                │               │  retouching-face/ │
│  .atlas/             │   contracts   │    CLAUDE.md ←── scoped context
│    contracts/ ───────┼──────────────►│    contracts/
│    rules/            │               │    src/
│    registry.json     │               │      business/
│  CLAUDE.md           │   merge back  │      ui/
│                      │◄──────────────│      data/
└─────────────────────┘               │      tests/
                                      └──────────────────┘
                                      Claude ONLY sees this ↑
```

**Key concepts:**
- **Main Pipeline** — Your core project with contracts defining integration boundaries
- **Feature Lab** — Isolated directory where Claude focuses on ONE feature with minimal context
- **Contract** — Language-agnostic interface that defines how features plug into the main project
- **Labs Repo** — Sibling git repo (`{project}-labs/`) holding all feature labs

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and working

### Option 1: Install as Claude Code Plugin (Recommended)

```bash
# Add Atlas as a plugin marketplace
claude /plugin marketplace add https://github.com/YOUR_USERNAME/super-development-atlas

# Install Atlas
claude /plugin install atlas
```

### Option 2: Manual Installation

1. Clone this repository:
```bash
git clone https://github.com/YOUR_USERNAME/super-development-atlas.git ~/.claude/atlas
```

2. Copy the skills to your Claude Code skills directory:
```bash
cp -r ~/.claude/atlas/skills/* ~/.claude/skills/
```

3. The skills will be available on your next Claude Code session.

### Option 3: Project-Local Installation

If you want Atlas only for a specific project:

1. Clone into your project:
```bash
cd your-project
git clone https://github.com/YOUR_USERNAME/super-development-atlas.git .atlas-framework
```

2. Add to your project's `.claude/settings.local.json`:
```json
{
  "skills": {
    "directories": [".atlas-framework/skills"]
  }
}
```

## Quick Start

### 1. Initialize Atlas in your project

```bash
cd your-project
claude
```

```
You: /atlas-init
```

Atlas will:
- Scan your project (or set up a new one)
- Generate coding rules (universal + project-specific)
- Propose contracts based on cross-cutting concerns
- Create the labs repo next to your project
- Generate an Atlas-enhanced CLAUDE.md

### 2. Create a Feature Lab

```
You: /atlas-lab-create payment-system
```

Atlas will:
- Ask you to describe the feature
- Expand requirements through a short Q&A
- Identify which contracts the feature must satisfy
- Check complexity and suggest splitting if needed
- Create a fully isolated lab with scoped CLAUDE.md

### 3. Develop in Isolation

```bash
# Open Claude in the feature lab — clean, focused context
cd ../your-project-labs/payment-system
claude
```

Claude only sees the feature's code and contracts. No bloat. Maximum efficiency.

### 4. Split Complex Features (Optional)

```
You: /atlas-lab-split payment-system
```

Creates sub-labs that can be developed in parallel:
```
payment-system/
└── labs/
    ├── stripe-integration/    ← Developer A
    ├── invoice-generator/     ← Developer B
    └── payment-hooks/         ← Developer C
```

### 5. Merge Back

```bash
# Return to main project
cd your-project
claude
```

```
You: /atlas-lab-merge payment-system
```

Atlas will:
- Validate all contracts are satisfied
- Review code quality against rules
- Generate an integration plan
- Execute the merge
- Run post-merge validation (tests, build, lint)

## All Commands

| Command | Description |
|---------|-------------|
| `/atlas-init` | Initialize Atlas in a new or existing project |
| `/atlas-lab-create {name}` | Create an isolated feature lab |
| `/atlas-lab-split {name}` | Split a feature lab into sub-labs |
| `/atlas-lab-merge {name}` | Merge a feature lab back into parent |
| `/atlas-lab-sync {name}` | Sync contracts when parent changes |
| `/atlas-lab-list` | List all active labs with status tree |
| `/atlas-contract add {name}` | Add a new contract |
| `/atlas-contract list` | List all contracts |
| `/atlas-contract edit {name}` | Edit an existing contract |
| `/atlas-contract remove {name}` | Remove a contract |
| `/atlas-rules` | Show all active coding rules |
| `/atlas-rules add` | Add a custom coding rule |
| `/atlas-rules suggest` | Claude suggests rules based on codebase analysis |

## Project Structure After Init

```
your-project/                          ← Main Pipeline
├── CLAUDE.md                          ← Atlas-enhanced (guides Claude's behavior)
├── .atlas/
│   ├── config.json                    ← Project configuration
│   ├── contracts/                     ← Contract definitions
│   │   ├── _index.json               ← Contract registry
│   │   ├── error-handling.contract.md
│   │   └── auth.contract.md
│   ├── rules/
│   │   ├── universal.md              ← Built-in rules (clean code, SOLID, etc.)
│   │   ├── custom.md                 ← Your project-specific rules
│   │   └── suggested.md              ← Claude-suggested rules
│   └── registry.json                 ← Tracks all active labs
└── src/

your-project-labs/                     ← Labs Repo (sibling, separate git repo)
├── feature-a/
│   ├── CLAUDE.md                      ← Scoped — Claude only sees this feature
│   ├── .atlas-lab/
│   │   ├── manifest.json
│   │   ├── origin.json
│   │   └── requirements.md
│   ├── contracts/                     ← Copied from main (read-only reference)
│   └── src/
│       ├── business/                  ← Business logic ONLY
│       ├── ui/                        ← UI components ONLY
│       ├── data/                      ← Data access ONLY
│       └── tests/                     ← Tests for business logic + contracts
└── feature-b/
    └── ...
```

## The Rules System

Atlas enforces a three-layer rule system:

### Universal Rules (Always Active)
Built-in, non-negotiable defaults:
1. **Clean Code** — meaningful names, small functions, no magic numbers
2. **Separation of Concerns** — business logic and UI strictly separated
3. **SOLID Principles** — single responsibility, open/closed, etc.
4. **Loose Coupling** — communicate through interfaces, no circular deps
5. **DRY with Judgment** — extract at 3+ repetitions, don't over-abstract
6. **Error Handling** — handle at right level, fail fast, no silent failures
7. **Testing** — isolated business logic tests, test behavior not implementation
8. **Documentation** — self-documenting code, comments explain why
9. **Security** — validate inputs, no secrets in code
10. **Performance** — don't optimize prematurely

### Custom Rules (Developer-Defined)
Your project-specific standards. Add with `/atlas-rules add`.

### Suggested Rules (Claude-Analyzed)
Claude analyzes your codebase and suggests rules based on existing patterns and tech stack. Generated during `/atlas-init` and refreshable with `/atlas-rules suggest`.

## Contracts — The Merge Mechanism

Contracts are the core of Atlas. They define **how features plug into the main project** in a language-agnostic way.

Example contract (`editing-history.contract.md`):
```markdown
# Contract: Editing History

## Purpose
Every feature that modifies the image must integrate with the undo/redo system.

## Interface
interface HistoryAction {
  execute(): void
  undo(): void
  description: string
}

## Behavior
1. Every edit creates a HistoryAction
2. Actions are pushed to the history stack
3. Undo reverses the last action
4. Redo re-applies the last undone action

## Validation
- [ ] Feature creates HistoryAction for every edit
- [ ] Undo fully reverses the edit
- [ ] Redo re-applies the edit
- [ ] History stack correctly tracks all actions
```

When a feature lab merges, Atlas validates that **every contract is satisfied**. No compliance = no merge.

## Working with Existing Projects

Atlas handles existing codebases with a **retrofit** approach:

1. `/atlas-init` analyzes your codebase
2. It identifies strengths, issues (tight coupling, mixed concerns), and cross-cutting concerns
3. It proposes contracts based on what exists
4. It suggests refactoring areas as feature labs — **use Atlas to refactor into Atlas**
5. You work through refactoring labs one by one, each merging cleanly

## Multi-Developer Workflow

Atlas supports parallel development out of the box:

```
Developer A: cd project-labs/retouching-face/labs/face-detection && claude
Developer B: cd project-labs/retouching-face/labs/skin-smoothing && claude
Developer C: cd project-labs/background-blur && claude
```

Each developer works in complete isolation. Contracts ensure their work integrates cleanly.

## Why Atlas Works

| Problem | Atlas Solution |
|---------|---------------|
| AI reads entire project → context bloat | Feature labs isolate context to one feature |
| Big features → bugs and rework | Split into sub-labs, each focused and manageable |
| Merging isolated work → conflicts | Contracts define integration boundaries upfront |
| Code quality degrades over time | Three-layer rules enforced in every lab |
| Multiple devs stepping on each other | Sibling labs work in parallel without interference |
| Existing project is messy | Retrofit with refactoring labs |

## License

MIT
