<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Google Ad Library API

[![SocQ Google Ad Library API](https://img.shields.io/badge/API-SocQ%20Google%20Ad%20Library%20API-F64C31)](https://socq.ai/apis/google-ad-library?utm_source=github&utm_medium=readme&utm_campaign=socq-google-ad-library-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/google-ad-library/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/google-ad-library-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/google-ad-library-api/actions/workflows/check.yml)

3 endpoints for public Google Ad Library data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Google Ad Library APIs](https://socq.ai/apis/google-ad-library?utm_source=github&utm_medium=readme&utm_campaign=socq-google-ad-library-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-google-ad-library-api) · [Documentation](https://docs.socq.ai/api-manual/google-ad-library/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Google Ad Library Ad API](#google-ad-library-ad-api) | `google-ad-library/ad` | Retrieve one public Google Ads Transparency Center creative from its exact URL, returning advertiser identity, copy, media, delivery dates, regions, format, destination, and available impressions in a normalized record. | 0.5 credits ($0.0025) per result | 2026-08-11 |
| [Google Ad Library Advertiser Search API](#google-ad-library-advertiser-search-api) | `google-ad-library/advertiser-search` | Discover public Google Ads Transparency Center advertiser candidates by query and optional region, returning normalized identity, website, logo, region, and available ad-count fields. | 0.5 credits ($0.0025) per result | 2026-08-11 |
| [Google Ad Library Company Ads API](#google-ad-library-company-ads-api) | `google-ad-library/company-ads` | Collect public Google Ads Transparency Center creatives for a company domain or advertiser ID with optional topic, region, date, surface, format, and bounded result filters. | 0.5 credits ($0.0025) per result | 2026-08-11 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-google-ad-library-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/google-ad-library-api.git
cd google-ad-library-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs google-ad-library/ad

# Python 3.8+ (standard library only)
python3 examples/python/socq.py google-ad-library/ad payloads/ad.json

# cURL + jq
bash examples/curl/socq.sh google-ad-library/ad '{"url":"https://adstransparency.google.com/advertiser/AR01614014350098432001/creative/CR10449491775734153217"}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Google Ad Library Ad API

Retrieve one public Google Ads Transparency Center creative from its exact URL, returning advertiser identity, copy, media, delivery dates, regions, format, destination, and available impressions in a normalized record.

- **Request:** `POST https://api.socq.ai/v1/google-ad-library/ad`
- **publicId:** `google-ad-library/ad`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-08-11
- **Example body:** [`payloads/ad.json`](payloads/ad.json)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/google-ad-library/ad" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://adstransparency.google.com/advertiser/AR01614014350098432001/creative/CR10449491775734153217"
}'
```

Full docs: [https://docs.socq.ai/api-manual/google-ad-library/ad](https://docs.socq.ai/api-manual/google-ad-library/ad) · Landing page: [socq.ai/apis/google-ad-library/ad](https://socq.ai/apis/google-ad-library/ad?utm_source=github&utm_medium=readme&utm_campaign=socq-google-ad-library-api)

### Google Ad Library Advertiser Search API

Discover public Google Ads Transparency Center advertiser candidates by query and optional region, returning normalized identity, website, logo, region, and available ad-count fields.

- **Request:** `POST https://api.socq.ai/v1/google-ad-library/advertiser-search`
- **publicId:** `google-ad-library/advertiser-search`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-08-11
- **Example body:** [`payloads/advertiser-search.json`](payloads/advertiser-search.json)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `query` | string | Yes | Search query. |
| `region` | string | No | - |

```bash
curl -X POST "https://api.socq.ai/v1/google-ad-library/advertiser-search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "Nike",
  "region": "US"
}'
```

Full docs: [https://docs.socq.ai/api-manual/google-ad-library/advertiser-search](https://docs.socq.ai/api-manual/google-ad-library/advertiser-search) · Landing page: [socq.ai/apis/google-ad-library/advertiser-search](https://socq.ai/apis/google-ad-library/advertiser-search?utm_source=github&utm_medium=readme&utm_campaign=socq-google-ad-library-api)

### Google Ad Library Company Ads API

Collect public Google Ads Transparency Center creatives for a company domain or advertiser ID with optional topic, region, date, surface, format, and bounded result filters.

- **Request:** `POST https://api.socq.ai/v1/google-ad-library/company-ads`
- **publicId:** `google-ad-library/company-ads`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-08-11
- **Example body:** [`payloads/company-ads.json`](payloads/company-ads.json)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `advertiser_id` | string | One of* | - |
| `domain` | string | One of* | - |
| `end_date` | string | No | - |
| `format` | string | No | - |
| `platform` | string | No | - |
| `region` | string | No | - |
| `results_limit` | integer | No | (value 40 to 2000; default `40`) |
| `start_date` | string | No | - |
| `topic` | string | No | - |

*Required rule: send at least one of `advertiser_id`, `domain`.

```bash
curl -X POST "https://api.socq.ai/v1/google-ad-library/company-ads" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "domain": "lululemon.com",
  "results_limit": 40
}'
```

Full docs: [https://docs.socq.ai/api-manual/google-ad-library/company-ads](https://docs.socq.ai/api-manual/google-ad-library/company-ads) · Landing page: [socq.ai/apis/google-ad-library/company-ads](https://socq.ai/apis/google-ad-library/company-ads?utm_source=github&utm_medium=readme&utm_campaign=socq-google-ad-library-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Google Ad Library data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
