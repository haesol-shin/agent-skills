# Repository Operations Standard

> **Status:** Draft v0
>
> **Purpose:** Define one production-quality operating model for repositories managed by a solo maintainer and implemented primarily by coding agents.

## 1. Operating principles

- The maintainer defines outcomes, required behavior, non-goals, acceptance criteria, risk, and authorization; agents own delegated planning, execution, verification, and pull request preparation.
- Every mutation is traceable through an issue-backed or explicitly permitted direct-pull-request scope, an isolated branch and worktree, commits, verification evidence, and a merge decision.
- Planning and implementation are separate gates for medium- and high-risk work; approval of an issue does not authorize an unreviewed implementation plan.
- An agent may prepare a branch and pull request but cannot approve or merge its own work. It may tag, publish, deploy, migrate, perform destructive cleanup, or change production credentials only when a separately recorded maintainer approval names that exact operation and authorizes the agent as executor.
- Repository merge, release, and production deployment are separate authorities; completing one never implies permission for the next.
- Policies are enforced by repository rules and CI where practical; prose-only rules must be treated as advisory until enforcement exists.
- Process complexity must be proportional to risk; a small documentation fix does not require the same review depth as authentication, public contracts, destructive behavior, or releases.

The operating model draws on [OpenAI's harness-engineering practice](https://openai.com/index/harness-engineering/), [Anthropic's guidance on simple composable agent patterns](https://www.anthropic.com/engineering/building-effective-agents), and the [Learn Harness Engineering](https://walkinglabs.github.io/learn-harness-engineering/ko/) curriculum. These are design inputs, not normative dependencies.

## 2. Policy ownership

The future public `haesol-shin/.github` repository is the canonical home for account-wide contribution guidance, security reporting, issue and pull request templates, reusable workflows, and this operating standard. Each product repository keeps only repository-specific commands, boundaries, and release details.

GitHub-provided default community files affect the contribution interface but are not copied into repository clones, so every instruction required by a local agent remains in that repository's `AGENTS.md` or `CONTRIBUTING.md`.

| Concern | Canonical owner |
| --- | --- |
| Account-wide workflow and review policy | `haesol-shin/.github` |
| Default issue and pull request templates | `haesol-shin/.github` |
| Reusable CI and release workflows | `haesol-shin/.github` |
| Engine-neutral agent roles, workflow contracts, prompts, and provenance schemas | `agent-skills`, initially as a proposed repository-operations package |
| Repository-specific agent rules and safety boundaries | Repository `AGENTS.md` |
| Repository-specific development commands | Repository `CONTRIBUTING.md` |
| Product release procedure and supported artifacts | Repository `RELEASE.md` |
| User-visible release history | Repository `CHANGELOG.md` |
| Cross-repository product compatibility | Owning bundle's `bundle.toml` |

Each repository declares the adopted standard version in `.github/repo-policy.yml`. Shared workflows are referenced by an immutable commit SHA; branches and tags are not trusted as immutable workflow inputs.

## 3. Work item lifecycle

```text
Interactive intake -> issue or permitted direct work -> risk and planning route -> optional plan and critique -> maintainer approval -> isolated execution -> verification -> code review -> maintainer merge -> optional separately approved release
```

The maintainer owns readiness, risk, plan or design approval, exceptions, merge, release, and production authorization. Implementation details are delegated unless they alter scope, public contracts, security boundaries, cost, or acceptance criteria.

| Gate | Decision owner | Permitted operator |
| --- | --- | --- |
| Mark issue ready and confirm proposed risk | Maintainer | Maintainer or agent recording the maintainer decision |
| Approve plan or design | Maintainer | Maintainer records approval; Planner records later revisions |
| Accept independent review | Maintainer | Independent reviewer reviews; executor resolves findings |
| Merge pull request | Maintainer | Maintainer |
| Approve release and exact release commit | Maintainer | Maintainer or explicitly authorized release automation |
| Deploy, migrate, clean up destructively, or change credentials | Maintainer | Named agent, automation, or maintainer |

An approval is valid only when recorded in the relevant GitHub issue or pull request by the maintainer and names the artifact and operation. Plan and design approvals bind to the plan path and the commit SHA containing the approved revision; review acceptance, merge approval, and release approval bind to an exact head SHA. A plan edit invalidates its approval. Ordinary implementation commits do not invalidate an approved plan unless they change its scope, contract, security boundary, or release target; any new commit invalidates head-bound review and merge evidence. Chat discussion may shape a proposal but is not durable approval until recorded there. The one exception is initial authorization for direct low-risk work: the maintainer may give it in the dispatching conversation, and the executor must reproduce the authorized scope and rationale in the first draft pull request before requesting review. Creating an issue never authorizes product implementation. `status:planning` permits only an isolated branch or worktree, plan-file edits, and plan-only commits; `status:ready` authorizes issue-backed product implementation against the approved plan revision.

The initial shared labels are `status:triage`, `status:planning`, `status:ready`, `status:executing`, `status:reviewing`, `status:merge-ready`, `status:blocked`, `risk:low`, `risk:medium`, and `risk:high`. Merged state is represented by the closed pull request rather than a persistent label. An account-level GitHub Project may be added when cross-repository priority can no longer be managed clearly through labels and queries.

```text
triage -> planning -> ready -> executing -> reviewing -> merge-ready -> merged
triage -> ready
reviewing -> executing
any active state -> blocked
blocked -> prior state after maintainer-recorded resolution
```

### 3.1 Issue contracts

An implementation issue states what must become true, not how to code it. The maintainer may write it directly or approve a draft produced through an interactive agent conversation.

```markdown
## Outcome
The user-visible or operational result.

## Must have
- Required behavior.
- Required verification.

## Non-goals
- Explicitly excluded work.

## Done when
- Conditions the maintainer can verify.

## Dependencies
- Related contracts, repositories, or releases.

## Planning
direct | brief | full

## Risk
low | medium | high

## Risk rationale
```

Bug issues use the following contract.

```markdown
## Actual behavior

## Expected behavior

## Reproduction
1.

## Affected version

## Environment

## Failure evidence

## Done when
-

## Planning
direct | brief | full

## Risk
low | medium | high
```

Planning depth and risk are assessed separately, but medium- and high-risk work requires at least a `brief` plan. `direct` means a low-risk accepted scope can be implemented as one clear reviewable change without a separate plan; `brief` means bounded investigation or a small design choice is required; `full` means intent, architecture, sequencing, or cross-repository coordination requires explicit planning. Maintainer interview is triggered by unclear outcome, constraints, non-goals, or acceptance criteria rather than estimated size. Work that cannot remain one coherent, independently verifiable pull request is split before it becomes Ready.

Cross-repository initiatives use one coordinating issue in the integration-owning repository and linked implementation issues in each affected repository. New features and structural changes require issue-backed intake; reproduced bugs may enter through a focused bug issue; ambiguous ideas remain in interactive discussion until their outcome is clear; trivial documentation and clearly low-risk maintenance may proceed directly to a pull request.

### 3.2 Plan contract

`brief` and `full` work keeps its mutable plan in the affected repository at `.ops/plans/<issue>-<slug>.md`; plans from different repositories are never combined into one directory. The issue remains the outcome contract and the plan references it instead of copying its outcome or non-goals. The Planner owns plan revisions, the Critic returns read-only findings, and their intermediate artifacts remain in the runtime ledger rather than GitHub comments. The accepted plan is committed on the issue branch before `status:ready`; the issue records its path, commit SHA, and maintainer approval. Plan documents remain in the repository after merge as decision history.

```markdown
# Plan: <title>

## Inputs
- Issue: #<issue>
- Specification:
- Research:

## Decision drivers
1.

## Options considered

### Option A

### Option B

## Chosen approach

## Contract and dependency impact

## Execution slices
1.

## Verification

## Rollout

## Rollback or mitigation

## Open decisions

## Pre-mortem
<required for high-risk work>
```

Planning stops when the Critic accepts the same plan revision that is presented to the maintainer. Planner-Critic revision is limited to three iterations per planning run; reaching the limit sets the work to `status:blocked` and requires the maintainer to split scope, clarify intent, or approve a documented exception. Any later change to the approved plan file, scope, public contract, security boundary, or release plan returns the work to planning and requires renewed approval against the new plan commit.

For high-risk work, the pre-mortem covers trust boundaries, credible failure modes, compatibility and migration impact, recovery, and required security evidence.

### 3.3 Pull request contract

Every pull request represents one coherent outcome with an explicit rollback or mitigation strategy and contains the following sections.

```markdown
## Outcome
## Scope and non-goals
## Plan
<plan path and approved commit SHA; omit for direct work>
## Contract and dependency impact
## Verification
## Risk and rollback
## Changelog impact
none | added | changed | fixed | deprecated | removed | security

<user-facing entry when applicable>
## Related
Closes #<issue> | Direct low-risk PR: <rationale>
```

Verification records exact commands and results, manual behavior checked, CI evidence, and anything not verified. Checkboxes without results are not evidence. A new commit invalidates prior head-specific review evidence and requires the applicable checks and review to run again.

## 4. Risk and review

| Risk | Typical changes | Required evidence |
| --- | --- | --- |
| Low | Documentation, tests, internal cleanup with no behavioral contract change | Focused checks, fresh independent agent review, maintainer merge decision |
| Medium | User-visible behavior, dependencies, performance, schemas, compatibility | Approved plan, full relevant CI, fresh independent agent review, maintainer merge decision |
| High | Authentication, credentials, security, destructive operations, public API, cross-repository contract, installer, release, deployment | Approved full plan or design, full CI and realistic smoke evidence, fresh independent adversarial review, explicit rollback, maintainer merge decision |

The planning agent proposes the initial risk with evidence, the maintainer confirms it when marking issue-backed work Ready, and the highest applicable category wins. An agent may raise risk when new evidence appears; lowering risk requires maintainer approval. Discoveries that affect credentials, destructive behavior, public or cross-repository contracts, installers, release, or deployment immediately pause work for reclassification.

Review borrows [Gajae Code's](https://github.com/Yeachan-Heo/gajae-code) role separation, explicit verdicts, severity discipline, immutable change-set binding, and bounded repair. After implementation verification, the head is frozen and the reviewer receives the issue contract, approved plan when present, `base...HEAD` diff, directly relevant contracts, and verification evidence without the authoring conversation. The verdict is `APPROVE`, `COMMENT`, or `REQUEST_CHANGES`, and every finding records severity, evidence, and a concrete correction. The executor cannot review its own work. Findings are consolidated before the executor starts one repair batch; repaired code is verified, frozen as a new review generation, and reviewed again. Review is limited to three generations, after which the work becomes `status:blocked` for maintainer intervention. Every verdict is bound to the exact head SHA, so a new commit invalidates it. Medium- and high-risk reviews should use a different model family from the executor when available.

Adversarial review is the risk-calibrated instruction used for high-risk work, not a separate workflow role, runtime agent, state, or receipt type. It uses the same Reviewer role and review contract with stronger attention to trust boundaries, abuse paths, recovery, and evidence quality.

Full review artifacts and intermediate generations remain beside the runtime ledger and are indexed by it. GitHub receives one final status for the clean head with the verdict, exact head SHA, evidence reference, and reviewer provenance; iterative findings do not accumulate as pull request comments. During the manual pilot this status may be a single final pull request comment. The target automated design publishes a trusted check and makes it a merge prerequisite. If the agent and maintainer use the same GitHub identity, distinct run provenance and exclusive maintainer merge authority provide the initial separation; a least-privilege agent identity or GitHub App may later make identity separation enforceable by GitHub.

Each internal review generation produces a receipt conforming to the following logical contract; the machine-readable schema, not this display form, becomes authoritative when implemented.

```markdown
## Contract version

## Review generation

## Reviewed head

## Verdict
APPROVE | COMMENT | REQUEST_CHANGES

## Findings

### <severity>: <title>
- Evidence:
- Impact:
- Required correction:
- Resolution: fixed | rebutted | accepted-risk

## Verification gaps

## Reviewer provenance
- Runtime:
- Runtime version:
- Model:
- Custom agent or prompt: <optional; omit when using the runtime default>
```

## 5. Branches, worktrees, commits, and merge

- `main` is the only long-lived branch unless sustained parallel release integration demonstrates a need for `dev`.
- Each pull request in each repository uses one isolated worktree and a short-lived branch named `<type>/<issue>-<short-kebab-description>`, for example `feat/42-local-evidence`; one issue may coordinate several linked pull requests with an explicit merge order.
- Valid branch and commit types are `feat`, `fix`, `docs`, `refactor`, `test`, `perf`, `ci`, `build`, `chore`, and `revert`.
- Commit subjects and pull request titles use `type(scope): imperative summary`, for example `feat(lectural): accept local media evidence`.
- Stage exact paths, review the staged diff, and never mix unrelated user changes into the branch.
- Pull requests use squash merge so the pull request title becomes the permanent `main` commit; merge commits are disabled and noisy intermediate commits do not enter `main`.
- Delete the source branch and worktree after merge; stale merged branches are operational drift and should be detected periodically.
- Direct pushes, force pushes, deletion, and history rewriting of protected branches and release tags are prohibited.

## 6. Repository rules

Every active repository protects `main` with a GitHub ruleset that requires a pull request, linear history, the stable aggregate `quality-gate` status, resolved review conversations, and blocks force pushes and deletion. Automatic merge is disabled. Required approving reviews are enabled only when PR authorship uses an identity distinct from the maintainer; otherwise the maintainer remains the sole merge authority and the limitation is documented.

Tag rules protect `v*` for distributable repositories. The only permitted creation path is the release workflow or maintainer identity named by the recorded release approval; a release tag must reference that approved, verified commit reachable from protected `main` and must not be moved or recreated.

## 7. Continuous integration

Every repository exposes one stable required status named `quality-gate`, even when its internal jobs differ. During the manual pilot, it aggregates implemented repository CI only; one final exact-head pull request comment records the manual policy and review result. After `repo-ops-gate` exists and has been validated in advisory mode, `quality-gate` also requires it to validate the adopted contract version, required issue and plan approval, iteration ceilings, unresolved blockers, final review receipt, and exact head binding. Pull request workflows use read-only permissions by default, receive no production secrets, pin third-party Actions to reviewed immutable revisions, and rerun for every new head.

### 7.1 Common checks

- Formatting, linting, and static validation appropriate to the repository.
- Unit and contract tests covering the changed behavior.
- Secret and forbidden-file checks.
- Manifest, schema, generated-file, and source-of-truth consistency checks where applicable.
- Clean-worktree verification after tests.
- A bounded smoke test of the public entrypoint.

### 7.2 Repository profiles

| Profile | Required additions |
| --- | --- |
| Python engine | Locked installation, supported Python versions, Windows and Linux tests, CLI JSON smoke |
| Campus provider | Mocked browser and credential tests, redaction tests, capability and status mapping contracts |
| Agent plugin or catalog | Skill validation, target manifest validation, dependency contract and compatibility checks |
| Operations repository | Configuration validation, shell checks, secret-free dry runs, production separation checks |

Tests requiring real LMS accounts, credentials, paid external services, or production systems remain opt-in and never run on untrusted pull requests. Their absence is recorded under unverified items and, when release-critical, must be supplied as explicit release evidence.

Shared workflows live in `haesol-shin/.github` and are versioned. Each repository contains a thin caller workflow and repository-specific commands in `.github/repo-policy.yml`; shared workflows must not infer arbitrary commands or receive inherited secrets by default.

## 8. Dependencies and cross-repository contracts

- Each runtime owns its dependency manifest, lockfile, environment, performance benchmarks, and release cadence.
- Repositories integrate through versioned CLI, JSON schema, file manifest, or protocol contracts rather than source imports or shared environments.
- The contract-owning repository publishes schemas and fixtures; consumers link to the owner and maintain compatibility tests without becoming a second schema owner.
- Released integrations pin exact compatible tags or commits in the owning bundle; mutable branches are development-only and block release.
- Dependency updates are isolated PRs. Heavy or compatibility-sensitive upgrades require lock regeneration, realistic smoke tests, and an explicit rollback version.
- Reusable implementation code is extracted only after at least two independent consumers prove a stable boundary, test suite, versioning need, and release owner.

## 9. Changelog and release

Repositories that publish installable software, plugins, libraries, CLIs, or versioned contracts use Semantic Versioning, annotated `vX.Y.Z` tags, `CHANGELOG.md`, and GitHub Releases. Operations-only repositories may use dated, append-only production change records instead of artificial product releases.

User-visible pull requests add an entry under `## [Unreleased]`; other pull requests state `none` in the changelog-impact field. Released changelog sections are not rewritten except for a separately reviewed correction or security redaction that preserves an audit note. A release pull request moves relevant entries into a dated version section, updates the single canonical version source and any validated mirrors, and includes compatibility, upgrade, known limitations, and rollback or mitigation information.

The release sequence is:

1. Merge a release PR into protected `main` after the normal quality gate.
2. Validate the intended release commit's version agreement, ancestry from `main`, changelog presence, and complete release test graph.
3. Build candidate artifacts from that exact commit in CI, generate checksums, and perform clean-install and public-entrypoint smoke tests.
4. After the aggregate release gate and maintainer approval succeed, create the annotated tag on the validated commit and publish the already-validated artifacts.
5. Create a GitHub Release from the changelog with artifacts, checksums, compatibility, upgrade, known limitations, and rollback instructions.
6. Preserve the previous release as a candidate rollback target, document data and schema compatibility, and provide mitigation when downgrade is unsafe; never move or recreate an existing tag.

Release notes use the following structure.

```markdown
## Added
## Changed
## Fixed
## Compatibility
## Upgrade
## Known limitations
## Rollback
```

For a multi-repository product, release engines and providers first, validate their published contracts, update exact versions in the integration bundle through a separate pull request, run the supported compatibility matrix, and release the bundle last.

## 10. Security and production

- Security reports use GitHub private vulnerability reporting or the address named in `SECURITY.md`, never a public issue.
- Credentials, tokens, cookies, browser profiles, and unsanitized sensitive production data never enter issues, pull requests, logs, fixtures, artifacts, or repositories; approved operational evidence must be minimized and redacted.
- Production changes require a separate approved runbook with exact target, preconditions, backup or rollback, commands, expected evidence, and post-change verification.
- A source merge or release never authorizes deployment, credential changes, migrations, or destructive cleanup.
- CI uses least privilege; write permissions are limited to the specific release job that requires them.
- The supported security-update window is stated per product rather than implied.

## 11. Agent workflow and execution

Agent responsibilities are engine-neutral. A workflow role defines responsibility, a runtime names the tool executing it, a model identifies the exact provider and model card selected for that run, and a run is one concrete execution against an issue, plan, or pull request. Runtime agents, model roles, and profiles remain runtime-managed implementation details unless a run uses a custom agent or prompt that materially changes its behavior.

| Workflow role | Responsibility |
| --- | --- |
| Planner | Own interactive intake, planning route, plan creation, revision, and decomposition without implementing product changes |
| Critic | Review a plan without editing it and return an approval or actionable blockers |
| Executor | Perform only approved implementation or research work and produce verification evidence |
| Reviewer | Review the frozen implementation without editing it and return evidence-backed findings and a verdict |

The standard adopts the [Gajae Code](https://github.com/Yeachan-Heo/gajae-code) design philosophy without requiring its runtime: interactive intake clarifies intent; a writer-reviewer loop stabilizes the plan; execution advances bounded work with evidence; a second writer-reviewer loop verifies the frozen implementation; and the maintainer owns authorization. The workflow deliberately keeps four roles and does not create separate research, QA, security, or repository-operation roles unless repeated evidence proves an independently triggered contract. Specialized runtime agents such as scouts and security reviewers remain helpers beneath these roles. Any copied or adapted prompt text must be compatible with its source license and retain required attribution; otherwise only the operating concept is reimplemented.

### 11.1 OMP adapter

[OMP](https://github.com/can1357/oh-my-pi) is the preferred interactive runtime, but the workflow also permits Codex, Claude Code, Gajae Code, or another compatible runtime. OMP separates runtime agents from model roles: prompts and tools belong to agents, while roles such as `plan`, `slow`, and `smol` select configured models. `task` is an agent, not a model role. The adapter reuses OMP rather than replacing its prompts.

| Workflow role | OMP surface | Model role |
| --- | --- | --- |
| Planner | Main interactive session | `@plan` when an explicit planning model is selected |
| Critic | One custom read-only `plan-critic` agent | `@slow` |
| Executor | Bundled `task` agent | Agent definition, configured override, or inherited model |
| Reviewer | Bundled `reviewer` agent | Agent definition, configured override, or inherited model |

The maintainer currently coordinates work through OMP's main interactive session and invokes `task`, `reviewer`, `plan-critic`, `scout`, or `sonic` only as temporary internal helpers. These helpers do not become separately managed repository actors. The custom `plan-critic` exists because OMP's bundled `reviewer` is patch-oriented rather than a plan reviewer. Repository operations supplies dispatch inputs, artifact schemas, iteration gates, and output normalization around these native agents; it does not copy their runtime prompts. Exact model selection remains runtime-configurable and is recorded with the provider-qualified identifier, such as `openai/gpt-5.6-luna`.

### 11.2 Contract and enforcement

One versioned, machine-readable workflow contract is the source of truth for required fields, allowed state transitions, role permissions, iteration limits, and artifact schemas. Templates are generated from or validated against that contract. Prompts and runtime adapters reference its contract ID and version instead of restating schemas. CI validates changed templates, prompts, adapters, and sample artifacts together so their required fields and verdicts cannot drift independently.

The durable states are `triage`, `planning`, `ready`, `executing`, `reviewing`, `merge-ready`, `merged`, and `blocked`. A direct change may move from `triage` to `ready` without a plan; all other transitions require the applicable issue, approval, approved plan commit, verification receipt, review receipt, or maintainer decision. GitHub is the source of truth for work state and human authorization. The runtime ledger is the source of truth for internal planning and review generations; it must not create a competing approval state.

During the manual pilot, each repository keeps its untracked ledger at `.ops/runtime/<issue-or-direct>/<run-id>/ledger.jsonl` and excludes `.ops/runtime/` from version control. Each append-only record contains the contract version, run ID, generation, role, artifact type, artifact-relative path, SHA-256 digest, timestamp, and provenance. The final GitHub status cites the run ID and digest of the final receipt. The trusted host persists and backs up this directory; after restart, missing or mismatched ledger evidence blocks further mutation until the maintainer reconciles it. Future automation may replace this storage location without changing the logical ledger contract.

The manual pilot enforces irreversible boundaries first: protected branches, maintainer-only merge, release separation, exact-head CI, and durable approval records. Future automation may enforce runtime transitions, operation IDs, per-issue execution locks, resumable reconciliation, and trusted GitHub checks. On restart or uncertain state, the runtime reads GitHub state, the approved plan commit, head SHA, and the ledger before mutation; it never repeats a mutating operation merely because the prior session response is missing.

### 11.3 Run provenance

Every material run records the runtime name, runtime version or exact development commit, provider-qualified model ID, related issue or pull request, base SHA, and head SHA. A custom runtime agent or prompt is recorded only when it materially differs from the runtime default; runtime-managed roles, helper agents, model-role aliases, reasoning effort, and default prompt versions are not duplicated in repository provenance. This document owns the requirement; a future repository-operations specification and machine-readable schema will own the exact representation if the workflow package is implemented in `agent-skills`.

[Herdr](https://github.com/herdrdev/herdr) may run on `pi-server` to keep OMP or other runtime sessions alive, expose their state, and permit remote reconnection; it is execution infrastructure, not the workflow authority or task ledger. GitHub remains the durable source for work contracts and approvals. Automatic dispatch from `status:ready` is deferred until a manual end-to-end pilot proves the required checks, concurrency limits, reconciliation behavior, and interruption handling.

If unattended operation is later justified, a persistent external orchestrator may follow Gajae Code's public operating pattern by accepting work, starting an isolated runtime session or worktree for each task, and collecting its terminal evidence. That orchestrator is an optional future component, not the OMP main session, Herdr, or a component assumed by this standard.

[Paperthin](https://github.com/LilMGenius/paperthin) skills are optional reasoning, review, quality, and retrospective aids. They do not own workflow state or approval. A run must not operate two competing planning or execution state machines, such as Gajae-style durable planning and Paperthin `re0-plan` or `re0-loop`, while independent critique and retrospective skills may supplement either workflow.

### 11.4 Agent operating contract

- The agent starts from an approved issue or a clearly low-risk direct-pull-request scope, reads repository instructions, inspects live state, and reports any conflict between the approved scope and current code before mutation.
- The agent creates or uses the issue-specific worktree and branch, changes only in-scope files, and preserves unrelated user work.
- The agent records implementation decisions in the pull request, not only in chat context.
- The agent runs the smallest relevant checks during development and the complete required profile before handoff.
- The agent performs a fresh self-review of the final diff and every code pull request receives a separate fresh-context reviewer calibrated to its risk.
- The agent may open or update a draft pull request and respond to review findings but must not approve its own work, merge, tag, release, deploy, or alter credentials without explicit authority.
- If execution state is uncertain, the agent reconciles it instead of repeating a potentially mutating operation.

### 11.5 Operational terms

- A material run can mutate files or external state, approve or review work, or produce evidence used for a gate.
- Fresh context means the reviewer receives the approved contract, relevant artifacts, diff, and evidence but not the authoring conversation or hidden rationale.
- Full relevant CI means every required check in the repository's declared profile for the affected surfaces.
- Realistic smoke evidence exercises the public entrypoint against the closest safe environment to actual use and records the result and known gap.
- Accepted risk is a specific unresolved finding that the maintainer explicitly records and accepts for the exact head SHA; silence or merge alone is not acceptance.

## 12. Exceptions and emergencies

The maintainer may authorize an exception only in the relevant issue or pull request with its reason, exact scope, expiry, compensating checks, and follow-up issue. Emergency security work may shorten planning and review but does not waive secret handling, exact-head verification, rollback or mitigation, or the durable decision record. CI or reviewer unavailability is not implicit permission to merge; the maintainer must record the degraded gate and accept its risk explicitly.

## 13. Adoption plan

1. Review this draft and resolve its open operating decisions; it is not enforceable policy yet.
2. Create the public `haesol-shin/.github` repository, define the repository-policy schema and conformance check, and publish the initial approved standard as version `v0.1.0`.
3. Derive the default `CONTRIBUTING.md`, security policy, issue forms, pull request template, repository policy schema, and reusable CI workflows.
4. Pilot the standard manually in LecturAL through interactive issue drafting, maintainer Ready approval, an isolated OMP run using main session, `task`, `reviewer`, and `plan-critic` only when planning requires it, exact-head fresh review, pull request, merge, and retrospective; use Herdr on `pi-server` only for session persistence and remote observation during the pilot.
5. Decide from pilot evidence whether reusable roles and schemas remain an `agent-skills` package and whether a separate dispatcher or repository is justified; do not create either merely to complete the draft architecture.
6. Revise the standard only from evidence produced by the pilot, then tag the revision.
7. Adopt the pinned standard in `agent-skills`, `campusctl`, `campusctl-cnu`, and other repositories through separate reviewable pull requests.
8. Audit policy drift periodically by comparing each repository's declared standard version, ruleset, required status, workflows, and release configuration.
