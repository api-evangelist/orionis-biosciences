---
name: Search Orionis Biosciences site content
description: Run keyword search across Orionis Biosciences news posts, pages and team profiles via the WordPress REST search endpoint, then dereference hits to full records.
api: openapi/orionis-biosciences-content-openapi.yml
base_url: https://orionisbio.com/wp-json
operations:
  - getWpV2Search
  - getWpV2PostsById
  - getWpV2PagesById
  - getWpV2OrionisTeamById
  - getWpV2Types
generated: '2026-08-04'
method: generated
---

# Search Orionis Biosciences site content

Use this when you need to answer a question about Orionis Biosciences from the company's own words —
a partnership, a platform, a pipeline program, a named executive — rather than from a third-party
summary.

## 1. Search

```
GET https://orionisbio.com/wp-json/wp/v2/search?search=molecular%20glue&per_page=20
```

`getWpV2Search`. Anonymous, no credentials. Each hit is deliberately lightweight:

```json
{ "id": 1470,
  "title": "Orionis Biosciences Announces Strategic Collaboration with Novartis…",
  "url": "https://orionisbio.com/2026/06/novartis-collaboration-2026/",
  "type": "post", "subtype": "post" }
```

Narrow the corpus with `subtype`, which accepts `post`, `page`, `orionis_team` or `any`:

```
GET .../wp/v2/search?search=draetta&subtype=orionis_team
GET .../wp/v2/search?search=allo-glue&subtype=page
```

`type` accepts `post`, `term` or `post-format`; leave it at the default `post` for content search.

## 2. Dereference the hit

`subtype` tells you which collection to read:

| `subtype` | operation | URL |
|---|---|---|
| `post` | `getWpV2PostsById` | `.../wp/v2/posts/{id}` |
| `page` | `getWpV2PagesById` | `.../wp/v2/pages/{id}` |
| `orionis_team` | `getWpV2OrionisTeamById` | `.../wp/v2/orionis_team/{id}` |

Or follow `_links.self[0].href` on the hit, which the API returns pre-resolved.

`getWpV2Types` (`GET .../wp/v2/types`) lists the post types registered on this host if you need to
confirm the set rather than assume it.

## 3. Know the limits of this search

- It matches **titles and content**, not metadata, and it does not rank by relevance in any
  documented way — treat result order as unspecified.
- The corpus is small: 22 posts, 16 pages, 15 team profiles. For broad questions it is cheaper to
  pull all three collections once and search locally than to issue many queries.
- There is no full-text highlighting, no faceting and no synonym handling.
- `per_page` caps at 100; totals come back in `X-WP-Total`.

## 4. Rules

- Reads are anonymous — do not send credentials.
- `content.rendered` and `title.rendered` are HTML; strip entities before quoting.
- Errors use `{ code, message, data.status }`, not RFC 9457. A `403` with an HTML body is the
  Sucuri firewall — back off rather than retry hard.
- Cite `link`/`url` from the record when quoting Orionis, so the claim traces to the company's page.

See `conventions/orionis-biosciences-conventions.yml` and
`errors/orionis-biosciences-problem-types.yml`.
