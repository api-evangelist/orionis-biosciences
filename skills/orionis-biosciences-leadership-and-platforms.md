---
name: Retrieve Orionis Biosciences leadership profiles and platform pages
description: Pull the orionis_team leadership/scientific-team profiles and the Allo-Glue, A-Kine and MCE platform pages (with portraits and images) from the Orionis Biosciences WordPress REST API.
api: openapi/orionis-biosciences-content-openapi.yml
base_url: https://orionisbio.com/wp-json
operations:
  - getWpV2OrionisTeam
  - getWpV2OrionisTeamById
  - getWpV2Pages
  - getWpV2PagesById
  - getWpV2Media
  - getWpV2MediaById
generated: '2026-08-04'
method: generated
---

# Retrieve Orionis Biosciences leadership profiles and platform pages

Two things live behind this API that are not obvious from the website: the leadership and scientific
team is a **custom post type** called `orionis_team`, and the platform pages are ordinary WordPress
pages arranged in a parent/child hierarchy. Both are anonymously readable — no credentials.

## 1. List the team

```
GET https://orionisbio.com/wp-json/wp/v2/orionis_team?per_page=100&_fields=id,slug,link,title,excerpt,featured_media
```

`getWpV2OrionisTeam`. 15 profiles at last harvest, covering senior management, scientific leadership
and the board. `title.rendered` is the person's name with credentials (e.g. "Giulio Draetta, MD,
Ph.D") and may contain non-breaking spaces and stray Unicode object-replacement characters — normalize
before matching on names.

Fetch one profile with `getWpV2OrionisTeamById`; `content.rendered` is the biography as HTML.

## 2. Resolve portraits

Each profile carries `featured_media`, an integer id into the media library (0 means no image).

```
GET https://orionisbio.com/wp-json/wp/v2/media/1446?_fields=id,source_url,alt_text,media_details
```

`getWpV2MediaById`. `source_url` is the full-size file; `media_details.sizes` holds the responsive
variants.

**Faster:** add `_embed` to the collection request and WordPress inlines the media record into
`_embedded['wp:featuredmedia']`, resolving all 15 portraits in a single call:

```
GET https://orionisbio.com/wp-json/wp/v2/orionis_team?per_page=100&_embed
```

## 3. Walk the platform pages

```
GET https://orionisbio.com/wp-json/wp/v2/pages?per_page=100&_fields=id,slug,link,title,parent,menu_order
```

`getWpV2Pages`. 16 pages. The hierarchy is carried in `parent`:

- page **460** `platforms` is the parent of `mce-platform` (MCE Cell Engager), `allo-glue-platform`
  and `a-kines` (A-Kine)
- page **462** `about-us` is the parent of `executive-team` and `board-of-directors`
- `pipeline`, `news`, `contact` and the legal pages sit at `parent: 0`

Filter children directly with `?parent=460`. Retrieve the page body with `getWpV2PagesById`;
`content.rendered` is HTML.

Note that `new-homepage-test` (id 1008) is a staging artifact published on the live site — exclude it
when presenting the page set as the company's public pages.

## 4. Rules

- Do not send `Authorization`. Reads are anonymous; writes are not available to the public.
- `per_page` caps at 100; read `X-WP-Total` for the count.
- Rendered fields are HTML strings, never plain text.
- A `403` with an HTML body is the site firewall, not a WordPress error — do not JSON-parse it.

See `data-model/orionis-biosciences-data-model.yml` for the full entity graph and
`conventions/orionis-biosciences-conventions.yml` for `_fields` / `_embed` semantics.
