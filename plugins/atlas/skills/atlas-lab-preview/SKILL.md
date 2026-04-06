---
name: atlas-lab-preview
description: Mount a lab or sub-lab into the shared preview harness and run it
argument-hint: "<lab-name>"
level: 3
---

<Purpose>
Mount a specific feature lab (or sub-lab) into the shared Preview Harness so its UI can be tested and previewed in isolation — without copying app scaffolding or duplicating UI code. Generates the lab's preview config if missing, wires up contract mocks, and launches the preview.
</Purpose>

<Use_When>
- User says "/atlas lab preview {name}", "preview lab", "test lab UI", "run lab"
- User wants to see how a lab's UI looks without merging
- User wants to run UI tests for a specific lab in isolation
- User wants to preview a sub-lab's UI
</Use_When>

<Do_Not_Use_When>
- Atlas is not initialized — prompt user to run `/atlas init` first
- Preview harness doesn't exist — prompt user to run `/atlas lab preview setup` first
- Lab doesn't exist — inform user and suggest `/atlas lab create`
- Lab has no UI components — inform user this lab has no UI to preview
</Do_Not_Use_When>

<Pre_Flight>
1. Verify `.atlas/config.json` exists and has `preview` section — if no preview section, tell user to run `/atlas lab preview setup`
2. Verify `.atlas-harness/` exists in the labs repo
3. Read `.atlas-harness/config.json` to understand the harness setup
4. Verify the target lab exists (check registry and filesystem)
5. Determine if target is a top-level lab or sub-lab
</Pre_Flight>

<Steps>

## Step 1: Locate Lab

Resolve the lab path:

1. **Top-level lab:** `{labs-repo}/{lab-name}/`
2. **Sub-lab:** `{labs-repo}/{parent-lab}/labs/{sub-lab-name}/`
3. **Deep sub-lab:** Follow the nesting: `{parent}/labs/{child}/labs/{grandchild}/`

If the user provides just a name, search:
1. Check direct match in `{labs-repo}/{name}/`
2. Search registry for matching lab name
3. If multiple matches (e.g., same sub-lab name in different parents), ask user to disambiguate

```
Found lab: {lab-name}
Path: {lab-path}
Status: {active|split}
Contracts: {list}
```

## Step 2: Scan Lab UI Components

Analyze the lab's `src/ui/` directory to identify:

1. **Entry component** — The main screen/widget/view that represents this feature
2. **Routes** — Any navigation routes this lab defines
3. **UI dependencies** — What the UI components need (contract interfaces, shared components)

```
Found UI components:
- Entry: {component_path} ({component_name})
- Screens: {count} screens
- Components: {count} presentational components
- Routes: {list of routes}
```

If `src/ui/` is empty or doesn't exist:
```
Lab "{lab-name}" has no UI components in src/ui/.
Nothing to preview. If this lab should have UI, create components in src/ui/ first.
```

## Step 3: Check or Generate Preview Config

Check if `{lab-path}/preview.config.json` exists:

### If exists:
Read and validate it. Ensure entry component still exists and routes are current.

### If missing:
Generate it based on the UI scan:

```json
{
  "version": "1.0.0",
  "preview": {
    "labName": "{lab-name}",
    "mountPoint": "screen",
    "entryComponent": "{MainScreen|MainWidget|MainView}",
    "entryPath": "src/ui/containers/{entry_file}"
  },
  "routes": [
    {
      "name": "{route-name}",
      "path": "/{route-path}",
      "component": "src/ui/{component_path}"
    }
  ],
  "dependencies": {
    "contracts": ["{contract-1}", "{contract-2}"],
    "mockOverrides": {}
  },
  "assets": {
    "include": [],
    "static": []
  }
}
```

Present to user for confirmation:
```
Generated preview config for "{lab-name}":
  Entry: {entry_component} at {entry_path}
  Routes: {route_list}
  Contract mocks: {contract_list}

Look correct? Any adjustments?
```

## Step 4: Verify Contract Mocks

For each contract this lab depends on:

1. Check if mock exists in `.atlas-harness/mocks/{contract}.mock.{ext}`
2. If mock is missing → generate it (same logic as preview setup Step 4)
3. If contract has been updated since mock was generated → regenerate mock
4. Apply any `mockOverrides` from the lab's `preview.config.json`

```
Contract mocks:
- {contract-1}: up to date
- {contract-2}: REGENERATED (contract updated)
- {contract-3}: NEW mock generated
```

## Step 5: Check Provider Compatibility

Verify the lab's UI is compatible with the harness providers:

1. **Theme** — Does the lab use theme tokens that exist in the harness theme?
2. **Navigation** — Does the lab's route structure work with the harness navigation?
3. **State** — Does the lab's state management match the harness state provider?

If incompatibilities found:
```
Compatibility issues:
- {issue-1}: {description} — {suggested fix}
- {issue-2}: {description} — {suggested fix}

Fix these before previewing, or proceed anyway?
```

## Step 6: Wire Up and Mount

Configure the harness to mount this specific lab:

1. Update the harness shell's lab pointer to target this lab's path
2. Register the lab's routes in the navigation provider
3. Inject the correct contract mocks (base + overrides)
4. Set up any lab-specific assets

**Implementation varies by tech stack:**

### React Native / Web React
- Update `metro.config.js` or webpack alias to resolve lab paths
- Set environment variable for lab entry
- Register lab's navigation screens

### Flutter
- Update `pubspec.yaml` path dependencies to include lab
- Set the lab widget as home in `MaterialApp`
- Register lab routes in `GoRouter` / `Navigator`

### SwiftUI
- Add lab directory to Xcode project references
- Set lab's root view as `ContentView`
- Inject environment objects

### Jetpack Compose
- Add lab module to Gradle settings
- Set lab composable as start destination
- Provide mock dependencies via Hilt/Koin

## Step 7: Launch Preview

Execute the preview:

1. Run the harness preview command from `.atlas-harness/config.json` → `scripts.preview`
2. Pass the lab name as an argument

```
Launching preview for "{lab-name}"...

Command: {preview_command} --lab {lab-name}
```

**If the preview command is not automated** (e.g., requires Xcode, Android Studio):
```
To preview "{lab-name}":

1. Open {ide} project at: {harness_project_path}
2. The lab's UI is mounted as the entry screen
3. Contract dependencies are mocked — see .atlas-harness/mocks/
4. Run the project normally

To switch to a different lab:
  /atlas lab preview {other-lab-name}
```

## Step 8: Summary

```
Preview ready for "{lab-name}"!

Mounted: {entry_component} from {entry_path}
Routes: {route_count} routes registered
Mocks: {mock_count} contract mocks active
  {mock_overrides_count} custom overrides applied

Preview: {running | ready to launch}
{command or IDE instructions}

Tips:
- Edit lab UI at: {lab-path}/src/ui/
- Override mocks: edit {lab-path}/preview.config.json → dependencies.mockOverrides
- Update harness: /atlas lab preview setup (reconfigure)
```

</Steps>

<Tool_Usage>
- Use `Read` to read lab manifest, contracts, preview config, and harness config
- Use `Glob` to scan lab's src/ui/ for components
- Use `AskUserQuestion` for preview config confirmation and disambiguation
- Use `Write` for generating preview.config.json and updating mocks
- Use `Bash` for running preview commands and checking file existence
- Use `Grep` to check for theme token usage and import patterns
</Tool_Usage>

<Validation>
After preview setup, verify:
- [ ] Lab has a valid `preview.config.json`
- [ ] Entry component specified in config exists at the declared path
- [ ] All contract mocks exist and are up to date in `.atlas-harness/mocks/`
- [ ] Mock overrides (if any) are valid and applied
- [ ] Harness shell is configured to mount this specific lab
- [ ] Navigation routes match the lab's route declarations
- [ ] No broken imports or missing dependencies
- [ ] Preview command runs without errors (or IDE instructions are clear)
</Validation>
