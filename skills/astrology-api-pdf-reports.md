---
name: astrology-api-pdf-reports
description: Generate a white-labeled astrology PDF report with AstrologyAPI — Kundli, matchmaking, gemstone, numerology, Varshaphal, natal, solar return, synastry or life forecast — and handle the branding fields, the field-name inconsistencies and the fact that generation is billed and irreversible.
api: astrology-api
host: https://pdf.astrologyapi.com/v1
operations:
  - mini_horoscope_pdf
  - basic_horoscope_pdf
  - pro_horoscope_pdf
  - basic_gemstone_report_pdf
  - pro_numerology_report
  - varshphal_horoscope_pdf
  - match_making_pdf
  - natal_horoscope_report_tropical
  - solar_return_report_tropical
  - life_forecast_report_tropical
  - synastry_couple_report_tropical
  - star_sign_compatibility_report
generated: '2026-09-07'
method: generated
source: openapi/astrology-api-pdf-openapi.yml + https://astrologyapi.com/developers/v1/pdf
---

# Generate a PDF report

PDF reports are served from a **different host**: `https://pdf.astrologyapi.com/v1/<endpoint>`, not
the JSON host. Each call returns a hosted `pdf_url`.

## Read this before your first call

**Generation is billed and cannot be undone.** Each report costs credits — from ₹1.50 for a Mini
Horoscope up to ₹100 for Natal, Solar Return, Life Forecast, Synastry and Star Sign Compatibility.
There is no cancel, void or delete operation, the provider's refund policy states all purchases are
final and non-refundable, and there is **no idempotency key**. A retry after a timeout generates and
bills a second report.

So: **never put a PDF call inside a generic retry wrapper.** If a PDF request times out, check
whether the report was produced before you re-send. This is the one place on this platform where a
naive retry costs real money.

## Field names are not consistent across these endpoints

This is the most common integration bug here. Both spellings are in use *within the same PDF
collection*:

- `mini_horoscope_pdf`, `basic_horoscope_pdf`, `pro_horoscope_pdf`, `basic_gemstone_report_pdf`,
  `pro_numerology_report` → `hour`, `min`, `lat`, `lon`, `tzone`
- `natal_horoscope_report/tropical`, `solar_return_report/tropical`,
  `life_forecast_report/tropical`, `star_sign_compatibility_report`,
  `synastry_couple_report/tropical` → `hour`, `minute`, `latitude`, `longitude`, `timezone`

Subject prefixes differ too: `match_making_pdf` uses `m_`/`f_`, while
`synastry_couple_report/tropical` uses `p_`. Check the operation's request schema in
`openapi/astrology-api-pdf-openapi.yml` before building the payload — a misnamed field is simply
absent and fails validation rather than mapping.

## Branding

Every PDF operation accepts the same eleven white-label fields, supplied **per request** — there is
no account-level branding resource:

`logo_url`, `domain_url`, `footer_link`, `company_name`, `company_info`, `company_email`,
`company_landline`, `company_mobile`, plus `name`, `gender` and `language`.

Keep these in one config object in your own code and spread it into every request.

## Delivery

The response carries `pdf_url`. The provider also advertises webhook and email delivery on its
pricing pages, but publishes no webhook documentation, payload schema, signature scheme or
registration endpoint — so build against `pdf_url` and treat webhook delivery as a sales conversation.

Retention, expiry and access control on `pdf_url` are undocumented. Since these reports contain a
named person's birth data, download and store the file yourself rather than handing the provider's
URL to an end user.

## Trial credits

The provider's pricing page states trial wallet credits cannot be used for PDF generation, while its
FAQ on the same page says free credits work across all endpoints including PDF. Assume you need a
funded wallet.
