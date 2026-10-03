# Logging contexts

Use `work.archaic.service.logging.v03` from `work.archaic.service.catalog` and
[Culpa](https://github.com/archaic-java/culpa) as the runtime provider. Depend on catalog types
in application code; avoid importing provider implementation classes.

## Contents

- Compose the provider and output
- Log from objects
- Scope execution and outcome
- Keep thread boundaries explicit
- Migrate applications and verify

## Compose the provider and output

Declare `requires work.archaic.service.catalog` and `uses work.archaic.service.logging.v03.Log`
in the composition module. Resolve `work.archaic.culpa` at launch. Select one factory explicitly:

```java
var providers = ServiceLoader.load(Log.class).stream().toList();
if (providers.size() != 1)
    throw new IllegalStateException("Exactly one logging provider required");
var logging = providers.getFirst().get();
var settings = Configuration.text(false, System.err);
```

Import `java.util.ServiceLoader` and the logging v03 types. Reuse the stateless factory and
immutable configuration; create a fresh context per complete execution. Capture debug, clock,
output and retention at creation. Avoid global installation or mutable provider-wide debug state.
Use the provider's `logging.context()` defaults only when stdout is appropriate.

Use `Configuration.text(debug, stream)` for standard timestamped text to stdout, stderr or another
PrintStream. Reuse catalog `TextOutput` with its `entry` and `failure` methods when supplying a
custom clock or limits. Render complete reports atomically across contexts sharing the stream;
leave stream ownership to the application. Follow PrintStream error semantics; do not imply durability.
Supply application-defined entry/report sinks for a different format or concise expected CLI errors.
Require shared sinks to support concurrency. Route LSP logs exclusively to stderr.

Use the three-argument Configuration constructor for custom sinks and default UTC, 256 retained
entries and 2048 UTF-16 units per field. Use the full constructor for a clock and retention limits
(capacity >= 1, field limit >= 2). Keep the newest evidence, mark clipping and report dropped entries.

## Log from objects

Implement `Logging` without a logger field:

```java
final class Orders implements Logging {
    String place() throws IOException {
        logOnFailure("Checking order requirements");
        logOnDebug(() -> expensiveStateDescription());
        // Perform the application work; let unrecovered failures escape.
        logImmediately("Order placed");
        return "accepted";
    }
}
```

Use `logImmediately(String)` to publish now and `logOnFailure(String)` to retain evidence.
Use `logOnDebug(Supplier<String>)` to compute and publish only when this context enables debug;
put expensive work inside the lambda. Require a non-null supplier even when disabled; evaluate
it exactly once on the caller thread when enabled and reject a null result. Propagate supplier
exceptions/errors normally. Capture entry timestamps after lazy message computation.
Use the default implementing-class `loggingName()` or override it for instance names.
Require an active context for every logging method; reject logging outside a scope.

## Scope execution and outcome

Wrap complete work, including its response where appropriate:

```java
logging.context(settings).run(() -> application.execute());
String outcome = logging.context(settings).call(() -> orders.place());
```

Use `run(Work<E>)` for void work and `call(Call<T, E>)` to return the exact value, including null,
without a mutable holder. Preserve checked exception types. Run synchronously on the calling
thread; logging creates no executor or thread. Share one single-use lifecycle between run and call;
reject reentrant, concurrent or completed-context reuse. Reject null work before consuming a context.

Discard evidence on normal completion unless explicitly marked failed. Publish once when an
exception or error escapes, then rethrow the original throwable. Use `context.fail(reason)` or
`Logging.context().fail(reason)` for a handled failure; keep the first reason and include subsequent
evidence. Keep syntax diagnostics and other ordinary domain results separate from logging failures;
do not infer failure from a returned value. Complete publication before returning a call result.

Treat nested contexts as independent: temporarily replace the binding and restore the parent
before publication/propagation. Recovering an inner failure leaves the parent successful; the same
exception escaping both scopes fails both with their own evidence. Reuse the current context across
ordinary helper calls rather than creating scopes for every method. Release evidence on every exit.
Suppress sink failure onto an escaping application throwable; let sink failure after a marked normal
return escape. Keep transactions, rollback and retries in application code.

## Keep thread boundaries explicit

Create independent contexts inside child tasks; ordinary threads do not inherit a context.
Allow creation on one thread and execution on another, but confine active operations to the executing
thread. Share configuration and thread-safe sinks, not active contexts. Observe task failures through
the application's completion mechanism; fail a parent only when its own outcome requires it.
Keep existing transport/parser concurrency when needed for responsiveness; remove threads whose
only purpose was an old logging boundary. Do not add preview or structured-concurrency dependencies.

## Migrate applications and verify

Replace named Goals with ordinary operational methods/objects implementing Logging; keep pure
helpers and independently usable service providers free of an implicit context requirement.
Use one context per command or independently executing task. Let completion own failure rendering;
let outer catches select exit status without reporting the same exception again. Render composition
failures separately when no context exists. Preserve compiler diagnostics, protocol framing and
expected CLI errors; avoid duplicate-report flags and switches created solely for logging.

Use [Knit](https://github.com/archaic-java/knit) for a command boundary and
[Shrink](https://github.com/archaic-java/shrink) for independent session/parser contexts and stderr
protocol separation. Inspect current source and dependency pins before adapting either example.

Keep Minau TestTrail independent. Use catalog `LoggingV03ProviderContract` for provider checks.
Test successful evidence discard, handled/escaping failures, recovery, nullable call results,
checked exceptions, disabled/enabled debug suppliers, sink failure and thread confinement where
relevant. For protocol servers, verify real-process stdout framing with debug enabled.
Consult the [catalog contract](https://github.com/archaic-java/service-catalog/blob/main/docs/logging-v03.md)
for exact lifecycle, retention and text-rendering semantics.
