# agent-skills

Portable agent skills and plugin bundles for recurring research, project, and study workflows.

## Start here

- [Architecture](docs/architecture.md): ownership, repository boundaries, dependencies, and cross-agent packaging.
- [Lecture Tools specification](packages/lecture-tools/SPEC.md): the first bundle, combining lecture discovery and playback with local audiovisual evidence extraction and lecture-to-code generation.
- [Repository operations](docs/repository-operations.md): the draft workflow for issues, plans, agent execution, review, merge, and release.

## Packages

| Package | Status | Purpose |
| --- | --- | --- |
| `lecture-tools` | Experimental, not runnable | Check and play selected lectures, then build a verified project from authorized course evidence and selected requirements. |

Packages own agent-facing behavior and integration contracts. Standalone engines remain in their own repositories and expose versioned JSON CLIs.

`lecture-tools` currently documents and validates the intended skill boundary. Do not install it for operational use until `bundle.toml` names tagged engine releases and published contracts.

## External repositories

Human-facing documentation links to the canonical GitHub repository. Machine-readable bundle metadata records the repository, required contract, and release status separately. Released profiles pin tags or commits; mutable branches are allowed only for unreleased development.

## License

This repository is licensed under the [MIT License](LICENSE).
