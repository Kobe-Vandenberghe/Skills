---
name: write-dotnet-tests
description: Write, organize, repair, and review focused C#/.NET tests that follow the repository's architecture, mirror production project and folder structure, and exercise behavior at the correct boundary. Use when adding tests for a feature or module, creating a regression test for a defect, reorganizing a .NET test suite, reviewing test quality, or deciding between domain unit, application use-case, adapter integration, API integration, and end-to-end tests. Preserve the repository's established test framework and conventions; do not use merely because production code is being discussed when no test work is requested or needed.
---

# Write .NET Tests

Create the smallest set of tests that gives strong confidence in observable behavior. Treat tests as part of the architecture: locate them beside the corresponding logical module and layer, and preserve the same internal structure as the production code.

Read [references/test-structure-and-patterns.md](references/test-structure-and-patterns.md) before creating a new test project, reorganizing tests, or writing C# test code in a repository without a clear established example.

## Operating rules

- Follow repository instructions and existing test conventions before applying defaults from this skill.
- Test behavior through public boundaries. Do not test private methods, reproduce implementation steps in assertions, or expose internals solely for tests.
- Select the lowest test level that can catch the relevant failure with realistic confidence.
- Keep architecture, module ownership, project boundaries, folder paths, and namespaces visible in the test layout.
- Keep tests deterministic, isolated, readable, and safe to run repeatedly or in parallel unless the suite explicitly documents otherwise.
- Ask a focused question when uncertainty about the expected behavior, external dependency, or test environment would materially change the test. Do not invent business expectations.
- Do not add a new test framework, mocking library, assertion library, fixture framework, or container dependency when the repository already has a suitable convention.

## Workflow

### 1. Inspect the test architecture

Read only enough to establish the local pattern:

- `AGENTS.md` and repository documentation
- solution, project, central package, and build configuration files
- production project references and module boundaries
- existing test projects, folders, namespaces, fixtures, and naming conventions
- CI commands and test categories when present
- one representative production flow and its closest tests

Identify the production project and logical path being tested. Do not create tests in a generic catch-all project when the repository already separates modules or layers.

### 2. Choose the test boundary

Classify each behavior before writing a test:

| Behavior | Preferred test |
| --- | --- |
| Value object, entity, aggregate, or pure domain policy | Domain unit test |
| Application use case coordinating domain behavior and ports | Application use-case test |
| EF Core mapping, repository, database query, broker, file system, or external client | Adapter integration test |
| Routing, serialization, authentication, authorization, status code, or transport mapping | API or driving-adapter integration test |
| Critical journey across deployed boundaries | End-to-end test, used sparingly |

Avoid proving the same rule at every level. Test a domain invariant thoroughly in the domain project, then use higher-level tests only to prove wiring, translation, persistence, or collaboration that the lower test cannot cover.

### 3. Mirror production structure

Map each production project to a corresponding test project under the same logical module. Mirror all meaningful folders beneath the project root.

For example:

```text
src/Modules/Payments/Payments.Application/UseCases/CapturePayment/CapturePayment.cs
tests/Modules/Payments/Payments.Application.Tests/UseCases/CapturePayment/CapturePaymentTests.cs
```

Apply these defaults when the repository has no stronger convention:

- `<Project>.Tests` for fast unit or use-case tests
- `<Project>.IntegrationTests` for tests that use real infrastructure
- `<TypeOrBehavior>Tests.cs` for the corresponding production type or behavior
- a test namespace rooted in the test project name, followed by the mirrored source folders

Do not flatten tests by generic categories such as `UnitTests/`, `Mocks/`, or `Services/` when doing so hides the production module and behavior. Keep test-only builders, fakes, fixtures, and data close to the feature using them. Promote helpers into shared test building blocks only after multiple test projects have a stable, genuine need.

### 4. Define scenarios from behavior

Derive tests from observable examples:

- successful behavior and resulting state
- rejected inputs or illegal state transitions
- boundary values and meaningful equivalence classes
- idempotency, concurrency, or retry behavior when required
- persisted representation or external contract where relevant
- regression scenario that previously failed

Use Arrange, Act, Assert or Given, When, Then consistently with the repository. Name tests as readable behavior statements. If no convention exists, prefer `Operation_Scenario_ExpectedOutcome`.

Assert all parts of one observable outcome when they belong together. Do not split a coherent state transition into many tiny tests merely to enforce one assertion per test.

### 5. Use test doubles deliberately

- Do not mock entities, value objects, aggregates, or pure domain services.
- For application tests, invoke the use case through its public driving boundary and replace driven ports with small fakes or stubs.
- Use mocks only when an interaction is itself part of the contract, such as publishing once after a successful commit or not calling a gateway after rejection.
- Avoid asserting incidental call order, exact internal method counts, or implementation-specific query shapes.
- Prefer a real infrastructure dependency for behavior owned by that dependency. Do not use EF Core's in-memory provider to claim confidence in relational constraints, SQL translation, transactions, or provider-specific behavior.
- Keep remote systems out of normal test runs. Use a controlled fake server, sandbox, or explicit contract suite according to repository convention.

### 6. Preserve domain semantics

For domain-rich code, cover the semantics the model promises:

- strongly typed ID parsing, serialization, and persistence conversion at boundaries
- entity equality based on identity
- value-object equality, normalization, validation, and replacement semantics
- valid construction and rejected invalid values
- aggregate invariants, ownership, and encapsulated child mutation
- legal and illegal state transitions, including absence of partial mutation on failure
- domain events as facts emitted only after successful behavior

Construct test data through production factories and behavior methods. Do not bypass invariants with reflection, public setters added for tests, or invalid object mothers.

### 7. Implement and verify incrementally

For a defect, write a regression test that fails for the reported reason before fixing production code when practical. For new behavior, write the smallest representative scenarios first.

Run tests in increasing scope:

1. the new or changed test
2. the containing test project
3. related module tests
4. the full solution when cost and repository practice justify it

Use the repository's established commands. Investigate failures rather than weakening assertions, adding arbitrary delays, disabling parallelism globally, or retrying flaky tests without finding the cause.

## Review checklist

Before finishing, verify:

- every test lives in the test project for the production layer it exercises
- folders and namespaces mirror the corresponding production path
- the selected test level matches the failure mode
- assertions describe externally observable behavior
- domain rules are not duplicated across multiple layers without a reason
- clocks, IDs, randomness, culture, and ordering are controlled where relevant
- test data uses valid production construction paths
- infrastructure tests exercise the real semantics they claim to verify
- tests can run independently and leave no persistent state behind
- focused and relevant broader test runs pass

Report which behaviors were covered, why each test level was chosen, the commands run, and any remaining untested risk. Do not present coverage percentage alone as evidence of quality.
