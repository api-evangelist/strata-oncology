---
name: Track Strata Oncology publications and press releases
description: >-
  Retrieve Strata Oncology's press releases and published research from the public website content API,
  filter them by date or publication type, and page through the full archive without scraping HTML.
api: openapi/strata-oncology-content-api-openapi.yml
base_url: https://strataoncology.com/wp-json
operations:
  - listPublications
  - getPublication
  - listTaxonomies
  - themePublications
---

# Track Strata Oncology publications and press releases

Strata Oncology publishes no developer program. The one anonymous, machine-readable way to read its
announcements is the WordPress REST content API behind `strataoncology.com`. Use it instead of scraping
`https://strataoncology.com/news-and-publications/`.

## Authentication

None. Every operation below returns 200 with no credential. Do not send an `Authorization` header.

## Steps

1. **List the most recent publications.** Call `listPublications` against
   `GET https://strataoncology.com/wp-json/publications`. Set `per_page` (max 100, default 10) and
   `orderby=date`, `order=desc` for newest first.

2. **Read the paging signals from the headers, not the body.** The response body is a bare JSON array.
   Total item count is in `X-WP-Total`, total page count is in `X-WP-TotalPages`, and the next page is in the
   RFC 8288 `Link` header as `rel="next"`. Walk pages with the `page` parameter until `page` exceeds
   `X-WP-TotalPages`.

3. **Filter incrementally.** For a delta since your last run, pass `modified_after` (or `after` for
   publication date) as an ISO 8601 datetime rather than re-reading the archive. `search` does full-text
   matching; `slug` fetches an exact item.

4. **Narrow by publication type.** `listPublications` accepts `publication_type` (and
   `publication_type_exclude`). Discover the valid terms with `listTaxonomies` before hard-coding any value —
   the site's own "Press releases" section is one such term.

5. **Fetch one item in full.** Call `getPublication` with the numeric `id` from the list response. Titles
   and bodies come back as rendered HTML under `title.rendered`, `content.rendered` and `excerpt.rendered`;
   strip or sanitize before display. `link` is the canonical public URL. `featured_media` is a media id you
   can resolve with `getMediaitem`.

6. **Only if you need the site's own display grouping**, call `themePublications`
   (`GET /wp-json/api/publications`). It returns a different, presentation-shaped envelope —
   `{success, data:{layout, sections:[{name, btnParam, currentPage, totalPosts, totalPages, postsPerPage,
   posts[]}]}}` — with pre-paged sections and no query parameters. Prefer `listPublications` for anything
   programmatic.

## Error handling

Errors are **not** RFC 9457. They come back as `application/json` shaped `{code, message, data:{status}}`:

- `rest_no_route` (404) — wrong namespace or path. Re-check against `https://strataoncology.com/wp-json/`.
- `rest_post_invalid_id` (404) — the id does not exist or is not anonymously readable.
- `rest_invalid_param` (400) — a parameter failed validation, most often `per_page` above 100 or an
  `orderby` value outside the allowed enum.
- `rest_user_cannot_view` (401) — you have strayed onto a privileged route (e.g. `/wp/v2/users`). It is
  outside this public surface; do not retry with guessed credentials.

## Rules

- There is no idempotency contract and no documented rate limit. Treat the surface as read-only, request it
  politely (serial paging, not parallel fan-out), and cache by `modified`.
- Never write to this API. Write methods exist on these collections but require WordPress authentication that
  Strata Oncology does not publish; attempting them is out of scope.
- This is the **marketing website's** content API. It carries no patient, clinical, laboratory or test-result
  data, and there is no public API for those. Do not represent it as a clinical interface.
