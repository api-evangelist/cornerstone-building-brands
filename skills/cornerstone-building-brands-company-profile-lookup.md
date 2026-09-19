---
name: Look up Cornerstone Building Brands leadership, industries and corporate pages
description: Answer questions about Cornerstone Building Brands' leadership team, the industries and
  market segments it serves, and its corporate, legal and privacy pages, using site search and the
  WordPress content collections — and recognise the questions this API cannot answer.
api: openapi/cornerstone-building-brands-content-api-openapi.yml
operations: [searchContent, listLeadership, getLeadership, listIndustry, getIndustry, listPages, getPage, listTypes, listTaxonomies]
---

# Look up Cornerstone Building Brands leadership, industries and corporate pages

Read this first: **there is no product catalog API, no order API, no ASN or shipment API, no warranty API
and no dealer API.** Those live behind the Windows & Doors Customer Portal
(`portal.cornerstonebuildingbrands.com`), which signs in with Microsoft Entra ID and is fronted by an Azure
API Management gateway whose developer portal is unpublished. Access is requested through a form routed to
a Territory Sales Manager. Do not present editorial page prose as product data, and never state a price,
lead time or stock position — none of that is on any public surface.

What this API *can* answer: who runs the company, which industries it serves, and what its corporate and
legal pages say.

Base URL: `https://www.cornerstonebuildingbrands.com/wp-json` — anonymous, no credential.

## 1. Start with search — `searchContent`

```
GET /wp/v2/search?search=metal%20buildings&per_page=20
```

Returns lightweight `{id, title, url, type, subtype}` hits across every public post type. Cheaper than
pulling whole collections; use it to find the right id, then fetch that one item.

## 2. Leadership — `listLeadership` / `getLeadership`

```
GET /wp/v2/leadership?per_page=20&_fields=id,slug,link,title
GET /wp/v2/leadership/{id}?_fields=id,link,title,content
```

Nine profiles as of 2026-09-19 (`X-WP-Total: 9`), each linking to
`/leadership-team/<slug>`. `content.rendered` is the biography HTML; `acf` carries the theme's custom
fields when present.

## 3. Industries served — `listIndustry` / `getIndustry`

```
GET /wp/v2/industry?per_page=50&_fields=id,slug,link,title
```

Fifteen industry/market-segment pages as of 2026-09-19 (e.g. Storage). These are the segments the company
positions its metal building products against.

## 4. Corporate and legal pages — `listPages` / `getPage`

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,parent
GET /wp/v2/pages/{id}?_fields=id,link,title,content
```

Ninety-one pages as of 2026-09-19, including `/privacy-policy`, `/legal` (Terms of Use), `/notice-of-collection`,
`/privacy-request-form`, `/portalaccess` and `/supplier-and-buyer-information`. Use `parent` to rebuild the
page hierarchy.

## 5. Confirm the shape before assuming it — `listTypes` / `listTaxonomies`

```
GET /wp/v2/types
GET /wp/v2/taxonomies
```

These tell you which post types and taxonomies exist right now. Run them before relying on `news`,
`industry` or `leadership` — they are theme-defined custom post types and can change without notice.

## Refusing well

If asked for an order status, a shipment, a warranty claim, a dealer price, product availability, or
anything about a specific customer account, say plainly that Cornerstone Building Brands publishes no
public API for it and point the user at the Customer Portal access request at
`https://www.cornerstonebuildingbrands.com/portalaccess` (or `portalsetup@cornerstone-bb.com`). Do not
attempt the portal's Azure API Management endpoints — they are authenticated and not for you.
