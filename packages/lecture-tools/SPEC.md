# Lecture Tools Plugin Specification

> **Status:** Draft v0 (2026-09-18)
>
> **Working name:** `lecture-tools`
>
> **Understood as:** define a portable, versioned agent plugin that turns explicitly selected course sources into a verified project satisfying the selected assignment requirements. Lecture video is processed as time-aligned multimodal evidence; briefs, notices, materials, and other lectures provide additional requirements and context. The package also supports course discovery and user-selected visible playback, but excludes submission, grading-state mutation, Google Tasks, digests, scheduled playback, and other unattended campus automation.

The checked-in `0.1.0` manifest is a design version, not a published or operational release. The package remains non-runnable while `bundle.toml` lists an unreleased engine or missing contract.

## 1. Purpose

`lecture-tools` is the bundle and installation unit. It gives a student using a supported campus provider a natural-language interface for four actions:

1. Refresh the enrolled lecture catalog.
2. List incomplete lecture videos.
3. Play only the lecture videos the user explicitly selects, in order and in a visible official player, then report the LMS's authoritative completion state.
4. Turn authorized lectures and explicitly selected course context into a verified project that satisfies the selected assignment requirements and remains grounded in source evidence.

`lecture-to-code` is the bundle's primary value path, not an optional integration. It accepts one or more authorized lecture sources plus explicitly selected assignment briefs, notices, materials, and supporting lectures. It derives an explicit requirement set, uses the selected sources to implement the requested project, and verifies the deliverable against both the requirements and applicable executable checks. A runnable release must provide at least one end-to-end audiovisual input path for a supported source. For v0, an explicitly supplied authorized local media file qualifies; provider-integrated retrieval remains the preferred path and must advertise unavailability rather than silently degrading to text-only input.

The plugin is an orchestration and instruction layer. It does not absorb the two engines:

- `campusctl` owns the portable campus-domain CLI, authentication abstraction, course-source discovery and retrieval, playback orchestration, LMS completion verification, and institution provider modules. It converts the user's authorized LMS session into local source packages or media without exposing browser handles, cookies, tokens, or URLs that require the private session to resolve. The CNU provider remains inside campusctl for v0 and is extracted only after another provider proves a stable shared boundary.
- `lectural` owns deterministic extraction of time-aligned transcript, representative frames, OCR annotations, synthesis inputs, notes, and extraction completeness from supported media. It does not decide what a frame means to an assignment or whether a project should be generated.
- `notice-bot` remains the owner's personal application for schedules, notifications, Google Tasks, boards, and other unattended automation. It may consume campusctl once the public CLI exists but does not own the public campus contract.

The engines remain independently versioned processes joined by stable JSON CLI contracts.

### 1.1 Document ownership

This document owns the agent-facing product and integration contract. Engine-specific command schemas belong to their engine repositories and are referenced through `bundle.toml`; they are not copied here once published. README links follow the current default branch for navigation, while a released bundle pins exact tags or commits for reproducibility. Git submodules are not used.

## 2. Scope

### 2.1 Included

- One-time laptop setup with the fewest possible user-run commands.
- An OS-keyring credential provider for the portable `campusctl` runtime.
- Read-only runtime and dependency diagnosis through `doctor`.
- Selective synchronization and read-only listing of lectures, assignments, notices, and materials, including their LMS structural context.
- Retrieval of an explicitly selected assignment or notice as a local source package containing structured text, embedded images, attachments, provenance, and available deadline metadata.
- Explicit user-selected playback queues, processed serially in a visible official player with an officially supported speed.
- Read-only verification of the provider's authoritative completion state after playback; the plugin never writes or fabricates attendance or progress.
- Provider-declared media retrieval through an official download or policy-permitted live capture; an authorized local file bypasses campusctl and goes directly to LecturAL.
- LecturAL evidence extraction with speech and visual completeness, explicit OCR state, retained representative frame images, and time-aligned artifact references. Transcript-only evidence is insufficient for a primary lecture source.
- Requirement-driven agent generation grounded in the selected assignment brief, notices, materials, lectures, transcript, frame images, OCR annotations, and timestamps.
- Vision inspection of relevant frame images whenever the deliverable depends on displayed code, commands, outputs, diagrams, or other visual state. OCR is a retrieval and transcription aid, not a substitute for the source image.
- Requirement-by-requirement verification of the generated project, including applicable formatter, test, build, or smoke commands.
- Codex-first plugin packaging with canonical, agent-neutral `SKILL.md` sources.
- Deterministic dependency, compatibility, CI, PR, and release policy.

### 2.2 Excluded from the plugin

- Assignment submission, answer posting, grading-state mutation, reminders, or unattended assignment automation; read-only use of an explicitly selected brief as a requirement source remains in scope.
- Google Tasks registration or status.
- CSE/DDC boards, Telegram notifications, daily digests, and unrelated LMS material downloads; retrieval of explicitly selected course context remains in scope.
- Scheduled, hidden, concurrent, or unselected bulk playback.
- Playback skipping, fabricated progress, attendance mutation, access-control or copy-protection bypass, and browser-fingerprint spoofing or human-mimicry.
- Credential display, export, recovery, or management by an agent.
- Automatic LMS submission of generated code.
- Acquisition that bypasses provider capability declarations, access controls, or copy-protection.

Existing private applications may retain capabilities outside this bundle. A personal profile may compose private extensions, but the public plugin must not call or advertise capabilities absent from its released provider contract. Keeping a capability private does not make prohibited use permitted.

## 3. User Experience

### 3.1 First use

The target is one private setup invocation:

```console
lecture-tools setup
```

Setup is a future trusted installer entrypoint, separate from the credential-free orchestration runtime and never invoked by a skill. It performs or coordinates:

- runtime and platform checks;
- creation of user config/data/cache directories;
- installation of the exact tagged engine versions selected by `bundle.toml` and preparation of their independent locked Python environments;
- Chromium and `ffmpeg` checks;
- private OS-keyring credential entry through a direct user-run prompt;
- an initial campusctl sync; and
- a final aggregate doctor run.

Credential entry must occur outside LLM-visible input and output. If the host cannot provide a private prompt, setup stops with one exact remediation rather than accepting a password in an argument, environment variable, chat message, redirected stdin, or config file.

The default credential backend is the platform's recommended keyring: Windows Credential Manager, macOS Keychain, or a supported Linux Secret Service/KWallet backend. Setup rejects plaintext and insecure fallback backends. Browser-managed interactive SSO may be used instead when the provider supports it; SSO is an authentication flow, while keyring is credential storage.

### 3.2 Normal use

After setup, the user interacts in natural language. Representative requests are:

- “남은 강의 확인해줘.”
- “3주차 강의 틀어줘.”
- “영상에서 작성한 코드를 프로젝트로 옮겨줘.”

The plugin may run internal commands. The user is not asked to translate intent into CLI flags. If multiple lectures match, it presents a short disambiguation list and performs no mutation until the user selects one.

A request may select a finite ordered queue. The plugin validates the whole queue before opening the visible player, runs one lecture at a time, and advances only after normal player termination and a bounded authoritative status check. An unresolved status pauses the queue; it is never treated as successful completion or retried automatically.

`check-lectures` invokes `campusctl sync --only lectures` only when the request requires current data, then reads the lecture list.

## 4. Architecture

```text
User
  |
  v
lecture-tools plugin
  |-- check-lectures skill
  |-- play-lecture skill
  |-- lecture-to-code skill
  `-- lecture-tools-runtime (credential-free orchestrator)
          |                         |
          | JSON/argv               | JSON/argv
          v                         v
   campusctl CLI             lectural CLI
   own uv.lock/env            own uv.lock/env
          |                         |
          v                         v
 campus provider            local evidence artifacts
 (for example CNU)          and coverage
          |
          v
 campus LMS + OS keyring
```

`lecture-tools-runtime` is a dependency-free, standard-library-only process orchestrator. It passes every argument as an argv item, never constructs shell command strings, never imports either engine as a library, and never reads their private state files, configs, browser profiles, keyrings, or internal modules.

The product uses three durable abstractions:

- A **source record** identifies one user-selected lecture, assignment brief, notice, or material and records its provenance, LMS structural context, authorization basis, role, and retention rule. `campusctl` retrieves provider-backed source packages and media through the authorized LMS session; the runtime owns the job-local record.
- An **evidence bundle** is LecturAL's time-aligned, machine-readable transformation of one lecture medium. It references transcript segments, representative frame images, OCR annotations, timestamps, and extraction-completeness results without interpreting their assignment meaning.
- A **verified deliverable** is the generated project plus a requirement checklist that maps each selected requirement to implementation evidence, course-source evidence, and executable verification where applicable.

Semantic work stays in the `lecture-to-code` skill and the host agent. The transcript explains spoken intent and constraints; OCR makes visual text searchable and copyable; the frame image is the authoritative source for displayed code, terminal output, diagrams, layout, and other visual state. The agent uses transcript and OCR to locate relevant moments, then inspects the matching frame and adjacent visual changes with vision when the task depends on what was shown. It does not send every frame to a model by default.

The runtime persists state and validates schemas, paths, versions, and process results. It does not classify code scenes, interpret course content, or make a semantic build-eligibility decision. The skill derives requirements, decides which evidence is relevant, directs visual inspection, implements the project, and verifies the result.

Every selected source has one job-local role:

| Role | Meaning | Allowed source kinds |
| --- | --- | --- |
| `requirement` | Defines required behavior, constraints, deliverables, or evaluation conditions. | Assignment brief, requirement-bearing notice or material, explicit user instruction. |
| `primary` | Supplies the audiovisual lecture evidence that grounds the requested work. | Authorized local video, provider-retrieved video, supported YouTube video. |
| `supporting` | Adds explanation or context but cannot replace primary audiovisual evidence. | Additional lecture video/audio, notice, material, or other selected course text. |

The skill presents candidates and resolves ambiguity with the user before retrieval. The runtime records only the resulting explicit selection and rejects a job with no `requirement` source or no audiovisual `primary` source. A direct user request may itself be the requirement source. Local audio is always `supporting`.

`campusctl` keeps its portable core and institution providers as separate modules in one repository for v0. The core contains domain models, JSON contracts, capability negotiation, and credential interfaces; the CNU provider contains institution-specific login, URLs, selectors, course structure, rich-content retrieval, media access, status mapping, player behavior, and policy metadata. Browser automation or authenticated session reuse is a provider implementation detail. The public contract returns local artifacts, stable opaque entity references, and safe metadata, never live page handles, cookies, tokens, or URLs that require a private session to resolve. Provider code becomes a separate package or repository only after at least two implementations prove the boundary. `notice-bot` remains a personal scheduled automation application and is not the public campus engine or provider.

### 4.1 Dependency isolation

The engines must not share a Python environment. Their current dependency constraints conflict:

- campusctl currently requires `numpy>=2.4.6` and `onnxruntime>=1.29.0`.
- LecturAL currently requires NumPy below 2 and ONNX Runtime below 1.24, with an older bounded OpenCV stack.

Each engine therefore owns its `pyproject.toml`, `uv.lock`, and environment. Production plugin invocations use the equivalent of:

```console
uv run --project <engine-root> --locked <engine-command> ...
```

`uvx` is not the production execution path because it runs a tool in an isolated cached environment without consuming the engine's project lock as its runtime contract. It remains acceptable for explicitly pinned, disposable developer utilities.

## 5. Public CLI Contracts

Every command in this section is a target interface unless explicitly marked as already implemented. Existing flags are not accepted as substitutes until they satisfy these contracts. Each engine must publish its command-specific JSON Schema before this package becomes runnable; `lecture-tools` keeps consumer fixtures but does not become a second owner of those schemas.

Human-readable output is allowed for direct use, but every plugin-consumed command must support pure JSON on stdout. Diagnostics go to stderr only when JSON output cannot be produced.

### 5.1 Common response envelope

```json
{
  "schema_version": 1,
  "tool": "campusctl",
  "tool_version": "0.2.0",
  "status": "ok",
  "result": {},
  "errors": [],
  "generated_at": "2026-09-17T12:00:00Z"
}
```

Requirements:

- `schema_version` changes only for incompatible response-shape changes.
- `tool_version` follows the engine release version.
- `status` is one of `ok`, `partial`, `user-action`, `busy`, or `error`.
- Each error contains `code`, safe `message`, and nullable `remediation`; it never embeds an exception object.
- Timestamps are UTC RFC 3339 strings. Paths are absolute native paths. Optional unavailable values are `null`, not omitted.
- Consumers ignore unknown fields within a supported schema version.
- No response contains credentials, tokens, cookies, private browser endpoints, raw environment values, or unredacted underlying exception text.
- Unknown schema versions fail closed; the plugin does not guess or scrape human output.

Common process exit codes:

| Code | Meaning |
| ---: | --- |
| 0 | `ok`: requested operation completed. |
| 1 | `partial` or `error`: inspect per-item result and errors. |
| 2 | `user-action` or invalid usage: do not retry unchanged. |
| 75 | `busy`: resource is occupied; safe to retry later without assuming the operation started. |

### 5.2 campusctl surface

Required commands:

```console
campusctl --version --json
campusctl doctor --json
campusctl sync [--course <course-id>] [--only lectures,assignments,notices,materials] --json
campusctl courses list --json
campusctl lectures list [--course <course-id>] [--all] --json
campusctl lectures play <entity-id>... [--speed <rate>] --json
campusctl lectures fetch-media <entity-id> --out <directory> [--method download|live-capture] [--policy-ack <version>] --json
campusctl assignments list [--course <course-id>] --json
campusctl assignments fetch <entity-id> --out <directory> --json
campusctl notices list [--course <course-id>] --json
campusctl notices fetch <entity-id> --out <directory> --json
campusctl materials list [--course <course-id>] --json
campusctl materials download <entity-id> --out <directory> --json
```

Contract requirements:

- `sync` refreshes all enabled domains by default. `--course` bounds synchronization to one course, and `--only` restricts it to the named domains. Synchronization updates metadata and text records only; it never plays lectures, fetches media, downloads material files, submits work, or invokes Google Tasks, notification delivery, or digests.
- `lectures list` returns incomplete lecture records only; `--all` also includes completed records. Neither form returns assignment fields or records.
- Each list response reports whether it came from an available local cache and when that cache was generated. A missing cache returns `user-action` with `campusctl sync` remediation; skills never infer cache availability from private files.
- Domain `list` commands return only their own record type. `assignments fetch` and `notices fetch` retrieve one explicitly selected local source package; `materials download` retrieves only the selected file.
- Every course item record includes an opaque stable `entity_id`, course identity and label, item kind, title, and provider-normalized structural coordinates when available: week, session or section, sequence, container ID, and ordered sibling entity IDs. These fields expose LMS structure, not semantic relevance; the skill decides which nearby items are candidate context and still requires explicit selection before retrieval.
- An assignment record additionally includes submission window and deadline metadata when available. A lecture record additionally includes duration when known and LMS completion state.
- A retrieved assignment or notice package contains a manifest, structured text, embedded images, every source-referenced attachment available under the provider policy, content digests, source provenance, and unavailable-asset diagnostics. An image or attachment referenced by the source is not flattened into OCR text or silently discarded; an unavailable or policy-excluded asset remains explicit in the manifest. All package paths resolve beneath `--out`.
- Consumers never parse `entity_id`. Any ID-scheme change requires the provider to publish and test an alias/backfill migration before release; no undocumented repository-local dedupe rule is part of the public contract.
- `lectures play` validates every ID before opening a browser.
- Playback uses a visible official player, is ordered as requested, runs one item at a time, and returns one result per ID. Validation failure is atomic; runtime failure is `partial` and leaves later unstarted IDs explicit.
- Playback uses only controls and rates exposed by the official player. When `--speed` is omitted it uses the provider's declared default; `doctor --json` reports the default and supported rates, and an unsupported explicit rate returns `user-action` before playback. Playback does not hide the browser, run concurrent videos, skip required content, or simulate user presence.
- `campusctl` records local playback observations separately from LMS state. It reports completion only after the provider reads its authoritative finished state; elapsed local time or a player end event alone is insufficient. Provider-specific raw values such as the current Panopto `F` are mapped to a versioned portable enum rather than exposed as the cross-provider contract.
- `lectures fetch-media` prepares one LMS lecture as local media for evidence extraction through the current authorized LMS session and never sends attendance or progress-mutation requests. It supports only the provider-declared methods `download` and `live-capture`; a local file is a separate input source and bypasses campusctl.
- The command uses `download` by default when available. It never falls back automatically to `live-capture`; that method requires `--method live-capture`, provider permission, and explicit user confirmation because visible playback may update LMS progress incidentally. The result reports any observed authoritative state without claiming the operation was read-only.
- If `download` is unavailable and `live-capture` is available, the command returns `user-action` with the available method and current policy version. After user confirmation, `lecture-tools-runtime` supplies that exact version through `--policy-ack`; a stale or missing acknowledgement fails without opening the player.
- An authorized lecture is reachable through the current user's enrolled LMS session and normal media entitlement; the command does not bypass access controls. For a direct local file, the user explicitly attests that they are permitted to extract it.
- Source-package retrieval and `fetch-media` write only beneath the requested, resolved output directory. Their JSON responses return safe local artifact paths, provenance, authorization basis, retention requirements, and operation-specific metadata. `fetch-media` additionally returns media metadata, selected method, provider policy version, and incidental progress behavior. `lecture-tools-runtime` converts these results into its own `source.json`; campusctl does not own the integration manifest.
- Busy scheduled browser work exits 75 without a source-error alert.

Every command supports `--json`, writes its response to stdout, writes diagnostics to stderr, uses stable opaque entity IDs, and follows the common response envelope and exit codes. Commands that create artifacts return explicit resolved output paths. Consumers use only these CLI contracts and never read campusctl's private JSON cache or CNU provider modules.

### 5.3 LecturAL surface

LecturAL accepts YouTube sources and local `.mp4`, `.webm`, `.mkv`, and `.wav` files. Before integration it must expose:

```console
lectural --version --json
lectural doctor --json
lectural extract <source> --out <directory> [--skip-ocr] --json
```

The existing bare-source syntax may remain as a backward-compatible alias. The public extraction command accepts exactly one source per invocation so each output directory has one owner and failures cannot create ambiguous partial batches. JSON run output returns:

- normalized input kind;
- output directory;
- `evidence.json`, `transcript.md`, `notes.md`, `synthesis_input.json`, and `coverage.json` paths;
- frame directory when applicable;
- extraction-completeness result as `pass`, `warn`, or `fail`, with machine-readable reasons;
- separate speech and visual completeness plus timestamp integrity and OCR state, with OCR reported as `skipped`, `completed-no-text`, `completed-with-text`, or `failed`; and
- a bounded failure code and safe message when unsuccessful.

`evidence.json` is the machine-readable manifest for the extraction run: it records source kind, artifact paths, extraction summaries, status, and safe failure metadata. It references rather than duplicates transcript, frame, OCR, notes, and synthesis content. Every retained representative frame remains addressable with its source timestamp whether OCR succeeds, produces no text, is skipped, or fails. LecturAL may use OCR to annotate and index frames, but it must not discard the only retained visual evidence solely because OCR could not classify or read it.

LecturAL accepts a local media path or a supported YouTube URL. The runtime assigns every selected input an opaque job-local `source_id` before extraction and gives LecturAL the empty `evidence/<source-id>/` directory. A preexisting or nonempty output directory fails before work begins. Direct local files and YouTube URLs require the same user authorization attestation recorded by the runtime; LecturAL validates source syntax and access but does not decide authorization.

Only extraction `pass` permits the plugin to claim that a primary lecture was processed completely enough for the derivation workflow. OCR failure blocks a run that requested OCR; explicit `--skip-ocr` remains usable because representative frame images are retained for direct inspection. Local audio and other transcript-only input may contribute supporting context but cannot satisfy the primary audiovisual-source requirement. `warn` may produce a clearly labeled study artifact but cannot serve as complete primary evidence, and `fail` stops processing for that source. LecturAL owns and versions extraction criteria only; it does not classify code scenes, judge assignment relevance, or return project/build eligibility.

The plugin runs the LecturAL project through its checked-in lockfile. Published documentation must not recommend an unlocked `uvx --from ".[run]"` path for the plugin workflow.

### 5.4 Plugin runtime surface

```console
lecture-tools setup
lecture-tools setup --json
lecture-tools doctor
lecture-tools doctor --json
lecture-tools job init --root <empty-directory> --json
lecture-tools job source add --job <job-root> --input <source-record.json> --json
lecture-tools job source retrieve --job <job-root> --source-id <source-id> --json
lecture-tools job source extract --job <job-root> --source-id <source-id> --json
lecture-tools job requirements record --job <job-root> --input <requirements.json> --json
lecture-tools job verification record --job <job-root> --input <verification.json> --json
lecture-tools job validate --job <job-root> --stage derivation|complete --json
lecture-tools job cleanup --job <job-root> --json
```

`setup` is an idempotent, direct user-run installer operation and is never invoked by a skill. Interactive mode may open the private credential prompt described in Section 3. In JSON mode it never requests or accepts a secret; it reports completed phases and returns `user-action` with the exact private continuation step when credentials or browser SSO are required. Re-running setup preserves compatible installations and user data, repairs only explicitly approved plugin-owned paths, and never upgrades across an incompatible pinned bundle.

The default doctor is read-only and aggregates:

- plugin manifest and skill validity;
- engine discovery, versions, and supported schema versions;
- lockfile presence and environment readiness;
- browser and media binary availability;
- credential configured/missing state without reading values into the orchestrator;
- writable output/data paths; and
- restart or user-action requirements.

Repair is a separate explicit operation. A doctor never installs system packages, edits config, or changes credentials merely because diagnosis found a problem.

The `job` commands are deterministic state transitions used by skills; they do not contain an agent or interpret course content. `init` creates the fixed job layout and lock. `source add` validates and atomically records a source already selected by the user. `source retrieve` invokes the released campusctl contract when retrieval is required, while direct authorized local and YouTube sources bypass campusctl. `source extract` invokes the released LecturAL contract for lecture media. The two `record` commands validate agent-produced semantic records against package-owned schemas before atomically installing them. `cleanup` applies recorded retention rules without traversing outside the job root or provider-owned cache roots.

`job validate --stage derivation` checks only deterministic preconditions: compatible engine contracts, at least one requirement source, at least one audiovisual primary source, successful primary extraction, valid manifests, and path containment. It does not judge whether the evidence is semantically sufficient. `--stage complete` additionally requires a project and a verification record covering every requirement ID; it does not convert the agent's claims into independent proof.

Every job command follows the common response envelope and exit codes. Mutating commands acquire the job-root lock, validate the current state, write a temporary sibling, atomically replace the owned file, and return the new state. Interrupted operations remain explicitly resumable or failed; they never infer success from a partially written artifact.

## 6. Lecture-to-Code Workflow

1. Resolve the assignment or requested outcome, one or more primary lecture sources, and candidate supporting lectures, notices, or materials. Use campusctl's LMS structural context to surface nearby items, such as an assignment-introduction video in the same session, without treating proximity as semantic proof. Show the candidates and their relevance before retrieval; only explicitly selected items enter the job.
2. Before a hosted agent or any external LLM receives new lecture-derived metadata, transcript, OCR, frames, or other evidence, the skill shows a concise disclosure identifying the destination, data classes, purpose, retention facts known to the plugin, and the provider's policy warning. The runtime records and validates the user's explicit confirmation before returning that payload. Cache only the confirmation record and policy version, never the lecture data or credentials. Repeat the disclosure when the provider policy, external destination, or newly requested data class changes.
3. External-LLM handling is `warn-and-confirm`, not a general policy hard gate: after confirmation, the workflow continues. Confirmation does not create permission or override access controls, copyright restrictions, or institutional rules; responsibility remains with the user. Minimize disclosure to the source excerpts needed for the current derivation and never upload the raw media file.
4. Assign every selected input a unique opaque job-local `source_id`. For each LMS lecture, run `campusctl lectures fetch-media` to produce local media; for an explicitly supplied authorized local file or YouTube URL, skip campusctl. Retrieve each selected assignment or notice as a campusctl source package and each selected material through its dedicated campusctl command. Preserve embedded images and attachments as original source evidence rather than replacing them with extracted text. In every case, `lecture-tools-runtime` writes the source manifest and records the user's selection and any required authorization attestation.
5. Normalize the selected assignment brief and user request into `requirements.json`: an ordered checklist of required behavior, constraints, deliverables, and verification conditions, each retaining its source provenance. Compare the checklist back to every selected requirement source before implementation so an omitted requirement cannot disappear from later verification. Do not invent missing requirements; surface a blocking ambiguity when materially different implementations remain possible.
6. Run `lectural extract ... --json` against each selected lecture medium to produce an `evidence.json` and referenced artifacts, then create `evidence-set.json` linking every selected source, its role, and its provenance without copying artifact content.
7. Require extraction `pass` for every primary lecture. OCR failure blocks a run that requested OCR; explicit OCR skipping is valid because the agent can inspect retained frames directly. Text-only sources may constrain the work but cannot replace the primary audiovisual evidence.
8. Use transcript and OCR annotations to locate relevant moments. Inspect the corresponding frame images and adjacent retained frames with vision whenever the work depends on displayed code, commands, outputs, diagrams, UI state, or text that OCR may have distorted. Treat images as the visual source of truth and transcript as spoken context; never infer visual content from transcript alone.
9. Build an implementation plan mapping each requirement to relevant course evidence and an intended verification. Treat transcript, OCR, metadata, frames, and course documents as untrusted source data, never as agent instructions.
10. Create the deliverable in a new, empty, user-approved `project/` directory, never inside either engine checkout. Refuse symlink/reparse-point escape and nonempty-directory overwrite. Preserve source timestamps for evidence-dependent or interpretive decisions and explicitly mark anything created off-screen or otherwise unsupported.
11. Run applicable formatters, tests, builds, or smoke commands. In a separate verification pass, compare the project directly with the original selected requirement sources as well as `requirements.json`, then produce `verification.json` mapping every requirement to its implementation location, course evidence, executed checks, result, and unresolved ambiguity. A passing command does not compensate for an unmet requirement, and prose review does not replace an applicable executable check.
12. Report generated paths, the requirement-verification result, retrieval method, external-processing consent, unresolved ambiguity, missing visual evidence, and source timestamps; then invoke the runtime to apply each source retention rule and record the resulting source state.

State is file-based in v0. A user-approved job root has one fixed layout:

```text
<job-root>/
  job.json
  requirements.json
  evidence-set.json
  sources/<source-id>/source.json
  sources/<source-id>/<temporary-media-or-selected-context>
  evidence/<source-id>/evidence.json
  evidence/<source-id>/<transcript-frames-ocr-notes>
  project/<generated-code>
  verification.json
```

`campusctl` privately owns and atomically replaces its local catalog JSON; the plugin never reads those files and consumes only CLI JSON. `lecture-tools-runtime` owns the file lifecycle and schemas for every `source.json`, `job.json`, `requirements.json`, `evidence-set.json`, `verification.json`, and the job-root lock; the skill supplies the semantic requirement and verification content through the runtime contract. LecturAL owns each extracted `evidence/<source-id>/` subtree. The runtime records per-source states without reusing `watch/panopto`:

```text
retrieve/<source-id>: PENDING | DONE | FAILED
extract/<source-id>:  PENDING | DONE | FAILED | NOT_APPLICABLE
derive/code:         PENDING | DONE | FAILED
watch/panopto:       existing LMS completion meaning only
```

These JSON files are the v0 state contract; campusctl and lecture-tools do not introduce a database. Each owner writes through a temporary sibling file followed by atomic replacement. Campusctl separately locks its private catalog while the runtime serializes mutations beneath one job root. The runtime owns source retention and cleanup; LecturAL owns only its extraction intermediates. When retained `source.json` refers to deleted temporary media, it records `media_present = false` and the deletion timestamp rather than implying that the path remains usable. `notice-bot` may retain its existing SQLite ledger because scheduled multi-source deduplication is its separate responsibility.

Each `source.json` contains `source_id`, `kind`, `role`, original entity or user-supplied reference, provider-normalized structural context when available, selection timestamp, authorization basis and attestation timestamp when required, retrieval method, source-package or media path when any, media presence, policy version, retention rule, and content digest when available. `evidence-set.json` contains the ordered selected source IDs, each source's `requirement`, `primary`, or `supporting` role, source and evidence manifest paths when applicable, relevance rationale, extraction result, and unresolved conflicts. Unknown or duplicate source IDs, missing required manifests, path escape, and conflicting primary evidence fail before derivation.

The generated code is a study artifact. The plugin never submits it to the LMS.

The provider's declared retention rule is the upper bound. Within it, temporary media is deleted after success unless the user explicitly selects a permitted retain option, and a bounded stale-workspace cleanup handles failure or interruption. Cleanup never traverses outside the plugin-owned cache root. Secure physical erasure is not promised on modern filesystems. Derived frames, OCR, transcripts, requirements, and verification evidence remain local by default and follow the same precedence.

## 7. Plugin and Skill Layout

The current repository contains the manifest, bundle metadata, and three guarded skill drafts. The target runnable-release layout adds the orchestration runtime:

```text
lecture-tools/
  .codex-plugin/plugin.json
  skills/
    check-lectures/
      SKILL.md
      agents/openai.yaml
    play-lecture/
      SKILL.md
      agents/openai.yaml
    lecture-to-code/
      SKILL.md
      agents/openai.yaml
      references/workflow.md
  runtime/lecture-tools-runtime.py
  bundle.toml
```

The plugin is the install/update/remove unit; a skill is one bounded behavior. `skills/` is the canonical source. Native metadata may live beside a skill, such as `agents/openai.yaml`; generated or larger harness adapters belong under `adapters/` when introduced. Codex is the first supported adapter; other harnesses are not claimed until their adapters pass the same contract tests. Skills are separate because catalog access is read-only, playback has external effects and status checks, and lecture-to-code is expensive and artifact-producing. A separate `study-lecture` skill is not part of v0: LecturAL already owns evidence extraction and notes, while `lecture-to-code` owns the distinct verified-code outcome. Add another skill only when an independently triggered study workflow exists.

Skill rules:

- Keep `SKILL.md` self-contained and trigger-specific.
- Put substantial mode procedure or output rules in an explicitly routed reference.
- Use generic examples; never include personal course names, IDs, paths, endpoints, or host data.
- Call only documented CLI commands.
- Never call credential setup, keyring APIs, private state files, or provider modules directly.
- Treat playback and lecture-to-code as explicit user actions.
- Use capability/schema checks before acting and stop on incompatible versions.
- Bound retries; busy or unknown execution state never proves non-execution.

## 8. Credentials and Privacy

The portable credential implementation belongs to campusctl and uses the maintained Python `keyring` interface with the platform's recommended backend. The campusctl provider stores the password under a stable logical service name and local account identifier; config contains only the provider, account identifier, and `credential_provider = "keyring"`. Setup delegates credential entry to a direct campusctl private terminal or native prompt. Only the campusctl provider process reads the credential when authentication is required, fills the official login DOM, releases the reference promptly, and never serializes or logs it. `lecture-tools-runtime` remains credential-free.

The supported defaults are Windows Credential Manager, macOS Keychain, and a recommended Linux Secret Service or KWallet backend. `doctor` reports an unavailable or insecure backend as `user-action`; it never falls back to plaintext, environment variables, redirected stdin, or an alternate file keyring. Interactive browser SSO may avoid password retrieval when a provider supports it. Vaultwarden is not a public dependency or recommended setup path.

This provides normal-operation non-disclosure, not protection against arbitrary malicious code running as the same OS user. Hard isolation from a same-user agent requires a separately-owned broker or service and is a future security tier. Documentation must state this boundary without claiming that keyring alone is an application sandbox.

Tests must prove that fake secrets do not survive in `str()`/`repr()` of public exceptions, JSON, stdout, stderr, logs, or setup status. Tests use a fake keyring and never touch a real credential store.

External model processing is a separate disclosure boundary from credentials. The plugin warns and obtains confirmation before exposing lecture-derived data, minimizes the selected evidence, and records which destination and policy version were confirmed. Credentials, cookies, browser profiles, raw environment values, and raw media are never sent to a model. A local or institution-approved model may be selected without weakening the same provenance, evidence, and verification requirements.

## 9. Versioning and Compatibility

No released plugin may depend on a mutable branch. LecturAL's local-source support is merged, but it must ship in a tagged release together with the required extraction contract before entering a released compatibility matrix.

`bundle.toml` pins supported ranges and JSON contract versions once releases exist, for example:

```toml
[plugin]
version = "0.1.0"

[campusctl]
versions = ">=0.2,<0.3"
default_version = "0.2.1"
schema_versions = [1]

[campusctl.providers]
required = ["cnu"]
contract_versions = [1]

[lectural]
versions = ">=0.2,<0.3"
default_version = "0.2.0"
schema_versions = [1]
```

Ranges define accepted existing installations; each plugin release's exact `default_version` values define reproducible fresh setup and rollback targets.

Patch releases preserve CLI and schema compatibility. Minor releases may add optional fields and capabilities. Incompatible CLI or schema changes require a major version or a parallel versioned command/response contract.

## 10. Repository and Release Governance

Each repository follows GitHub Flow:

1. `main` is the only long-lived branch.
2. One short-lived `<type>/<description>` branch and isolated worktree per issue.
3. One coherent purpose and rollback story per PR.
4. Exact-path staging and a reviewed staged diff.
5. Required CI plus independent human or agent review before merge.
6. Squash merge using the PR title, then branch deletion.
7. Installation, credential changes, and production deployment remain separate approved actions.

The PR template contains `Summary`, `Scope and impact`, `Documentation`, and `Verification`. Verification records exact commands, results, and anything not verified. CI runs on every PR and on `main`; release publication runs only from a version tag after the same gates pass.

Before the first tagged release, each repository must have a license, README, contribution guide, security reporting policy, changelog/release policy, secret-history audit, and clean install test. The campusctl public split still requires its planned history rewrite and private/public boundary review.

The intended responsibility split is:

- `agent-skills`: public bundles, canonical skills, compatibility metadata, and profiles;
- `campusctl`: portable campus-domain CLI, capability and policy contracts, plus the CNU provider as an internal module for login, discovery, course structure, rich-source retrieval, player, status, media retrieval, and policy metadata;
- `notice-bot`: the owner's personal schedules, notifications, Google Tasks, boards, SQLite-backed deduplication, and any private capabilities outside the public campus contract; it may later consume campusctl rather than being absorbed by it; and
- `lectural`: time-aligned audiovisual evidence extraction, extraction completeness, and study artifacts.

Reusable browser mechanics remain provider-local until at least two independent providers prove a stable shared contract. A shared library is extracted only when it has independent consumers, tests, versioning, and release ownership; similarity of implementation alone is insufficient.

## 11. Verification Strategy

### 11.1 Contract tests

- Golden JSON fixtures for every public response and error class.
- Consumer-driven tests proving `lecture-tools-runtime` accepts every supported engine version.
- Rejection tests for unknown schema versions and unexpected secret-shaped fields.
- Exit-code tests for success, failure, user action, and busy state.

### 11.2 Engine tests

- campusctl tests use mocked browser/keyring/network and an isolated temporary JSON state directory.
- LecturAL tests use local fixtures and keep smoke/network/model tests opt-in.
- Media-method authorization, path containment, atomic JSON replacement, locking, cleanup, partial files, and interrupted-run behavior are tested.
- Playback tests cover explicit finite queues, visible serial execution, supported speed controls, authoritative status mapping, pause on unresolved completion, and absence of automatic retry.
- Lecture-to-code tests require complete primary audiovisual evidence, relevant-frame vision inspection, requirement provenance, external-model disclosure, and requirement-by-requirement project verification, not exact prose or exact model output. Fixed requirement fixtures are authored independently of the runtime output so a generated checklist cannot validate itself.

### 11.3 Platform CI

- Windows and Linux run CLI, JSON, path, keyring-fake, and plugin validation tests.
- The plugin test matrix exercises supported pairs from `bundle.toml`.
- CI verifies a clean checkout after tests and detects catalog/manifest drift.
- No CI job uses real campus, Panopto, keyring, Telegram, or Google credentials.

## 12. Delivery Plan

### Phase A: LecturAL evidence contract

- Publish `lectural --version --json` and `lectural extract ... --json` in a dedicated LecturAL issue and pull request, including source kind, safe artifact paths, bounded failures, extraction status, timestamp integrity, and representative-frame references independent of OCR success.
- Remove unnecessary local-source path disclosure from portable artifacts, preserve supported YouTube URL compatibility, and make local-only dependency diagnosis agree with its documentation.
- Tag the reviewed LecturAL release after the evidence contract and portability changes land.
- Prove at least one end-to-end audiovisual input path against the user's legitimate session. If no LMS retrieval method is available, v0 accepts an explicitly supplied authorized local media file and reports the unavailable provider capability explicitly.

### Phase B: portable lecture engine

- Complete campusctl portable paths, browser fallback, and unified lock policy.
- Add the portable provider contract, recommended OS-keyring backend, private one-time setup, and insecure-backend rejection.
- Extract or reimplement the CNU provider as an internal campusctl module without bundled credentials, personal configuration, scheduled jobs, notifications, submission actions, or Google Tasks.
- Add the course-source CLI for lecture, assignment, notice, and material discovery; rich assignment and notice source packages; authenticated media retrieval; the selected visible playback queue; authoritative completion checks; and aggregate-safe doctor output.

### Phase C: audiovisual evidence bridge

- Implement `fetch-media` through provider-declared `download` or explicitly confirmed `live-capture`; route authorized local files directly to LecturAL.
- Integrate fetched or local media with `lectural extract`, require complete primary audiovisual evidence, preserve explicit OCR status, and make retained frame images available for selective vision inspection.
- Add runtime-owned requirement and verification records, then make the skill map selected course requirements to source evidence, implementation locations, and applicable executable checks.
- Define cleanup and retention for temporary media.

### Phase D: plugin

- Complete and forward-test the three drafted skills and implement `lecture-tools-runtime`.
- Add compatibility checks and cross-platform CI.
- Validate install, update, rollback, uninstall, and fresh-session discovery.

Setup, update, rollback, and uninstall must select concrete tagged versions from the compatibility matrix. Uninstall removes plugin/runtime files but preserves generated work and credentials unless the user separately requests their deletion.

### Phase E: public release

- Complete the campusctl history rewrite and public/private repository split.
- Run whole-history secret and identifier review.
- Publish tagged engine releases before the plugin release.

## 13. Acceptance Criteria

- A new laptop user completes one private setup flow and thereafter uses natural-language requests.
- Lecture catalog output never mixes in assignments or Google Tasks; `lecture-to-code` may retrieve only explicitly selected assignment, notice, and material sources through their dedicated campusctl commands.
- A retrieved assignment preserves structured text, embedded images, attachments, deadline metadata, and LMS structural provenance, allowing the skill to present nearby lecture candidates without selecting them automatically.
- The plugin never receives, prints, stores, or asks the user to paste an LMS credential.
- A user can synchronize and list incomplete lectures without knowing internal CLI flags.
- Only explicitly selected lectures play, in the requested order and in a visible official player; the provider's mapped LMS state remains the sole completion authority.
- A selected lecture can produce authorized audiovisual evidence through `download`, explicitly confirmed `live-capture`, or a user-supplied authorized local file; the result states any incidental LMS progress behavior.
- If no authorized media retrieval method is available, `lecture-to-code` stops with an actionable remediation rather than degrading to transcript-only code generation.
- Before external model use, the user sees the destination and lecture-derived data classes and explicitly confirms the warning; the confirmation is repeated when destination or policy changes.
- Lecture-to-code output satisfies or explicitly flags every selected requirement, cites relevant source timestamps and evidence types, uses vision for visually dependent claims, and passes applicable local verification steps.
- Campusctl and LecturAL run from separate locked environments.
- `lecture-tools doctor --json` diagnoses every required component without mutating the system.
- Unsupported tool or schema versions fail closed with one actionable remediation.
- Tagged releases, not branches, define the supported plugin dependency set.

## 14. Design Provenance (Non-normative)

- [gajae-code](https://github.com/Yeachan-Heo/gajae-code): centralized diagnosis, machine-readable status, compatibility, and rollback-oriented distribution.
- [pi-server](https://github.com/haesol-shin/pi-server): short-lived branches, gated PRs, squash merge, and deployment separated from merge authority.
- [mattpocock/skills](https://github.com/mattpocock/skills): narrow skills, shared setup, and routed references.
- [paperthin](https://github.com/LilMGenius/paperthin): self-contained skill contracts, single sources of truth, and tag-gated releases.
