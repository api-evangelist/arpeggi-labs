---
name: Separate vocals and stems from a track
description: Submit a mixed audio file for vocal separation or stem splitting, then poll for the isolated vocal and instrument stem URLs.
api: openapi/arpeggi-labs-kits-openapi.yml
operations: [createVocalSeparation, getVocalSeparation, createStemSplit, getStemSplit]
---

# Separate vocals and stems from a track

Use the Kits AI API (base `https://arpeggi.io/api/kits/v1`) to isolate vocals or
split a track into instrument stems.

## Auth
Send `Authorization: Bearer <api-key>`. Rate limit: 1 POST/min per account.

## Choose the job
- **Vocals only** → vocal separation.
- **Full stem breakdown** (vocals + individual instruments) → stem splitter.

## Steps
1. **Create the job** — POST `multipart/form-data` with `inputFile`
   (wav/webm/mp3/flac, max 50MB):
   - `createVocalSeparation` (POST `/vocal-separations`), or
   - `createStemSplit` (POST `/stem-splits`).
   The response is a SeparationJob with `status: "running"`.
2. **Poll for the result** — call `getVocalSeparation` (GET
   `/vocal-separations/{id}`) or `getStemSplit` (GET `/stem-splits/{id}`) until
   `status` is `success`.
3. **Read outputs** — use `vocalAudioFileUrl` for the isolated vocal and
   `stemFileUrls[]` (each `{instrument, url}`) for instrument stems. Signed URLs
   expire in 4 hours.

## Rules
- `backingAudioFileUrl` is DEPRECATED — use `stemFileUrls`/`lossyStemFileUrls`
  with instrument `"backing"` instead.
- Only `success` yields output URLs; handle `error`/`cancelled`.
- Respect the 1 POST/min limit; on 429 back off, on 422 fix the upload.
