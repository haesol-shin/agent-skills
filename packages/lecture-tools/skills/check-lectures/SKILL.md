---
name: check-lectures
description: Check or synchronize a user's lecture catalog and report remaining lectures through the campusctl JSON CLI. Use for lecture availability or completion-status requests, not assignments, notices, or general task management.
---

# Check Lectures

This package is currently experimental. Do not execute the workflow unless the installed bundle declares tagged engine releases and `campusctl doctor --json` reports the required lecture capability and supported schema. Otherwise report that the capability is not released.

Use only the documented campusctl JSON interface. Never import engine modules, read its private state files, or access credentials directly.

1. Verify the supported command and schema through `campusctl doctor --json` when compatibility is not already established.
2. Run `campusctl sync --only lectures --json` only when the user asks for current data or `lectures list --json` reports that its cache is unavailable.
3. List incomplete lectures by default. Use `campusctl lectures list --all --json` only when all records are requested and doctor reports that capability.
4. Present course, title, sequence, due date, and completion state without exposing unrelated assignments, notices, identifiers, or provider details.
5. If the command or schema is unavailable, stop with the engine's safe remediation. Do not scrape human-readable output or fall back to legacy flags.
