# agent-skills

Portable agent skills and plugin bundles for recurring research, project, and study workflows.

## Start here

- [Architecture](docs/architecture.md): ownership, repository boundaries, dependencies, and cross-agent packaging.
- [Lecture Tools specification](packages/lecture-tools/SPEC.md): the first multi-engine workflow for turning selected lecture and course evidence into a verified project.
- [Writing Tools specification](packages/writing-tools/SPEC.md): logical, evidence-grounded writing for reports, papers, presentations, and design documents.
- [Repository operations](docs/repository-operations.md): the draft workflow for issues, plans, agent execution, review, merge, and release.

## Packages

| Package | Status | Purpose |
| --- | --- | --- |
| `lecture-tools` | Experimental, not runnable | Build a verified project from authorized course evidence and selected requirements. |
| [`writing-tools`](packages/writing-tools/SPEC.md) | Experimental package; canonical skill usable as text | Draft and review logical narratives, with synthetic feedback cases and an improvement loop. |

Packages own higher-level agent-facing behavior and integration contracts; an engine may also ship at most one thin, single-tool skill that translates user intent to its own stable CLI or contract. Standalone engines remain in their own repositories and expose versioned JSON CLIs.
Direct lecture catalog and playback requests remain in the campusctl engine-owned skill; they are not duplicated in this bundle.

`lecture-tools` remains experimental and non-runnable: its multi-engine runtime is not implemented. Do not install this package for operational use.

`writing-tools` provides [canonical logical-writing instructions](packages/writing-tools/skills/logical-writing/SKILL.md) without a runtime or external engine dependency. No target-specific plugin or installation compatibility is claimed.

## External repositories

Human-facing documentation links to the canonical GitHub repository. Machine-readable bundle metadata records the repository, required contract, and release status separately. Released profiles pin tags or commits; mutable branches are allowed only for unreleased development.

## License

This repository is licensed under the [MIT License](LICENSE).
