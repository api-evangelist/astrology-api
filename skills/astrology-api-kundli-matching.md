---
name: astrology-api-kundli-matching
description: Score Vedic compatibility between two birth charts with AstrologyAPI — Ashtakoot and Dashakoot guna milan, Manglik/dosha analysis, obstruction detection and a narrative matchmaking report, for matrimonial and dating products.
api: astrology-api
host: https://json.astrologyapi.com/v1
operations:
  - match_birth_details
  - match_astro_details
  - match_planet_details
  - match_ashtakoot_points
  - match_dashakoot_points
  - match_percentage
  - match_manglik_report
  - match_obstructions
  - match_making_report
  - match_making_detailed_report
generated: '2026-09-07'
method: generated
source: openapi/astrology-api-json-openapi.yml + https://astrologyapi.com/developers/v1/guides/kundli-matching
---

# Kundli matching (guna milan)

Every matchmaking operation takes **two** birth moments in one request, prefixed `m_` and `f_`:

```
m_day m_month m_year m_hour m_min m_lat m_lon m_tzone
f_day f_month f_year f_hour f_min f_lat f_lon f_tzone
```

Resolve each subject's place and offset with `geo_details` and `timezone_with_dst` first, exactly as
in the single-chart flows.

## Pick the scoring system

| Call | Use it for |
|---|---|
| `match_ashtakoot_points` | Ashtakoot guna milan — the 36-point North Indian system |
| `match_dashakoot_points` | Dashakoot — the South Indian system |
| `match_percentage` | A single headline compatibility number |
| `match_making_report` | Narrative report |
| `match_making_detailed_report` | Long-form report |
| `match_manglik_report` | Manglik (Mangal dosha) analysis for the pair |
| `match_obstructions` | Obstructions/blockers in the match |
| `match_birth_details` / `match_astro_details` / `match_planet_details` | The underlying chart data for both subjects |

Choose one scoring system and stay with it. Ashtakoot and Dashakoot are different traditions on
different scales — showing both side by side to an end user reads as a contradiction, not as rigour.

## Handle a missing birth time

Matrimonial and dating profiles frequently lack one. Birth time drives the ascendant, the houses and
the moon's nakshatra pada, so it drives most of the guna score. If you do not have it, say so in the
product rather than defaulting to noon and presenting the result as exact. The provider publishes
guidance on this at `/developers/v1/guides/unknown-birth-time`.

## Present dosha results responsibly

Manglik and obstruction results carry real social weight in matrimonial contexts. Return the
provider's own text rather than paraphrasing it into a verdict, and avoid rendering a dosha as a
pass/fail gate on a profile.

## Caching

A pair's score never changes. Cache on the concatenated tuple of both birth moments plus the
ayanamsha. In a matrimonial product the same two profiles are compared repeatedly, so this is a large
saving — every call is billed per request.

## Errors

Retry 5xx and network errors only, with exponential backoff. Never retry a 4xx. There is no
idempotency key, so a retried call after a timeout bills a second time.
