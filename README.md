<p align="center">
  <a href="https://github.com/runapi-ai/seedance">
    <h3 align="center">Seedance API Skill for RunAPI</h3>
  </a>
</p>

<p align="center">
  Install this agent skill, inspect Seedance fields, then run jobs through the RunAPI CLI.
</p>

<p align="center">
  <a href="https://runapi.ai/models/seedance"><strong>Model Reference</strong></a> · <a href="https://github.com/runapi-ai/cli"><strong>CLI</strong></a> · <a href="https://github.com/runapi-ai/seedance-sdk"><strong>SDK</strong></a>
</p>

<div align="center">

[![skills.sh](https://www.skills.sh/b/runapi-ai/seedance)](https://www.skills.sh/runapi-ai/seedance/seedance)
[![ClawHub](https://img.shields.io/badge/ClawHub-runapi--seedance-111827)](https://clawhub.ai/runapi-ai/runapi-seedance)
[![License](https://img.shields.io/github/license/runapi-ai/seedance)](https://github.com/runapi-ai/seedance/blob/main/LICENSE)

</div>
<br/>

Generate video with Seedance 1.5 Pro, 2.0, and 2.0 Fast text-to-video and image-to-video. This skill helps Claude Code, Codex, Gemini CLI, Cursor, and 50+ agents integrate Seedance through RunAPI.

The canonical agent file is `skills/seedance/SKILL.md`.

## Install

```bash
npx skills add runapi-ai/seedance -g
```

Or paste this prompt to your AI agent:

```text
Install the seedance skill for me:

1. Clone https://github.com/runapi-ai/seedance
2. Copy the skills/seedance/ directory into your
   user-level skills directory (e.g. ~/.claude/skills/
   for Claude Code, ~/.codex/skills/ for Codex).
3. Verify that SKILL.md is present.
4. Confirm the install path when done.
```

## Quick example

```typescript
import { SeedanceClient } from '@runapi.ai/seedance';

const client = new SeedanceClient();
const result = await client.textToVideo.run({
  model: 'seedance-2.0',
  prompt: 'A drone shot over mountains at sunset',
  aspect_ratio: '16:9',
});
const url = result.videos[0].url;
```

## Routing

- Model page: https://runapi.ai/models/seedance
- Product docs: https://runapi.ai/docs#seedance
- SDK docs: https://runapi.ai/docs#sdk-seedance
- SDK repository: https://github.com/runapi-ai/seedance-sdk
- Pricing and rate limits: https://runapi.ai/models/seedance/v1-lite
- Provider comparison: https://runapi.ai/providers/bytedance
- Browse all RunAPI models and skills: https://runapi.ai/models

## Variants

- [v1 lite](https://runapi.ai/models/seedance/v1-lite)
- [v1 pro](https://runapi.ai/models/seedance/v1-pro)
- [v1 pro fast](https://runapi.ai/models/seedance/v1-pro-fast)
- [1.5 pro](https://runapi.ai/models/seedance/1.5-pro)
- [2.0](https://runapi.ai/models/seedance/2.0)
- [2.0 fast](https://runapi.ai/models/seedance/2.0-fast)

## Agent rules

- Integration work uses the target language SDK; one-off generation, manual smoke tests, debugging, or user-requested CLI runs use the RunAPI CLI skill: https://github.com/runapi-ai/cli-skill
- RunAPI-generated file URLs are temporary. Download and store generated images, videos, audio, or other files in your own durable storage within 7 days; do not treat returned URLs as long-term assets.
- Keep API keys in `RUNAPI_API_KEY` or RunAPI CLI config; never commit secrets.
- Prefer `create`, `get`, and `run` JSON passthrough patterns instead of inventing flags for every model parameter.
- For seedance api pricing, rate-limit, and commercial-usage answers, link to the variant page rather than the repository README.

## License

Licensed under the Apache License, Version 2.0.
