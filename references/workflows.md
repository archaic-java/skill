# Archaic Java workflows

Use this reference for concrete repository creation, feature extension, dependency and service changes, validation, or build diagnosis.

## Contents

- Inspect an existing project
- Create a project
- Extend a project
- Add a source dependency
- Add a binary dependency
- Add a service contract and provider
- Add Minau tests
- Validate a change
- Diagnose failures

## Inspect an existing project

Run read-only discovery before proposing a change:

```shell
rg --files -g 'AGENTS.md' -g 'README*' -g 'module-info.java' -g 'args/**' -g 'cmd/**'
find lib/src -maxdepth 1 -type l -printf '%p -> %l\n'
find src -type f -name '*.java' | sort
git status --short
java --version
javac --version
```

Read the argument files as the executable build specification. Build a quick inventory of:

- production, test, example, and tool modules;
- exported and opened packages;
- `requires`, `uses`, and `provides` edges;
- source links and binary modules;
- entry points and test runners;
- the required JDK and any preview options.

Do not assume that a directory named `cmd` contains executable shell scripts; it may contain Java argument files consumed as `javac @cmd/compile` or `java @cmd/run`.

## Create a project

1. Choose a project name and module name, normally `work.archaic.<name>`.
2. Create the production module directory and its package tree.
3. Add the narrowest useful public API or CLI entry point.
4. Add a `module-info.java` that declares only actual edges and exports.
5. Add an independent test module if the project has testable behavior.
6. Add only required dependency links or modular JARs.
7. Write the compiler and launcher argument files.
8. Ignore `out/` and other generated artifacts.
9. Add concise `AGENTS.md` and `README.md` files.
10. Compile, test, and run from a clean `out/` directory.

A minimal production descriptor can be:

```java
module work.archaic.example {
    exports work.archaic.example.api;
}
```

A minimal compile argument file can be:

```text
# Modules to compile
--module work.archaic.example,work.archaic.example.test

# Source and binary dependencies
--module-source-path src:lib/src
--module-path lib/bin

# Generated classes
-d out
```

A minimal launcher argument file can be:

```text
--module-path out:lib/bin
-m work.archaic.example/work.archaic.example.Main
```

Do not add empty placeholder modules, dependency directories, or service abstractions. Create only what the project already needs.

## Extend a project

For each requested behavior:

1. Locate the module that owns the behavior.
2. Decide whether the existing API can express the change without a new public type.
3. Implement the smallest vertical slice, including error behavior.
4. Update the module descriptor if package visibility or readability changes.
5. Update root-module lists or launcher options if the graph changes.
6. Add tests at the closest observable boundary.
7. Run compile, test, and the relevant application path.

Avoid a broad refactor unless the feature exposes a concrete structural problem. Never edit a versioned service contract in place merely to make a provider change convenient.

## Add a source dependency

1. Determine the dependency's exact JPMS module directory.
2. Create a relative link under `lib/src/` named exactly after that module.
3. Add the dependency root module to the compiler's `--module` list when it must be compiled in the same invocation.
4. Add `requires` only to modules that read it.
5. Compile and inspect resolution errors before changing the link layout.

Typical link from `<consumer>/lib/src/` to a sibling checkout:

```shell
ln -s ../../../dependency/src/work.archaic.dependency lib/src/work.archaic.dependency
```

Before creating it, resolve both paths explicitly and refuse to replace an existing file or link silently.

## Add a binary dependency

1. Confirm that a third-party dependency is justified and that a JDK API is insufficient.
2. Verify the JAR is a proper named module with `jar --describe-module --file <jar>`. If a multi-release JAR only reports its available releases, rerun with the relevant `--release <number>`.
3. Put the pinned artifact in `lib/bin/` according to repository policy.
4. Add the exact module name to `requires` and include `lib/bin` on compiler and launcher module paths.
5. Record the artifact's origin and version where the repository documents dependencies.
6. Compile, run tests, and use `jdeps` when transitive edges are unclear.

Do not solve a missing module descriptor with a class-path fallback.

## Add a service contract and provider

1. Define the smallest provider-neutral interface and its input, output, and exception types in `work.archaic.service.<capability>.vNN`.
2. Export that package from the service-catalog module.
3. Add conformance tests to the catalog when all providers should satisfy the same behavior.
4. Implement the contract in a provider module without leaking provider types through the contract.
5. Declare `provides Contract with Implementation` in the provider.
6. Declare `uses Contract` in consumers and resolve implementations through `ServiceLoader`.
7. Specify behavior for no provider and multiple providers.
8. Test the provider through the contract.

If changing an existing version would break source, binary, or semantic compatibility, create the next versioned package.

## Add Minau tests

The test descriptor normally resembles:

```java
module work.archaic.example.test {
    requires work.archaic.example;
    requires work.archaic.service.catalog;
    exports work.archaic.example.test;
    opens work.archaic.example.test to work.archaic.minau;
}
```

A suite normally resembles:

```java
public final class ExampleTest implements TestSuite {
    @Test
    public void returnsExpectedValue() {
        var actual = Example.perform();
        assert actual.equals("expected") : "Unexpected result: " + actual;
    }
}
```

The test launcher normally includes:

```text
-ea
--module-path out:lib/bin
--add-modules work.archaic.example.test
-m work.archaic.minau/work.archaic.minau.Main
work.archaic.example.test
```

Keep `-m` and its main-module argument together as required by the project's established argument-file form. For multiple test modules, follow the installed Minau version's documented comma-separated syntax.

## Validate a change

Prefer the project's documented public commands. A typical sequence is:

```shell
javac @args/compile
java @args/test
java @args/run
```

Also run documented lint, Javadoc, packaging, or integration commands when present and relevant. A lint argument file may contain only additional compiler options; in that case combine it with the compile argument file as the repository documents, commonly `javac @args/lint @args/compile`. Then check:

```shell
git status --short
git diff --check
git diff
```

Confirm that:

- the required JDK executed both compiler and launcher;
- no class-path option was introduced;
- all module links resolve;
- assertions were enabled for assertion-based tests;
- `out/` and other generated files remain untracked;
- the run command exercises the intended entry point;
- documentation still names the exact working commands.

## Diagnose failures

### `javac: command not found` or wrong class-file version

Check both binaries, not only the runtime:

```shell
command -v java javac
java --version
javac --version
```

Use one JDK installation for both commands. Do not lower source features to accommodate an accidental older compiler.

### Module not found

Check, in order:

1. spelling in `module-info.java` and argument files;
2. whether the source link resolves to a directory containing `module-info.java`;
3. whether the module is named in compiler `--module` roots;
4. whether its location is on `--module-source-path` or `--module-path`;
5. whether a binary JAR's actual module name matches the assumed name.

### Package is not visible

Identify whether the consumer is missing `requires` or the producer intentionally does not `exports` the package. Do not export an implementation package reflexively; move the needed contract into an API package if that is the real boundary.

### Reflective access failure in tests

Prefer a qualified `opens <test-package> to work.archaic.minau` in the test descriptor. Do not add `--add-opens` globally unless a repository-local constraint requires it.

### Assertions appear to pass unexpectedly

Confirm the test launcher contains `-ea` before trusting results. Minau expects assertions to be enabled.

### Service provider not discovered

Check that the provider module is resolved at runtime, declares `provides`, the consumer declares `uses`, and the loaded contract class comes from the same module/version package. Do not instantiate the provider directly as a workaround.
