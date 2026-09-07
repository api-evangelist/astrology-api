---
name: astrology-api-western-natal
description: Build a Western tropical natal chart with AstrologyAPI — planets, house cusps, aspects, interpretation text, a rendered wheel image, and daily or weekly transits against that natal chart.
api: astrology-api
host: https://json.astrologyapi.com/v1
operations:
  - geo_details
  - timezone_with_dst
  - planets_tropical
  - house_cusps_tropical
  - western_horoscope
  - natal_chart_interpretation
  - general_ascendant_report_tropical
  - natal_wheel_chart
  - natal_transits_daily
  - natal_transits_weekly
  - tropical_transits_daily
generated: '2026-09-07'
method: generated
source: openapi/astrology-api-json-openapi.yml + https://astrologyapi.com/developers/v1/guides/western-birth-chart-api-integration
---

# Build a Western natal chart

All operations are `POST` to `https://json.astrologyapi.com/v1/<endpoint>` with
`day, month, year, hour, min, lat, lon, tzone`. Western endpoints are tropical and take
`house_type` rather than `ayanamsha`.

## House systems

`house_type` defaults to `placidus`. Documented alternatives: `koch`, `topocentric`, `poryphry`,
`equal_house`, `whole_sign`. It changes house placements, so it belongs in your cache key.

## Step 1 — birth moment

Same as the Vedic flow: `geo_details` to resolve the place, then `timezone_with_dst` for the offset
at that date. Do not assume the current offset.

## Step 2 — the chart

| Call | Returns |
|---|---|
| `planets/tropical` | Tropical planetary positions |
| `house_cusps/tropical` | House cusp degrees for the chosen `house_type` |
| `western_horoscope` | Consolidated chart data — planets, houses and aspects in one response |
| `natal_chart_interpretation` | Narrative interpretation of the chart |
| `general_ascendant_report/tropical` | Ascendant traits |
| `natal_wheel_chart` | A rendered wheel image — the API returns the artwork, you do not draw it |

`western_horoscope` is the efficient starting point: one call instead of three, and one credit
instead of three.

## Step 3 — transits

Two different things, easy to confuse:

- `tropical_transits/daily|weekly|monthly` — where the planets are now, independent of any person.
  Compute once and serve to every user. Cache by date.
- `natal_transits/daily|weekly` — transiting planets aspecting *this* natal chart, with aspect
  windows and exact timing. Per person, so cache per `(birth tuple, date)`.

Getting this split right is most of the cost control in a transit product.

## Caching

Natal responses never change — cache indefinitely on
`(day, month, year, hour, min, lat, lon, tzone, house_type)`. Daily tropical transits change once a
day and are identical for everyone; cache one copy per date, never one per user.

## Errors

Retry 5xx and network errors with exponential backoff; never retry 4xx. `401` returns
`{"status": false, "msg": "API authentication failed!"}`, and `401` is also what you get when the
endpoint sits outside your subscription plan's package rather than when the key itself is wrong —
check package coverage before assuming a bad credential.
