# Agent Skills Architecture

## Purpose

This repository is the single entry point for reusable agent behavior. It owns skills, bundles, cross-agent packaging, and compatibility declarations. It does not absorb the applications that perform domain work.

## Ownership model

| Layer | Owns | Does not own |
| --- | --- | --- |
| Engine | Authentication, data access, deterministic processing, JSON CLI | Agent prompting or multi-engine workflow |
| Skill | When and how an agent uses one bounded capability | Engine implementation or credentials |
| Bundle | Related skills, user experience, cross-engine workflow, compatibility | Dependency source code |
| Adapter | Harness-specific manifest and discovery metadata | Canonical skill instructions |
| Profile | A user's selected bundles and pinned releases | Shared product defaults |

Current engines remain independent:

- [`campusctl`](https://github.com/haesol-shin/notice-bot) is the target public CLI extracted from the existing repository. Its CNU implementation remains an internal provider module until another institution proves a stable separation boundary.
- [`lectural`](https://github.com/haesol-shin/lectural) extracts evidence and study artifacts from supported video sources.
- `notice-bot` remains the owner's personal scheduled automation application for notifications, tasks, and boards; it may consume campusctl later but does not define the public campus contract.

## External reference policy

Use three forms of reference for three different jobs:

1. README and navigation use a repository or default-branch GitHub link for discoverability.
2. Specs link to the engine-owned contract path and name the required schema version.
3. Released `bundle.toml` and lock data pin a tag or commit plus an integrity hash.

Do not use git submodules. A bundle consumes released CLI contracts rather than importing engine source. Development branches may appear in an experimental bundle, but publication is blocked until every required engine has a reviewed tag.

## Repository policy

A capability gets a separate repository when it has an independent runtime, dependency graph, security boundary, or release cadence. Skills and bundles stay in this monorepo until they need independent maintainers or releases. Canonical `SKILL.md` content is shared across harnesses; platform-specific metadata stays in adapters or native manifests.
