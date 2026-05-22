---
name: seedance
description: Generate videos (Seedance 1.5 Pro, Seedance 2 / 2-fast text-to-video, image-to-video, frame mode, reference mode) through RunAPI.ai using the @runapi.ai/seedance Node/TypeScript SDK. Use when the user asks to add Seedance video generation, or writes against @runapi.ai/seedance. Triggers on "seedance", "即梦视频", "video generation", "生成视频", "@runapi.ai/seedance".
documentation: https://runapi.ai/models/seedance
provider_page: https://runapi.ai/providers/bytedance
catalog: https://runapi.ai/models
metadata:
  openclaw:
    homepage: https://runapi.ai/models/seedance
    primaryEnv: RUNAPI_API_KEY
    requires:
      env:
      - RUNAPI_API_KEY
    envVars:
    - name: RUNAPI_API_KEY
      required: true
      description: RunAPI API key from https://runapi.ai/api_keys.
---
# @runapi.ai/seedance — RunAPI.ai Seedance video generation

Build Node / TypeScript integrations that generate Seedance video through RunAPI.ai.

## Setup

Requires **Node 18+** (global `fetch`).

```bash
npm install @runapi.ai/seedance
```

Set your API key in the environment:

```dotenv
# .env
RUNAPI_API_KEY=runapi_xxx   # get one at https://runapi.ai/api_keys
```

```ts
import { SeedanceClient } from '@runapi.ai/seedance';

// The SDK reads RUNAPI_API_KEY from the environment automatically.
const client = new SeedanceClient();
```

Pass `{ apiKey }` explicitly if you manage secrets differently. `baseUrl` defaults to `https://runapi.ai`; override only for local development.

## Core recipe — text to video

```ts
const result = await client.textToVideo.run({
  model: 'seedance-2',
  prompt: 'A drone shot over mountains at sunset',
  aspect_ratio: '16:9',
  duration: 8,
});

const url = result.videos[0].url;
```

`run()` creates the task, auto-polls, and resolves only when the task completes — `videos[0].url` is guaranteed on the resolved value. On failure it throws `TaskFailedError`; on polling timeout it throws `TaskTimeoutError`. Use `run()` for scripts and short-lived processes. For request handlers, split it:

```ts
const { id } = await client.textToVideo.create({ model: 'seedance-2', prompt: '...' });
// return 202 immediately; fetch later:
const status = await client.textToVideo.get(id);
if (status.status === 'completed') { /* ... */ }
```

Do not hold a web worker open waiting on `run()`. Split + webhook is the production pattern.

`run()` polls every 2 s for up to 15 min by default. Tune when needed:

```ts
await client.textToVideo.run(params, { maxWaitMs: 30 * 60_000, pollIntervalMs: 5_000 });
```

If `TaskTimeoutError` fires, the task is still running server-side — resume with `textToVideo.get(id)` or finish via webhook.

## Image-to-video (seedance-1.5-pro)

```ts
await client.textToVideo.run({
  model: 'seedance-1.5-pro',
  prompt: 'The flower blooms and petals scatter',
  aspect_ratio: '16:9',
  input_urls: ['https://cdn.example.com/flower.jpg'],
  resolution: '1080p',
  duration: 8,
  lock_camera: true, // lock camera movement
});
```

## Frame mode (seedance-2 / 2-fast)

Guide with first (required) and optional last frame. **Mutually exclusive with reference mode.**

```ts
await client.textToVideo.run({
  model: 'seedance-2',
  prompt: 'A sunrise over the ocean',
  first_frame_url: 'https://cdn.example.com/start.jpg',
  last_frame_url: 'https://cdn.example.com/end.jpg',
});
```

## Reference mode (seedance-2 / 2-fast)

Guide with reference images, videos, or audio. **Mutually exclusive with frame mode.**

```ts
await client.textToVideo.run({
  model: 'seedance-2',
  prompt: 'A person dancing in the same style',
  reference_video_urls: ['https://cdn.example.com/dance.mp4'],
  reference_audio_urls: ['https://cdn.example.com/beat.mp3'],
});
```

## Models

| `model` | Modes | Notes |
|---|---|---|
| `seedance-1.5-pro` | text-to-video, image-to-video (`input_urls`), `lock_camera` | `aspect_ratio` required. `resolution`: `480p` / `720p` / `1080p`. `duration`: 4 / 8 / 12. |
| `seedance-2` | text / frame / reference | `resolution`: `480p` / `720p` / `1080p`. `duration`: integer 4–15. |
| `seedance-2-fast` | text / frame / reference | Faster, cheaper variant of `seedance-2`. `resolution`: `480p` / `720p`. |
| `seedance-v1-lite` | text-to-video, image-to-video (auto-detected via `input_urls`) | `aspect_ratio` required in text mode (`…, 9:21`). `resolution`: `480p` / `720p` / `1080p`. `duration`: `"5"` or `"10"`. Image mode supports `last_frame_url`. Extras: `seed`, `enable_safety_checker`, `lock_camera`. |
| `seedance-v1-pro` | text-to-video, image-to-video (auto-detected via `input_urls`) | Same as v1-lite but `aspect_ratio` set is `…, 21:9`. No `last_frame_url`. |
| `seedance-v1-pro-fast` | image-to-video only | `input_urls` required. `resolution`: `720p` / `1080p`. `duration`: `"5"` or `"10"`. Narrow param set — no `aspect_ratio` / `lock_camera` / `seed` / `enable_safety_checker`. |

Exact credit costs per model are shown at https://runapi.ai/pricing and in the dashboard — do not hardcode prices in application code.

## Callbacks (webhooks)

Pass `callback_url` on `create()` (or any `run()` call) and RunAPI will POST the final payload to you:

```ts
await client.textToVideo.create({
  model: 'seedance-2',
  prompt: '...',
  callback_url: 'https://your.app/webhooks/runapi/seedance',
});
```

Payload shape:

```ts
{ id: string; status: 'completed' | 'failed'; videos?: { url: string }[]; last_frame_url?: string; error?: string }
```

**Always verify the signature before trusting the body.** RunAPI signs every callback with your account's Callback Secret (rotate at `/accounts/callback_secret`). Headers:

- `X-Callback-Id` — UUID, store to make handler idempotent
- `X-Callback-Timestamp` — unix seconds, reject if `|now - ts| > 300`
- `X-Callback-Signature` — base64 HMAC-SHA256 over `` `${id}.${ts}.${rawBody}` `` using the base64-decoded secret

```ts
import crypto from 'node:crypto';

function verify(raw: string, id: string, ts: string, sig: string, secret: string) {
  const key = Buffer.from(secret, 'base64');
  const mac = crypto.createHmac('sha256', key)
    .update(`${id}.${ts}.${raw}`)
    .digest('base64');
  return crypto.timingSafeEqual(Buffer.from(mac), Buffer.from(sig));
}
```

Reply `2xx` within 10s; any non-2xx triggers retries.

## Errors

All errors are re-exported from `@runapi.ai/core`. Always `instanceof` — never string-match messages.

| Error | Status | Action |
|---|---|---|
| `AuthenticationError` | 401 | abort; surface "reconnect your API key" |
| `InsufficientCreditsError` | 402 | prompt user to top up at runapi.ai/billing |
| `ValidationError` | 400 / 422 | fix params; do not retry |
| `RateLimitError` | 429 | sleep `err.retryAfterMs`, then retry |
| `ServiceUnavailableError` | 503 / 455 | retry with backoff; transient service issue |
| `TaskFailedError` | — | show `err.details` to user; do not auto-retry |
| `TaskTimeoutError` | — | re-poll with `textToVideo.get(id)` |

```ts
import { InsufficientCreditsError, TaskFailedError } from '@runapi.ai/seedance';

try {
  await client.textToVideo.run({ model: 'seedance-2', prompt: '...' });
} catch (err) {
  if (err instanceof InsufficientCreditsError) { /* surface top-up CTA */ }
  else if (err instanceof TaskFailedError)       { /* show err.details */ }
  else throw err;
}
```

## Gotchas

- `model` is required on every call.
- For `seedance-1.5-pro`, `aspect_ratio` is required and `resolution`/`duration` use a different set of values than the 2.x models.
- For `seedance-2` / `seedance-2-fast`, **frame mode and reference mode are mutually exclusive** — do not send both `first_frame_url` and `reference_*_urls` in the same request.
- `aspect_ratio: 'auto'` is 2.x-only (useful with frame mode to inherit dimensions from the reference frame).
- `reference_audio_urls` requires at least one `reference_image_urls` or `reference_video_urls` to be set.
- `last_frame_url` is optional in the response and may be absent.
- `callback_url` must be reachable from the public internet. `localhost` / `127.0.0.1` URLs will never fire — use a tunnel (cloudflared, ngrok, tailscale funnel) when developing locally.

## Dig deeper

Package README (full API surface, all params): `node_modules/@runapi.ai/seedance/README.md`. Types: `@runapi.ai/seedance/dist/types.d.ts`. Product docs: https://runapi.ai/docs.

## RunAPI public routing

seedance api public links use the API-379 catalog route map. The main seedance api page is https://runapi.ai/models/seedance. SDK docs live at https://runapi.ai/docs#sdk-seedance and product docs live at https://runapi.ai/docs#seedance.

Pricing, rate limits, and commercial usage for seedance api should point to the most specific variant page:
- [v1 lite](https://runapi.ai/models/seedance/v1-lite)
- [v1 pro](https://runapi.ai/models/seedance/v1-pro)
- [v1 pro fast](https://runapi.ai/models/seedance/v1-pro-fast)
- [1.5 pro](https://runapi.ai/models/seedance/1.5-pro)
- [2.0](https://runapi.ai/models/seedance/2.0)
- [2.0 fast](https://runapi.ai/models/seedance/2.0-fast)

Compare Seedance with other Bytedance models at https://runapi.ai/providers/bytedance. Browse every RunAPI model and skill at https://runapi.ai/models. SDK repository: https://github.com/runapi-ai/seedance-sdk. Skill repository: https://github.com/runapi-ai/seedance.
