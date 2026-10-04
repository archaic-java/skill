# Coding guidance

Read this reference when writing or reviewing Java methods, including tests. Follow
the mandatory rules in [Archaic Java](../SKILL.md#keep-control-flow-flat-and-validate-before-work).
Keep the main operation readable as a sequence after its guards. Reduce the number
of conditions, lifetimes and recovery decisions a reader must retain at once.

## Contents

- Establish preconditions before work
- Flatten decisions without hiding them
- Communicate foreseeable failures explicitly
- Keep recovery and resource scopes coherent
- Review each changed method

## Establish preconditions before work

Reject invalid inputs and unmet preconditions before preparing or executing the
operation. Return immediately for a legitimate trivial result. Use one guard per
distinct reason when that makes dependencies and failure messages clearer.

Avoid enclosing the operation in successive success conditions:

```java
Receipt submit(Order order) throws OrderRejectedException {
    Objects.requireNonNull(order, "order");
    if (!order.items().isEmpty()) {
        if (order.ready()) {
            return execute(prepare(order));
        } else {
            throw new OrderRejectedException("Order is not ready");
        }
    } else {
        throw new OrderRejectedException("Order must contain items");
    }
}
```

Write the equivalent guards followed by the operation:

```java
Receipt submit(Order order) throws OrderRejectedException {
    Objects.requireNonNull(order, "order");
    if (order.items().isEmpty()) {
        throw new OrderRejectedException("Order must contain items");
    }
    if (!order.ready()) {
        throw new OrderRejectedException("Order is not ready");
    }

    var plan = prepare(order);
    return execute(plan);
}
```

Treat these as method fragments in an order-processing object: `prepare` creates a
plan and `execute` performs it. Treat a null reference here as incorrect API usage;
an empty or unready order is a foreseeable domain rejection. If missing input comes
from an external request, translate it at that boundary into the declared domain
failure instead of exposing a programming-error exception.

Order guards by their dependencies and documented error precedence. Prefer cheap
checks first when that order is otherwise unconstrained. Do not open a resource,
fetch remote data or build an expensive plan merely to create a validation block.
Check conditions requiring those results immediately after obtaining them. Preserve
any required close, rollback or cleanup on rejection.

Do not treat a prior check of mutable shared state as a guarantee. Perform checks
and dependent changes under the required lock or transaction. Avoid speculative
filesystem existence/access checks as a substitute for handling failure of the
actual file operation.

## Flatten decisions without hiding them

Use `continue` to reject loop items before processing them. A short guard inside a
loop does not require extraction merely because it is indented. For deeper work,
extract a method naming the complete operation for one item. Use a flat `switch`
when it expresses a genuine choice among alternatives.

Do not replace nested branches with nested ternaries, streams containing branching
lambdas or an opaque boolean expression. Retain small expressions that describe one
clear predicate, such as an inclusive range check. Extract methods for coherent
responsibilities, not arbitrary line ranges or generic `step1`/`step2` helpers.

## Communicate foreseeable failures explicitly

Use custom exceptions extending `Exception` for foreseeable unhappy paths callers
must handle or deliberately propagate. Choose domain names and provide useful
messages, structured details when needed, and a cause constructor for translation.
Reuse a suitable existing domain exception; do not create one type per guard unless
callers need distinct handling. Never require callers to parse message text.

Reserve unchecked exceptions for programming errors and broken internal invariants.
Use ordinary return values for expected alternatives within a successful operation,
such as a search finding no match. Follow the owning contract's documented distinction
between a successful alternative and a failed operation.

Declare shared exception types in the owning API or service catalog, keeping provider
details out of the public failure surface. Declare the specific checked type in
`throws`; do not erase it to `Exception`, wrap it in `RuntimeException`, swallow it,
or return a success-shaped fallback merely to fit a callback. Choose an API that
supports propagation or handle the failure at the actual policy boundary. Preserve
published contracts; propose a new contract version when a new checked exception
would break compatibility.

## Keep recovery and resource scopes coherent

Catch only failures for which the method owns recovery or translation. Let callers
handle other failures. Keep exception translation close to the operation that raises
it, rather than wrapping an entire business method or nesting recovery blocks.

```java
String readConfiguration(Path path) throws ConfigurationReadException {
    Objects.requireNonNull(path, "path");
    try {
        return Files.readString(path);
    } catch (IOException failure) {
        throw new ConfigurationReadException(
            "Cannot read configuration: " + path, failure);
    }
}
```

Treat this as a method fragment with `ConfigurationReadException` extending
`Exception` and accepting a message and cause. Keep parsing and validation outside
this I/O translation scope so their failures retain their own meaning.

Use try-with-resources when the method owns a resource. Combine resources in one
header when their lifetimes coincide; retain separate scopes when their lifetimes
or recovery semantics differ. A resource scope or synchronization scope is not
itself a conditional decision. Preserve close order, suppressed exceptions,
transaction boundaries and required recovery even when that requires indentation.
Do not flatten code by leaking resources or changing which failures a catch sees.

## Review each changed method

Trace guards, the main operation and failure exits. Refactor nested decisions,
unnecessary `else` branches after termination, overly broad catches and avoidable
work before rejection. Check foreseeable failures use the declared custom checked
types and remain visible to callers. Apply this review to tests too; retain inline
expected-exception assertions as required by the testing conventions.

For any retained nesting or exception-policy deviation, report the specific reason
that correctness or readability requires it. Keep non-obvious invariants beside the
code. Check observable behavior, including failure precedence, cause preservation,
cleanup and the absence of expensive work or external changes on rejected input
where relevant. Avoid adding tests that merely restate the implementation's shape.
