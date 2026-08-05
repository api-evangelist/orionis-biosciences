---
name: Read the Orionis Biosciences newsroom
description: Pull Orionis Biosciences news posts — partnership announcements, clinical milestones and leadership appointments — from the company's WordPress REST API, with correct pagination and HTML handling.
api: openapi/orionis-biosciences-content-openapi.yml
base_url: https://orionisbio.com/wp-json
operations:
  - getWpV2Posts
  - getWpV2PostsById
  - getWpV2Categories
generated: '2026-08-04'
method: generated
---

# Read the Orionis Biosciences newsroom

Orionis Biosciences runs no developer program. The newsroom at <https://orionisbio.com/news/> is
served by the site's WordPress REST API, which is anonymously readable. **No API key, token or
sign-up is required.** Do not send an `Authorization` header — writes are not available to you and
sending credentials will not unlock anything.

## 1. List posts

```
GET https://orionisbio.com/wp-json/wp/v2/posts?per_page=100&orderby=date&order=desc
```

`getWpV2Posts`. There were 22 published posts at last harvest, so one page at `per_page=100` covers
the whole newsroom. Trim the payload with `_fields` — the full record carries rendered HTML plus
Yoast SEO blocks and is large:

```
GET https://orionisbio.com/wp-json/wp/v2/posts?per_page=100&_fields=id,date,slug,link,title,excerpt,categories
```

## 2. Respect the pagination contract

- `per_page` is capped at **100**. Requesting more returns `400 rest_invalid_param`.
- Read `X-WP-Total` (item count) and `X-WP-TotalPages` from the response headers rather than
  guessing when to stop.
- `Link: …; rel="next"` is returned when more pages exist. Follow it; do not construct page numbers
  past `X-WP-TotalPages`.

## 3. Filter by date or category

```
GET .../wp/v2/posts?after=2026-01-01T00:00:00&orderby=date&order=desc
GET .../wp/v2/posts?categories=6
```

Resolve category ids first with `getWpV2Categories` (`GET .../wp/v2/categories?_fields=id,name,slug,count`).
The site publishes 5 terms; `latest-updates` is id 6. The `tags` collection is registered but empty
(0 terms) — do not filter on it.

## 4. Fetch one post

```
GET https://orionisbio.com/wp-json/wp/v2/posts/1470
```

`getWpV2PostsById`. `content.rendered` and `excerpt.rendered` are **HTML strings**, not plain text —
strip or render them; they contain entities (`&#8230;`), non-breaking spaces and trailing
"Learn more" anchor markup. `title.rendered` is HTML-escaped too.

## 5. Handle errors

The error envelope is **not** RFC 9457. It is `{ "code": "...", "message": "...", "data": { "status": n } }`.

- `400 rest_invalid_param` — a parameter failed the route schema (usually `per_page > 100`).
- `404 rest_post_invalid_id` — unknown or unpublished post id.
- `403` with an **HTML** body — the Sucuri firewall in front of the origin, not WordPress. Do not
  try to parse it as JSON. Back off and retry more slowly.

No rate-limit headers are published. Stay under a few requests per second and cache; the whole
newsroom is 22 records and changes a handful of times a year.

See `errors/orionis-biosciences-problem-types.yml` and
`conventions/orionis-biosciences-conventions.yml`.
