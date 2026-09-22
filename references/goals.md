# Goal API details

Use `work.archaic.service.logging.v02`. Resolve Diagnostics and Log explicitly with ServiceLoader at composition time; share the selected instances. Require the catalog and declare `uses` for services loaded by the module. Peep is a runtime provider; do not import its implementation types.

Define distinct reusable goals for the application's intents:

```java
Goal placeOrder = diagnostics.goal("orders.place", log);
Goal cancelOrder = diagnostics.goal("orders.cancel", log);
```

Execute a complete attempt at its application entry point (operations below are application-defined):

```java
placeOrder.run(() -> {
    Order order = orders.place(request);
    respondWithConfirmation(order);
});
```

- Construction starts no work. A Goal supports repeated and concurrent attempts. `run` executes on the calling thread, preserves checked exceptions and returns no value; ordinary methods inside it may return values.
- Normal return discards the trail. An escaping exception or error publishes failure evidence and rethrows the original throwable. When responding to failure inside the goal, rethrow afterward; attach response failures as suppressed. An error status or failure-valued return alone does not fail a goal.
- Use `diagnostics.note(...)` or `Trail.note(...)` for temporary evidence; `log.write(...)` publishes immediately. Notes are optional. Trails are bounded, confined to the active attempt's thread and invalid after completion.
- Use `goal.newExecutor()` for independent attempts on virtual threads; own and close the executor and observe futures. No nested goals or implicit trail inheritance are supported. Place the boundary where failures actually escape; server tasks can swallow handler exceptions.
- Goals imply no transactions, rollback or retries. Keep Minau TestCase/TestTrail independent from application goals.

Consult the [catalog contract](https://github.com/archaic-java/service-catalog/blob/main/docs/logging-v02.md) for exact semantics and [Peep](https://github.com/archaic-java/peep) for provider behavior.
