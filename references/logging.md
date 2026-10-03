# Logging API details

Use `work.archaic.service.logging.v03` from `work.archaic.service.catalog`.
Use [Culpa](https://github.com/archaic-java/culpa) as the runtime provider.
Application objects depend on the catalog, not provider implementation types.

## Compose once

Declare `requires work.archaic.service.catalog` and `uses work.archaic.service.logging.v03.Log`
in the module that selects the provider. Resolve `work.archaic.culpa` at runtime.
Select exactly one provider explicitly and install it once at startup:

```java
var providers = ServiceLoader.load(Log.class).stream().toList();
if (providers.size() != 1)
    throw new IllegalStateException("Exactly one logging provider required");
Logging.install(providers.getFirst().get());
Logging.debug(false);
```

Import `java.util.ServiceLoader` and the logging v03 types. Use before installation and
provider replacement fail explicitly. The shared provider holds application-wide debug state;
debug is disabled initially. Configure custom output only at composition time when needed.

## Log from objects

Implement `Logging` without declaring a logger field:

```java
final class Orders implements Logging {
    void place() throws IOException {
        logOnFailure("Checking order requirements");
        // Perform the application work; let unrecovered failures escape.
        logImmediately("Order placed");
    }
}
```

- Use `logImmediately(String)` to publish now, inside or outside a trail.
- Use `logOnDebug(String)` to publish now only when `Logging.debug(true)` is enabled.
- Use `logOnFailure(String)` to retain evidence in the active trail.
- Let the default `loggingName()` identify the implementing class; override it for instance names.
- Capture timestamps and source names at submission. String arguments are eager, even when debug is disabled.

## Establish the failure boundary

Wrap complete work at the application entry point, including its response where appropriate:

```java
Logging.trail(() -> orders.place());
```

- Execute on the calling thread. Logging creates no thread, executor or asynchronous task.
- Preserve checked exception types through `Work<E extends Exception>` and rethrow the original exception or error.
- Discard evidence on normal return; publish once when a failure escapes the outer boundary.
- Mark a handled or result-based failure with `Logging.failure(String)` inside the trail. It does not throw; the first reason is retained and publication occurs at completion.
- Join the active trail on nested `Logging.trail` calls. An inner exception caught and recovered by the outer work does not independently publish or mark failure.
- Keep independent executions isolated, even when they share application objects. Child threads establish independent trails; no supported cross-thread propagation exists.
- Require an active trail for `logOnFailure` and `Logging.failure`; calls outside one fail explicitly.
- Keep evidence bounded. Culpa retains the newest 256 entries, limits each text field to 2048 UTF-16 units, marks clipping and reports dropped entries.
- Keep thread creation, retries, transactions and rollback in application code. Observe asynchronous task results at the appropriate application boundary.
- Keep Minau TestTrail independent. Use the catalog's `LoggingV03ProviderContract` for provider-conformance tests.

Consult the [catalog contract](https://github.com/archaic-java/service-catalog/blob/main/docs/logging-v03.md)
for exact semantics and Culpa for configuration and output behaviour.
