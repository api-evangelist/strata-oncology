---
name: Assemble a Strata Oncology company profile
description: >-
  Build a current, sourced profile of Strata Oncology — leadership, product and resource pages, and site
  structure — from the public website content API rather than from cached or scraped HTML.
api: openapi/strata-oncology-content-api-openapi.yml
base_url: https://strataoncology.com/wp-json
operations:
  - listTeam
  - getTeammember
  - listPages
  - getPage
  - listResources
  - listTestimonials
  - search
  - listTypes
---

# Assemble a Strata Oncology company profile

Everything below is anonymous and read-only on `https://strataoncology.com/wp-json`. No credential is
required or accepted.

## Steps

1. **Confirm the content types before assuming any collection exists.** Call `listTypes`
   (`GET /wp-json/types`). Strata Oncology registers custom types beyond WordPress core — `publications`,
   `resources`, `team`, `testimonials`. If a type is missing on a later run, the site changed; do not guess a
   path.

2. **Pull leadership.** Call `listTeam` with `per_page=100` to get every profile in one page, then
   `getTeammember` for any individual you need in full. Each profile's `link` resolves to
   `https://strataoncology.com/team/<slug>/`, and `content.rendered` carries the bio as HTML.

3. **Pull the product and company pages.** Call `listPages` with `per_page=100`. Twelve pages were present
   at capture. Use `slug` to target a known page directly — for example `about`, `pdl1ihc`, `create-account`,
   `privacy-practices`, `no-surprises-act`. Follow `parent` to reconstruct the page hierarchy; see
   `data-model/strata-oncology-data-model.yml`.

4. **Pull patient and provider resources.** Call `listResources`. These are the documents behind
   `https://strataoncology.com/resources/…`, including patient billing and financial assistance.

5. **Add testimonials if the profile needs voice-of-customer.** Call `listTestimonials`.

6. **Search across everything** with `search` (`GET /wp-json/search`), passing `search`, and narrowing with
   `type` and `subtype` when you want results from one collection only.

## Rules

- Read the totals from `X-WP-Total` / `X-WP-TotalPages` and the next page from the `Link` header; the body is
  a bare array with no envelope. `per_page` caps at 100.
- All text fields are **rendered HTML**, not plain text. Sanitize before rendering and never execute embedded
  markup.
- Errors follow the WordPress envelope `{code, message, data:{status}}` — see
  `errors/strata-oncology-problem-types.yml`. A 401 `rest_user_cannot_view` means you have left the public
  surface; stop rather than retrying.
- Cite `link` (the canonical public URL) and `modified` for every fact you extract, so the profile is
  traceable and freshness is visible.
- Strata Oncology publishes no API changelog or status page. If the shape of a response changes, re-read
  `https://strataoncology.com/wp-json/` — that discovery document is the only source of truth for this
  surface.
