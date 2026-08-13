# .NET Test Structure and Patterns

Use these defaults only when the repository has no established alternative.

## Contents

- [Architecture-aligned layout](#architecture-aligned-layout)
- [Domain test shape](#domain-test-shape)
- [Application use-case test shape](#application-use-case-test-shape)
- [Infrastructure integration shape](#infrastructure-integration-shape)
- [API integration shape](#api-integration-shape)
- [Test data and deterministic inputs](#test-data-and-deterministic-inputs)
- [Contract suites](#contract-suites)

## Architecture-aligned layout

Mirror both the production project and the folders beneath it, eg:

```text
src/
└── Modules/
    └── Payments/
        ├── Payments.Domain/
        │   ├── Payments/
        │   │   ├── Payment.cs
        │   │   └── PaymentId.cs
        │   └── ValueObjects/
        │       └── Money.cs
        ├── Payments.Application/
        │   └── UseCases/
        │       └── CapturePayment/
        │           └── CapturePayment.cs
        └── Payments.Infrastructure/
            └── Persistence/
                ├── PaymentConfiguration.cs
                └── PaymentRepository.cs

tests/
└── Modules/
    └── Payments/
        ├── Payments.Domain.Tests/
        │   ├── Payments/
        │   │   ├── PaymentTests.cs
        │   │   └── PaymentIdTests.cs
        │   └── ValueObjects/
        │       └── MoneyTests.cs
        ├── Payments.Application.Tests/
        │   └── UseCases/
        │       └── CapturePayment/
        │           └── CapturePaymentTests.cs
        └── Payments.Infrastructure.IntegrationTests/
            └── Persistence/
                ├── PaymentConfigurationTests.cs
                └── PaymentRepositoryTests.cs
```

Recommended project references preserve the architectural direction:

```text
Payments.Domain.Tests
└── Payments.Domain

Payments.Application.Tests
├── Payments.Application
└── Payments.Domain

Payments.Infrastructure.IntegrationTests
├── Payments.Infrastructure
├── Payments.Application
└── Payments.Domain
```

Do not reference Infrastructure from Domain or Domain tests. Keep shared test utilities in a dedicated test building-block project only when more than one test project genuinely reuses them.

## Domain test shape

Test a domain object directly without mocks. Prefer meaningful state and result assertions:

```csharp
public sealed class PaymentTests
{
    [Fact]
    public void Capture_WhenAuthorized_MarksPaymentAsCaptured()
    {
        var payment = Payment.Create(
            PaymentId.New(),
            Money.Euros(125));
        payment.Authorize();

        var result = payment.Capture();

        Assert.True(result.IsSuccess);
        Assert.Equal(PaymentStatus.Captured, payment.Status);
        Assert.Contains(
            payment.DomainEvents,
            domainEvent => domainEvent is PaymentCaptured);
    }

    [Fact]
    public void Capture_WhenAlreadyRejected_DoesNotChangeState()
    {
        var payment = Payment.Create(
            PaymentId.New(),
            Money.Euros(125));
        payment.Reject();

        var result = payment.Capture();

        Assert.True(result.IsFailure);
        Assert.Equal(PaymentStatus.Rejected, payment.Status);
        Assert.DoesNotContain(
            payment.DomainEvents,
            domainEvent => domainEvent is PaymentCaptured);
    }
}
```

Adapt attributes and assertion syntax to the repository's test framework. Do not introduce xUnit merely because the example uses it.

## Application use-case test shape

Treat the test as a driving adapter. Invoke the use case and supply controlled implementations of its driven ports:

```csharp
public sealed class CapturePaymentTests
{
    [Fact]
    public async Task Execute_WhenPaymentIsAuthorized_PersistsCapturedPayment()
    {
        var payment = PaymentTestData.Authorized();
        var payments = new InMemoryPaymentStore(payment);
        var useCase = new CapturePayment(payments);

        var result = await useCase.Execute(
            new CapturePaymentCommand(payment.Id),
            CancellationToken.None);

        Assert.True(result.IsSuccess);
        Assert.Equal(PaymentStatus.Captured, payments.Saved.Status);
    }
}
```

Keep `InMemoryPaymentStore` next to this use case if it is local test support. Promote it only when its behavior is stable and several test projects use the same port contract.

## Infrastructure integration shape

Use the actual database engine when testing:

- EF Core mappings and value converters
- indexes, unique constraints, foreign keys, and concurrency tokens
- SQL translation, full-text search, ordering, or collation
- transaction and rollback behavior
- repository save and rehydration semantics

Isolate test data with a fresh database, schema, transaction, or reliable reset strategy consistent with parallel execution. Keep container and fixture lifecycle at the test-project boundary instead of hiding it in domain test helpers.

## API integration shape

Use the application's supported in-process host when available. Replace only external driven adapters. Verify concerns owned by the transport boundary:

- route and method
- authentication and authorization
- request and response serialization
- validation-to-response mapping
- status codes and headers
- dependency wiring

Do not repeat every domain invariant through HTTP. One representative API rejection can prove mapping while domain tests cover the rule matrix.

## Test data and deterministic inputs

- Prefer builders with valid defaults and explicit overrides.
- Name scenario-specific values; avoid unexplained GUIDs and numbers.
- Use a fake or repository-standard `TimeProvider` for time-dependent behavior.
- Supply fixed randomness, IDs, culture, and ordering when they affect results.
- Avoid `Task.Delay`, wall-clock waiting, and global mutable fixtures.
- Do not create invalid aggregates by bypassing production construction paths.

## Contract suites

When multiple adapters implement the same important port, define reusable behavioral contract tests and run them against each adapter. Keep adapter-specific tests for provider behavior outside the shared contract. Do not create a contract suite for a single trivial implementation.
