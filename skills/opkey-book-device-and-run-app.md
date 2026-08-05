---
name: Book a pCloudy device and exercise an app on it
description: Authenticate to pCloudy, upload an app build, book a real device, install and launch the app, drive it, then release the device cleanly.
api: openapi/opkey-pcloudy-openapi.yml
operations:
  - authentication
  - uploadApp
  - getAvailableApps
  - getDeviceList
  - bookDevice
  - installLaunchApp
  - sendText
  - rotateDevice
  - killApp
  - releaseDeviceLegacy
generated: '2026-08-04'
method: generated
source: openapi/opkey-pcloudy-openapi.yml + conventions/opkey-conventions.yml
---

# Book a pCloudy device and exercise an app on it

This is the spine of every pCloudy flow. A **reservation id (`rid`)** is the join key for
almost everything, so the order below is not optional.

## Before you start

- Base URL is per tenant. The docs call it `<Cloud URL>`; production tenants seen in
  Opkey's own tooling are `device`, `us`, `sg`, `uae`, `ind-west`, `ind-west2`.pcloudy.com,
  plus private/on-prem hosts. Default to `https://device.pcloudy.com`.
- Everything is `POST` and `Content-Type: application/json` **except** authentication.

## 1. Mint an access token — `authentication`

`GET <Cloud URL>/api/access` with HTTP Basic: username = your pCloudy email,
password = the access key from **Account Settings → API**.

The token comes back at `result.token`. **Watch the envelope**: this legacy path can return
HTTP 200 with `result.error` set. Read the body, not the status line.

Carry the token two different ways depending on the operation — this is the sharpest edge
in the API:

- `/api/v2/*` operations: a **`token` request header**
- legacy `/api/*` operations: a **`token` field in the JSON body**

There is no documented TTL. Opkey's own MCP server caches the token for 5 days and
re-authenticates when a call is rejected; do the same.

## 2. Upload the build — `uploadApp`

`POST /api/upload_file` as `multipart/form-data` with `file`, `source_type=raw`,
`token`, and `filter` (`all` / `apk` / `ipa`). The binary lands in the account cloud drive.

Confirm with `getAvailableApps` (`POST /api/drive`, takes `token`, `limit`, `filter`) —
this is the only operation in the whole API with anything resembling a page size, and
there is no cursor, so do not build a paging loop.

## 3. Pick and book a device — `getDeviceList`, `bookDevice`

`getDeviceList` (`POST /api/devices`) returns the available handsets for a platform.
`bookDevice` (`POST /api/book_device`) reserves one and **returns the `rid`**.

Booking allocates a billable real device. Opkey's MCP server books for 30 minutes by
default. Persist the `rid` immediately — if you lose it you cannot release the device.

## 4. Install, launch and drive — `installLaunchApp` and friends

- `installLaunchApp` (`POST /api/install_app`) — install and launch in one call
- `sendText` (`POST /api/v2/send-text`) — type into the focused field
- `rotateDevice` (`POST /api/v2/rotate-device`) — `rotate` is `"L"` or `"P"`
- `sendHome` (`POST /api/v2/send-home`) — background the app
- `killApp` (`POST /api/v2/generic/kill-app`) — force-stop without uninstalling
- `executeAdb` (`POST /api/v2/execute-adb`) — Android escape hatch

All of these take `rid`. The `/api/v2/*` ones return
`{traceId, requestId, statusCode, status, message}` — quote `traceId` and `requestId`
in any support conversation.

## 5. Release — `releaseDeviceLegacy`

`POST /api/release_device`. **Always call it, including on the failure path.** pCloudy
documents that failing to release can get the caller's IP temporarily blocked. Wrap steps
3–4 so release runs in a `finally`.

## Rules an agent must respect

- **No idempotency.** There is no `Idempotency-Key` on any of the 90 documented
  operations. A retried `bookDevice` after a timeout can allocate a *second* device. On
  any timeout, do not blind-retry a booking or an install — reconcile by checking whether
  you already hold an `rid`.
- **No rate-limit contract.** No `429` semantics, no `Retry-After`, no quota headers are
  published. Back off conservatively on your own schedule.
- **401 means re-mint.** Every operation declares `401` for a missing/expired/invalid
  token. Re-run `authentication`, do not retry with the same token.
- **Two API generations coexist.** `/api/*` and `/api/v2/*` overlap, with the token in a
  different place and a different response envelope. Never assume the shape; check which
  generation the operation you called belongs to.
