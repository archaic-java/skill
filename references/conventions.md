# Archaic Java conventions

Use this reference when creating a project, choosing module boundaries, adding dependencies, designing services, or reviewing whether a change fits the school of Archaic Java.

## Contents

- Design priorities
- Canonical repository shape
- Module and package design
- Dependencies
- Service contracts and providers
- Testing
- Source and documentation style
- Documentation ownership
- Existing examples

## Design priorities

The school favors explicit mechanics over ecosystem convenience:

1. **JDK first.** Search the current JDK for a sufficient API before introducing a library.
2. **Modules as architecture.** Compile-time readability, exports, services, and qualified opens describe real boundaries.
3. **Commands as build interface.** Checked-in argument files make the compiler and launcher invocation reviewable and reproducible without a build-tool model.
4. **Source-level composition.** Small sibling projects can be compiled together through module-directory links rather than published merely to satisfy a local build.
5. **Contracts before containers.** Java interfaces plus `ServiceLoader` supply decoupling without a dependency-injection framework.
6. **Intent in code.** Name ordinary methods for application intents; collect failure evidence in a configured context around a complete execution on the calling thread, using `run` or value-returning `call`.
7. **Small code over scaffolding.** Add machinery only when it removes more complexity than it creates.

These are decision criteria, not permission to rewrite a working repository. Match local conventions and preserve deliberate exceptions.

## Canonical repository shape

```text
<project>/
├── AGENTS.md
├── README.md
├── skills/
│   └── maintain-example/
│       ├── SKILL.md
│       └── references/          # detailed guidance, created as needed
├── cmd/
│   ├── compile
│   ├── run
│   └── test
├── lib/
│   ├── src/
│   │   └── work.archaic.dependency -> ../../../dependency/src/work.archaic.dependency
│   └── bin/
│       └── deliberate-modular-dependency.jar
├── src/
│   ├── work.archaic.example/
│   │   ├── module-info.java
│   │   └── work/archaic/example/...
│   └── work.archaic.example.test/
│       ├── module-info.java
│       └── work/archaic/example/test/...
└── out/                       # generated and ignored
```

Some established projects use `args/` instead of `cmd/`. Preserve the chosen directory and keep its documentation consistent. Eclipse `.project` and `.classpath` files may exist as editor metadata; they do not replace the command-line build.

## Module and package design

- Name first-party modules `work.archaic.<project-or-capability>`.
- Mirror the module name in package roots, using a deliberately named subpackage such as `.api`, `.cli`, or `.internal` when useful.
- Give tests their own `<production-module>.test` module.
- Do not export implementation packages by default.
- Use `requires transitive` only when consumers of the current module's public API must also read the dependency.
- For test-module visibility, follow the selected runner’s maintenance guidance. See the [Minau workflow](workflows.md#add-minau-tests) for the catalog and runner entry points; do not add broad reflective access workarounds.
- Keep `module-info.java` changes in the same change set as the Java code that needs them.

A direct API is simpler than a service boundary when the implementation is intrinsic to the module. Introduce a service when consumers should depend on a capability independently of its provider.

## Dependencies

### Source modules

Represent a source dependency as a link whose name equals the JPMS module:

```text
lib/src/work.archaic.minau -> ../../../minau/src/work.archaic.minau
```

The target is the module directory containing `module-info.java`, not the dependency repository root. Prefer relative links so sibling checkouts can move together. List the links and then check for broken targets before compiling:

```shell
find lib/src -maxdepth 1 -type l -printf '%p -> %l\n'
find -L lib/src -maxdepth 1 -type l -print
```

The second command prints only links whose targets cannot be resolved; no output is the successful result.

A broken source link is a checkout/dependency problem, not a reason to copy dependency source into the consumer.

### Binary modules

Put intentionally accepted modular JARs in `lib/bin/`. Inspect their module identity before referring to them:

```shell
jar --describe-module --file lib/bin/dependency.jar
```

For a multi-release JAR, the first command may only list available releases. Rerun it with the listed descriptor release, for example `--release 9`.

Avoid unnamed or automatic-module behavior. If the dependency is not a proper named module, stop and discuss the architectural exception instead of silently using the class path.

### Compile roots

List every root module that must be compiled in the compile argument file. A typical compiler graph uses:

```text
--module work.archaic.minau,work.archaic.service.catalog,work.archaic.example,work.archaic.example.test
--module-source-path src:lib/src
--module-path lib/bin
-d out
```

Keep one option or conceptual group per line and retain comments that explain why an option exists.

## Service contracts and providers

Use the service catalog to separate a capability from implementations:

```java
package work.archaic.service.example.v01;

public interface Example {
    Result perform(Request request) throws ExampleException;
}
```

Use one catalog per namespace; applications may depend on catalogs from multiple namespaces. The catalog module exports versioned contract packages. It contains the types and shared mechanics needed to express contracts and, where useful, provider-conformance cases. It does not depend on a provider. Those cases attest documented expectations rather than introducing stronger promises; actual execution evidence belongs with the provider being verified. Read the [catalog skill](https://github.com/archaic-java/service-catalog/blob/main/skills/maintain-service-catalog/SKILL.md) when evolving contracts or checking provider compliance.

A provider descriptor states:

```java
module work.archaic.example.provider {
    requires work.archaic.service.catalog;
    provides work.archaic.service.example.v01.Example
        with work.archaic.example.provider.JdkExample;
}
```

A consumer descriptor states:

```java
module work.archaic.example.consumer {
    requires work.archaic.service.catalog;
    uses work.archaic.service.example.v01.Example;
}
```

For service-loaded capabilities, load providers explicitly with `ServiceLoader.load(Example.class)` and define what zero or multiple providers mean. Follow the capability’s composition rules where explicit construction is supported; do not hide selection in a global container.

Treat each published version package as immutable. A breaking signature or semantic change creates `v02`; keep `v01` while consumers or providers still use it.

## Testing

Prefer testing v02 and Minau for new suites when the selected dependencies support them. Keep tests in a separate named module and depend on catalog interfaces rather than runner internals. Organize cases around observable behavior; prefer a public suite record with package-private case records and immutable input/expected-result components. Use ordinary registration loops for data-driven cases rather than introducing parameterized-test machinery.

Use Java `assert condition : "reason";` for every assertion. Always include the colon message: briefly state the expectation or contract whose violation makes the test fail, rather than merely saying "assertion failed" or dumping actual/expected values. Include observed values when useful and run with `-ea`. Apply this style to v01, v02 and assertion-based integration checks.

Do not create assertion helper methods, wrappers or a local assertion DSL such as `assertEquals`, `check` or `assertThrows`. Keep verification, including expected-exception checks, inline with Java `assert`. If that becomes cumbersome, propose a test API extension and agree on contract changes before implementing them. Ordinary helpers for preparing inputs, launching processes and collecting observations remain useful.

Create mutable fixtures where the test owns their lifetime, use synchronization to observe concurrency, and bound external work. Keep test evidence independent of application logging; add logging dependencies when logging itself is under test. Preserve existing v01 suites and purpose-built main-method integration checks unless migration is requested.

Read the [testing workflow](workflows.md#add-minau-tests) for task routing. The catalog owns registration, case and evidence contracts; Minau owns discovery, CLI selection, scheduling and retention limits. The consuming project owns its fixtures and test commands. Do not repeat those specifications here.

## Source and documentation style

- Match the repository's formatter and indentation; do not perform unrelated formatting.
- Prefer small classes, interfaces, records, and plain methods over framework abstractions.
- Use current language features when they improve clarity and the selected JDK supports them.
- Keep mutable global state rare and explicit.
- Use checked or domain-specific exceptions when callers can act on a failure; include a useful message.
- Write Javadoc for exported contracts and non-obvious invariants. Keep implementation comments focused on why.
- Keep CLI output and exit codes deliberate. Use the repository's logging convention rather than introducing a new facade casually.

## Documentation ownership

Provide a shared project maintenance skill, normally under `skills/maintain-<project>/`, and route both README readers and agents to it. Keep its detailed guidance in references. Follow [documentation.md](documentation.md) for ownership, file responsibilities, progressive disclosure and completion checks. Javadoc stays beside declarations; command files stay executable build specifications.

## Existing examples

These repositories illustrate the school but are examples, not a substitute for local instructions:

- `archaic-work/rami`: small application, separate test module, argument-file commands, source-linked Minau and service catalog.
- `archaic-java/minau`: JDK-first test runner and test-service consumer.
- `archaic-java/service-catalog`: versioned service catalog.
- `archaic-work/jules`: JPMS service provider with an intentional modular binary dependency.

If inspecting them online, use the current repository state and note that conventions can evolve.
