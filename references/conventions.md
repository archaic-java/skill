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
- Existing examples

## Design priorities

The school favors explicit mechanics over ecosystem convenience:

1. **JDK first.** Search the current JDK for a sufficient API before introducing a library.
2. **Modules as architecture.** Compile-time readability, exports, services, and qualified opens describe real boundaries.
3. **Commands as build interface.** Checked-in argument files make the compiler and launcher invocation reviewable and reproducible without a build-tool model.
4. **Source-level composition.** Small sibling projects can be compiled together through module-directory links rather than published merely to satisfy a local build.
5. **Contracts before containers.** Java interfaces plus `ServiceLoader` supply decoupling without a dependency-injection framework.
6. **Small code over scaffolding.** Add machinery only when it removes more complexity than it creates.

These are decision criteria, not permission to rewrite a working repository. Match local conventions and preserve deliberate exceptions.

## Canonical repository shape

```text
<project>/
├── AGENTS.md
├── README.md
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

Some established projects use `args/` instead of `cmd/`. change them and their respective docs. Eclipse `.project` and `.classpath` files may exist as editor metadata; they do not replace the command-line build.

## Module and package design

- Name first-party modules `work.archaic.<project-or-capability>`.
- Mirror the module name in package roots, using a deliberately named subpackage such as `.api`, `.cli`, or `.internal` when useful.
- Give tests their own `<production-module>.test` module.
- Do not export implementation packages by default.
- Use `requires transitive` only when consumers of the current module's public API must also read the dependency.
- Use `opens <test-package> to work.archaic.minau` for Minau discovery rather than opening the whole module.
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

The catalog module exports the versioned contract package. It contains types required to express the contract and, where useful, provider-conformance tests. It does not depend on a provider.

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

Load providers explicitly with `ServiceLoader.load(Example.class)` and define what zero or multiple providers mean. Do not hide selection in a global container.

Treat each published version package as immutable. A breaking signature or semantic change creates `v02`; keep `v01` while consumers or providers still use it.

## Testing

When the project uses Minau:

- Put tests in a separate named module.
- Require the production module and `work.archaic.service.catalog`.
- Export the package containing suites and open it specifically to `work.archaic.minau`.
- Implement `work.archaic.service.test.v01.TestSuite`.
- Annotate zero-argument instance methods with `@Test`.
- Use Java `assert` with an explanatory failure message.
- Run the JVM with `-ea`.
- Add the test module with `--add-modules` and pass its name to Minau's main class.

Test observable contracts and edge cases. Keep tests independent because a runner may execute them concurrently. Use `setup()` and `teardown()` only for suite-level resources, and preserve the original failure if cleanup also fails.

An established project may contain purpose-built main-method integration or compatibility tests. Preserve that testing style where it tests process boundaries or JDK compatibility better than an in-process suite.

## Source and documentation style

- Match the repository's formatter and indentation; do not perform unrelated formatting.
- Prefer small classes, interfaces, records, and plain methods over framework abstractions.
- Use current language features when they improve clarity and the selected JDK supports them.
- Keep mutable global state rare and explicit.
- Use checked or domain-specific exceptions when callers can act on a failure; include a useful message.
- Write Javadoc for exported contracts and non-obvious invariants. Keep implementation comments focused on why.
- Keep CLI output and exit codes deliberate. Use the repository's logging convention rather than introducing a new facade casually.

## Existing examples

These repositories illustrate the school but are examples, not a substitute for local instructions:

- `archaic-work/rami`: small application, separate test module, argument-file commands, source-linked Minau and service catalog.
- `archaic-work/minau`: JDK-first test runner and test-service consumer.
- `archaic-work/contract`: versioned service catalog.
- `archaic-work/jules`: JPMS service provider with an intentional modular binary dependency.

If inspecting them online, use the current repository state and note that conventions can evolve.
