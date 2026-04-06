---
name: atlas-lab-preview-setup
description: Initialize the shared UI preview harness in the labs repo
argument-hint: ""
level: 3
---

<Purpose>
Initialize a shared Preview Harness in the labs repo that allows any feature lab (or sub-lab) to run its UI in isolation — without duplicating app scaffolding, theme providers, navigation, or state management. The harness is created ONCE per project and shared across all labs.
</Purpose>

<Use_When>
- User says "/atlas lab preview setup", "setup preview", "init preview harness"
- User wants to test UI of feature labs without merging back to main
- First time setting up UI preview for a project's labs
</Use_When>

<Do_Not_Use_When>
- Atlas is not initialized — prompt user to run `/atlas init` first
- Preview harness already exists at `{labs-repo}/.atlas-harness/` — inform user, offer to reconfigure
- Project has no UI components (pure backend/library project)
</Do_Not_Use_When>

<Pre_Flight>
1. Verify `.atlas/config.json` exists — if not, tell user to run `/atlas init`
2. Read `.atlas/config.json` to get project name and labs path
3. Check if `.atlas-harness/` already exists in the labs repo — if yes, offer reconfigure or abort
4. Scan the main pipeline to detect tech stack (framework, language, platform)
</Pre_Flight>

<Steps>

## Step 1: Detect Tech Stack

Analyze the main pipeline project to determine:

1. **Platform:** iOS (SwiftUI/UIKit), Android (Jetpack Compose/XML), Cross-platform (React Native, Flutter, Kotlin Multiplatform), or Web (React, Vue, Angular)
2. **Language:** Swift, Kotlin, Dart, TypeScript, JavaScript
3. **UI Framework:** SwiftUI, UIKit, Jetpack Compose, Flutter Widgets, React Native, React, etc.
4. **Navigation:** React Navigation, Flutter Navigator/GoRouter, Jetpack Navigation, SwiftUI NavigationStack, etc.
5. **State management:** Redux, Riverpod, BLoC, MobX, SwiftUI @Observable, ViewModel, etc.
6. **Theme system:** Material, Cupertino, custom design system, styled-components, etc.

**Detection sources:** `package.json`, `pubspec.yaml`, `build.gradle`, `Podfile`, `Package.swift`, `tsconfig.json`, framework imports in source files.

Present findings to user for confirmation:
```
Detected tech stack:
  Platform: {{platform}}
  Framework: {{framework}}
  Language: {{language}}
  Navigation: {{navigation}}
  State: {{state_management}}
  Theme: {{theme_system}}

Is this correct? Any adjustments?
```

## Step 2: Identify Shared Providers

Based on the tech stack, determine which providers the harness needs:

**Common providers (most stacks):**
- **Theme provider** — Wraps lab UI with the project's design tokens (colors, typography, spacing)
- **Navigation provider** — Minimal navigation shell so lab screens can navigate
- **State provider** — Minimal state management setup (empty stores/providers)

**Stack-specific providers:**
- **React Native:** `SafeAreaProvider`, `GestureHandlerRootView`, `NavigationContainer`
- **Flutter:** `MaterialApp` / `CupertinoApp`, `ProviderScope`, `MediaQuery`
- **SwiftUI:** `@main App` struct, `EnvironmentObject` injection
- **Jetpack Compose:** `Theme {}`, `NavHost`, `CompositionLocalProvider`
- **Web React:** `BrowserRouter`, `ThemeProvider`, `QueryClientProvider`

Ask user if any additional providers are needed:
```
The harness will include these providers:
1. {{provider_1}} — {{purpose}}
2. {{provider_2}} — {{purpose}}
3. {{provider_3}} — {{purpose}}

Need any additional providers? (e.g., localization, accessibility, analytics mock)
```

## Step 3: Locate Theme Source

Find the project's existing theme/design system to reference (not copy):

1. Search for theme definition files in the main pipeline
2. Determine if the theme can be imported as a package/module, or needs to be extracted
3. **Preferred approach:** Reference theme from main pipeline as a dependency
4. **Fallback:** Create a minimal theme stub that matches the project's design tokens

```
Found theme at: {{theme_path}}
Strategy: {{reference_as_dependency | extract_tokens | minimal_stub}}

This means lab previews will use the same look and feel as the main app.
```

## Step 4: Generate Contract Mocks

For each contract in `.atlas/contracts/`:

1. Read the contract's **Interface** section
2. Generate a mock implementation that:
   - Implements every method/property defined in the contract
   - Returns sensible default/sample data
   - Logs calls for debugging (optional, configurable)
   - Is clearly marked as a mock (comments, naming)

3. Write mock to `.atlas-harness/mocks/{contract-name}.mock.{ext}`

**Mock generation rules:**
- Methods that return data → return realistic sample data matching the contract's types
- Methods that perform actions → no-op with console log
- Event listeners → support registration but never fire (unless triggered manually)
- Async methods → resolve immediately with sample data

```
Generated mocks for {{N}} contracts:
- {{contract-1}}.mock.{{ext}} — {{methods_count}} methods
- {{contract-2}}.mock.{{ext}} — {{methods_count}} methods
```

## Step 5: Create Harness Directory Structure

Create the harness in the labs repo:

```
{labs-repo}/.atlas-harness/
├── config.json            ← From harness-config.json template
├── CLAUDE.md              ← From claude-harness.md template
├── shell/
│   └── {{entry_file}}     ← Minimal app entry point
├── providers/
│   ├── theme.{{ext}}      ← Theme provider wrapper
│   ├── navigation.{{ext}} ← Navigation shell
│   └── state.{{ext}}      ← State management setup
├── mocks/
│   └── {contract}.mock.{{ext}}  ← One per contract
└── scripts/
    └── mount.{{ext}}      ← Lab mounting/resolution logic
```

## Step 6: Generate Shell Entry Point

Create the minimal app shell that:

1. Reads a lab's `preview.config.json` to know what to mount
2. Wraps the lab's entry component with all providers
3. Resolves the lab's path and imports its entry component
4. Injects contract mocks as dependencies

**The shell must be tech-stack-specific.** Examples:

### React Native
```jsx
// shell/App.tsx
import { registerRootComponent } from 'expo';
// ... providers, mock injection, dynamic lab loading
```

### Flutter
```dart
// shell/main.dart
import 'package:flutter/material.dart';
// ... MaterialApp, provider setup, lab widget mounting
```

### SwiftUI
```swift
// shell/PreviewApp.swift
import SwiftUI
// ... App struct, environment injection, lab view mounting
```

### Jetpack Compose
```kotlin
// shell/PreviewActivity.kt
import androidx.compose.material3.*
// ... Theme, NavHost, lab composable mounting
```

The shell should accept a `--lab` argument or environment variable to specify which lab to mount.

## Step 7: Generate Mount Script

Create `scripts/mount.{ext}` that handles:

1. **Lab resolution** — Given a lab name, find its directory and `preview.config.json`
2. **Config parsing** — Read the lab's preview config
3. **Dependency injection** — Load contract mocks, apply any mock overrides from the lab
4. **Component mounting** — Dynamically load and mount the lab's entry component
5. **Sub-lab support** — Handle nested paths like `parent-lab/labs/sub-lab-name`

```
// Resolution logic:
// 1. Check {labs-repo}/{lab-name}/preview.config.json
// 2. Check {labs-repo}/{parent}/labs/{sub-lab}/preview.config.json
// 3. Error if not found
```

## Step 8: Generate Provider Files

Create each provider file:

### Theme Provider
- Imports/references the project's theme
- Wraps children with theme context
- Provides design tokens (colors, typography, spacing)

### Navigation Provider
- Creates a minimal navigation container
- Reads routes from the lab's `preview.config.json`
- Provides navigation primitives (push, pop, replace)

### State Provider
- Sets up the project's state management with empty/default state
- Labs inject their own state as needed
- Provides the state context wrapper

## Step 9: Update Main Config

Add preview harness configuration to `.atlas/config.json`:

```json
{
  "preview": {
    "harness": ".atlas-harness",
    "autoMocks": true,
    "scripts": {
      "preview": "{{preview_command}}",
      "build": "{{build_command}}",
      "test": "{{test_command}}"
    }
  }
}
```

## Step 10: Summary

```
Preview Harness initialized!

Location: {labs-repo}/.atlas-harness/
Tech Stack: {{framework}} ({{language}}) on {{platform}}

Providers:
- Theme: {{theme_strategy}}
- Navigation: {{navigation_type}}
- State: {{state_management}}

Contract Mocks: {{N}} generated
- {{contract-1}}, {{contract-2}}, ...

To preview a lab:
  /atlas lab preview {lab-name}

Existing labs without preview config:
- {{lab-1}} — run /atlas lab preview {lab-1} to generate its preview config
- {{lab-2}} — run /atlas lab preview {lab-2} to generate its preview config
```

</Steps>

<Tool_Usage>
- Use `Read` to analyze project files for tech stack detection
- Use `Glob` to find framework config files (package.json, pubspec.yaml, etc.)
- Use `AskUserQuestion` for tech stack confirmation and provider selection
- Use `Bash` for creating directories
- Use `Write` for generating all harness files
- Use `Read` to read existing contracts for mock generation
</Tool_Usage>

<Validation>
After harness creation, verify:
- [ ] `.atlas-harness/` directory exists in the labs repo
- [ ] `config.json` has correct tech stack and provider settings
- [ ] `CLAUDE.md` exists with harness scope and rules
- [ ] `shell/` has the entry point file for the detected framework
- [ ] `providers/` has theme, navigation, and state files
- [ ] `mocks/` has one mock file per contract from main pipeline
- [ ] `scripts/mount.{ext}` handles lab resolution and mounting
- [ ] `.atlas/config.json` in main pipeline is updated with preview section
- [ ] Entry point can accept a lab name parameter
- [ ] Mock implementations match contract interfaces
</Validation>
