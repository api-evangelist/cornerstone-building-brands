---
name: Read the Cornerstone Building Brands news feed
description: Retrieve, filter and summarise Cornerstone Building Brands press releases and company news
  from the anonymously readable WordPress content API on www.cornerstonebuildingbrands.com, including the
  company-specific news_category and news_solution taxonomies.
api: openapi/cornerstone-building-brands-content-api-openapi.yml
operations: [listNews, getNews, listNewsCategory, getNewsCategory, listNewsSolution, getNewsSolution, getMedia]
---

# Read the Cornerstone Building Brands news feed

Cornerstone Building Brands publishes press releases as a WordPress custom post type called `news`. The
feed is anonymously readable — no key, no token, no sign-up. There were **91 news items** as of
2026-09-19.

Base URL: `https://www.cornerstonebuildingbrands.com/wp-json`

## 1. Authentication

None. Every operation below returned HTTP 200 anonymously on 2026-09-19. Do not send an Authorization
header; the OAuth server on this host protects the MCP endpoint, not these routes.

## 2. List news — `listNews`

```
GET /wp/v2/news?per_page=20&page=1&orderby=date&order=desc&_fields=id,date,modified,slug,link,title,news_category,news_solution
```

Read the count from the **`X-WP-Total`** header and the page count from **`X-WP-TotalPages`**; both are
exposed to browsers via `Access-Control-Expose-Headers`. Follow `rel="next"` in the RFC 8288 `Link` header
rather than incrementing `page` blindly.

`per_page` is capped at **100**. Asking for more returns `400 rest_invalid_param` with
`data.details.per_page.code = rest_out_of_bounds` — not a 429.

## 3. Narrow by date or text

```
GET /wp/v2/news?after=2026-01-01T00:00:00&search=metal%20roofing&per_page=50
```

`after`, `before`, `modified_after`, `modified_before`, `search`, `slug`, `include`, `exclude`, `order` and
`orderby` are all accepted — they come from the provider's own route index, not from guesswork.

## 4. Filter by taxonomy — `listNewsCategory` / `listNewsSolution`

```
GET /wp/v2/news_category?per_page=100&_fields=id,name,slug,count
GET /wp/v2/news?news_solution=<term-id>&per_page=50
```

`news_solution` maps a release to a product solution (siding, windows & doors, metal building products);
`news_category` is the editorial classification. Resolve the term id first, then filter — the filter
parameters take ids, not slugs.

## 5. Fetch one item — `getNews`

```
GET /wp/v2/news/{id}?_fields=id,link,title,content,date,modified,featured_media
```

`content.rendered` is HTML. A bad id returns `404 rest_post_invalid_id`.

## 6. Resolve the image — `getMedia`

```
GET /wp/v2/media/{featured_media}?_fields=id,source_url,alt_text,media_details
```

## Conventions that matter here

- **Cite the `link`, not the id.** Every item carries an absolute `link` to the public press release; that
  is the stable citation, and the integer id is an internal surrogate key.
- **Errors are not RFC 9457.** The envelope is `{"code","message","data":{"status"}}`; match on `code`.
  See `errors/cornerstone-building-brands-problem-types.yml`.
- **No rate-limit headers exist.** Nothing tells you the ceiling before you hit it, and the site sits
  behind Cloudflare and WP Engine. Keep concurrency low and honour `cache-control: max-age=600`.
- **No versioning or deprecation policy is published.** `wp/v2` is WordPress core's namespace, not a
  commitment by the company; a plugin upgrade can change this surface without notice.
