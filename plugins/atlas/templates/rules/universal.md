# Universal Coding Rules

These rules ALWAYS apply when writing code in any Atlas project. They are non-negotiable defaults that ensure code quality, maintainability, and clean architecture.

---

## 1. Clean Code

- **Meaningful names** — Variables, functions, and classes must have descriptive names that reveal intent. No abbreviations unless universally understood (e.g., `id`, `url`).
- **Small functions** — Each function does ONE thing. If you need to describe what a function does with "and", split it.
- **No magic numbers/strings** — Extract constants with meaningful names.
- **No dead code** — Remove unused variables, functions, imports, and commented-out code.
- **Consistent formatting** — Follow the project's existing style. When no style exists, follow the language's community standard.

## 2. Clean Architecture — Separation of Concerns

- **Business logic and UI are STRICTLY separated.** Never put business logic in UI components. Never put UI concerns in business logic.
- **Layer structure:**
  - **UI Layer** — Components, views, templates. Only handles rendering and user interaction.
  - **Business Logic Layer** — Services, use cases, domain models. Contains ALL business rules.
  - **Data Layer** — Repositories, API clients, database access. Handles data persistence and retrieval.
- **Dependencies point inward** — UI depends on Business Logic. Business Logic depends on Data interfaces (not implementations). Data implements interfaces defined by Business Logic.

## 3. SOLID Principles

- **Single Responsibility** — A class/module has one reason to change.
- **Open/Closed** — Open for extension, closed for modification. Use interfaces and composition.
- **Liskov Substitution** — Subtypes must be substitutable for their base types.
- **Interface Segregation** — Many small, specific interfaces over one large general interface.
- **Dependency Inversion** — Depend on abstractions, not concretions. Inject dependencies.

## 4. Loose Coupling

- **Modules communicate through interfaces/contracts**, not direct implementation references.
- **No circular dependencies** — If A depends on B, B must NOT depend on A.
- **Event-driven where appropriate** — Use events/callbacks for cross-cutting concerns instead of direct calls.
- **Dependency injection** — Pass dependencies in, don't create them internally.

## 5. DRY (Don't Repeat Yourself) — With Judgment

- **Shared logic goes in shared modules.** If the same logic exists in 3+ places, extract it.
- **But don't over-abstract.** Two similar-looking pieces of code that change for different reasons should remain separate. Duplication is better than wrong abstraction.

## 6. Error Handling

- **Handle errors at the appropriate level.** Don't catch errors you can't handle meaningfully.
- **Fail fast** — Validate inputs early. Return/throw on invalid state immediately.
- **No silent failures** — Every error must be either handled or propagated. Never swallow exceptions.
- **User-facing errors are friendly.** Internal errors are detailed for debugging.

## 7. Testing

- **Business logic must be testable in isolation** — no UI, no database, no network required.
- **Test behavior, not implementation** — Tests should survive refactoring.
- **Each feature lab must include tests** that verify contract compliance.

## 8. Documentation

- **Code should be self-documenting** through good naming and structure.
- **Comments explain WHY, not WHAT** — If you need to explain what the code does, the code is too complex.
- **Contracts are the documentation** — Interface contracts serve as the primary API documentation.

## 9. Security

- **Validate all external inputs** — User input, API responses, file contents.
- **No secrets in code** — Use environment variables or secret managers.
- **Principle of least privilege** — Request only the permissions you need.

## 10. Performance

- **Don't optimize prematurely** — Write clean code first, optimize when measured.
- **Know your data structures** — Choose appropriate data structures for the use case.
- **Async where appropriate** — Don't block on I/O operations.
