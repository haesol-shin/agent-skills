# Repository Operations Standard

> **Status:** Draft v0
>
> **Purpose:** Define one production-quality operating model for repositories managed by a solo maintainer and implemented primarily by coding agents.

## 1. Operating principles

- The maintainer defines outcomes, required behavior, non-goals, acceptance criteria, risk, and authorization; agents own delegated planning, execution, verification, and pull request preparation.
- Every mutation is traceable through an issue-backed or explicitly permitted direct-pull-request scope, an isolated branch and worktree, commits, verification evidence, and a merge decision.
- Planning and implementation are separate gates for medium- and high-risk work; accepted intent authorizes bounded medium-risk execution, while high-risk execution also requires approval of the reviewed plan.
- An executing agent may prepare a branch and pull request but cannot authorize or merge its own work. It may tag, publish, deploy, migrate, perform destructive cleanup, or change production credentials only when a separately recorded maintainer approval names that exact operation and authorizes the agent as executor.
- Repository merge, release, and production deployment are separate authorities; completing one never implies permission for the next.
- Policies are enforced by repository rules and CI where practical; prose-only rules must be treated as advisory until enforcement exists.
- Process complexity must be proportional to risk; a small documentation fix does not require the same review depth as authentication, public contracts, destructive behavior, or releases.

The operating model draws on [OpenAI's harness-engineering practice](https://openai.com/index/harness-engineering/), [Anthropic's guidance on simple composable agent patterns](https://www.anthropic.com/engineering/building-effective-agents), and the [Learn Harness Engineering](https://walkinglabs.github.io/learn-harness-engineering/ko/) curriculum. These are design inputs, not normative dependencies.

## 2. Policy ownership

The public `haesol-shin/.github` repository is the canonical home for account-wide workflow policy, receipt and repository-policy schemas, trusted validators, reusable workflows, contribution guidance, security reporting, and default issue and pull request templates. Each product repository keeps repository-specific commands, boundaries, and release details. `agent-skills` owns installable runtime behavior and adapters that consume the central contract; it does not become a second policy or schema owner.

GitHub-provided default community files affect the contribution interface but are not copied into repository clones, so every instruction required by a local agent remains in that repository's `AGENTS.md` or `CONTRIBUTING.md`. A repository-local issue-template directory replaces the account defaults as a set rather than extending them. A local template may change front matter and instructional text and may add product-specific level-two sections, but it reproduces every central required heading exactly once and in contract order. Required headings are never renamed or removed.

| Concern | Canonical owner |
| --- | --- |
| Workflow, risk, review, receipt, and provenance contracts | `haesol-shin/.github` |
| Trusted policy validators and reusable workflows | `haesol-shin/.github` |
| Default issue and pull request templates | `haesol-shin/.github` |
| Runtime adapters and installable agent behavior | `agent-skills` |
| Repository-specific agent rules and safety boundaries | Repository `AGENTS.md` |
| Repository-specific development and verification commands | Repository `CONTRIBUTING.md` and `.github/repo-policy.yml` |
| Product release procedure and supported artifacts | Repository `RELEASE.md` |
| User-visible release history | Repository `CHANGELOG.md` |
| Cross-repository product compatibility | Owning bundle's `bundle.toml` |

Each repository declares the adopted contract revision in `.github/repo-policy.yml`. Shared workflows are referenced by an immutable commit SHA; branches and tags are not trusted as immutable workflow inputs.

## 3. Work item lifecycle

```text
Interactive intake -> issue or permitted direct pull request -> accepted intent -> planned or direct route -> draft pull request -> internal plan review when required -> high-risk plan approval when required -> isolated execution -> verification -> internal implementation review -> exact-head merge-ready review -> maintainer merge -> optional separately approved release
```

GitHub is the control plane for human intent, authorization, exact-head merge evidence, and final decisions. The runtime owns internal planning and implementation rounds. Intermediate rounds and findings stay inside the runtime unless their round limit is exhausted or maintainer action is required.

| Gate | Decision owner | Durable GitHub record |
| --- | --- | --- |
| Accept intent and risk | Maintainer | Issue comment bound to the intent digest |
| Approve a high-risk plan | Maintainer | Draft pull request comment bound to the intent, plan digest, and plan commit |
| Declare an exact head merge-ready | Fresh-context Reviewer | Final GitHub Review, or pull request comment under a shared identity, with the merge-review receipt |
| Merge pull request | Maintainer | GitHub merge event |
| Approve release and exact release commit | Maintainer | Pull request or release approval bound to the exact commit |
| Deploy, migrate, clean up destructively, or change credentials | Maintainer | Separate approval naming the operation and executor |

Creating an issue does not authorize implementation. An accepted medium-risk intent authorizes the reviewed plan and bounded implementation unless the plan changes the outcome, non-goals, public contract, security boundary, cost, or acceptance criteria. High-risk work additionally requires explicit approval of the final reviewed plan. Initial authorization for a direct low-risk pull request may be given in the dispatching conversation, and its scope and rationale must appear in the draft pull request. Any material intent or plan change invalidates the dependent authorization. Any new commit invalidates exact-head merge evidence.

The workflow uses GitHub's native issue, draft pull request, review, check, and merge states rather than status labels. An account-level GitHub Project may later provide cross-repository portfolio fields if native search is insufficient.

### 3.1 Issue and intent contract

An issue organizes one bounded work item. It records why the work exists, the outcome that must become true, the result-level work, observable completion conditions, scope exclusions, and dependencies or authority context. It does not prescribe file edits or implementation order; those belong in the plan when planning is required.

Issue titles use a concise natural-language outcome, such as `Publish a versioned extraction JSON contract`. Conventional Commit prefixes are reserved for commit subjects and pull request titles.

```markdown
## Problem

Current behavior, impact, and relevant evidence. A bug includes reproduction and environment here when known.

## Desired outcome

What must become true, including material constraints.

## Work

- [ ] Result or deliverable, not a file-by-file implementation step.

## Acceptance

- [ ] Observable behavior or evidence that proves completion.

## Non-goals

- Explicitly excluded adjacent work, or `None`.

## Context

Dependencies, responsibility boundaries, approval constraints, and related issues, pull requests, or contracts; use `None` when no context is needed.
```

`Work` is the maintainable task list, `Acceptance` is the completion contract, and `Non-goals` prevents scope growth. A repository may append specialized level-two sections such as an operational target or security boundary anywhere that preserves the shared heading order, but it does not duplicate, rename, or remove the shared headings.

When discussion has stabilized the intent, the maintainer records one accepted-intent comment that states the resolved outcome, invariants, non-goals, risk, and content digest. The comment, not a copy in the pull request, is the durable intent authority. Editing or deleting it revokes dependent authorization until the trusted validator accepts a replacement.

```text
Intent accepted.

<resolved outcome, invariants, and non-goals>

Risk: <low|medium|high>
Intent digest: sha256:<digest>
```

Canonical digest construction is part of the central contract and its conformance fixtures. Structured payloads are parsed into schema-defined fields, text fields normalize CRLF and CR to LF with one terminal newline, and the result is hashed as UTF-8 [JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785) bytes. The diff digest hashes the raw output of `git -c core.quotePath=true diff --binary --full-index --no-color --no-ext-diff --no-textconv --no-renames <base>...<head> --`. A permitted direct low-risk pull request without an accepted-intent comment uses the literal `intent:none`; its authorization scope and rationale remain in the pull request body.
Each machine-readable record occupies one standalone line, uses the fields in the shown order with one ASCII space between fields, and permits human-readable Markdown only on other lines of the same comment.

Low-risk work defaults to the direct route when its accepted scope is clear and can be implemented as one reviewable change. Medium- and high-risk work use the planned route. Risk is justified in the accepted-intent comment or, for a direct pull request, in the pull request body. An agent may raise risk when new evidence appears; lowering risk requires maintainer approval. Work that cannot remain one coherent, independently verifiable pull request is split before execution.

Cross-repository initiatives use one coordinating issue in the integration-owning repository and linked implementation pull requests or issues in affected repositories. The coordinating issue closes only after every required repository change is integrated. Trivial documentation and clearly low-risk maintenance may proceed directly to a pull request.

### 3.2 Plan contract

Planned work keeps its current plan at `.ops/plans/<issue>-<slug>.md` in the affected repository. The plan references the issue and accepted-intent digest instead of copying the issue. The Planner owns revisions; the Critic is read-only. A candidate plan commit opens the draft pull request, internal review may replace it on the branch, and only the accepted plan content remains in the squash-merged decision history; intermediate review artifacts remain runtime-owned.

```markdown
# <plan title>

Issue: #<issue>
Intent: `sha256:<digest>`

## Approach

The chosen design, material alternatives, decision drivers, contract and dependency effects, and high-risk pre-mortem when required.

## Execution

Ordered, independently verifiable implementation slices.

## Verification and recovery

Behavioral checks, realistic smoke evidence, rollout, and rollback or mitigation.
```

One plan review round is review of one immutable plan digest followed by either approval or one consolidated revision request. Medium-risk planning permits at most three rounds; high-risk planning permits at most five. A valid reviewer verdict consumes a round; timeouts, malformed output, unavailable reviewers, and cancelled runs do not. Rebase-only head changes with an unchanged plan and diff digest require provenance rebinding but do not consume a round. Exhausting the limit with blockers produces one GitHub blocker handoff and requires maintainer intervention.

Medium-risk work proceeds automatically after internal plan approval because accepted intent already delegated bounded execution. High-risk work pauses after internal plan approval and requires one maintainer comment on the draft pull request:

```text
repo-ops.plan-approval.v1 decision:approved risk:high issue:<number> intent:sha256:<digest> plan:sha256:<digest> plan-commit:<sha>
```

The trusted validator accepts the approval only from an authorized maintainer when the named plan commit is in the pull request history and its content still matches the intent and plan digests. Later implementation commits do not invalidate that approval; changing the intent or plan does. For high-risk work, the plan covers trust boundaries, credible failure modes, compatibility and migration impact, recovery, and required security evidence.

### 3.3 Pull request contract

Every pull request represents one coherent outcome. A candidate plan commit opens the draft pull request; implementation, verification, and internal review continue there after the applicable plan gate passes. The executing agent marks the pull request Ready only after the implementation review joins cleanly and required evidence is current.

```markdown
## Summary

The problem and impact, why the work was needed, and the delivered outcome. This must stand on its own without opening the issue.

## Changes

- Material user-visible, behavioral, API, schema, dependency, or operational changes.
- Important compatibility effects; omit file-by-file inventories.

## Impact

- User/runtime: <effect | none>
- API/schema/dependencies: <effect | none>
- Operations/deployment: <effect | none>
- Not changed: <important preserved boundary | none>

## Verification

- `<behavior>` — `<command or evidence>` — `<result>`
- Unverified: none | <gap and reason>

## Risk and rollback

Risk: <low|medium|high> — <rationale>

Rollback: <exact reversal or mitigation>

## Related

Fixes #<issue> | Direct low-risk PR: <rationale>
Plan: <link | none>
```

The pull request is independently understandable but does not copy the accepted intent verbatim. `Summary` carries the final context and outcome, `Changes` records what was delivered, `Impact` separates product, contract, dependency, and production effects, `Verification` records exact commands, evidence, results, and gaps, `Risk and rollback` makes failure exposure and recovery visible, and `Related` provides provenance rather than required reading. Checkboxes without results are not evidence. Changelog entries and file inventories remain in their canonical or generated surfaces instead of becoming pull request sections. Repository-local templates may change instructional text and add sections such as contract impact, deployment procedure, or documentation state while retaining each shared heading exactly once and in contract order.

## 4. Risk and review

| Risk | Typical changes | Plan review limit | Implementation review limit | Required evidence |
| --- | --- | ---: | ---: | --- |
| Low | Documentation, tests, internal cleanup with no behavioral contract change | None | 2 rounds | Focused checks, fresh-context review, maintainer merge |
| Medium | User-visible behavior, dependencies, performance, schemas, compatibility | 3 rounds | 3 rounds | Internally approved plan, full relevant CI, fresh-context review, maintainer merge |
| High | Authentication, credentials, security, destructive operations, public API, cross-repository contract, installer, release, deployment | 5 rounds | 5 rounds | Maintainer-approved plan with pre-mortem, full CI and realistic smoke evidence, adversarial fresh-context review, explicit rollback, maintainer merge |

Changes to the workflow contract or schema, trusted validators, repository policy, risk classifier, verification commands, or release and deployment workflows are always high-risk. They are evaluated under the immutable base policy and cannot authorize themselves.

Review borrows [Gajae Code's](https://github.com/Yeachan-Heo/gajae-code) role separation, explicit verdicts, severity discipline, immutable change-set binding, joined findings, and bounded repair. The reviewer receives accepted intent, the approved plan when present, the full `merge-base...head` diff, relevant contracts, and verification evidence without the authoring conversation. Internal verdicts are `APPROVE`, `REQUEST_CHANGES`, or `INCONCLUSIVE`; only a valid completed verdict consumes a round. Findings are joined before one consolidated fix batch starts. High-risk review uses the same Reviewer role with stronger attention to trust boundaries, abuse paths, recovery, and evidence quality.

Intermediate implementation rounds remain runtime-owned and do not accumulate as GitHub comments. When the exact head joins cleanly, the reviewer publishes one final human-readable conclusion and machine-readable receipt as a native GitHub Review when it uses a distinct identity, or as a pull request comment when it shares the author's identity:

```text
repo-ops.merge-review.v1 verdict:merge-ready risk:<risk> intent:<sha256:digest|none> plan:<sha256:digest|none> plan-round:<n/max|none> implementation-round:<n/max> base:<sha> head:<sha> diff:sha256:<digest> runtime:<runtime/version> model:<provider/model>
```

The trusted validator independently recomputes the head, merge base, binary diff digest, intent and plan digests, applicable round ceilings, reviewer authority, and the live `CI / quality` conclusion. A receipt indexes evidence; it is not evidence by itself. A new implementation commit makes the merge-ready receipt stale. Rebase-only rebinding of an unchanged diff does not consume a round. If the limit is exhausted with blockers, the runtime posts one blocker handoff with the unresolved findings instead of a merge-ready receipt.

When the executor, reviewer, and maintainer share one GitHub identity, GitHub cannot authenticate role independence. The receipt is posted as a pull request comment, the validator applies the same schema and exact-head rules, and the maintainer remains the sole merge authority. A distinct least-privilege GitHub App or bot is required before the policy can claim independently authenticated reviewer identity or require an approving review.

## 5. Branches, worktrees, commits, and merge

- `main` is the only long-lived branch unless sustained parallel release integration demonstrates a need for `dev`.
- Each pull request in each repository uses one isolated worktree and a short-lived branch named `<type>/<issue>-<short-kebab-description>`, for example `feat/42-local-evidence`; a permitted direct low-risk pull request uses `<type>/<short-kebab-description>`. One issue may coordinate several linked pull requests with an explicit merge order.
- Valid branch and commit types are `feat`, `fix`, `docs`, `refactor`, `test`, `perf`, `ci`, `build`, `chore`, and `revert`.
- Commit subjects and pull request titles use `type(scope): imperative summary`, for example `feat(lectural): accept local media evidence`.
- Stage exact paths, review the staged diff, and never mix unrelated user changes into the branch.
- Pull requests use squash merge so the pull request title becomes the permanent `main` commit; merge commits are disabled and noisy intermediate commits do not enter `main`.
- Delete the source branch and worktree after merge; stale merged branches are operational drift and should be detected periodically.
- Direct pushes, force pushes, deletion, and history rewriting of protected branches and release tags are prohibited.

## 6. Repository rules

Every active repository protects `main` with a GitHub ruleset that requires a pull request, linear history, the stable `CI / quality` and `Repository policy / contract` checks, resolved review conversations, and blocks force pushes and deletion. Automatic merge is disabled. Required approving reviews are enabled only when PR authorship uses an identity distinct from the maintainer; otherwise the maintainer remains the sole merge authority and the limitation is documented.

Tag rules protect `v*` for distributable repositories. The only permitted creation path is the release workflow or maintainer identity named by the recorded release approval; a release tag must reference that approved, verified commit reachable from protected `main` and must not be moved or recreated.

## 7. Continuous integration

Every repository exposes two stable required checks. `CI / quality` runs on `pull_request`, executes repository code with read-only permissions and no production secrets, and aggregates all applicable repository verification with an always-running final job. `Repository policy / contract` runs trusted base-owned validation on pull request, review, and issue-comment changes; it treats head content as data, never checks out or executes untrusted head code, and validates intent, high-risk plan approval, round ceilings, unresolved blockers, the final merge-review receipt, and exact-head binding. Required workflows are not path-filtered, and check names are unique so skipped or duplicate contexts cannot accidentally satisfy the ruleset.

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

Shared workflows live in `haesol-shin/.github` and are versioned. Each repository contains thin caller workflows and repository-specific commands in `.github/repo-policy.yml`; shared workflows must not infer arbitrary commands or receive inherited secrets by default. Policy-comment and review edits or deletions retrigger validation and revoke stale authorization. Pull requests from forks receive the same metadata validation but never production secrets or write-capable execution.

## 8. Dependencies and cross-repository contracts

- Each runtime owns its dependency manifest, lockfile, environment, performance benchmarks, and release cadence.
- Repositories integrate through versioned CLI, JSON schema, file manifest, or protocol contracts rather than source imports or shared environments.
- The contract-owning repository publishes schemas and fixtures; consumers link to the owner and maintain compatibility tests without becoming a second schema owner.
- Released integrations pin exact compatible tags or commits in the owning bundle; mutable branches are development-only and block release.
- Dependency updates are isolated PRs. Heavy or compatibility-sensitive upgrades require lock regeneration, realistic smoke tests, and an explicit rollback version.
- Reusable implementation code is extracted only after at least two independent consumers prove a stable boundary, test suite, versioning need, and release owner.

## 9. Changelog and release

Repositories that publish installable software, plugins, libraries, CLIs, or versioned contracts use Semantic Versioning, annotated `vX.Y.Z` tags, `CHANGELOG.md`, and GitHub Releases. Operations-only repositories may use dated, append-only production change records instead of artificial product releases.

User-visible changes add one fragment per change under `changelog.d/<issue>-<slug>.md`; a permitted direct low-risk pull request uses `changelog.d/direct-<slug>.md`. Repositories with multiple release units place the same filename under `packages/<package>/changelog.d/`. A fragment contains one or more `### Added`, `### Changed`, `### Deprecated`, `### Removed`, `### Fixed`, `### Security`, `### Breaking Changes`, `### Documentation`, `### Performance`, or `### Tests` sections with bullet entries. Ordinary pull requests never edit `CHANGELOG.md`; CI validates fragment syntax and ownership, rejects direct edits to the shared unreleased area, and rejects deletion of unconsumed fragments. The trusted release workflow prepares the release pull request by deterministically folding pending fragments into the dated version section and deleting the consumed files. Released sections are append-only except for a separately reviewed correction.

The release sequence is:

1. Merge a release PR into protected `main` after both required checks pass.
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

Agent responsibilities are engine-neutral. A workflow role defines responsibility; a runtime names the tool executing it; a model identifies the exact provider-qualified model selected for that run.

| Workflow role | Responsibility |
| --- | --- |
| Planner | Create and revise the technical plan without implementing product changes |
| Critic | Review a plan without editing it and return approval or actionable blockers |
| Executor | Perform only authorized implementation or research and produce verification evidence |
| Reviewer | Review the frozen implementation without editing it and return evidence-backed findings and a verdict |

The standard adopts the [Gajae Code](https://github.com/Yeachan-Heo/gajae-code) separation between external authority and internal writer-reviewer loops without requiring its runtime. Interactive intake stabilizes intent; plan review stabilizes the approach; execution advances bounded work with evidence; implementation review verifies the frozen change; and the maintainer owns authorization and merge. Specialized scouts and security reviewers remain helpers beneath these roles. Copied prompt text must retain license-required attribution; otherwise only the operating concept is reimplemented.

### 11.1 OMP adapter

[OMP](https://github.com/can1357/oh-my-pi) is the preferred interactive runtime, while Codex, Claude Code, Gajae Code, or another compatible runtime may implement the same contract. OMP separates runtime agents from configurable model roles.

| Workflow role | OMP surface | Model selection |
| --- | --- | --- |
| Planner | Main interactive session | `@plan` when explicitly selected |
| Critic | Custom read-only `plan-critic` | `@slow` role, resolved at runtime |
| Executor | Bundled `task` agent | Agent definition or configured override |
| Reviewer | Bundled `reviewer` agent | Bundled definition, currently resolved through its configured role |

The main session coordinates temporary helpers; they do not become separate repository actors. The custom `plan-critic` exists because the bundled reviewer is patch-oriented. The adapter supplies dispatch inputs, artifact schemas, round gates, and output normalization without copying native prompts. Receipts record the resolved provider-qualified model ID, not aliases such as `@slow`.

### 11.2 Contract and enforcement

One versioned contract bundle in `haesol-shin/.github` owns the enforceable workflow contract: `contract.json` declares the governed heading level, required occurrence and order, extension policy, round limits, and record order; referenced schemas define structured records and repository policy; the trusted validator implements authorization and risk behavior; conformance fixtures bind those pieces together. Templates and runtime adapters reference the bundle's contract revision instead of restating its rules. CI validates changed schemas, templates, validators, adapters, and samples together.

GitHub is the durable control plane for accepted intent, high-risk plan approval, the exact-head merge-ready review, checks, merge, release, and exceptional human decisions. Native issue, draft pull request, review, check, and merge states replace a duplicate label state machine. The runtime is the execution plane for internal plan and implementation rounds, findings, retries, and handoffs.

Interactive OMP use does not require a repository-local runtime ledger. A future unattended orchestrator must persist its own append-only or transactional state for round artifacts, source hashes, operation identities, locks, blockers, and restart reconciliation; its database or durable storage is runtime-owned and must not create competing human approval state. On uncertain state it reconciles GitHub authority, the current plan digest, head, and its own records before repeating a mutation.

### 11.3 Run provenance and unattended operation

The final merge-review receipt records the runtime name and version, resolved provider-qualified model ID, intent and plan digests, round counts, base, head, and binary diff digest. Its GitHub location supplies the pull request identity, and the intent record supplies the issue identity when one exists. The trusted validator binds it to the live `CI / quality` conclusion. Custom agents or prompts are recorded only when they materially differ from runtime defaults.

[Herdr](https://github.com/herdrdev/herdr) may keep sessions alive and permit reconnection; it is execution infrastructure, not workflow authority or a task ledger. A future persistent orchestrator may accept an intent-authorized work item, create an isolated worktree and runtime session, manage internal rounds, wait for high-risk GitHub plan approval, and publish the final merge-ready review. Automatic merge remains out of scope.

[Paperthin](https://github.com/LilMGenius/paperthin) skills may supplement reasoning, critique, and retrospectives but do not own approval or run a competing planning or execution state machine.

### 11.4 Agent operating contract

- The agent starts from accepted intent or a clearly low-risk direct-pull-request scope, reads repository instructions, inspects live state, and reports conflicts before mutation.
- The agent creates or uses the issue-specific worktree and branch, changes only in-scope files, and preserves unrelated user work.
- The agent opens a draft pull request after committing the candidate plan when planning is required, updates that branch through internal plan review, and keeps the pull request's Summary, Changes, Impact, Verification, Risk and rollback, and Related sections current.
- The agent runs the smallest relevant checks during development and the complete required profile before handoff.
- The agent performs a fresh self-review and obtains the required fresh-context review rounds without posting normal intermediate rounds to GitHub.
- The agent may post the final merge-ready review or a terminal blocker handoff but must not merge, tag, release, deploy, or alter credentials without authority.
- If execution state is uncertain, the agent reconciles it instead of repeating a potentially mutating operation.

### 11.5 Operational terms

- A review round evaluates one immutable plan or change-set digest and ends in one valid verdict; infrastructure failures do not consume it.
- Fresh context means the reviewer receives accepted intent, relevant artifacts, the full diff, and evidence but not the authoring conversation or hidden rationale.
- Full relevant CI means every required check in the repository's declared profile for the affected surfaces.
- Realistic smoke evidence exercises the public entrypoint against the closest safe environment to actual use and records the result and known gap.
- Accepted risk is a specific unresolved finding that the maintainer explicitly accepts for the exact head; silence or merge alone is not acceptance.

## 12. Exceptions and emergencies

The maintainer may authorize an exception only in the relevant issue or pull request with its reason, exact scope, expiry, compensating checks, and follow-up issue. Emergency security work may shorten planning and review but does not waive secret handling, exact-head verification, rollback or mitigation, or the durable decision record. CI or reviewer unavailability is not implicit permission to merge; the maintainer must record the degraded gate and accept its risk explicitly.

## 13. Adoption plan

1. Finalize this standard in `agent-skills` as the design source for the initial implementation.
2. Create the public `haesol-shin/.github` repository and publish the central contract, schemas, trusted validators, and default community files as `v0.1.0`.
3. Implement the Markdown issue template, pull request template, repository-policy schema, `CI / quality` caller contract, `Repository policy / contract` validator, and changelog-fragment checks.
4. Replay the validator against representative existing pull requests to calibrate risk and receipt handling without treating the work as a new pilot.
5. Enable the contract check in advisory mode, fix false positives and missing evidence from real use, then make both stable checks required.
6. Adopt the immutable contract revision in `agent-skills` and subsequent repositories through separate reviewable pull requests.
7. Add a persistent unattended orchestrator only when automated dispatch, restart reconciliation, or concurrent runs require it.
8. Audit policy drift periodically by comparing each repository's contract revision, ruleset, required checks, workflows, and release configuration.
