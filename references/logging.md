# Application logging guidance

Use logging v03 and Culpa for new application logging. This reference owns Archaic
Java's application-design recommendations; the catalog owns API semantics and Culpa
owns provider mechanics. Read the
[catalog skill](https://github.com/archaic-java/service-catalog/blob/main/skills/maintain-service-catalog/SKILL.md)
and its [logging v03 guide](https://github.com/archaic-java/service-catalog/blob/main/skills/maintain-service-catalog/references/logging-v03.md)
when composing or changing logging. Inspect declarations at the dependency revision
actually used; the links to `main` are navigation, not version pins.

## Compose explicitly

Select the logging provider at application composition, using catalog types in
application code. Follow the contract's selection rules and the provider project's
checkout/runtime guidance. Record provider choice and dependency revision in the
application's maintenance documentation.

Choose output, debug policy and scope boundaries explicitly for the application.
Use the catalog's standard text rendering when appropriate; supply custom sinks when
protocol or CLI needs require another format. Keep log output off a protocol channel:
LSP logs belong on stderr. Decide stream ownership and expected-error rendering in
the application, with the catalog's sink semantics as constraints.

Use provider defaults only after checking their documented behavior. Keep exact
Configuration constructors, retention bounds, field clipping and publication semantics
in catalog/provider references rather than maintaining another specification here.

## Model application intents

Prefer ordinary methods and objects implementing `Logging` over named logging goals
or logger fields. Choose scopes around complete operational intents: one command,
request handler or independently executing task. Include response handling when it
belongs to the operation. Reuse the current context through ordinary helper calls;
do not create another context for every method.

Use the catalog's void and value-returning context entry points for the relevant work,
avoiding mutable holders solely to retrieve results. Put expensive debug computations
inside lazy suppliers. Use failure evidence to explain an unsuccessful operation and
immediate output for information whose publication belongs to the intent.

Keep syntax diagnostics and ordinary domain results distinct from logging failures.
For handled failures, apply the contract's explicit failure mechanism when the operation
has failed; do not make every diagnostic or negative result a logging failure.
Keep transactions, rollback and retry decisions in application code. Keep independently
usable service providers and pure helpers free of an incidental logging-scope requirement.

## Preserve existing execution boundaries

Apply the catalog's ownership and nesting rules to the application's concurrency.
Give independently executing worker tasks their own scopes and use the application's
completion mechanism to observe failures. Keep transport/parser concurrency when it
serves responsiveness; do not introduce threads merely to establish logging.
Do not add preview or structured-concurrency dependencies for this purpose.
Read the catalog for exact binding restoration, thread confinement and outcome semantics.

## Migrate and verify application policy

Let the logging boundary own failure rendering and let outer catches choose exit
status without reporting the same failure twice. Render composition failures separately
when no context exists. Preserve compiler diagnostics, protocol framing and expected CLI
errors; avoid duplicate-report flags introduced only to coordinate logging layers.

Use [Knit](https://github.com/archaic-java/knit) as an example of a command boundary and
[Shrink](https://github.com/archaic-java/shrink) for session/parser contexts and protocol
separation. Read their current project guidance and dependency pins before adapting them.
Document actual application choices and integration checks in the consuming project.

For provider compliance, follow the catalog's reusable cases and the provider project's
verification guidance. For application integration, test the chosen scope boundaries,
handled/escaping failure rendering and configured destinations as relevant. Protocol
servers should verify real-process stdout framing with debug enabled. Keep Minau's test
evidence independent; use its own maintenance guidance for test reports and trails.
