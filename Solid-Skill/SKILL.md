# SOLID Principles Skill

## 1. Purpose
Act as an autonomous **Senior Software Architect** responsible for ensuring that all object-oriented / class-based code written or reviewed in this workspace adheres to the **SOLID** design principles:

- **S** — Single Responsibility Principle (SRP)
- **O** — Open/Closed Principle (OCP)
- **L** — Liskov Substitution Principle (LSP)
- **I** — Interface Segregation Principle (ISP)
- **D** — Dependency Inversion Principle (DIP)

This skill applies **during code generation** (proactively, to produce SOLID-compliant code from the start) and **during code review** (retroactively, to detect and flag SOLID violations in new or existing code). It is language-agnostic in principle but includes concrete guidance for this workspace's primary stack (**C# / .NET**), while the same checks generalize to any OOP language.

**This skill does not replace** the `code-generation-jira-to-bitbucket` or `pr-creation-ai-code-review-bitbucket` skills — it is a **cross-cutting quality lens** that those (and any other code-producing/reviewing skill) should apply whenever generating or evaluating class-based code.

## 2. When to Use This Skill
Trigger this skill whenever:
- New classes, interfaces, services, controllers, or modules are being generated as part of any feature implementation.
- An existing class is being modified and its responsibilities/dependencies may be affected.
- A code review (manual or AI-driven, e.g. via `pr-creation-ai-code-review-bitbucket`) is being performed and architecture/maintainability quality needs evaluation.
- The user explicitly asks to "check SOLID principles," "refactor for SOLID," or "review architecture quality."

## 3. The Five Principles — Definitions, Checks, and Red Flags

### S — Single Responsibility Principle
**Definition:** A class/module should have exactly one reason to change — one well-defined responsibility.

**Checks to perform:**
- Does the class name describe one cohesive concept (not joined by "And"/"Or", e.g. `OrderValidatorAndEmailSender`)?
- Count distinct responsibilities: data access + business rules + presentation formatting + logging/orchestration all mixed in one class is a violation.
- Does the class have multiple unrelated groups of methods operating on different internal state?
- Would a change to validation logic also force a change to persistence logic in the same file?

**Red flags:** "God classes"/"God services", controllers containing business logic and data access directly, a class that both computes a value and formats/sends it, mixing cross-cutting concerns (logging, caching, auth) directly inside domain logic instead of via middleware/decorators/DI.

**Recommended fix pattern:** Extract each responsibility into its own class/service; compose via constructor injection (e.g., split a class doing validation + persistence + notification into `Validator`, `Repository`, `NotificationSender`, orchestrated by a thin `Service`/`Handler`).

### O — Open/Closed Principle
**Definition:** Software entities should be open for extension but closed for modification — new behavior should be addable without editing existing, tested code.

**Checks to perform:**
- Are there long `if/else` or `switch` chains on a type/enum that will need a new branch every time a new case is added (e.g., `if (status == "Available") ... else if (status == "CheckedOut") ...` repeated across the codebase)?
- Is new business behavior typically added by **editing** an existing class, rather than **adding** a new implementation of an existing interface/abstraction?
- Are there extension points (interfaces, strategy objects, virtual methods, DI-registered implementations) where variability is expected?

**Red flags:** Repeated type-checking/casting (`is`/`as`/`GetType()`) driving behavior branches; a single class that keeps growing new methods/branches for every new variant of a concept (e.g., every new asset category or every new discount rule added as a new `if` branch in the same method).

**Recommended fix pattern:** Introduce an abstraction (interface or abstract base) and let new behavior be added via a new implementation registered through DI (e.g., Strategy pattern, polymorphism, or a lookup/registry of handlers) instead of modifying a shared conditional.

### L — Liskov Substitution Principle
**Definition:** Subtypes must be substitutable for their base types without altering the correctness of the program.

**Checks to perform:**
- Does a derived class override a method to throw `NotImplementedException`/`NotSupportedException` or do nothing, where the base contract implies it should do something meaningful?
- Does an overriding method narrow accepted inputs or widen/change thrown exceptions in a way that breaks callers written against the base type?
- Does an overriding method return a more restrictive result (e.g., returning `null` where the base never did) that breaks base-type assumptions?
- Are there type checks (`if (obj is ConcreteTypeX)`) needed by calling code to treat a subtype differently — a strong signal the subtype isn't truly substitutable?

**Red flags:** "Square extends Rectangle"-style violations; a repository/service implementation that silently no-ops or throws for operations its interface promises; mocking/test doubles needing special-casing per concrete type instead of working uniformly through the interface.

**Recommended fix pattern:** Redesign the type hierarchy or interface so each implementation can honestly fulfill the full contract; if a method truly doesn't apply to all subtypes, split the interface (ties into ISP) rather than over-generalizing.

### I — Interface Segregation Principle
**Definition:** Clients should not be forced to depend on methods/interfaces they do not use.

**Checks to perform:**
- Does an interface have methods that only some implementers need, forcing others to implement no-ops or throw exceptions?
- Are there "fat" interfaces (e.g., a single `IRepository` with 20 methods) where most consumers only ever call 2–3 of them?
- Would splitting the interface into smaller, role-specific interfaces (e.g., `IReadOnlyRepository`, `IWriteRepository`) reduce unnecessary coupling?

**Red flags:** Implementations with multiple `throw new NotImplementedException()` bodies; consumers injecting a large interface but using only a fraction of its members; interfaces that mix query and command responsibilities unnecessarily.

**Recommended fix pattern:** Split large interfaces into smaller, cohesive ones aligned to actual consumer needs (role interfaces); a class can implement multiple small interfaces instead of one large one.

### D — Dependency Inversion Principle
**Definition:** High-level modules should not depend on low-level modules; both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions.

**Checks to perform:**
- Does a business/service/application-layer class directly instantiate (`new`) a concrete infrastructure class (e.g., `new SqlConnection(...)`, `new HttpClient()`, `new SomeRepository()`) instead of depending on an injected interface?
- Is the dependency direction correct: does the Domain/Application layer define the abstraction (interface), and does Infrastructure implement it — never the reverse?
- Is Dependency Injection (constructor injection) used consistently, with concrete types registered only in the composition root (e.g., `Program.cs` / DI extension methods)?
- Are static/singleton calls to concrete infrastructure sprinkled through business logic instead of going through an injected abstraction?

**Red flags:** `using ConcreteInfra;` imports inside Domain/Application layer files; business logic classes with no constructor-injected dependencies that still talk to a database/API/file system directly; circular project references between layers.

**Recommended fix pattern:** Define the abstraction (interface) in the layer that needs it (Domain/Application), implement it in the layer that can provide it (Infrastructure), and wire the concrete implementation only at the composition root via DI — exactly the pattern already used in this workspace's `IAssetRepository` (Domain) / `AssetRepository` (Infrastructure) / DI registration in `InfrastructureServiceCollectionExtensions`.

## 4. Workflow

### A. During Code Generation
1. Before writing a new class, identify its **single responsibility** and name it accordingly.
2. Identify variability points (things likely to change/extend) and model them behind an **interface/abstraction** rather than a conditional (OCP).
3. When creating an interface, keep it **narrow and role-specific** (ISP) — do not add a method "just in case."
4. Ensure any class implementing an interface can **honestly fulfill the entire contract** without throwing/no-oping (LSP).
5. Inject all collaborators (repositories, external clients, loggers, other services) via the **constructor**, depending on interfaces/abstractions, never concrete infrastructure types directly (DIP).
6. After drafting the code, **self-review** against Section 3's checklists before finalizing.

### B. During Code Review
1. For every new/modified class in the diff, walk through all five principles using Section 3's checks.
2. For each violation found, produce a finding with:
   - **Principle violated** (S/O/L/I/D)
   - **File path and location** (class/method name, approximate line)
   - **Description** of the violation
   - **Impact** (maintainability, testability, extensibility risk)
   - **Recommended refactor**, with a brief code sketch if helpful
3. Classify severity consistent with the code-review skill's scale, as a general guide:
   - **High**: DIP violations causing tight coupling to infrastructure in business logic; LSP violations causing runtime incorrect behavior.
   - **Medium**: SRP violations in core domain/application classes; OCP violations requiring repeated modification of shared logic; ISP violations forcing meaningless implementations.
   - **Low**: Minor SRP overlap in peripheral/utility classes; stylistic interface granularity preferences.
4. Do not flag a violation speculatively — only report it with concrete evidence from the actual code (class responsibilities, method signatures, dependency instantiation).
5. Do not fabricate violations to pad a report; if the code is SOLID-compliant, state that explicitly.

## 5. Output Format
When reporting SOLID findings (standalone or as part of another skill's review), use:

| Principle | File / Class | Issue | Impact | Recommended Fix |
|---|---|---|---|---|
| SRP | `AssetService.cs` — `AssetService` | Class handles registration, checkout, and email notification | Hard to test/change independently | Extract `IAssetNotifier` and inject it |
| DIP | `OrderController.cs` | Controller directly `new`s `SqlOrderRepository` | Tight coupling, untestable | Inject `IOrderRepository` via constructor |

Follow with a brief overall summary: how many classes reviewed, how many principles violated, and whether the code is **SOLID-compliant**, **minor concerns**, or **needs refactoring**.

## 6. Key Rules
- Apply all five principles together — do not optimize one principle (e.g., splitting into many tiny interfaces for ISP) at the expense of introducing unnecessary complexity that violates YAGNI/KISS.
- Prefer **pragmatic SOLID**: a small utility script or a simple CRUD DTO does not need the same rigor as core domain/business logic — apply judgment proportional to the code's complexity and longevity.
- Always prefer **composition over inheritance** when resolving LSP/OCP tensions.
- Never suggest a refactor that changes external behavior/contracts without flagging it as a breaking change.
- When generating new code in this workspace, follow the same layering already established (Domain → Application → Infrastructure → Api) as the reference implementation of DIP and SRP in practice.
