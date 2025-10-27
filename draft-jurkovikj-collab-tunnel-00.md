---
title: "The Collaboration Tunnel Protocol"
abbrev: "Collab-Tunnel"
docname: draft-jurkovikj-collab-tunnel-00
category: info
ipr: trust200902
submissiontype: IETF

author:
 - name: Antun Jurkovikj
   email: antunjurkovic@gmail.com
   country: North Macedonia

normative:
  RFC9110:
    title: HTTP Semantics
    author:
      - name: R. Fielding
      - name: M. Nottingham
      - name: J. Reschke
    date: 2022-06
    target: https://www.rfc-editor.org/rfc/rfc9110

  RFC9111:
    title: HTTP Caching
    author:
      - name: R. Fielding
      - name: M. Nottingham
      - name: J. Reschke
    date: 2022-06
    target: https://www.rfc-editor.org/rfc/rfc9111

  RFC8288:
    title: Web Linking
    author:
      - name: M. Nottingham
    date: 2017-10
    target: https://www.rfc-editor.org/rfc/rfc8288

  RFC6596:
    title: The Canonical Link Relation
    author:
      - name: M. Ohye
      - name: J. Kupke
    date: 2012-04
    target: https://www.rfc-editor.org/rfc/rfc6596

  RFC2119:
    title: Key words for use in RFCs to Indicate Requirement Levels
    author:
      - name: S. Bradner
    date: 1997-03
    target: https://www.rfc-editor.org/rfc/rfc2119

  RFC8174:
    title: Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words
    author:
      - name: B. Leiba
    date: 2017-05
    target: https://www.rfc-editor.org/rfc/rfc8174

informative:
  RFC9530:
    title: Digest Fields
    author:
      - name: L. Pardue
      - name: R. Polli
    date: 2024-02
    target: https://www.rfc-editor.org/rfc/rfc9530

  RFC9421:
    title: HTTP Message Signatures
    author:
      - name: A. Backman
      - name: J. Richer
      - name: M. Sporny
    date: 2024-02
    target: https://www.rfc-editor.org/rfc/rfc9421

  RFC6973:
    title: Privacy Considerations for Internet Protocols
    author:
      - name: A. Cooper
      - name: H. Tschofenig
      - name: B. Aboba
      - name: J. Peterson
      - name: J. Morris
      - name: M. Hansen
      - name: R. Smith
    date: 2013-07
    target: https://www.rfc-editor.org/rfc/rfc6973

  ResourceSync:
    title: ResourceSync Framework Specification
    author:
      - org: Open Archives Initiative
    date: 2017-05
    target: https://www.openarchives.org/rs/1.1/resourcesync

  AMP:
    title: Accelerated Mobile Pages (AMP) HTML Specification
    target: https://amp.dev/documentation/guides-and-tutorials/learn/spec/amphtml/

  XMLSitemaps:
    title: Sitemap XML format
    target: https://www.sitemaps.org/protocol.html
    author:
      - org: sitemaps.org

--- abstract

This document specifies the Collaboration Tunnel Protocol, a method for efficient, verifiable content delivery between web publishers and automated agents. The protocol achieves 60-90% bandwidth reduction through bidirectional URL discovery, template-invariant content fingerprinting, sitemap-first verification, and strict conditional request discipline.

--- middle

# Introduction

Automated agents (AI crawlers, search engines, content aggregators) increasingly consume web content at scale. Traditional HTML delivery designed for human browsers imposes unnecessary overhead: presentational boilerplate (navigation, footers, advertisements), large CSS/JavaScript bundles, and redundant fetches of unchanged content.

Existing approaches address portions of this problem:

- XML Sitemaps {{XMLSitemaps}} provide discovery but lack content fingerprints
- AMP {{AMP}} reduces HTML overhead but lacks synchronized hashing
- ResourceSync {{ResourceSync}} provides digest-based synchronization but lacks endpoint-level validator discipline

The Collaboration Tunnel Protocol (TCT) integrates these concepts into a cohesive system optimized for machine consumption while preserving human-readable canonical URLs for SEO and web compatibility.

## Problem Statement

Current AI crawler behavior (2025) demonstrates:

1. **Bandwidth Waste**: Fetching full HTML documents when only core content is needed
2. **Token Overhead**: Processing boilerplate (navigation, footers) consumes 86% of tokens
3. **Redundant Fetches**: No efficient skip mechanism when content unchanged
4. **Lack of Verification**: No cryptographic proof of content delivery

Measured impact (from live deployments):

- HTML-only retrieval: 103 KB average (13,900 tokens)
- TCT JSON delivery: 17.7 KB average (1,960 tokens)
- **Savings: 83% bandwidth, 86% tokens**

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

**C-URL (Canonical URL)**: The human-readable URL of a web resource, typically serving HTML.

**M-URL (Machine URL)**: A deterministically mapped endpoint serving machine-readable structured content for the same resource.

**Template-Invariant Fingerprint**: A cryptographic hash computed from normalized core content, stable across presentation changes.

**Core Content**: The primary informational content of a resource, excluding presentational boilerplate.

# Protocol Overview

The Collaboration Tunnel Protocol consists of four coordinated mechanisms:

1. **Bidirectional Discovery**: Explicit C-URL <-> M-URL handshake preventing SEO conflicts
2. **Template-Invariant Fingerprinting**: Normalized content hashing for stable cache validation
3. **Conditional Request Discipline**: Strict If-None-Match precedence and 304 responses
4. **Sitemap-First Verification**: Zero-fetch skip logic when content unchanged

## Architecture

~~~
+------------------------+          +----------------------+
|  Publisher (Origin)    |          |  Automated Agent     |
+------------------------+          +----------------------+
|                        |          |                      |
|  C-URL (HTML)          |<---------|  1. Fetch Sitemap    |
|  +- <link rel=         |          |                      |
|  |  "alternate"        |          |  2. Compare Hashes   |
|  |  type="application/ |          |     (Zero-Fetch)     |
|  |       json"         |          |                      |
|  |  href="/llm/">      |          |  3. If Changed:      |
|                        |          |     GET /llm/        |
|  M-URL (/llm/)         |<---------|     If-None-Match    |
|  +- Link: canonical    |          |                      |
|  +- ETag: sha256-...   |          |                      |
|  +- Content-Type: JSON |--------->|  4. 304 or 200+JSON  |
|                        |          |                      |
|  /llm-sitemap.json     |          |  5. Cache ETag       |
|  +- {cUrl, mUrl,       |<---------|                      |
|  |   contentHash}      |          |                      |
+------------------------+          +----------------------+
~~~

# Protocol Requirements

This section defines the normative requirements for TCT compliance.

## MUST Requirements

Implementations MUST:

1. **Bidirectional Discovery**
   - C-URL HTML MUST include `<link rel="alternate" type="application/json">` pointing to M-URL
   - M-URL response MUST include `Link: <c-url>; rel="canonical"` HTTP header

2. **Validators**
   - M-URL response MUST include `ETag` header
   - M-URL response SHOULD include `Last-Modified` header when available
   - Sitemap MUST include `contentHash` field for each URL

3. **Conditional Requests**
   - Server MUST honor `If-None-Match` header
   - Server MUST return `304 Not Modified` when ETag matches
   - Server MUST give `If-None-Match` precedence over `If-Modified-Since` ({{RFC9110, Section 13.1.2}})

4. **304 Response**
   - Response MUST NOT include message body
   - Response MUST include `ETag` header
   - Response SHOULD include `Cache-Control` header

5. **HEAD Support**
   - Servers SHOULD support HEAD requests for all M-URLs and sitemaps
   - HEAD request MUST return same headers as GET
   - HEAD request MUST NOT include message body

6. **Sitemap Parity**
   - Sitemap `contentHash` value MUST equal M-URL `ETag` value (excluding quotes and `W/` prefix)
   - Example: M-URL ETag `W/"sha256-abc"` -> Sitemap contentHash `sha256-abc`

7. **Canonical Verification (Agents)**
   - Agents MUST verify that the M-URL response includes `Link: <c-url>; rel="canonical"` and that the canonical URL matches the expected C-URL before processing
   - If the canonical link is missing or mismatched, agents SHOULD treat the endpoint as non-compliant and skip ingestion
   - If HTML `<link rel="alternate">` discovery and the M-URL’s self-declared canonical conflict, agents SHOULD prefer the M-URL’s self-declared canonical and flag for operator review

## SHOULD Recommendations

Implementations SHOULD:

1. **Cache-Control**
   - Use: `max-age=0, must-revalidate, stale-while-revalidate=60, stale-if-error=86400`
   - Rationale: Enables revalidation with graceful stale serving

2. **Vary Header**
   - Include: `Vary: Accept-Encoding`
   - Rationale: Content varies by compression (gzip, br)

3. **Weak ETags**
   - Prefer weak ETags (`W/"sha256-..."`) for semantic fingerprints
   - Rationale: Template-invariant fingerprint is semantic, not byte-for-byte
   - Note: Strong ETags acceptable if computed per representation variant

4. **Content Pages Only**
   - Return 404 for M-URL requests to archive pages
   - Include only content pages in sitemap

## MAY Extensions

Implementations MAY:

1. **Policy Descriptor**
   - Provide machine-readable policy at `/llm-policy.json`
   - Link policy via `Link: </llm-policy.json>; rel="describedby"; type="application/json"`

2. **Additional Integrity**
   - Use `Content-Digest` header (RFC 9530)
   - Use HTTP Message Signatures (RFC 9421)

3. **Additional Formats**
   - Provide PDF JSON alternates
   - Provide receipt/proof systems

# Bidirectional Discovery

Note: Path names in examples are non-normative. This specification does not require any specific URL paths. Examples such as "/llm/" and "/tct/" are illustrative. Servers MAY choose alternative slugs or publish mappings. Agents MUST NOT assume a fixed path and SHOULD discover M-URLs via HTML `rel="alternate"`, HTTP `Link` headers, or the JSON sitemap.

## C-URL to M-URL Mapping

The C-URL MUST include an HTML `<link>` element in the document `<head>`:

~~~html
<link rel="alternate"
      type="application/json"
      href="https://example.com/post/llm/">
~~~

**Attributes:**

- `rel="alternate"`: Indicates alternate representation ({{RFC8288}})
- `type="application/json"`: Machine-readable format
- `href`: Absolute or relative URL to M-URL

## M-URL to C-URL Canonicalization

The M-URL response MUST include an HTTP `Link` header with `rel="canonical"`:

~~~http
Link: <https://example.com/post/>; rel="canonical"
~~~

This establishes bidirectional verification and prevents SEO duplication.

## Deterministic Mapping

M-URLs SHOULD follow a deterministic pattern from C-URLs.

**Non-normative examples**: Append a slug to the C-URL path, such as `/tct/` or `/llm/`.

Example:
- C-URL: `https://example.com/post/`
- M-URL: `https://example.com/post/tct/` (or `.../llm/`)

**Guidance**:
- These path patterns are examples, not protocol requirements. If a preferred slug collides with existing site routes, publishers MAY choose an alternate (e.g., `/tct/`, `/api/tct/`) or publish a mapping in a site-level manifest.
- Agents MUST NOT assume a fixed path. Agents SHOULD discover M-URLs via HTML `<link rel="alternate" type="application/json">`, HTTP `Link` headers, or the JSON sitemap.

**Migration Note**: Publishers MAY use HTTP 308 Permanent Redirect to migrate from legacy paths to preferred endpoints and SHOULD list only the primary M-URL in the sitemap.

**Discovery Hint**: Publishers MAY include a `Link` header on the homepage or C-URL HTML responses to advertise the sitemap location (example paths only):

~~~http
Link: </tct-sitemap.json>; rel="index"; type="application/json"
Link: </llm-sitemap.json>; rel="index"; type="application/json"
~~~

## Content Pages Only

M-URL endpoints SHOULD be provided only for **content pages** (posts, articles, pages) and NOT for archive pages (category listings, tag archives, search results, date archives).

**Rationale:**

- Archive pages contain navigation and lists, not primary content
- Template-invariant fingerprinting is designed for stable content, not dynamic lists
- Archive pages change frequently as new content is published

**Implementation:**

- Publishers SHOULD return HTTP 404 for M-URL requests to archive pages
- Sitemaps SHOULD include only content page URLs, not archives
- Homepage MAY be included if it represents stable content

**Content page examples:**
- Blog posts: `/blog/understanding-tct/`
- Articles: `/news/2025/protocol-launch/`
- Static pages: `/about/`, `/contact/`

**Archive page examples (should NOT have M-URLs):**
- Category archives: `/category/technology/`
- Tag archives: `/tag/web-protocols/`
- Date archives: `/2025/10/`
- Search results: `/search/?q=protocol`
- Author archives: `/author/john/`

# Template-Invariant Fingerprinting

## Normalization Algorithm

To generate a template-invariant fingerprint:

1. **Parse**: Extract core content from HTML (Article, main content region)
2. **Filter**: Remove boilerplate (navigation, footer, sidebar, ads, scripts, styles)
3. **Strip Tags**: Convert HTML to plain text
4. **Normalize**:
   - Decode HTML entities
   - Apply Unicode normalization (NFKC) and Unicode casefolding (locale-independent)
   - Remove control characters
   - Collapse whitespace runs to a single ASCII space
   - Optionally normalize punctuation (e.g., map curly quotes to ASCII)
   - Trim leading/trailing whitespace
5. **Hash**: Compute SHA-256 of normalized text

**Example implementation (pseudocode):**

~~~
function generateFingerprint(html):
  dom = parseHTML(html)
  content = extractPrimaryContent(dom)  // <article>, <main>, etc.
  text = stripTags(content)

  normalized = text
    .decodeEntities()
    .unicodeNormalize('NFKC')
    .casefold()
    .removeControlChars()
    .replace(/\s+/g, ' ')
    .normalizePunctuation()
    .trim()

  return "sha256-" + sha256(normalized)
~~~

**Note**: This is illustrative pseudocode. Production implementations must use global replacement for all normalization patterns, handle Unicode edge cases, and select appropriate content extraction strategies for their platform.

## ETag Generation

The M-URL response MUST include an `ETag` header containing the template-invariant fingerprint.

**Weak ETag (RECOMMENDED):**

~~~http
ETag: W/"sha256-2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
~~~

**Strong ETag (acceptable):**

~~~http
ETag: "sha256-2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
~~~

**Rationale:**
- Template-invariant fingerprints represent semantic equivalence, not byte-for-byte identity
- Weak ETags (`W/"..."`) better express this semantic validator property
- Strong ETags acceptable if computed per representation variant (including compression)

**Format Requirements:**
- MUST start with `sha256-` prefix (after optional `W/"` weak prefix)
- MUST contain 64 hexadecimal characters (256 bits)
- Weak ETags SHOULD be used for semantic fingerprints

**Note:** `If-Range` requests require strong ETags; respond with `412 Precondition Failed` if client sends `If-Range` with weak ETag.

# Conditional Request Discipline

## If-None-Match Precedence

When both `If-None-Match` and `If-Modified-Since` headers are present, servers MUST give `If-None-Match` precedence per {{RFC9110, Section 13.1.2}}. This means:

1. Evaluate `If-None-Match` first
2. If ETag matches, return `304 Not Modified` (ignore `If-Modified-Since`)
3. If ETag doesn't match, process `If-Modified-Since` (if present)

**Rationale:** ETags provide stronger validation than modification dates, especially for semantic fingerprints.

## 304 Not Modified Response

When the ETag matches the `If-None-Match` value:

1. Server MUST respond with `304 Not Modified`
2. Response MUST NOT include a message body
3. Response MUST include `ETag` header; SHOULD include `Last-Modified` and `Cache-Control` headers

**Example:**

~~~http
HTTP/1.1 304 Not Modified
ETag: W/"sha256-2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
Last-Modified: Wed, 21 Oct 2025 07:28:00 GMT
Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60, stale-if-error=86400
Vary: Accept-Encoding
~~~

## Cache-Control Directives

M-URL responses SHOULD use:

~~~http
Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60, stale-if-error=86400
~~~

**Directives:**
- `max-age=0`: Require revalidation before serving from cache
- `must-revalidate`: Do not serve stale without successful revalidation
- `stale-while-revalidate=60`: Serve stale while revalidating in background (60s window)
- `stale-if-error=86400`: Serve stale if origin unavailable (24h window)

Note: Avoid `private` on M-URLs and sitemaps (they are cacheable by shared caches).

## Vary Header

Responses SHOULD include `Vary: Accept-Encoding` to indicate compression variance:

~~~http
Vary: Accept-Encoding
~~~

M-URL responses primarily vary by compression (gzip, brotli), not by content type (always `application/json`).

## HEAD Request Support

Servers SHOULD support HEAD requests for all M-URLs and sitemaps.

HEAD responses MUST:
- Return same HTTP headers as equivalent GET request
- NOT include a message body
- Include all validators (`ETag`, `Last-Modified`, `Cache-Control`)

**Example:**

~~~http
HEAD /post/llm/ HTTP/1.1
Host: example.com

HTTP/1.1 200 OK
ETag: W/"sha256-abc123..."
Last-Modified: Wed, 21 Oct 2025 07:28:00 GMT
Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60
Vary: Accept-Encoding
Content-Type: application/json
Content-Length: 1234
~~~

This enables efficient validation without transferring the full response body.

# Sitemap-First Verification

## JSON Sitemap Format

Publishers SHOULD provide a machine-readable sitemap at a well-known location (e.g., `/llm-sitemap.json`).

**Schema:**

~~~json
{
  "version": 1,
  "profile": "tct-1",
  "items": [
    {
      "cUrl": "https://example.com/post/",
      "mUrl": "https://example.com/post/llm/",
      "modified": "2025-10-01T12:34:56Z",
      "contentHash": "sha256-2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
    }
  ]
}
~~~

**Fields:**

- `version` (integer): Sitemap format version (currently 1)
- `profile` (string, RECOMMENDED): Protocol version identifier (e.g., "tct-1"). Enables clients to detect protocol capabilities and maintain forward compatibility as the specification evolves.
- `items` (array): List of URL pairs
  - `cUrl` (string, required): Canonical URL
  - `mUrl` (string, required): Machine URL
  - `modified` (string, RFC 3339): Last modification timestamp
  - `contentHash` (string, required): Template-invariant fingerprint (same as M-URL ETag)

**Parity Rule:**

The sitemap `contentHash` value MUST match the M-URL `ETag` header value, excluding quotes and the `W/` weak prefix.

**Forward Compatibility:**

Clients MUST ignore unknown fields in the sitemap JSON. Servers MAY add additional fields to support future protocol versions.

## Sitemap Scalability

Publishers and agents SHOULD consider scalability for large sites:

- Publishers MAY split sitemaps into an index (a sitemap-of-sitemaps) to segment large URL sets
- Servers SHOULD support compression (e.g., `Content-Encoding: gzip` or `br`) for sitemap responses; agents SHOULD accept compressed responses
- Agents SHOULD use streaming JSON parsers for large sitemaps and enforce a maximum sitemap size (e.g., 100 MB)

**Example:
- M-URL returns: `ETag: W/"sha256-2c26b46b..."`
- Sitemap contains: `"contentHash": "sha256-2c26b46b..."`

This enables zero-fetch skip optimization: clients compare sitemap hash to cached ETag without fetching the M-URL.

**Homepage Handling:**

Publishers SHOULD include the site homepage as the **first item** in the sitemap array. This provides automated agents with immediate access to site-level context (site name, description, purpose) before processing individual content pages.

For homepages that display dynamic content listings (blog roll, latest posts), publishers MAY synthesize stable content representing the site overview rather than the dynamic list.

**Example with homepage first:**

~~~json
{
  "version": 1,
  "profile": "tct-1",
  "items": [
    {
      "cUrl": "https://example.com/",
      "mUrl": "https://example.com/llm/",
      "modified": "2025-10-15T08:00:00Z",
      "contentHash": "sha256-abc123..."
    },
    {
      "cUrl": "https://example.com/about/",
      "mUrl": "https://example.com/about/llm/",
      "modified": "2025-10-01T10:00:00Z",
      "contentHash": "sha256-def456..."
    }
  ]
}
~~~

**Sitemap HTTP Response:**

~~~http
GET /llm-sitemap.json HTTP/1.1
Host: example.com

HTTP/1.1 200 OK
Content-Type: application/json
ETag: W/"sha256-sitemap-fingerprint"
Last-Modified: Wed, 21 Oct 2025 12:00:00 GMT
Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60
Vary: Accept-Encoding
Content-Length: 4567

{
  "version": 1,
  "profile": "tct-1",
  "items": [...]
}
~~~

**Conditional Sitemap Fetch:**

~~~http
GET /llm-sitemap.json HTTP/1.1
Host: example.com
If-None-Match: W/"sha256-sitemap-fingerprint"

HTTP/1.1 304 Not Modified
ETag: W/"sha256-sitemap-fingerprint"
Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60
~~~

Clients SHOULD use conditional requests for sitemap to avoid unnecessary bandwidth when sitemap unchanged.

## Zero-Fetch Skip Logic

Automated agents SHOULD:

1. Fetch `/llm-sitemap.json` periodically
2. Compare `contentHash` values to locally cached hashes
3. **If hash unchanged**: Skip fetching both C-URL and M-URL (zero-fetch optimization)
4. **If hash changed**: Issue conditional GET to M-URL with `If-None-Match`

This enables 90%+ skip rate for unchanged content.

**Example workflow:**

~~~
Agent: Fetch /llm-sitemap.json
Agent: item.contentHash = "sha256-abc123..."
Agent: cachedHash = lookup(item.mUrl)

if (item.contentHash === cachedHash):
  // Zero-fetch: Content unchanged, skip all requests
  skip()
else:
  // Hash changed, fetch with conditional request
  GET item.mUrl
  Headers: If-None-Match: "sha256-abc123..."

  if (response.status == 304):
    // Still matched at endpoint, update cache
    cache(item.mUrl, item.contentHash)
  else:
    // Content changed, process new data
    process(response.body)
    cache(item.mUrl, response.headers['ETag'])
~~~

# Publisher Policy Descriptor

Publishers MAY provide a machine-readable policy descriptor at a well-known location (e.g., `/tct-policy.json` or `/llm-policy.json`) to communicate usage terms, rate limits, and content licensing preferences to automated agents. Example paths are non-normative.

## Policy Endpoint

The policy descriptor SHOULD be available at a stable URL. Example paths (non-normative):

~~~
https://example.com/tct-policy.json
https://example.com/llm-policy.json
~~~

## JSON Schema

~~~json
{
  "profile": "tct-policy-1",
  "version": 1,
  "effective": "2025-10-01T00:00:00Z",
  "updated": "2025-10-15T12:00:00Z",

  "policy_urls": {
    "terms_of_service": "https://example.com/terms/",
    "payment_info": "https://example.com/pricing/",
    "contact": "https://example.com/contact/"
  },

  "purposes": {
    "allow_ai_input": true,
    "allow_ai_train": false,
    "allow_search_indexing": true
  },

  "requirements": {
    "attribution_required": true,
    "link_back_required": false,
    "notice_required": true
  },

  "rate_hints": {
    "max_requests_per_second": null,
    "max_requests_per_day": 10000,
    "note": "Advisory limits, honor system"
  }
}
~~~

**Fields:**

- `profile` (string): Policy schema version identifier (e.g., "tct-policy-1")
- `version` (integer): Policy revision number
- `effective` (string, RFC 3339): When policy took effect
- `updated` (string, RFC 3339): Last policy modification
- `policy_urls` (object): URLs to human-readable policy documents
  - `terms_of_service`: Legal terms URL
  - `payment_info`: Pricing/billing information URL (for paid access)
  - `contact`: Publisher contact for licensing inquiries
- `purposes` (object): Usage permissions
  - `allow_ai_input`: Content may be used as AI input (RAG, context)
  - `allow_ai_train`: Content may be used for model training
  - `allow_search_indexing`: Content may be indexed for search
- `requirements` (object): Usage conditions
  - `attribution_required`: Must credit publisher when using content
  - `link_back_required`: Must link to canonical URL when republishing
  - `notice_required`: Must notify publisher of commercial use
- `rate_hints` (object): Advisory crawl rate limits (non-binding)
  - `max_requests_per_second`: Requests per second limit (null = no limit)
  - `max_requests_per_day`: Requests per day limit
  - `note`: Additional guidance

Rate hints are advisory only; enforcement, payment, and economic arrangements are out of scope for this specification. Vocabulary alignment with IETF AIPREF is expected as that work matures.

## Discovery

The policy descriptor SHOULD be linked from the sitemap with a `describedby` Link header (example paths only):

~~~http
Link: </tct-policy.json>; rel="describedby"; type="application/json"
Link: </llm-policy.json>; rel="describedby"; type="application/json"
~~~

Automated agents SHOULD:

1. Fetch `/llm-policy.json` before crawling
2. Honor stated usage restrictions
3. Respect rate hints to avoid overwhelming origin
4. Review terms before commercial use

## Alignment with IETF AIPREF

This policy format is designed to complement the IETF AIPREF (AI Preferences) proposal, providing machine-readable expressions of publisher preferences for automated agent behavior.

# M-URL Response Format

## Content-Type

M-URL responses MUST set `Content-Type: application/json`:

~~~http
Content-Type: application/json; charset=utf-8
~~~

## JSON Payload Schema

**Minimal required fields:**

~~~json
{
  "profile": "tct-1",
  "canonical_url": "https://example.com/post/",
  "title": "Article Title",
  "content": "Core article content...",
  "hash": "2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
}
~~~

**Fields:**

- `profile` (string, RECOMMENDED): Protocol version identifier (e.g., "tct-1"). Future versions (e.g., "tct-2") can introduce new fields while maintaining backward compatibility.
- `canonical_url` (string, required): The C-URL for this resource
- `title` (string, required): Resource title
- `content` (string, required): Core content text
- `hash` (string, required): Template-invariant fingerprint. MUST equal the ETag value excluding quotes and the `W/` weak prefix (e.g., if ETag is `W/"sha256-abc123"`, hash is `sha256-abc123`)

**Forward Compatibility:**

Clients MUST ignore unknown fields in the JSON payload. Servers MAY add additional fields to support future protocol versions or domain-specific metadata.

**Extended fields example:**

~~~json
{
  "profile": "tct-1",
  "canonical_url": "https://example.com/post/",
  "title": "Article Title",
  "language": "en-US",
  "published": "2025-10-01T10:00:00Z",
  "modified": "2025-10-15T14:30:00Z",
  "content": "Core article content...",
  "hash": "2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae",
  "structured_data": {
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "Article Title"
  }
}
~~~

## Complete Response Example

~~~http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
ETag: W/"sha256-2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
Link: <https://example.com/post/>; rel="canonical"
Last-Modified: Wed, 15 Oct 2025 14:30:00 GMT
Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60, stale-if-error=86400
Vary: Accept-Encoding
Content-Length: 1234

{
  "profile": "tct-1",
  "canonical_url": "https://example.com/post/",
  "title": "Understanding the Collaboration Tunnel Protocol",
  "language": "en",
  "published": "2025-10-01T10:00:00Z",
  "modified": "2025-10-15T14:30:00Z",
  "content": "The Collaboration Tunnel Protocol enables...",
  "hash": "2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
}
~~~

# Operational Considerations (Informative)

This section provides non-normative operational guidance for deployers and automated agents.

**Migration and Path Collisions**
- If a preferred slug (e.g., `/llm/`) collides with existing routes, choose an alternate (e.g., `/tct/`, `/api/tct/`) and publish a 308 Permanent Redirect from legacy to primary endpoints
- List only the primary M-URL in the sitemap to avoid duplication; agents SHOULD follow redirects but rely on sitemap entries for canonical endpoint discovery
- Ensure robots.txt permits the chosen machine paths as appropriate for your policy

**Caching and CDNs**
- Configure intermediaries to honor validators and serve 304 responses; avoid `private` on M-URLs and sitemaps to enable shared caching
- Enable compression (gzip or br) for sitemaps and JSON payloads

**Discovery and Verification**
- Prefer discovery via HTML `<link rel="alternate" type="application/json">`, HTTP `Link` headers, or sitemap entries; agents MUST NOT assume a fixed path
- Agents SHOULD verify the M-URL’s canonical link header before processing and skip ingestion if missing or mismatched

**Network and WAFs**
- Allow the HEAD method (some WAFs block by default) to enable efficient validation
- If using an edge worker/proxy, ensure required headers (`Link`, `ETag`, `Cache-Control`, `Vary`) are preserved or injected consistently

# Security Considerations

## Content Integrity

The template-invariant fingerprint (SHA-256) provides:
- **Tamper detection**: Agents can verify content integrity
- **Cache validation**: Ensures served content matches expected hash

However, SHA-256 alone does NOT provide:
- **Authentication**: Does not prove publisher identity
- **Non-repudiation**: Publisher can deny serving content

For authenticated content delivery, publishers MAY implement digital signatures (outside scope of this specification).

## Privacy

Sitemap exposure MAY reveal:
- **Content inventory**: All published URLs
- **Modification patterns**: Publishing/update frequency

Publishers SHOULD:
- Apply access controls if sitemaps contain sensitive URLs
- Use robots.txt to restrict crawler access if needed

For general privacy considerations related to Internet protocols, see {{RFC6973}}.

## Denial of Service

### Sitemap Abuse

Large sitemaps MAY be used for DoS attacks. Agents SHOULD:
- Implement request rate limiting
- Set maximum sitemap size limits (e.g., 100 MB)
- Use streaming JSON parsers for large sitemaps

### HEAD vs GET Bandwidth

HEAD requests provide DoS mitigation by enabling validation without body transfer:
- HEAD request: ~500 bytes (headers only)
- GET request: 1-100 KB (full JSON body)

Agents SHOULD use HEAD for validation before GET.

## Injection Surface Reduction

M-URL JSON responses contain only structured text, reducing injection attack surface compared to HTML:
- No `<script>` tags or inline JavaScript
- No CSS injection vectors
- No HTML parsing ambiguities

However, agents MUST:
- Validate JSON syntax before processing
- Sanitize content before rendering to users
- Treat URLs in JSON as untrusted input

## Content Provenance (Optional)

Publishers MAY enhance content authenticity using:

**Content-Digest (RFC 9530):**
~~~http
Content-Digest: sha-256=:X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=:
~~~

**HTTP Message Signatures (RFC 9421):**
~~~http
Signature: sig1=:MEUCIQDXlI...;
Signature-Input: sig1=("content-digest" "content-type");created=1618884473;keyid="key-1"
~~~

These mechanisms are OPTIONAL extensions outside TCT core requirements.

## Privacy and PII

M-URL JSON MAY contain personally identifiable information (PII) or sensitive content. Publishers SHOULD:
- Apply same access controls as C-URL HTML
- Respect user privacy preferences
- Comply with applicable data protection regulations (GDPR, CCPA, etc.)

Agents MUST:
- Honor robots.txt and meta robots directives (agents SHOULD respect robots.txt as site-wide policy across all resources)
- Respect HTTP authentication requirements
- Not expose cached content beyond publisher-specified Cache-Control directives

## Access Control

Publishers MAY restrict M-URL access using standard HTTP mechanisms:
- HTTP Authentication (Basic, Bearer, etc.)
- IP allowlisting
- Rate limiting per client
- Geographic restrictions

Access controls SHOULD be consistent between C-URL and M-URL for the same resource.

**Security Note**: Authenticated M-URLs MUST NOT leak protected content via publicly accessible sitemaps. Publishers MUST either exclude protected resources from sitemaps or apply equivalent access controls to the sitemap itself.

# Energy Efficiency Considerations

## Overview

The Collaboration Tunnel Protocol's bandwidth and token reduction directly translates to significant energy savings across network infrastructure and AI model inference operations. This section quantifies the environmental impact of TCT deployment at scale.

## Network Energy Consumption

Data transmission consumes energy at every network layer: routers, switches, content delivery networks (CDNs), data centers, and end-user devices. Current estimates for network energy intensity range from 0.03 kWh/GB (fixed networks) to 0.14 kWh/GB (mobile networks), with a conservative industry standard of 0.06 kWh/GB for mixed-mode transmission.

**TCT bandwidth reduction (measured):**
- HTML-only retrieval: 103 KB average
- TCT JSON delivery: 17.7 KB average
- **Reduction: 85.3 KB per fetch (83% savings)**

**Energy savings per fetch:**
- Bandwidth saved: 85.3 KB = 0.0000853 GB
- Energy saved: 0.0000853 GB x 0.06 kWh/GB = **0.0000051 kWh** (0.0051 Wh) per fetch

**Scaled impact (1 million fetches/day):**
- Daily energy savings: 5.1 kWh (5,100 Wh)
- Annual energy savings: 1,861.5 kWh
- **Carbon equivalent**: ~930 kg CO2 avoided (assuming 0.5 kg CO2/kWh grid average)

## AI Inference Impact (Informative Summary)

TCT reduces tokenized payloads by ~86% through normalized JSON delivery. The impact on inference energy depends on model size, hardware, batching, and caching; therefore specific kWh figures vary significantly across deployments. See Appendix "Energy Methodology" for an illustrative, non-normative scenario and assumptions.

## Sitemap-First Zero-Fetch Optimization

Beyond per-fetch savings, TCT's sitemap-first verification enables complete request elimination when content is unchanged.

**Measured skip rate:** 90%+ for unchanged content

For a typical deployment with 1,000 URLs checked daily:
- Traditional crawler: 1,000 full fetches/day
- TCT deployment: ~100 fetches/day (90% skipped via sitemap comparison)
- **Additional savings: 900 fetches avoided**

**Combined energy impact:**
- Network: Fewer full responses reduce transmission energy proportionally to bytes avoided
- Computation: Fewer fetches and elimination of local rendering/extraction reduce crawler CPU/memory work
- Inference: Reduced token processing (impact varies by model and deployment; see Energy Methodology appendix)

## Comparison to Existing Approaches

| Method | Avg Size | Tokens | Bandwidth Reduction |
|--------|----------|--------|---------------------|
| Full HTML page | 350 KB | 47,000 | Baseline |
| HTML body only | 103 KB | 13,900 | 71% |
| AMP HTML | 52 KB | 7,000 | 85% |
| **TCT JSON** | **17.7 KB** | **1,960** | **95%** |

Energy impact per fetch depends on network infrastructure, model architecture, and deployment patterns. See Energy Methodology appendix for illustrative calculations.

## Cumulative Environmental Impact

TCT's bandwidth reduction (typically 80-90%) and elimination of client-side rendering translate to measurable energy savings at scale. Actual impact depends on:

- Deployment scope (number of sites, request volume)
- Network infrastructure efficiency
- Model provider's hardware and batching strategies
- Grid carbon intensity in serving regions

**Measured improvements:**
- 95% smaller payloads (measured across production deployments)
- 90%+ fetch elimination via sitemap-first logic
- 94% reduction in crawler CPU/memory usage

Organizations deploying TCT should measure baseline energy consumption and monitor post-deployment savings specific to their infrastructure. See Energy Methodology appendix for example calculation approaches.

## Relationship to IETF GREEN Working Group

The IETF Getting Ready for Energy-Efficient Networking (GREEN) Working Group focuses on infrastructure-level energy optimization through monitoring, measurement, and network-wide traffic optimization.

TCT complements GREEN's infrastructure focus by addressing **application-layer efficiency**:

- GREEN targets: Device power consumption, network-wide traffic flow optimization
- TCT targets: Content delivery efficiency, redundant fetch elimination, token cost reduction

Together, these approaches create a comprehensive energy reduction strategy spanning both network infrastructure (GREEN WG) and application protocols (TCT).

## Recommendations for Implementers

Publishers deploying TCT SHOULD:

1. **Monitor metrics**: Track bandwidth savings, 304 hit rates, and skip rates
2. **Report impact**: Document energy savings for sustainability reporting
3. **Optimize aggressively**: Minimize JSON payload size to maximize efficiency
4. **Promote adoption**: Encourage crawler operators to implement sitemap-first logic

Automated agents consuming TCT endpoints SHOULD:

1. **Implement zero-fetch**: Use sitemap hashes to skip unchanged content
2. **Respect 304 responses**: Honor conditional request discipline
3. **Cache aggressively**: Store ETags and avoid redundant fetches
4. **Measure savings**: Track bandwidth, token, and energy reduction

## Future Work

Potential enhancements for energy optimization:

- **Delta encoding**: Transmit only content changes instead of full payloads
- **Push notifications**: WebSub or IndexNow integration to eliminate polling
- **Compression**: Brotli/gzip to further reduce transmission size
- **Edge caching**: CDN-level 304 responses to reduce origin load

# IANA Considerations

This document has no IANA actions.

(Future versions may register:
- Link relation type: `rel="machine-alternate"`
- Well-known URI: `/.well-known/llm-sitemap.json`)

# Comparison to Prior Art

## ResourceSync

ResourceSync {{ResourceSync}} provides sitemap-based synchronization with content digests (`md5`, `sha-256`) in ResourceSync Change Lists.

**Key differences:**

1. **Endpoint validator**: ResourceSync does NOT specify using the digest as endpoint ETag
2. **Handshake**: No bidirectional C-URL <-> M-URL discovery
3. **Zero-fetch**: No specification for skipping endpoint fetch when sitemap hash matches
4. **Precedence discipline**: No requirement for If-None-Match precedence

TCT integrates the SAME hash in both sitemap AND endpoint ETag, enabling zero-fetch optimization.

## AMP

Accelerated Mobile Pages {{AMP}} provides:
- C-URL -> AMP-URL mapping via `<link rel="amphtml">`
- Lighter HTML (no JavaScript allowed)

**Key differences:**

1. **Format**: AMP uses HTML; TCT uses JSON
2. **Fingerprinting**: AMP has no content hashing
3. **Sitemap**: AMP sitemaps lack content hashes
4. **Conditional requests**: No specified validator discipline

## XML Sitemaps

Traditional XML sitemaps provide:
- URL discovery
- `<lastmod>` timestamp

**Key differences:**

1. **Hashes**: XML sitemaps lack content fingerprints
2. **Timestamps**: `lastmod` insufficient for template changes
3. **Conditional requests**: No integration with HTTP caching

TCT adds content hashes enabling efficient change detection.

# Implementation Status

[Note to RFC Editor: Remove this section before publication.]

As of October 2025, TCT has been implemented in:

**Publisher implementations:**
- WordPress plugin (970 URLs across 3 production sites, 100% compliance)
- Cloudflare Worker (edge-based implementation)

**Measurements:**
- HTML-only: 103 KB average (13,900 tokens)
- TCT JSON: 17.7 KB average (1,960 tokens)
- **Bandwidth savings: 83%**
- **Token savings: 86%**

**Sitemap-first performance:**
- Skip rate: 90%+ for unchanged content
- 304 hit rate: 95%+ on changed content

# Acknowledgments

Thanks to the WordPress, Cloudflare, and IETF HTTP Working Group communities for feedback on early drafts.

--- back

# Example Implementation (WordPress)

This section is informative.

**PHP implementation** (simplified):

~~~php
function handle_llm_endpoint() {
  $canonical_url = remove_suffix($_SERVER['REQUEST_URI'], '/llm/');

  // Extract core content
  $html = get_canonical_html($canonical_url);
  $content = extract_core_content($html);

  // Generate ETag
  $normalized = normalize_text($content);
  $hash = hash('sha256', $normalized);
  $etag = "W/\"sha256-{$hash}\""; // weak ETag recommended for semantic fingerprints

  // Check If-None-Match
  if ($_SERVER['HTTP_IF_NONE_MATCH'] === $etag) {
    header('HTTP/1.1 304 Not Modified');
    header("ETag: {$etag}");
    header("Link: <{$canonical_url}>; rel=\"canonical\"");
    exit;
  }

  // Return JSON
  header('Content-Type: application/json');
  header("ETag: {$etag}");
  header("Link: <{$canonical_url}>; rel=\"canonical\"");
  header('Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60, stale-if-error=86400');
  header('Vary: Accept-Encoding');

  echo json_encode([
    'canonical_url' => $canonical_url,
    'content' => $content,
    'hash' => $hash
  ]);
}
~~~

# Example Sitemap

This section is informative.

~~~json
{
  "version": 1,
  "items": [
    {
      "cUrl": "https://example.com/article-1/",
      "mUrl": "https://example.com/article-1/llm/",
      "modified": "2025-10-01T12:00:00Z",
      "contentHash": "sha256-abc123..."
    },
    {
      "cUrl": "https://example.com/article-2/",
      "mUrl": "https://example.com/article-2/llm/",
      "modified": "2025-09-15T08:30:00Z",
      "contentHash": "sha256-def456..."
    }
  ]
}
~~~
# Energy Methodology (Informative)

**Scope**: This appendix provides illustrative, non-normative estimates to contextualize the potential energy impact of TCT deployment. Actual results will vary significantly based on infrastructure, model architecture, and deployment patterns.

## Metrics and Sources

**Network Energy Intensity:**
- Range: 0.03-0.14 kWh/GB (varies by network type: fixed vs mobile)
- Conservative estimate used: 0.06 kWh/GB (industry midpoint for mixed-mode transmission)
- Variability: Actual consumption depends on routing, CDN utilization, and regional infrastructure

**Crawler Computational Energy:**
- Measured reductions from controlled benchmarks (primary evidence):
  - CPU time: -94% (median)
  - Memory usage: -95% (median)
- Method: Headless browser + extraction vs direct TCT JSON retrieval
- Limitations: Results vary by site complexity, extractor implementation, and hardware

**AI Inference Energy:**
- Highly variable: depends on model size (parameters), quantization, hardware (GPU/TPU type), batching, and prompt caching
- No single authoritative figure exists; published estimates vary by 10x or more

## Example Calculations

**Network Energy (per fetch):**
- Bandwidth saved: 85.3 KB = 0.0000853 GB
- Energy saved: 0.0000853 GB x 0.06 kWh/GB = **0.0000051 kWh** (0.0051 Wh) per fetch

**Scaled Network Impact (1 million fetches/day):**
- Daily savings: 0.0000051 kWh x 1,000,000 = 5.1 kWh (5,100 Wh)
- Annual savings: 1,861.5 kWh
- Carbon equivalent (at 0.5 kg CO2/kWh grid average): ~930 kg CO2 avoided annually

**AI Inference (Illustrative Scenario - Not Normative):**

*Hypothetical assumptions for illustration only:*
- Assume a model with energy cost E_token = 0.000187 kWh/token (one published estimate for large models)
- TCT reduces tokens by 11,940 per fetch (measured from our benchmarks)
- Estimated savings per query: 11,940 x 0.000187 kWh = 2.23 kWh per query

*Sensitivity:*
- If E_token = 0.0001 kWh (smaller model, better batching): savings ~ 1.19 kWh per query
- If E_token = 0.0003 kWh (larger model, poor batching): savings ~ 3.58 kWh per query
- **Range varies by 3x based on deployment parameters**

*Important caveats:*
1. Published per-token energy estimates vary widely and often lack transparency about measurement conditions
2. Prompt caching, batching, and model quantization can reduce inference costs by 5-10x
3. Token savings do not translate linearly to energy savings in all scenarios
4. Use these figures only for order-of-magnitude context, not precise predictions

## Uncertainty and Variability

Results depend on:
- **Hardware**: GPU/TPU model, fabrication node, power management
- **Grid carbon intensity**: 0.2-0.9 kg CO2/kWh depending on region
- **Model architecture**: Parameter count, attention mechanism, quantization
- **Deployment patterns**: Batch size, request concurrency, cache hit rates
- **Network path**: CDN caching, routing efficiency, edge compute

## Reproducibility

**Benchmark dataset:**
- Multiple production sites with varying complexity
- Representative URL samples across content types
- Repeated runs (~3 per URL) with median values reported
- Outliers inspected; clear anomalies excluded

**Measurement environment:**
- Stable network conditions
- Same host for both methods (browser vs direct fetch)
- Tools: Headless browser (Puppeteer/Playwright), standard HTTP clients

**Reporting recommendations:**
- Always report ranges and confidence intervals
- State assumptions explicitly
- Cite sources for energy intensity figures
- Acknowledge deployment-specific variability

## Recommendations for Operators

When estimating TCT energy impact for your deployment:

1. **Measure your baseline**: Actual HTML payload sizes and token counts
2. **Use conservative estimates**: Start with proven network energy figures (0.06 kWh/GB)
3. **Focus on measurable metrics**: Bandwidth reduction, 304 hit rates, skip rates
4. **Report ranges**: Account for variability in model efficiency and batching
5. **Update with real data**: Monitor actual savings post-deployment

**Example conservative reporting:**
> "TCT deployment reduced our network transfer volume by 83% (measured). Assuming network energy intensity of 0.06 kWh/GB, we estimate network energy savings of ~5 MWh annually. Inference energy impact depends on our model provider's infrastructure and is not separately quantified."

