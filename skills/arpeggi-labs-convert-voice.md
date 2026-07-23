---
name: Convert a performance to a Kits AI voice
description: Pick a voice model, submit an audio file for voice conversion, and poll until the converted output URL is ready.
api: openapi/arpeggi-labs-kits-openapi.yml
operations: [listVoiceModels, createVoiceConversion, getVoiceConversion]
---

# Convert a performance to a Kits AI voice

Use the Kits AI API (base `https://arpeggi.io/api/kits/v1`) to render an input
vocal through a target voice model.

## Auth
Send `Authorization: Bearer <api-key>` on every request. Create a key at
https://app.kits.ai/api-access. Rate limit: 1 POST/min per account.

## Steps
1. **Find a voice model** — call `listVoiceModels` (GET `/voice-models`). Filter
   with `instruments=true` for instrument models or `myModels=true` for your own.
   Note a usable model's `id` (`isUsable: true`).
2. **Submit the conversion** — call `createVoiceConversion` (POST
   `/voice-conversions`) as `multipart/form-data` with `voiceModelId` and
   `soundFile` (wav/mp3/flac, max 100MB). Optional: `conversionStrength`,
   `modelVolumeMix`, `pitchShift` (-24..24), and `pre`/`post` effect objects.
   The response is an InferenceJob with `status: "running"`.
3. **Poll for the result** — call `getVoiceConversion` (GET
   `/voice-conversions/{id}`) until `status` is `success`. Read `outputFileUrl`
   (a signed URL that expires in 4 hours). If `status` is `error` or
   `cancelled`, stop and surface the failure.

## Rules
- POST is capped at 1/min per account — do not retry create calls in a tight loop.
- Do not treat a `running` job as complete; only `success` yields output URLs.
- Download the output within 4 hours before the signed URL expires.
- On 401 re-check the bearer key; on 429 back off; on 422 fix the request body.
