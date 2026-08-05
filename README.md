# Opkey

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Opkey (Smart Software Testing Solutions, Inc.) is a Cloud Application Lifecycle Management and
AI-powered test automation vendor for enterprise packaged applications — Oracle Cloud/EBS, Workday,
Salesforce, SAP, Coupa, Veeva, UKG and Microsoft Dynamics.

Opkey's public developer surface lives on **pCloudy**, the real-device mobile and browser testing
cloud it operates. The core Opkey product help and user guides are **not** public — everything under
`www.opkey.com/Help/docs/` 301-redirects to a Freshdesk customer hub that requires a login. pCloudy's
documentation, API reference and release notes are public.

## What is profiled here

| Surface | Where |
|---|---|
| pCloudy Device Cloud API — 90 documented operations | `openapi/` (transcribed; Opkey publishes no OpenAPI) |
| pCloudy MCP server — 36 tools, `uvx pcloudy-mcp` | `mcp/` |
| MCP → REST tool crosswalk | `mcp/opkey-tool-crosswalk.yml` |
| Auth, conventions, errors, lifecycle, data model | `authentication/` `conventions/` `errors/` `lifecycle/` `data-model/` |
| Packages, CLI utilities, sandbox, changelog | `packages/` `cli/` `sandbox/` `changelog/` |
| Agent skills grounded in real operationIds | `skills/` |

**Provenance note.** `openapi/opkey-pcloudy-openapi.yml` is an API Evangelist transcription of the
operations, parameters and response samples Opkey publishes at
<https://content.pcloudy.com/apidocs/>. It is a third-party artifact, not a provider document.
Opkey does not publish a machine-readable specification.

## Links

- Company — https://www.opkey.com/
- pCloudy docs — https://www.pcloudy.com/docs/
- pCloudy API reference — https://content.pcloudy.com/apidocs/index.html
- pCloudy MCP server — https://www.pcloudy.com/docs/mcp-server/
- Security & Trust — https://www.opkey.com/security-and-trust
- GitHub — https://github.com/Smart-Software-Testing-Solutions-Opkey
- Secondary-market listing (original harvest source) — https://forgeglobal.com/opkey_stock/
