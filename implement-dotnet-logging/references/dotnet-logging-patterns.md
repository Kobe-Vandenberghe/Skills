# .NET Logging Patterns

Adapt these examples to the repository. Prefer `Microsoft.Extensions.Logging` abstractions even when Serilog, OpenTelemetry, Application Insights, or another provider is configured underneath.

## Contents

1. [Structured templates](#structured-templates)
2. [Exception ownership](#exception-ownership)
3. [Operation scopes](#operation-scopes)
4. [Background work and retries](#background-work-and-retries)
5. [Source-generated logging](#source-generated-logging)
6. [HTTP and dependency boundaries](#http-and-dependency-boundaries)
7. [Sensitive data](#sensitive-data)
8. [Common corrections](#common-corrections)

## Structured templates

Use a constant template and named properties:

```csharp
logger.LogInformation(
    "Action proposal {ActionProposalId} created for contribution {ContributionId} with {ChangeCount} changes",
    proposal.Id,
    contribution.Id,
    proposal.Changes.Count);
```

Avoid interpolation, which turns queryable properties into formatted text:

```csharp
// Avoid
logger.LogInformation(
    $"Action proposal {proposal.Id} created for {contribution.Id}");
```

Use consistent names and avoid dots in property names unless the configured provider explicitly supports them.

## Exception ownership

Log once where the failure becomes an owned operational outcome:

```csharp
try
{
    await processor.Process(contributionId, cancellationToken);
}
catch (OperationCanceledException) when (cancellationToken.IsCancellationRequested)
{
    logger.LogInformation(
        "Contribution processing {ContributionId} was cancelled",
        contributionId);
    throw;
}
catch (Exception exception)
{
    logger.LogError(
        exception,
        "Contribution processing {ContributionId} failed",
        contributionId);
    throw;
}
```

If centralized middleware logs every unhandled exception with request and trace context, do not add the generic catch above inside an API handler. Log there only if the handler adds a distinct outcome, such as persisting `ProcessingFailed`, applying a fallback, or exhausting retries.

Never discard the exception object:

```csharp
// Avoid: loses structured exception and stack information
logger.LogError("Processing failed: {Error}", exception.Message);
```

## Operation scopes

Add stable context once for a logical operation:

```csharp
using var scope = logger.BeginScope(new Dictionary<string, object?>
{
    ["ContributionId"] = contributionId,
    ["RepositoryId"] = repositoryId,
    ["Operation"] = "ProcessContribution"
});

logger.LogInformation("Contribution processing started");
await processor.Process(contributionId, cancellationToken);
logger.LogInformation("Contribution processing completed");
```

Prefer the active `Activity.TraceId` supplied by ASP.NET Core or OpenTelemetry. Only expose it explicitly when the sink does not already enrich logs:

```csharp
var traceId = Activity.Current?.TraceId.ToString();
```

Do not generate a new ID inside every layer.

## Background work and retries

Make attempts and the terminal outcome distinguishable:

```csharp
for (var attempt = 1; attempt <= maxAttempts; attempt++)
{
    try
    {
        await gateway.Push(repositoryId, cancellationToken);
        logger.LogInformation(
            "Repository synchronization {RepositoryId} completed on attempt {Attempt}",
            repositoryId,
            attempt);
        return;
    }
    catch (HttpRequestException exception) when (attempt < maxAttempts)
    {
        logger.LogWarning(
            exception,
            "Repository synchronization {RepositoryId} failed on attempt {Attempt}; retrying",
            repositoryId,
            attempt);
    }
}
```

The owner of retry exhaustion should emit one `Error` with total attempts and final context. When a resilience library already emits retry telemetry, enrich or configure it instead of duplicating every attempt.

## Source-generated logging

Use source generation for high-throughput paths or a stable event catalog. It is optional for normal application logs.

```csharp
internal static partial class KnowledgeLog
{
    [LoggerMessage(
        EventId = 2101,
        Level = LogLevel.Information,
        Message = "Knowledge repository {RepositoryId} synchronized at revision {Revision}")]
    public static partial void RepositorySynchronized(
        ILogger logger,
        Guid repositoryId,
        string revision);

    [LoggerMessage(
        EventId = 2102,
        Level = LogLevel.Error,
        Message = "Knowledge repository {RepositoryId} synchronization failed")]
    public static partial void RepositorySynchronizationFailed(
        ILogger logger,
        Guid repositoryId,
        Exception exception);
}
```

Group events by owning module or component. Keep event IDs unique only within the scheme the repository has chosen. Do not introduce arbitrary IDs into a codebase that does not use them operationally.

## HTTP and dependency boundaries

ASP.NET Core already produces request and exception telemetry when configured. Avoid repeating route, status, duration, and unhandled exception in every endpoint.

At a driven adapter, log provider context only when handled there:

```csharp
try
{
    return await githubClient.GetRepository(owner, name, cancellationToken);
}
catch (RateLimitExceededException exception)
{
    logger.LogWarning(
        exception,
        "GitHub rate limit prevented repository read for {RepositoryOwner}/{RepositoryName}; reset at {ResetAt}",
        owner,
        name,
        exception.ResetAt);

    throw new RepositoryTemporarilyUnavailableException(exception.ResetAt, exception);
}
```

Repository owner and name may themselves be sensitive in some systems. Follow the product's classification policy rather than assuming every identifier is safe.

## Sensitive data

Log an opaque identifier and a bounded classification, not the content:

```csharp
logger.LogWarning(
    "Contribution {ContributionId} was rejected with reason code {ReasonCode}",
    contribution.Id,
    rejection.Code);
```

Avoid:

```csharp
// Avoid: potentially stores private free text indefinitely
logger.LogWarning(
    "Contribution by {Email} was rejected: {ContributionContent}",
    user.Email,
    contribution.Content);
```

Also avoid logging JWTs, API keys, cookies, claims collections, connection strings, raw prompts, document bodies, and complete serialized DTOs.

## Common corrections

| Problem | Correction |
| --- | --- |
| `LogError(ex.Message)` | Pass `ex` as the exception argument and add stable context |
| Interpolated message | Use a constant template with named arguments |
| Same failure in every layer | Log once where handled; use traces for the call path |
| Validation logged as error | Return normal validation result; log only if operationally meaningful |
| Domain entity injects logger | Return a result or raise a domain event; log in application/adapter layer |
| Entire DTO serialized | Log safe IDs, counts, state, and reason codes |
| New correlation GUID per service | Propagate `Activity` trace context and use scopes |
| Success log for every row | Emit an aggregate completion log and metrics |
| Retry warning plus repeated outer errors | Log attempts consistently and one terminal error |
| Secret masked in one sink | Remove or redact before the logging call |
