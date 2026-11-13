# Writing Tools Specification

## 1. Outcome and scope

Understood as: provide one reusable logical-writing skill for drafting and reviewing reports, papers, presentations, and design documents, with synthetic feedback cases and a documented improvement loop.

`writing-tools` is the package and future release unit for `logical-writing`. The skill helps a reader understand a conclusion, assess its evidence, and make the intended judgment. It applies across subjects and document formats; it does not impose a fixed number of arguments or a universal document template.

The canonical skill instructions are usable as text. Package release and platform installation remain experimental: no target-specific plugin, adapter, executable runtime, external engine dependency, or automatic verification service is supplied or claimed. Writing guidance does not replace source verification, domain review, or a venue's formatting requirements.

## 2. Canonical layout

- `bundle.toml`: package identity, version, status, and canonical target; no runtime or engine tables because neither is required.
- `skills/logical-writing/SKILL.md`: trigger-specific operational rules and reference routing.
- `skills/logical-writing/references/checklist.md`: final review questions.
- `skills/logical-writing/references/feedback-cases.md`: synthetic examples of writing failures and repairs.
- `skills/logical-writing/references/related-skills.md`: boundaries with complementary review capabilities, not dependencies.
- `skills/logical-writing/references/feedback-log.md`: privacy-safe feedback recording and rule-promotion procedure.
- `changelog.d/`: package-owned user-visible change fragments.

Canonical skill content is platform-neutral. Any future installation metadata belongs in a separately validated native manifest or adapter, not in the writing rules. Package identity and version live in `bundle.toml`; the repository catalog registers the package for discovery.

## 3. Behavior contract

Start with the reader's judgment and an explicit conclusion. Organize a small set of supporting claims before polishing sentences. Include a sentence only if later explanation or the reader's judgment needs it; retain material warnings and limitations.

For an explanation of a decision, connect observation, diagnosis, hypothesis, action, result, and next implication. A proposal states its verification plan instead of pretending an outcome has been observed. Comparisons identify the baseline, changed and fixed conditions, and decision criterion. Observed association is not automatically a causal explanation.

Define terms consistently and identify each number's population, numerator, denominator, and unit. Distinguish verified facts, post-hoc justification, inference, and missing evidence. Do not fabricate reasons or cite an unopened source. Abstract analysis into decision-relevant propositions without discarding contrary evidence.

References supplement rather than redefine the core rules. All examples and feedback entries must be synthetic and contain no identifying details, raw evidence, private infrastructure, or local filesystem paths. No original source guide is moved, deleted, or replaced by this package.

## 4. Evolution and validation

Record new feedback using [the feedback log](skills/logical-writing/references/feedback-log.md). Promote recurring, independently supported lessons into the canonical skill; keep exceptional cases in references. Every user-visible change adds a package-local fragment following [repository operations](../../docs/repository-operations.md#9-changelog-and-release).

Before handoff, validate skill frontmatter and reference resolution, parse bundle and catalog TOML, check package/catalog agreement and fragment syntax, and scan every changed file for forbidden information. Follow repository-provided validation commands when available. The current repository supplies policy requirements, but no local validator script or executable test suite; do not claim platform compatibility without its own validation.
