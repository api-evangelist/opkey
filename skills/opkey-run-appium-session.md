---
name: Run an Appium automation session on pCloudy devices
description: Book devices for Appium, initialise the pCloudy Appium hub, drive a WebDriver session against real devices, and collect the report and performance files.
api: openapi/opkey-pcloudy-openapi.yml
operations:
  - authentication
  - uploadApp
  - bookDevicesAppium
  - initAppiumhub
  - getAppiumEndpoint
  - appiumSession
  - updateAppiumSession
  - getAppiumReport
  - getAppiumFileList
  - downloadAppiumPerf
  - appiumShareableLink
  - releaseDeviceLegacy
generated: '2026-08-04'
method: generated
source: openapi/opkey-pcloudy-openapi.yml + conventions/opkey-conventions.yml
---

# Run an Appium automation session on pCloudy devices

pCloudy's automation surface is a **WebDriver hub in front of real handsets**. The REST
API books the devices and hands you the hub; the actual test driving happens over the
Appium protocol, not over this API.

## 1. Authenticate — `authentication`

`GET <Cloud URL>/api/access` with HTTP Basic (email + access key). Keep the token; see
`conventions/opkey-conventions.yml` for the header-vs-body split.

## 2. Upload the build — `uploadApp`

`POST /api/upload_file` (multipart) with `file`, `source_type=raw`, `token`, `filter`.
The app must be on the cloud drive before the Appium session can reference it.

## 3. Book devices for Appium — `bookDevicesAppium`

`POST /api/appium/init` reserves the devices the run will use. The legacy equivalent is
`bookDevicesAppiumLegacy` (`POST /api/appium/devices`) — prefer the current path.

## 4. Bring up the hub — `initAppiumhub`, `getAppiumEndpoint`

`initAppiumhub` starts the hub for the booking; `getAppiumEndpoint` returns the WebDriver
URL to point your Appium client at. pCloudy's documented hub is
`https://device.pcloudy.com/appiumcloud/wd/hub`.

From here you are speaking Appium/WebDriver, using pCloudy's documented desired
capabilities. Nothing in this API drives the test steps.

Single-device convenience: `lonelyappium` (`POST /api/lonelyappium`) exists for the
one-device case.

## 5. Track and annotate the session — `appiumSession`, `updateAppiumSession`

`updateAppiumSession` (`POST /api/appium/update_session`) is how you write run metadata
back so the report reflects what the run actually was.

## 6. Collect artifacts — `getAppiumReport`, `getAppiumFileList`, `downloadAppiumPerf`

- `getAppiumReport` — the run report
- `getAppiumFileList` (`POST /api/appium/get_appium_file_list`) — what was produced
- `downloadAppiumPerf` — performance capture from the run
- `appiumShareableLink` — a link to hand to someone who is not in the account

## 7. Release — `releaseDeviceLegacy`

`POST /api/release_device` for every `rid` you booked. Non-negotiable: pCloudy warns that
not releasing can get your IP temporarily blocked, and in CI that blocks the whole runner.

## Rules an agent must respect

- **Booking is the expensive, non-idempotent step.** No idempotency key exists. If
  `bookDevicesAppium` times out, query state before retrying — a blind retry can double
  your device allocation and your bill.
- **The hub is not the API.** Do not try to model test steps as REST calls; there are no
  operations for them. Hand the endpoint to a real Appium client.
- **Two envelopes.** `/api/v2/*` returns `{traceId, requestId, statusCode, status, message}`;
  legacy `/api/*` returns `{result: {...}}` and can report failure inside a 200.
- **No event surface.** There is no webhook and no published AsyncAPI for run completion.
  Poll `getAppiumReport` / `getAppiumFileList`; do not wait for a callback that will never
  arrive.
