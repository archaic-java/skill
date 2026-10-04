# Shared documentation and progressive disclosure

Read this when creating, reorganizing or reviewing documentation, or deciding where
a fact belongs. Use the same readable Markdown entry points for humans and coding
agents. Organize for the smallest sufficient context for a task, including enough
orientation to recognize relevant constraints.

## Contents

- [Choose the authoritative home](#choose-the-authoritative-home)
- [Establish the project's entry points](#establish-the-projects-entry-points)
- [Route readers by task](#route-readers-by-task)
- [Summarize without creating another specification](#summarize-without-creating-another-specification)
- [Maintain and verify](#maintain-and-verify)
- [Follow the examples](#follow-the-examples)

## Choose the authoritative home

| Information | Home |
|---|---|
| Shared engineering conventions and the documentation method | Archaic Java skill and its references. |
| A project's mental model, task routing, implementation, configuration and local verification | That project's maintenance skill and references. |
| Portable APIs, shared data types and provider expectations | The relevant service-catalog skill, contract guides and declarations. |
| Exact type/method contracts and invariants | Javadoc beside their declarations, in the catalog for shared contracts and in the project for its own API. |
| Architectural rationale or a policy decision | A reference in the skill that owns that decision; implementation rationale can also live beside the code. |
| Repository checkout requirements and canonical command invocations | Its README; checked-in command files define executable compiler/launcher options. |
| Portable compliance assertions and actual coverage | Catalog cases tied to public promises; verification evidence records the provider, revisions and checks actually executed. |

A catalog has its own maintenance skill. It organizes shared contracts as well as
maintenance of that repository. Provider algorithms, platform constraints and defaults
allowed by a contract belong to the provider; selection, configuration and application
policy belong to the consumer. Catalog-owned shared mechanics remain in the catalog:
ownership follows the promise rather than whether a file contains implementation code.

## Establish the project's entry points

Use the following conventional locations unless the repository has an established
alternative. Link the actual location from both README and AGENTS.md.

| Location | Responsibility |
|---|---|
| `README.md` | Purpose, first successful use, checkout requirements, canonical commands and maintenance-skill link. |
| `AGENTS.md` | Essential repository instructions and routing to the same skill, including a direct-reading fallback if local skills are not automatically discovered. |
| `skills/maintain-<project>/SKILL.md` | Skill metadata, compact mental model, essential constraints and task-based reading map. |
| Skill `references/` | Integrated detailed project documentation: architecture, behavior, rationale, workflows, troubleshooting and verification. |
| Source Javadoc | Precise API contracts beside the declarations. |
| Checked-in argument files | Executable compiler and launcher specification. |

Keep detailed prose inside the owning skill's references instead of creating a parallel
human documentation tree or agent-only manual. Keep source Javadoc and command files
in their natural locations and link to them. No installation or particular agent tool
should be needed for a human to follow the Markdown.

Scale to the work already present: a small project may need only its entry point and
one reference. Split coherent concepts or tasks when readers benefit from choosing
between them. Do not generate empty documents, arbitrary file counts or nested mazes.

## Route readers by task

Keep essential constraints in the foundation so selective reading cannot bypass them.
Link every reference directly from SKILL.md and explain when to read it. For maintenance
tasks, include the relevant behavior, owning code and verification in the selected guide.
For example: "When changing discovery, read the discovery reference and ModuleScanner;
verify packaged modules and failure before execution."

Read the project skill's foundation first, then relevant task references. Load catalog
guidance only for the affected capabilities and versions. Inspect the actual dependency
checkout or pinned source; links to a repository's moving main branch aid navigation
but do not establish the version used by the project.

Keep shared conventions here instead of copying them into each project. Project skills
may give concise orientation and state deliberate local variations. Ensure readers can
identify and access the shared guidance without requiring a specific agent installation.

## Summarize without creating another specification

Allow short linked summaries and task-specific examples elsewhere. They help readers
recognize why a constraint matters. Keep the detailed rule in one authoritative home;
do not maintain independent specifications that readers must reconcile.

Within a catalog, put precise method/type details beside declarations and use guides
for the connected behavioral model. Tests attest documented promises; a stricter test
or one provider's behavior does not silently create another guarantee. Clarifying an
ambiguity must not conceal an incompatible requirement in a published contract.

If guidance conflicts, determine the fact's owner and compare the declarations, guide
and cases at the selected revision. Flag and correct inconsistencies in that home.
Shared conventions cannot add methods or guarantees to a versioned API. Repository-local
instructions guide the work but do not alter a dependency's published contract.

## Maintain and verify

For a new project, create the entry points with its first working implementation.
For an existing project, follow its established routing and migrate older documentation
incrementally when documentation work is in scope. Do not turn an unrelated fix into
a repository-wide documentation rewrite.

When changing behavior, policy, architecture, commands or verification, update the owning
source/guide in the same change. Update links and task maps when responsibilities move.
Preserve useful rationale; recipes alone do not explain why a boundary must remain.

Before completing a change, check that a contributor can find the relevant constraints,
owning code and verification through the project skill. Validate changed skill metadata,
local links/anchors and examples; compare claims with actual code and dependency revisions.
Check documentation ownership and whether compliance assertions exceed their promises.
Run execution checks when changed examples or guarantees need them; otherwise validate
structure and accuracy. Automated link/metadata checks help, but review must judge whether
the guidance is sufficient. Report actual checks and any coverage or tool limitations.

## Follow the examples

[Minau's skill](https://github.com/archaic-java/minau/blob/main/skills/maintain-minau/SKILL.md)
organizes runner behavior, implementation responsibilities and regression checks.
The [catalog skill](https://github.com/archaic-java/service-catalog/blob/main/skills/maintain-service-catalog/SKILL.md)
organizes versioned contract guides and provider-conformance expectations. For testing,
Archaic Java owns assertion/case-design conventions, the catalog owns suite/case/trail
semantics, Minau owns discovery and execution mechanics, and each consumer owns its
fixtures and launch commands. Apply the ownership pattern, not a fixed document count.
