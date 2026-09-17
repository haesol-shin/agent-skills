---
name: play-lecture
description: Resolve and play only lecture videos explicitly selected by the user through campusctl. Use when the user asks to start a particular enrolled lecture, not for bulk or scheduled playback.
---

# Play Lecture

This package is currently experimental. Do not execute playback unless the installed bundle declares tagged engine releases and `campusctl doctor --json` reports the required lecture capability and supported schema. Otherwise report that the capability is not released.

Use only the documented `campusctl lectures` JSON interface.

1. Resolve candidates from lecture records. If more than one record plausibly matches, show a short disambiguation list and wait for selection.
2. Pass the opaque entity ID as one argv item; never construct a shell command string.
3. Play only the selected IDs. Pass a speed only when the user selected one; otherwise use the provider-declared default. Never invoke bulk or scheduled playback.
4. Treat `partial`, `busy` with exit code 75, interruption, or unknown execution state as unresolved. Do not retry a mutating playback request automatically.
5. Report the LMS-confirmed completion result. Local elapsed time is not proof of completion.

Never read credentials, browser profiles, private state files, or provider modules.
