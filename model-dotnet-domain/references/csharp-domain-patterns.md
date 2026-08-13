# C# Domain Modeling Patterns

Use these patterns selectively. Names such as `Result`, `Error`, and `IDomainEvent` represent the host repository's equivalents; do not introduce a second competing abstraction.

## Contents

1. [Strongly typed aggregate ID](#strongly-typed-aggregate-id)
2. [Value object factory](#value-object-factory)
3. [Entity equality](#entity-equality)
4. [Aggregate root and domain events](#aggregate-root-and-domain-events)
5. [Encapsulated collections and transitions](#encapsulated-collections-and-transitions)
6. [Domain service placement](#domain-service-placement)
7. [Commands, results, and events](#commands-results-and-events)
8. [Persistence considerations](#persistence-considerations)

## Strongly typed aggregate ID

Use a distinct type per aggregate. Keep creation explicit and prevent public construction from known-invalid values.

```csharp
public readonly record struct BookingId
{
    public Guid Value { get; }

    private BookingId(Guid value) => Value = value;

    public static BookingId New() => new(Guid.NewGuid());

    public static BookingId From(Guid value) =>
        value != Guid.Empty
            ? new BookingId(value)
            : throw new ArgumentException("Booking ID cannot be empty.", nameof(value));

    public override string ToString() => Value.ToString();
}
```

Every struct still has `default`, so validate IDs when binding routes, deserializing, or reconstructing persisted data. Use a sealed reference type instead when an unrepresentable default is critical. Do not add implicit conversion between unrelated ID types.

## Value object factory

Use structural equality and create a normalized, valid value in one place.

```csharp
public sealed record EmailAddress
{
    public string Value { get; }

    private EmailAddress(string value) => Value = value;

    public static Result<EmailAddress> Create(string? input)
    {
        var normalized = input?.Trim().ToLowerInvariant();

        if (string.IsNullOrWhiteSpace(normalized))
            return Result.Failure<EmailAddress>("Email is required.");

        if (!System.Net.Mail.MailAddress.TryCreate(normalized, out _))
            return Result.Failure<EmailAddress>("Email is invalid.");

        return Result.Success(new EmailAddress(normalized));
    }

    public override string ToString() => Value;
}
```

Keep only context-free rules here. Uniqueness, deliverability, and ownership require external state and belong outside this value object.

For multi-component values, a `readonly record struct` is useful when value semantics, small size, and default-value handling are acceptable:

```csharp
public readonly record struct Money(decimal Amount, Currency Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Currencies must match.");

        return this with { Amount = Amount + other.Amount };
    }
}
```

Use a factory if negative amounts or unsupported currencies are invalid for the specific concept.

## Entity equality

Compare entities by stable identity and concrete type, never by mutable attributes.

```csharp
public abstract class Entity<TId> : IEquatable<Entity<TId>>
    where TId : notnull
{
    protected Entity(TId id) => Id = id;

    public TId Id { get; }

    public bool Equals(Entity<TId>? other) =>
        other is not null &&
        other.GetType() == GetType() &&
        EqualityComparer<TId>.Default.Equals(Id, other.Id);

    public override bool Equals(object? obj) =>
        obj is Entity<TId> other && Equals(other);

    public override int GetHashCode() => HashCode.Combine(GetType(), Id);
}
```

Prefer assigning IDs when entities are created. If the ORM assigns IDs later, define and test transient-entity equality explicitly instead of allowing two default IDs to compare equal.

## Aggregate root and domain events

Keep domain-event collection mechanics in the aggregate base only if the codebase actually uses domain events.

```csharp
public interface IDomainEvent { }

public abstract class AggregateRoot<TId> : Entity<TId>
    where TId : notnull
{
    private readonly List<IDomainEvent> _domainEvents = [];

    protected AggregateRoot(TId id) : base(id) { }

    public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();

    protected void Raise(IDomainEvent domainEvent) =>
        _domainEvents.Add(domainEvent);

    public void ClearDomainEvents() => _domainEvents.Clear();
}
```

Do not add this base class when plain entities are sufficient. Event dispatch and clearing are application or infrastructure responsibilities.

## Encapsulated collections and transitions

Expose state for observation and domain verbs for mutation.

```csharp
public sealed class Booking : AggregateRoot<BookingId>
{
    private readonly List<Passenger> _passengers = [];

    private Booking(BookingId id, CustomerId customerId)
        : base(id)
    {
        CustomerId = customerId;
        Status = BookingStatus.Draft;
    }

    public CustomerId CustomerId { get; }
    public BookingStatus Status { get; private set; }
    public IReadOnlyList<Passenger> Passengers => _passengers.AsReadOnly();

    public static Booking Create(CustomerId customerId) =>
        new(BookingId.New(), customerId);

    public Result AddPassenger(Passenger passenger)
    {
        if (Status != BookingStatus.Draft)
            return Result.Failure("Passengers can only be added to draft bookings.");

        if (_passengers.Any(x => x.TravelerId == passenger.TravelerId))
            return Result.Failure("Traveler is already part of this booking.");

        _passengers.Add(passenger);
        return Result.Success();
    }

    public Result Confirm(DateTimeOffset occurredAt)
    {
        if (Status != BookingStatus.Draft)
            return Result.Failure("Only draft bookings can be confirmed.");

        if (_passengers.Count == 0)
            return Result.Failure("A booking requires at least one passenger.");

        Status = BookingStatus.Confirmed;
        Raise(new BookingConfirmed(Id, occurredAt));
        return Result.Success();
    }
}
```

Keep setters private. An ORM-only constructor may be private and empty if the persistence framework needs it, but it must not become a second public creation path.

## Domain service placement

Use a domain service when a stateless domain policy requires multiple concepts and no entity naturally owns the decision.

```csharp
public sealed class RebookingPolicy
{
    public Result<Money> CalculateFee(
        Booking booking,
        FareRules fareRules,
        DateTimeOffset requestedAt)
    {
        if (!booking.CanBeRebookedAt(requestedAt))
            return Result.Failure<Money>("Booking can no longer be changed.");

        return Result.Success(fareRules.CalculateRebookingFee(booking));
    }
}
```

Loading the booking, reading fare rules from a remote system, opening transactions, and saving changes belong in an application handler. If the behavior only needs `Booking` state, put it on `Booking` instead.

## Commands, results, and events

Keep intent, outcome, and fact separate.

```csharp
public sealed record ConfirmBookingCommand(BookingId BookingId);

public sealed record BookingConfirmed(
    BookingId BookingId,
    DateTimeOffset OccurredAt) : IDomainEvent;

public sealed class ConfirmBookingHandler(
    IBookingStore bookings,
    IClock clock,
    IUnitOfWork unitOfWork)
{
    public async Task<Result> Handle(
        ConfirmBookingCommand command,
        CancellationToken cancellationToken)
    {
        var booking = await bookings.Get(command.BookingId, cancellationToken);
        if (booking is null)
            return Result.NotFound("Booking was not found.");

        var result = booking.Confirm(clock.UtcNow);
        if (result.IsFailure)
            return result;

        await unitOfWork.Commit(cancellationToken);
        return Result.Success();
    }
}
```

The command may fail; `BookingConfirmed` states a fact and exists only after `Confirm` succeeds. Convert it to a versioned integration event after commit when another module or system needs it.

## Persistence considerations

- Configure typed-ID value converters centrally and test round trips.
- Keep persistence constructors and backing fields from weakening public invariants.
- Use optimistic concurrency tokens for state transitions vulnerable to lost updates.
- Avoid lazy-loading behavior that lets aggregate methods silently fetch external state.
- Store value objects as owned/complex types or converted scalars according to query needs.
- Rehydrate aggregates without replaying creation behavior or raising new events.
- Keep EF configuration and migrations outside the domain project when the architecture separates infrastructure.
