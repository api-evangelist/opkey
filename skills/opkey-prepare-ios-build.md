---
name: Resign an iOS build (or instrument an APK) for pCloudy devices
description: Run the two asynchronous job flows in the pCloudy API — iOS app resigning and Android APK instrumentation — from initiate through polled progress to download and install.
api: openapi/opkey-pcloudy-openapi.yml
operations:
  - authentication
  - uploadApp
  - initiateResigning
  - resigningProgress
  - downloadResigned
  - instrumentInitiate
  - instrumentationProgress
  - downloadInstrumented
  - installLaunchApp
generated: '2026-08-04'
method: generated
source: openapi/opkey-pcloudy-openapi.yml + conventions/opkey-conventions.yml
---

# Resign an iOS build (or instrument an APK) for pCloudy devices

These are the only two **asynchronous, polled** flows in the pCloudy API. Everything else
is request/response. Both follow the identical shape:
`initiate → poll progress → download`.

## 0. Authenticate and upload

`authentication` (`GET /api/access`, HTTP Basic) then `uploadApp`
(`POST /api/upload_file`, multipart). Both jobs operate on a file already sitting on the
cloud drive, referenced by `file_name`.

## iOS resigning

An enterprise/internal iOS build usually has to be re-signed before pCloudy devices will
install it.

1. `initiateResigning` — `POST /api/resign/initiate`. Starts the job for `file_name`.
2. `resigningProgress` — `POST /api/resign/progress`. **Poll this.** Opkey's own MCP
   server caps the poll at 20 attempts (`MAXIMUM_RETRIES_RESIGN_PROGRESS = 20`) before
   giving up — use that as your ceiling rather than looping forever.
3. `downloadResigned` — `POST /api/resign/download`. Retrieves the resigned artifact.

Then install with `installLaunchApp` against a booked `rid`.

## Android APK instrumentation

Instrumentation adds test visibility to an APK.

1. `instrumentInitiate` — `POST /api/app_instrumentation/initiate`
2. `instrumentationProgress` — `POST /api/app_instrumentation/progress` (poll)
3. `downloadInstrumented` — download the instrumented APK

The legacy generation of the same flow is still documented at `/api/instrument/initiate`
and `/api/instrument/progress` (`instrumentInitiateLegacy`,
`instrumentationProgressLegacy`). Prefer the `app_instrumentation` paths; the legacy pair
is superseded but **carries no deprecation marker and no Sunset header**, so there is no
published signal for when it stops working.

## Rules an agent must respect

- **Never re-initiate on a timeout.** There is no idempotency key. If `initiateResigning`
  or `instrumentInitiate` times out, poll progress for the existing `file_name` before
  starting a second job — a duplicate initiate creates a duplicate job.
- **Bound your polling.** No `Retry-After`, no rate-limit contract, no terminal-state
  guarantee is published. Cap attempts (20 is the vendor's own number) and fail loudly.
- **These are the two operations where a stuck poll is normal.** Progress can sit
  unchanged for a while; that is not an error condition.
- **Read the envelope.** These paths are legacy-generation, so a failure can arrive as a
  200 with `result.error`. Check the body.
