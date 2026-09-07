---
name: astrology-api-daily-horoscope
description: Serve daily, weekly and monthly sun-sign horoscope content from AstrologyAPI at scale — generate once per sign per day, cache it, and serve every reader from the cache instead of calling the API per visitor.
api: astrology-api
host: https://json.astrologyapi.com/v1
operations:
  - sun_sign_prediction_daily_zodiacName
  - sun_sign_prediction_next_zodiacName
  - sun_sign_prediction_previous_zodiacName
  - sun_sign_consolidated_daily_zodiacName
  - horoscope_prediction_monthly_zodiacName
generated: '2026-09-07'
method: generated
source: openapi/astrology-api-json-openapi.yml + https://astrologyapi.com/blog/automate-daily-horoscope-content
---

# Daily horoscope content feeds

These endpoints are shaped differently from the rest of the platform. They take **no birth data** —
the sign is a path parameter and the only body field is `timezone`:

```
POST https://json.astrologyapi.com/v1/sun_sign_prediction/daily/{zodiacName}
Content-Type: application/json
x-astrologyapi-key: <ACCESS_TOKEN>

{"timezone": 5.5}
```

| Call | Returns |
|---|---|
| `sun_sign_prediction/daily/{zodiacName}` | Today's prediction for one sign |
| `sun_sign_prediction/next/{zodiacName}` | Tomorrow — pre-warm the cache with this |
| `sun_sign_prediction/previous/{zodiacName}` | Yesterday |
| `sun_sign_consolidated/daily/{zodiacName}` | Consolidated daily insight — love, career, health, luck |
| `horoscope_prediction/monthly/{zodiacName}` | Monthly prediction |

## The one thing that matters: 12 calls a day, not 12 per visitor

The content is generated **once per sign per day**. It is identical for every reader of that sign.

Cache server-side keyed on `(sign, date, timezone)` and serve every request from the cache. A daily
horoscope page for all twelve signs is **12 API calls per day** — roughly 360 a month. Calling per
visitor instead turns the same product into one call per pageview, and calls bill from a credit
wallet at roughly ₹20 per 1,000.

Pre-warm with `sun_sign_prediction/next/{zodiacName}` on a scheduled job before midnight so the
cache is never cold at the daily rollover.

## Timezone

`timezone` is a numeric offset (e.g. `5.5`). It determines which day's content you get, so it is part
of the cache key. If you serve readers in several regions, cache per offset rather than assuming one.

## Multi-language

Horoscope feeds are advertised in 22+ languages. Where an endpoint accepts `language`, add it to the
cache key too — otherwise you will serve one language's copy to everyone.

## Errors

Retry 5xx and network errors with exponential backoff; never retry a 4xx. On a cache miss combined
with an API failure, serve the previous day's cached copy rather than an empty page — the content is
editorial, and stale beats blank.

## SEO note

This content is identical for every site using the API. Treat it as a retention and engagement
feature rather than as original content that will rank.
