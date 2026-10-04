---
name: archaic-java
description: "Create, maintain, extend, diagnose, or review projects in the school of Archaic Java: JDK 25, explicit JPMS modules, javac/java argument files, source-linked or modular-JAR dependencies, JDK-first implementations, object logging with configured caller-thread contexts, Minau tests, versioned service contracts, shared project documentation through maintenance skills, and the Archaic Java web design system. Use for archaic.work repositories or when the user explicitly asks for Archaic Java conventions or visual design. Do not impose these conventions on unrelated Java projects."
---

# Archaic Java

Develop Java systems whose structure and mechanics remain visible in the repository. Prefer the JDK and JPMS over build tools, frameworks, class paths, generated configuration, and hidden dependency injection.

## Establish the local contract

Before changing a repository:

1. Read every applicable `AGENTS.md`, the project `README.md`, and, when present, its maintenance-skill foundation. Follow the task links relevant to the change; do not load every reference.
2. Inspect `args/` or `cmd/`, `module-info.java`, `lib/src`, `lib/bin`, `.gitignore`, and the test modules.
3. Check `java --version`, `javac --version`, and the worktree status.
4. When a change touches a service boundary, read the relevant catalog skill, contract guide and Javadoc at the dependency revision actually used.
5. Treat the repository as the authority. Preserve its chosen JDK version, naming, command-file directory, and established variations. For a new project, use JDK 25 unless the user chooses another version.

User instructions and repository-local instructions take precedence over this skill. Do not modernize an old repository merely to make it resemble another Archaic Java project.

## Preserve the defining constraints

- Compile and launch with `javac` and `java`, normally through checked-in `@argfiles`. Do not add Maven, Gradle, Ant, or a wrapper around them.
- Use named JPMS modules exclusively. Do not fall back to the class path or automatic modules. Use supported JDK APIs; do not use compiler/runtime internals, `Unsafe`, or reflection hacks.
- Put source modules in `src/<module-name>/`; put linked source dependencies in `lib/src/`; put deliberate binary modular JARs in `lib/bin/`; put generated classes in ignored `out/`.
- Prefer JDK APIs and small, explicit code. Admit third-party code only for clear leverage, record it explicitly on the module path, and keep the dependency boundary narrow.
- Express module relationships in `module-info.java`. Export only intended API packages; use qualified `opens` only where runtime discovery requires it.
- For service-loaded replaceable implementations, use Java `ServiceLoader`: stable contracts belong in a service-catalog module, providers declare `provides ... with ...`, and consumers declare `uses ...`.
- Keep a versioned service-contract package such as `work.archaic.service.<capability>.v01` immutable after publication. Add a new version instead of silently breaking the old one.
- Keep changes small and legible. Avoid generated source, annotation processors, broad reflection, framework lifecycle magic, and configuration whose effect cannot be seen from the command line and module descriptors.

Read [references/conventions.md](references/conventions.md) when creating a project, designing module or service boundaries, adding dependencies, or reviewing architectural fit.

## Log with configured contexts

Use `work.archaic.service.logging.v03` and Culpa for new application logging. Prefer objects implementing `Logging`, lazy debug computations and configured contexts around complete application intents. Keep provider selection explicit and application policy local. Read [application logging guidance](references/logging.md) for composition and migration, then the linked catalog references for exact API semantics and provider guidance for implementation details.

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
4. Add or update a separate test module. Prefer testing v02 and record-based Minau cases when supported by the selected dependencies. Keep assertions inline with a short explanation and run with `-ea`; do not introduce assertion wrappers. Preserve existing v01 suites unless migration is requested. Follow the [testing conventions](references/conventions.md#testing) and [testing workflow](references/workflows.md#add-minau-tests), which route to contract and runner details.
5. Compile, test, and run the relevant entry point. Also run lint or documentation commands when present.
6. Update the documentation that owns any changed behavior, policy, command or verification. Follow [documentation ownership](references/documentation.md), including compatibility review for changed contract expectations.
7. Inspect the final diff and confirm no compiled output, downloaded JDK, or incidental dependency files entered version control.

Do not create a service abstraction for code with no plausible alternate provider. Conversely, do not bypass an existing service contract by importing a provider implementation directly.

## Apply the Archaic Java design system

When designing a website, web application, or documentation interface with this skill, read [references/design-system.md](references/design-system.md). Apply the shared visual identity unless the user or existing project specifies another design. This visual system can also be requested independently of a Java implementation; it does not choose a frontend framework or impose Java build conventions on another stack.

Use [assets/design-system/foundation.css](assets/design-system/foundation.css) as a small starting point and [assets/design-system/index.html](assets/design-system/index.html) as the accepted visual specimen. Sans serif establishes structure; serif explains; monospace specifies. The reference defines Riot's identity and permitted variations.

## Organize shared documentation

Provide a project maintenance skill as the shared entry point for humans and coding agents, normally `skills/maintain-<project>/SKILL.md`. Link to it from `README.md` and `AGENTS.md`. Keep its foundation and task map compact; integrate detailed project documentation into its `references/` and load it by task. Scale the structure to the project and migrate older documentation incrementally when in scope.

Keep shared engineering conventions in Archaic Java, portable contracts and conformance expectations in the relevant catalog, and implementation/configuration guidance in the owning project. Keep precise API declarations and Javadoc beside their source. Short linked summaries and examples are useful; avoid independently maintained specifications of the same fact. Read [references/documentation.md](references/documentation.md) when creating, reorganizing, reviewing or resolving ownership of documentation.

## Verify completion

Report the exact checks run and their results. For Java changes, the JPMS graph must compile, tests must run with assertions enabled, and the intended module entry point must launch successfully when the project has one. For visual-only changes, verify the affected layout, assets, and interactions as described in the design-system reference; no Java build is required when Java behavior is untouched.

Check that a contributor can find the relevant constraints, owning code and verification through the project skill. Validate changed links and examples, check dependency revisions, and confirm conformance assertions attest documented promises. For documentation-only changes, validate structure, links and claims against source; run Java checks when changed examples or guarantees require execution.
