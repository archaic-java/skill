# Goals communicate application intent

Use `work.archaic.service.logging.v02.Goal` as an explicit statement of what an application is trying to accomplish. An application offers many possible goals; one execution is an attempt to fulfill one of them. Make that intent visible in code so readers can see both its purpose and where the attempt completes. Diagnostics are one consequence of this structure, not the only reason to use it.

## Contents

- Choose the intent and boundary
- Express the attempt in code
- Preserve completion semantics
- Separate intent, evidence and output
- Compose providers and execution

## Choose the intent and boundary

- Name the intended outcome: `orders.place`, `documents.import` or `releases.publish`. Use a readable variable such as `placeOrder`. Keep per-attempt identifiers in evidence rather than creating a new category for each entity.
- Establish the goal where the application accepts responsibility for that intent: a command, request handler or independently dispatched job. Include validation, work and communicating the result when those belong to the same intent.
- Keep inventory lookups, parsing and database writes as ordinary steps inside the goal. Do not add a goal to every method or force nested goals into the current contract.
- Use the goal even when no trail notes are needed: its name and execution boundary still explain the code. Do not add artificial notes merely to justify using a goal.
- Do not infer transactionality, rollback, retries or business success from the type. Implement those policies explicitly where needed.
- Keep libraries usable through ordinary methods. Let the application choose its intent boundaries; do not add goal scopes to generic helpers just to obtain logging.

## Express the attempt in code

Resolve and share Diagnostics and Log once at application composition time. Construct reusable goals separately from individual attempts:

```java
Goal placeOrder = diagnostics.goal("orders.place", log);
```

At the application entry point:

```java
placeOrder.run(() -> {
    diagnostics.note("Checking inventory");
    Order order = orders.place(request);
    respondWithConfirmation(order);
});
```

This example uses application-defined order and response operations. Import Goal, Diagnostics and Log from `work.archaic.service.logging.v02`. Ordinary functions such as `orders.place` may return values. Goal itself is run-only: keep the use of that result, including the confirmation, inside the complete intent. Do not add mutable result holders merely to emulate the removed `Goal.call` method.

Reuse a Goal for repeated or concurrent attempts; construction starts no work. Each invocation receives a fresh trail and execution identity. `run` executes on the calling thread and preserves declared checked exception types.

## Preserve completion semantics

Treat normal return as success and an escaping exception or error as failure. A completed successful attempt discards its trail. A failed attempt offers one failure report to its Log, then rethrows the original throwable. A reporting failure is suppressed on the original where possible.

When communicating a failed intent belongs inside the boundary, respond and rethrow:

```java
placeOrder.run(() -> {
    try {
        Order order = orders.place(request);
        respondWithConfirmation(order);
    } catch (OrderStorageException failure) {
        try {
            respondWithFailure(failure);
        } catch (IOException responseFailure) {
            failure.addSuppressed(responseFailure);
        }
        throw failure;
    }
});
```

The exception types and response operations above belong to the application; adapt them to its contract. Catch only failures the application knows how to handle. Catching a failure and returning normally declares success to the Goal, even if code sent an error response. A returned HTTP 500 or failure-valued result does not automatically fail it. Do not invent `finish`, `fail`, `close` or another outcome protocol for Goal. Keep response writing and resource closing synchronous within the boundary when they are part of completion.

## Separate intent, evidence and output

| Contract | Use |
| --- | --- |
| Goal | Name a reusable intent and delimit each attempt |
| Diagnostics | Define goals and access the selected instance's current trail |
| Trail / `diagnostics.note(...)` | Collect temporary evidence about the active attempt |
| `Log.write(String)` | Publish immediate information such as startup status |
| `Log.write(FailureReport)` | Receive completed failure evidence |

Keep notes useful for explaining failed attempts. Trail access outside an active attempt, after completion or from another thread is invalid. Do not assume propagation to spawned threads. Providers bound retained evidence and report loss; consult their documentation instead of hard-coding provider defaults into consumers.

Do not conflate Minau's TestCase and TestTrail with application Goal and Trail. Tests verify behavior; an expected application failure can make a test pass. Their independent scopes allow application goals inside test cases.

## Compose providers and execution

Require `work.archaic.service.catalog`. In the module that performs service discovery, declare:

```java
uses work.archaic.service.logging.v02.Diagnostics;
uses work.archaic.service.logging.v02.Log;
```

Resolve services explicitly with ServiceLoader; reject absent or ambiguous providers unless selecting a provider deliberately. Share the selected Diagnostics instance with participating code. Do not use a static global facade or separately discover instances in each helper. Peep supplies runtime providers and exports no implementation packages. Keep provider configuration in the catalog contract and retain v01 consumers unless migration is requested.

Use `goal.newExecutor()` only when each submitted task represents an independent attempt at that goal. Own and close the executor; observe submitted futures. The standard ExecutorService may return Callable results, but Goal's synchronous API remains run-only. No nested goals, goal inheritance or implicit cross-thread trail propagation are supported. Do not wrap an executor task in another goal on the same Diagnostics instance.

Check where exceptions are caught: a goal cannot observe failures consumed inside its body, a pre-wrapped FutureTask or a server's task. For example, establish `goal.run(...)` at an HTTP handler boundary with an ordinary server executor when the server would otherwise swallow handler exceptions.

Use the [catalog v02 contract](https://github.com/archaic-java/service-catalog/blob/main/docs/logging-v02.md) as the API authority and [Peep documentation](https://github.com/archaic-java/peep) for implementation behavior. Prefer `Diagnostics`, `Log.write`, `Goal.run`, `Goal.newExecutor` and `FailureReport.goalName` in new code; do not copy the superseded v01 names.
