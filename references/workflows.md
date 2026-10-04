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
rg --files -g 'AGENTS.md' -g 'README*' -g 'skills/**/SKILL.md' -g 'module-info.java' -g 'args/**' -g 'cmd/**'
find lib/src -maxdepth 1 -type l -printf '%p -> %l\n'
find src -type f -name '*.java' | sort
git status --short
java --version
javac --version
```

Read the maintenance-skill foundation and follow the relevant task links before exploring implementation. Discover its actual location through README/AGENTS.md rather than assuming the conventional path. Read catalog guidance at the selected dependency revision when touching a service boundary. Use [documentation ownership](documentation.md) to distinguish shared conventions, portable contracts and local mechanics.

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
9. Add README purpose, first use and canonical commands; add concise AGENTS.md routing. Create a shared maintenance skill with a compact foundation and task map, plus only the references already needed. Follow [documentation.md](documentation.md).
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

1. Follow the project skill’s relevant task links and locate the module that owns the behavior. Read the selected catalog contract when applicable.
2. Decide whether the existing API can express the change without a new public type.
3. Implement the smallest vertical slice, including error behavior.
4. Update the module descriptor if package visibility or readability changes.
5. Update root-module lists or launcher options if the graph changes.
6. Add tests at the closest observable boundary.
7. Update the owning documentation with changed behavior, commands and verification; link rather than copying another skill’s specification.
8. Run compile, test, and the relevant application path.

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

1. Read the selected catalog skill’s evolution and conformance guidance, then define the smallest provider-neutral interface and its input, output, and exception types in `work.archaic.service.<capability>.vNN`.
2. Export that package from the service-catalog module.
3. Add reusable conformance cases tied to documented portable promises where useful; keep provider-specific fault/platform checks in the provider project.
4. Implement the contract in a provider module without leaking provider types through the contract.
5. For service-loaded capabilities, declare `provides Contract with Implementation` in the provider.
6. Declare `uses Contract` in consumers and resolve through `ServiceLoader`, or follow documented explicit construction for capabilities that support it.
7. Specify selection policy in the owning application and document provider mechanics locally.
8. Test the provider through the contract.

If changing an existing version would break source, binary, or semantic compatibility, create the next versioned package.

## Add Minau tests

Use testing v02 for new suites when supported by the selected catalog and Minau revisions. Preserve existing v01 suites unless migration is requested. Follow the [shared testing conventions](conventions.md#testing) for case design and inline assertions.

1. Read the consuming project’s testing reference and canonical commands. Identify its production/test modules and dependency revisions.
2. Read the [catalog skill](https://github.com/archaic-java/service-catalog/blob/main/skills/maintain-service-catalog/SKILL.md) and its [testing v02 reference](https://github.com/archaic-java/service-catalog/blob/main/skills/maintain-service-catalog/references/test-v02.md) for suite registration, case execution and trail contracts. Inspect linked declarations in the actual dependency checkout.
3. Read [Minau’s maintenance skill](https://github.com/archaic-java/minau/blob/main/skills/maintain-minau/SKILL.md), especially its writing-tests, discovery and CLI references, for public constructors, JPMS visibility, launch/selection options and runner behavior. Use its README example rather than maintaining another complete example here.
4. Add cases at the closest observable boundary and update the consuming project’s descriptors/command files as needed. Keep its fixture choices and launch commands in that project’s documentation.
5. Compile, then run the consuming project’s documented test command. Inspect evidence and exit status using the selected runner’s guidance.

Do not assume Minau scans an `out/<module-name>` filesystem layout. Discovery behavior belongs to its versioned implementation and documentation; a different runner may schedule cases differently while honoring the catalog contract. Read only the references relevant to the task, not every catalog or runner document.

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
- documentation still names the exact working commands;
- a contributor can find changed constraints, owning code and verification through the project skill;
- changed links/anchors and examples resolve and match the dependency revisions used;
- detailed rules have one authoritative home and conformance assertions do not invent guarantees.

For documentation-only changes, validate metadata, links, routing and claims against source as described in [documentation.md](documentation.md). Run Java checks when changed examples or guarantees require execution; report unavailable tools rather than claiming checks passed.

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

Check the selected runner’s discovery/access reference and the catalog’s suite/case contract. For Minau, start from its [maintenance skill](https://github.com/archaic-java/minau/blob/main/skills/maintain-minau/SKILL.md). Diagnose the module descriptor and suite declaration before introducing access changes. Do not add global `--add-opens` or reflective workarounds merely to bypass the boundary.

### Assertions appear to pass unexpectedly

Confirm the test launcher contains `-ea` before trusting results. Minau expects assertions to be enabled.

### Service provider not discovered

Check that the provider module is resolved at runtime, declares `provides`, the consumer declares `uses`, and the loaded contract class comes from the same module/version package. Do not instantiate the provider directly as a workaround.
