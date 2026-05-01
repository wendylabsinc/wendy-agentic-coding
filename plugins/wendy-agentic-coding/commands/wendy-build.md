---
description: Build a Wendy app without deploying or starting it.
argument-hint: [app path, device hostname, or build type]
---

Use the `wendy-app-lifecycle` skill. Build the app for:

`$ARGUMENTS`

Prefer explicit `--device` and `--build-type` when needed. Do not rely on `--json` for structured build output; use exit status and text logs.
