# AGENTS.md

`agent-skills` is the canonical catalog for reusable agent skills and multi-skill bundles.

- Keep engine implementation and credentials out of this repository.
- Keep each behavior in one narrowly triggered `SKILL.md`.
- Put cross-engine product behavior in the owning package specification.
- Link to engine-owned contracts instead of copying their schemas or operational internals.
- Treat `bundle.toml` as the machine-readable source for external dependencies.
- Do not release a package that depends on a mutable branch or an unavailable private contract.
- Validate every changed skill and target-specific plugin manifest before release.

Commit messages use `type(scope): summary` in English. Stage exact paths only.
