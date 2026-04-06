# Contract: {{CONTRACT_NAME}}

## Purpose

{{Brief description of what this contract defines and why it exists.}}

## Consumers

{{Which features/modules must implement this contract.}}

- All features (cross-cutting)
OR
- Specific features: {{list}}

## Interface

{{Language-agnostic interface definition. Use code blocks for type signatures.}}

```
interface {{ContractName}} {
  // Define methods, properties, events that implementors must provide
}
```

## Behavior

{{Describe the expected behavior when this contract is implemented.}}

1. {{Behavior 1}}
2. {{Behavior 2}}
3. {{Behavior 3}}

## Integration Points

{{Where and how does this contract connect to the main pipeline?}}

- **Registration:** {{How does a feature register its implementation?}}
- **Invocation:** {{How/when is the implementation called?}}
- **Data flow:** {{What data goes in and comes out?}}

## Constraints

{{Any non-functional requirements or limitations.}}

- {{Constraint 1}}
- {{Constraint 2}}

## Examples

{{Optional: provide a concrete example of correct implementation.}}

```
// Example implementation
```

## Validation

{{How to verify a feature correctly implements this contract.}}

- [ ] {{Check 1}}
- [ ] {{Check 2}}
- [ ] {{Check 3}}
