---
name: implement-dotnet-logging
description: Design, add, repair, or review production logging in C#/.NET using structured `Microsoft.Extensions.Logging` conventions. Use when instrumenting an application flow, integration, background job, or failure path; reviewing log quality and levels; diagnosing missing context or duplicate exception logs; adding correlation and scopes; or preventing secrets and personal data from reaching logs. Place logs at meaningful application and adapter boundaries, preserve module ownership, and keep domain models logging-free. Do not use merely because code is being changed when no logging work or operational visibility is needed.
---

# Implement .NET Logging

Create logs that help an operator answer what happened, where, to which operation, and what to do next. Prefer a few stable, structured events over narration of every method call.

Read [references/dotnet-logging-patterns.md](references/dotnet-logging-patterns.md) before writing or reviewing logging code. Follow repository conventions and the configured logging stack when they differ in syntax without weakening the rules below.

## Inspect before adding logs

Inspect:

- logging abstractions and providers already in use
- middleware, exception handlers, tracing, request logging, and correlation setup
- nearby message templates, event IDs, scopes, and naming conventions
- configuration and per-category minimum levels
- privacy, retention, and redaction rules
- one representative flow from its driving adapter to its driven adapter

Do not add a second logging framework, custom wrapper, correlation mechanism, or logging vocabulary when the codebase already has one.

## Decide what deserves a log

Log meaningful operational events:

- start and completion of durable background work when progress cannot be inferred elsewhere
- important business or workflow transitions that operators must investigate
- calls across external boundaries when latency, failure, or retry state matters
- degraded behavior, retries, fallbacks, skipped work, and partial results
- unexpected failures at the boundary that owns handling or reporting them
- security-relevant outcomes when policy permits them, without exposing sensitive values

Usually do not log:

- method entry and exit
- every successful CRUD call or request already covered by framework request logging
- values that can be derived cheaply from traces, metrics, or existing logs
- expected validation failures at warning or error level
- exceptions that will be logged unchanged by an outer boundary
- domain entity internals merely because state changed

Every new event should have a plausible consumer or investigation question. If it has none, omit it.

## Place logs at the owning boundary

Keep the domain model free of `ILogger`, providers, scopes, and message templates. Domain code returns results, changes state, or raises domain events; it does not decide operational telemetry.

Place logs where enough context exists and responsibility is clear:

| Location | Appropriate logging |
| --- | --- |
| API middleware or exception handler | request outcome, unhandled exception, status, trace context |
| Application use case | meaningful workflow outcome, expected business rejection only when operationally useful |
| Driven adapter | dependency failure, latency or retry details, provider-specific context |
| Background worker or message consumer | item identity, attempt, completion, poison/dead-letter outcome |
| Composition root | startup configuration summary with secrets excluded |

Avoid logging the same event in endpoint, handler, repository, and middleware. Choose the layer that owns the outcome. A lower layer may add unique dependency context, but it must not repeat an unchanged exception just to rethrow it.

## Use structured events

- Use constant message templates with named properties.
- Pass values as arguments; do not interpolate strings or concatenate values into the message.
- Use stable, domain-relevant property names such as `ContributionId`, `RepositoryId`, `Operation`, `Attempt`, and `DurationMs`.
- Prefer identifiers and bounded summaries over serializing whole objects.
- Keep one semantic event shape stable across calls so logs remain queryable.
- Use `ILogger<TCategory>` or source-generated logging methods according to repository conventions.
- Use event IDs or stable event names when the system relies on them for alerts, dashboards, or tests.

Do not log object dumps, request bodies, access tokens, connection strings, authorization headers, cookies, secrets, private keys, or raw third-party payloads.

## Choose levels by operator action

| Level | Meaning |
| --- | --- |
| `Trace` | extremely detailed diagnostic steps, normally disabled |
| `Debug` | development or temporary diagnostic context |
| `Information` | meaningful successful lifecycle or operational events |
| `Warning` | abnormal but handled condition, retry, fallback, or approaching failure |
| `Error` | operation failed and requires investigation, but the process can continue |
| `Critical` | service or essential subsystem cannot safely continue |

Expected absence, validation, authorization denial, optimistic concurrency conflict, or domain rejection is not automatically an error. Match the level to operational impact and repository policy. Avoid using `Warning` as a generic record of anything undesirable.

## Handle exceptions once

Log an exception at the layer that handles it, converts it into an operational outcome, exhausts its retries, or prevents it from escaping silently.

- Pass the exception object to the logging API so stack and exception metadata are preserved.
- Add context the exception does not contain, such as stable IDs, dependency name, attempt, or operation.
- Do not log and rethrow unchanged when an outer boundary will log it.
- Do not swallow an exception after logging unless the flow deliberately converts it to a result, retry, fallback, or terminal state.
- Log transient failures as warnings during retries and log one error only after retry exhaustion, unless the platform already emits attempt telemetry.
- Preserve cancellation semantics. Do not log expected cooperative cancellation as an error.

## Carry correlation and context

Prefer the platform's `Activity` and distributed trace context. Do not create a parallel correlation ID when trace IDs already cross the relevant boundaries.

Use a logging scope for context shared by several events in one operation, such as `ContributionId`, `RepositoryId`, `JobId`, or `Attempt`. Start the scope at the driving boundary and let it flow through async calls. Do not manually repeat values already enriched globally unless the provider requires it.

For message processing, capture message identity and causation/correlation identifiers. For scheduled jobs, capture job/run identity. Keep identifiers consistent across logs, traces, and persisted workflow history.

## Protect data

Classify each property before logging it:

- **Safe:** non-sensitive internal IDs, operation names, statuses, counts, durations
- **Conditionally safe:** tenant IDs, filenames, URLs, user IDs, email domains; follow policy and retention rules
- **Unsafe by default:** names, email addresses, free text, document content, prompts, claims, tokens, headers, request bodies, credentials, secrets

Prefer opaque IDs over personal data. Redact at the source rather than relying only on a downstream sink. Never log a secret even at `Debug` or `Trace`. Avoid values supplied by users in message templates because they can create log-forging and cardinality problems.

## Distinguish logs, metrics, and traces

- Use logs for discrete events with investigation context.
- Use metrics for rates, counts, latency distributions, saturation, and alert thresholds.
- Use traces for the path and timing of one operation across components.

Do not emit a log per item solely to calculate a rate. Do not encode high-volume measurements as informational logs when a counter or histogram is the correct signal.

## Implement proportionally

For a small flow, a few `ILogger<T>` calls may be enough. Introduce source-generated logging, dedicated event catalogs, redaction infrastructure, or custom enrichers only when volume, performance, consistency, or policy justifies them.

When changing an existing flow:

1. Write the operator questions the logs must answer.
2. Map existing logs and framework telemetry.
3. Remove or avoid duplicates before adding events.
4. Select boundaries, levels, templates, and safe properties.
5. Add scopes or correlation only where missing.
6. Verify success, expected failure, unexpected failure, retry, and cancellation paths.
7. Check configured minimum levels so necessary events are actually emitted.

## Review checklist

Report concrete findings and repair the smallest useful surface:

- Is each log actionable or queryable?
- Is the event at the correct owning boundary?
- Are templates constant and properties named consistently?
- Are levels based on operational impact?
- Is each exception logged once with its object attached?
- Are retries, fallback, and cancellation represented accurately?
- Do scopes and traces carry correlation?
- Could any property contain a secret, personal data, or unbounded free text?
- Are logs duplicating metrics, traces, request middleware, or provider telemetry?
- Would production configuration retain the event at its chosen level?

Do not test implementation details of logging by default. Assert logs only when a specific audit, security, support, or operational event is part of the required contract.
