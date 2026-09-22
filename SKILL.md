---
name: archaic-java
description: "Create, maintain, extend, diagnose, or review projects in the school of Archaic Java: JDK 25, explicit JPMS modules, javac/java argument files, source-linked or modular-JAR dependencies, JDK-first implementations, Minau tests, versioned service contracts, and the Archaic Java web design system. Use for archaic.work repositories or when the user explicitly asks for Archaic Java conventions or visual design. Do not impose these conventions on unrelated Java projects."
---

# Archaic Java

Develop Java systems whose structure and mechanics remain visible in the repository. Prefer the JDK and JPMS over build tools, frameworks, class paths, generated configuration, and hidden dependency injection.

## Establish the local contract

Before changing a repository:

1. Read every applicable `AGENTS.md` and the project `README.md`.
2. Inspect `args/` or `cmd/`, `module-info.java`, `lib/src`, `lib/bin`, `.gitignore`, and the test modules.
3. Check `java --version`, `javac --version`, and the worktree status.
4. Treat the repository as the authority. Preserve its chosen JDK version, naming, command-file directory, and established variations. For a new project, use JDK 25 unless the user chooses another version.

User instructions and repository-local instructions take precedence over this skill. Do not modernize an old repository merely to make it resemble another Archaic Java project.

## Preserve the defining constraints

- Compile and launch with `javac` and `java`, normally through checked-in `@argfiles`. Do not add Maven, Gradle, Ant, or a wrapper around them.
- Use named JPMS modules exclusively. Do not fall back to the class path or automatic modules. Use supported JDK APIs; do not use compiler/runtime internals, `Unsafe`, or reflection hacks.
- Put source modules in `src/<module-name>/`; put linked source dependencies in `lib/src/`; put deliberate binary modular JARs in `lib/bin/`; put generated classes in ignored `out/`.
- Prefer JDK APIs and small, explicit code. Admit third-party code only for clear leverage, record it explicitly on the module path, and keep the dependency boundary narrow.
- Express module relationships in `module-info.java`. Export only intended API packages; use qualified `opens` only where runtime discovery requires it.
- Use Java `ServiceLoader` for replaceable implementations: stable contracts belong in a service-catalog module, providers declare `provides ... with ...`, and consumers declare `uses ...`.
- Keep a versioned service-contract package such as `work.archaic.service.<capability>.v01` immutable after publication. Add a new version instead of silently breaking the old one.
- Keep changes small and legible. Avoid generated source, annotation processors, broad reflection, framework lifecycle magic, and configuration whose effect cannot be seen from the command line and module descriptors.

Read [references/conventions.md](references/conventions.md) when creating a project, designing module or service boundaries, adding dependencies, or reviewing architectural fit.

## Work through the repository's public commands

Use the commands documented by the project, typically:

```shell
javac @args/compile
java @args/test
java @args/run
```

Run the compile command first because tests and execution consume `out/`. Do not substitute an IDE build. If a command fails, diagnose the module graph, source links, JDK version, and argument files before changing application code.

Read [references/workflows.md](references/workflows.md) for exact creation, extension, dependency, service-provider, testing, and troubleshooting workflows.

## Make coherent changes

When adding or changing functionality:

1. Identify the owning module and whether the change is implementation, public API, service contract, provider, CLI, or test-only behavior.
2. Change the narrowest suitable package and public surface.
3. Update `module-info.java` and the relevant compile/run/test argument files together with the code.
4. Add or update a separate test module. For new Minau tests, prefer testing v02: a public suite record registers package-private case records through `cases(Collection<TestCase>)`; each case implements `run(TestTrail)`. Use Java `assert condition : "reason";` for every assertion, with a short explanation of the violated expectation that makes it a test failure, and run with `-ea`. Do not create assertion helper methods or assertion wrappers; keep checks inline or propose an extension to the test API. Preserve existing v01 annotated suites unless migration is requested, and check that the selected catalog and runner support v02. See the testing workflow for a complete example.
5. Compile, test, and run the relevant entry point. Also run lint or documentation commands when present.
6. Inspect the final diff and confirm no compiled output, downloaded JDK, or incidental dependency files entered version control.

Do not create a service abstraction for code with no plausible alternate provider. Conversely, do not bypass an existing service contract by importing a provider implementation directly.

## Apply the Archaic Java design system

When designing a website, web application, or documentation interface with this skill, read [references/design-system.md](references/design-system.md). Apply the shared visual identity unless the user or existing project specifies another design. This visual system can also be requested independently of a Java implementation; it does not choose a frontend framework or impose Java build conventions on another stack.

Use [assets/design-system/foundation.css](assets/design-system/foundation.css) as a small starting point and [assets/design-system/index.html](assets/design-system/index.html) as the accepted visual specimen. Sans serif establishes structure; serif explains; monospace specifies. The reference defines Riot's identity and permitted variations.

## Document only the mechanics users need

Keep `AGENTS.md` concise and operational: project name, required JDK, build-tool prohibition, dependency locations, and unusual constraints. Keep `README.md` focused on purpose and the canonical build, test, and run commands. Document public modules, packages, and APIs with Javadoc where the contract is not obvious from types alone.

## Verify completion

Report the exact checks run and their results. For Java changes, the JPMS graph must compile, tests must run with assertions enabled, and the intended module entry point must launch successfully. For visual-only changes, verify the affected layout, assets, and interactions as described in the design-system reference; no Java build is required when Java behavior is untouched.
