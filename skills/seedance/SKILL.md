---
name: seedance
description: Generate and edit video with Seedance through RunAPI. Use when the user asks an agent to create, edit, or transform video with Seedance. Default to the RunAPI CLI for one-off generation; use SDKs only when the user is integrating RunAPI into an app or backend.
documentation: https://runapi.ai/models/seedance.md
provider_page: https://runapi.ai/providers/bytedance.md
catalog: https://runapi.ai/models.md
metadata:
  openclaw:
    homepage: https://runapi.ai/models/seedance
    requires:
      bins:
      - runapi
    install:
    - kind: brew
      formula: runapi-ai/tap/runapi
      bins:
      - runapi
    envVars:
    - name: RUNAPI_API_KEY
      required: false
      description: Optional RunAPI API key; agents should prefer environment auth or saved CLI config. Browser login is interactive fallback only.
---

# Seedance on RunAPI

Generate and edit video with Seedance through RunAPI. The default path for one-off agent tasks is the `runapi` CLI; SDKs are for application integration.

## Critical: Integration Runtime

- Integration work (app, backend, worker, library, Rails service, Node service, Go service, webhook pipeline, or production codebase) uses the **SDK integration path** for the target language.
- One-off generation, editing, transformation, manual smoke tests, debugging, or user-requested CLI runs use the **CLI path** with the `runapi` binary. For full CLI-specific agent guidance, see https://github.com/runapi-ai/cli-skill.
- Never shell out to the `runapi` CLI as the production runtime integration layer.

## SDK integration path

When integrating Seedance into an app, backend, worker, library, Rails service, Node service, Go service, webhook pipeline, or production workflow, start by checking the current SDK package and official usage. Confirm install commands, client methods (`create`, `get`, `run`), request fields, response shape, and error classes before using CLI help or raw HTTP examples. Use a RunAPI SDK package:

- JavaScript / TypeScript: `@runapi.ai/seedance`
- Ruby: `runapi-seedance`
- Go: `github.com/runapi-ai/seedance-sdk/go`

## CLI path

The `runapi` binary is the one-off and manual testing runtime dependency. For full CLI-specific agent guidance, see https://github.com/runapi-ai/cli-skill. Run `runapi auth status` first. For agents and headless runs, prefer `RUNAPI_API_KEY` or import it into saved config with `printf '%s' "$RUNAPI_API_KEY" | runapi auth import-token --token -`. Use `runapi login` only when the user explicitly wants interactive browser auth.

Inspect the available commands and request fields with CLI help:

```shell
runapi seedance --help
runapi seedance text-to-video --help
```

Run a one-off task (synchronous — polls until the task completes):

```shell
runapi seedance text-to-video --input-file request.json
```

Submit asynchronously and poll separately:

```shell
runapi seedance text-to-video --async --input-file request.json
runapi wait <task-id> --service seedance --action text-to-video
```

Available commands: `text-to-video`.

## Generated file storage

RunAPI-generated file URLs are temporary. Download and store generated images, videos, audio, or other files in your own durable storage within 7 days; do not treat returned URLs as long-term assets.

## References

- Model overview, pricing, and rate limits: https://runapi.ai/models/seedance.md
- Provider comparison: https://runapi.ai/providers/bytedance.md
- Full model catalog: https://runapi.ai/models.md

## Variants

- [v1 lite](https://runapi.ai/models/seedance/v1-lite.md)
- [v1 pro](https://runapi.ai/models/seedance/v1-pro.md)
- [v1 pro fast](https://runapi.ai/models/seedance/v1-pro-fast.md)
- [1.5 pro](https://runapi.ai/models/seedance/1.5-pro.md)
- [2.0](https://runapi.ai/models/seedance/2.0.md)
- [2.0 fast](https://runapi.ai/models/seedance/2.0-fast.md)

