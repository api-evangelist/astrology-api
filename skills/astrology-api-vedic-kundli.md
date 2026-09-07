---
name: astrology-api-vedic-kundli
description: Build a complete Vedic (Jyotish) birth chart with AstrologyAPI — resolve a birth place to coordinates, get the correct historical timezone offset, then compute birth details, planetary positions, divisional charts, the current Vimshottari dasha and planetary strength.
api: astrology-api
host: https://json.astrologyapi.com/v1
operations:
  - geo_details
  - timezone_with_dst
  - birth_details
  - astro_details
  - planets
  - horo_chart_chartId
  - current_vdasha
  - shadbala
  - bhavabala
generated: '2026-09-07'
method: generated
source: openapi/astrology-api-json-openapi.yml + https://astrologyapi.com/developers/v1/guides/kundli-api-integration
---

# Build a Vedic Kundli

Every Vedic operation is a `POST` to `https://json.astrologyapi.com/v1/<endpoint>`. Get the birth
moment right first — everything downstream is a pure function of it, and a wrong `tzone` produces a
chart that is confidently wrong rather than an error.

## Authenticate

Two credentials, and the choice changes the body encoding:

- **HTTP Basic** — User ID as username, API key as password, body as `application/x-www-form-urlencoded`.
- **Access token** — `x-astrologyapi-key: <token>` header, body as `application/json`.

Never put either in client-side code. `json.astrologyapi.com` sends no CORS headers, so a browser
call fails by design — route through your own server.

## Step 1 — resolve the birth place

`geo_details` with `place` (and optionally `maxRows`) returns candidate locations with coordinates.
Let the user pick; do not assume the first result.

## Step 2 — get the timezone offset for that moment

`timezone_with_dst` with `latitude`, `longitude` and `date`. Do **not** hardcode an offset and do not
reuse the modern one — DST rules and standard offsets have changed, and a birth in 1985 may not use
today's value. Note the field names change here: this endpoint wants `latitude`/`longitude`, while
the chart endpoints want `lat`/`lon`.

## Step 3 — compute the chart

With `day, month, year, hour, min, lat, lon, tzone` (and `ayanamsha`, default `LAHIRI`):

| Call | Returns |
|---|---|
| `birth_details` | Normalised birth data with the resolved ayanamsha |
| `astro_details` | Ascendant, sign lords, varna, tatva, yoga and the summary attributes |
| `planets` | Per planet: degree, sign, sign lord, nakshatra, nakshatra lord, pada, house, retrograde flag, awastha |
| `horo_chart/{chartId}` | A divisional chart — `D1` for the rasi chart, `D9` for navamsa |
| `current_vdasha` | The running Vimshottari mahadasha/antardasha |
| `shadbala` | Per-planet strength with `is_strong` and `strength_percent_of_minimum` |
| `bhavabala` | Per-house strength with `ranked_house_ids_desc` |

Use `shadbala` and `bhavabala` to decide what to *lead with*. Placements alone do not rank; these
endpoints return the ranking as numbers, so weight your output instead of treating every factor
equally.

## Cache aggressively

A natal chart never changes. Cache every response in step 3 indefinitely, keyed on the tuple
`(day, month, year, hour, min, lat, lon, tzone, ayanamsha)`. The provider documents this and it is
the single largest cost lever on the platform — calls are billed per request from a credit wallet.

Include `ayanamsha` in the cache key. Changing it changes the answer.

## Errors and retries

- Non-200 is a failure. Read the code before acting.
- Retry on **5xx and network errors only**, exponential backoff (1s/2s/4s), request timeout ~10s.
- **Never retry a 4xx.** A 401 with a wrong key fails identically forever; a 400 with a malformed
  date stays malformed. Retrying only spends credits.
- `401` → `{"status": false, "msg": "API authentication failed!"}` — bad credential, or an endpoint
  outside your plan's package.
- `404` → `{"status": false, "statusCode": 404, "error_msg": "..."}` — wrong endpoint name, wrong
  host, or a non-POST method.

## Things this API does not do

There is **no idempotency key**, so a retry after a successful-but-timed-out call bills twice. There
are **no rate-limit headers** and no published limit — you cannot detect throttling, and there is no
documented signal for running out of credits either, so monitor the wallet out of band.
