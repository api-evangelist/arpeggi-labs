---
name: Blend voice models into a new voice
description: Combine two to four Kits AI voice models with alpha weights to create a new voice model, then poll until it is usable.
api: openapi/arpeggi-labs-kits-openapi.yml
operations: [listVoiceModels, createVoiceBlend, getVoiceBlend]
---

# Blend voice models into a new voice

Use the Kits AI API (base `https://arpeggi.io/api/kits/v1`) to create a new voice
model by blending existing ones.

## Auth
Send `Authorization: Bearer <api-key>`. Rate limit: 1 POST/min per account.

## Steps
1. **Pick models** — call `listVoiceModels` (GET `/voice-models`) and choose 2-4
   usable model `id`s.
2. **Create the blend** — call `createVoiceBlend` (POST `/voice-blender`) as
   `application/json` with `modelId1`, `modelId2` (and optional `modelId3`,
   `modelId4`), `alpha`, and one fewer alpha value than models: provide `alpha2`
   if `modelId3` is set, `alpha3` if `modelId4` is set. The final alpha is
   computed automatically as `1 - sum(others)`. Optionally set `title`. The
   response includes `outputModelId` and `status: "running"`.
3. **Poll until usable** — call `getVoiceBlend` (GET `/voice-blender/{id}`) until
   `status` is `success`; the `outputModelId` is then usable for conversions.

## Rules
- Provide exactly one fewer alpha than the number of models; each alpha is 0..1.
- The output model is only usable after `status: success` — do not use
  `outputModelId` before then.
- Feed `outputModelId` into `createVoiceConversion` (see the convert-voice skill)
  to render audio with the blended voice.
- Respect the 1 POST/min limit; handle 401/422/429.
