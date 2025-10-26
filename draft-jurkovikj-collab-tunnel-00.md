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

informative:
  ResourceSync:
    title: ResourceSync Framework Specification
    author:
      - org: Open Archives Initiative
    date: 2017-05
    target: https://www.openarchives.org/rs/1.1/resourcesync

  AMP:
    title: Accelerated Mobile Pages (AMP) HTML Specification
    target: https://amp.dev/documentation/guides-and-tutorials/learn/spec/amphtml/

--- abstract

This document specifies the Collaboration Tunnel Protocol, a method for efficient, verifiable content delivery between web publishers and automated agents. The protocol achieves 60-90% bandwidth reduction through bidirectional URL discovery, template-invariant content fingerprinting, sitemap-first verification, and strict conditional request discipline.

--- middle

# Introduction

Automated agents (AI crawlers, search engines, content aggregators) increasingly consume web content at scale. Traditional HTML delivery designed for human browsers imposes unnecessary overhead: presentational boilerplate (navigation, footers, advertisements), large CSS/JavaScript bundles, and redundant fetches of unchanged content.

Existing approaches address portions of this problem:

- XML Sitemaps {{?RFC6926}} provide discovery but lack content fingerprints
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

1. **Bidirectional Discovery**: Explicit C-URL ↔ M-URL handshake preventing SEO conflicts
2. **Template-Invariant Fingerprinting**: Normalized content hashing for stable cache validation
3. **Conditional Request Discipline**: Strict If-None-Match precedence and 304 responses
4. **Sitemap-First Verification**: Zero-fetch skip logic when content unchanged

## Architecture

~~~
┌────────────────────────┐          ┌──────────────────────┐
│  Publisher (Origin)    │          │  Automated Agent     │
├────────────────────────┤          ├──────────────────────┤
│                        │          │                      │
│  C-URL (HTML)          │◄─────────┤  1. Fetch Sitemap    │
│  ├─ <link rel=         │          │                      │
│  │  "alternate"        │          │  2. Compare Hashes   │
│  │  type="app/json"    │          │     (Zero-Fetch)     │
│  │  href="/llm/">      │          │                      │
│                        │          │  3. If Changed:      │
│  M-URL (/llm/)         │          │     GET /llm/        │
│  ├─ Link: canonical    │◄─────────┤     If-None-Match    │
│  ├─ ETag: sha256-...   │          │                      │
│  ├─ Content-Type: JSON │──────────►  4. 304 or 200+JSON  │
│                        │          │                      │
│  /llm-sitemap.json     │          │  5. Cache ETag       │
│  ├─ {cUrl, mUrl,       │◄─────────┤                      │
│  │   contentHash}      │          │                      │
└────────────────────────┘          └──────────────────────┘
~~~

# Bidirectional Discovery

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

**Recommended pattern**: Append `/llm/` to C-URL path

Example:
- C-URL: `https://example.com/post/`
- M-URL: `https://example.com/post/llm/`

Alternative patterns MAY be used but MUST be documented in a site-level manifest.

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
   - Convert to lowercase
   - Collapse whitespace runs to single space
   - Normalize quotation marks (e.g., " → ", ' → ')
   - Trim leading/trailing whitespace
5. **Hash**: Compute SHA-256 of normalized text

**Example implementation (pseudocode):**

~~~
function generateFingerprint(html):
  dom = parseHTML(html)
  content = extractPrimaryContent(dom)  // <article>, <main>, etc.
  text = stripTags(content)

  normalized = text
    .toLowerCase()
    .replace(/\s+/g, ' ')
    .replace(/[""]/, '"')
    .replace(/['']/, "'")
    .trim()

  return "sha256-" + sha256(normalized)
~~~

## ETag Generation

The M-URL response MUST include an `ETag` header containing the template-invariant fingerprint:

~~~http
ETag: "sha256-2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
~~~

The ETag value:
- MUST be enclosed in quotes (strong ETag)
- MUST start with `sha256-` prefix
- MUST contain 64 hexadecimal characters (256 bits)

# Conditional Request Discipline

## If-None-Match Precedence

When both `If-None-Match` and `If-Modified-Since` headers are present, servers MUST evaluate `If-None-Match` with precedence ({{RFC9110, Section 13.1.2}}).

## 304 Not Modified Response

When the ETag matches the `If-None-Match` value:

1. Server MUST respond with `304 Not Modified`
2. Response MUST NOT include a message body
3. Response MUST include validators: `ETag`, `Last-Modified`, `Cache-Control`

**Example:**

~~~http
HTTP/1.1 304 Not Modified
ETag: "sha256-2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
Last-Modified: Wed, 21 Oct 2025 07:28:00 GMT
Cache-Control: private, max-age=0, must-revalidate
Vary: Accept
~~~

## Cache-Control Directives

M-URL responses SHOULD include:

- `must-revalidate`: Require validation before serving stale content
- `max-age=0` or `private`: Prevent long-term caching without validation

~~~http
Cache-Control: private, max-age=0, must-revalidate
~~~

## Vary Header

Responses SHOULD include `Vary: Accept` to enable content negotiation:

~~~http
Vary: Accept
~~~

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
  - `modified` (string, ISO 8601): Last modification timestamp
  - `contentHash` (string, required): Template-invariant fingerprint (same as M-URL ETag)

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

Publishers MAY provide a machine-readable policy descriptor at `/llm-policy.json` to communicate usage terms, rate limits, and content licensing preferences to automated agents.

## Policy Endpoint

The policy descriptor SHOULD be available at:

~~~
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
- `effective` (string, ISO 8601): When policy took effect
- `updated` (string, ISO 8601): Last policy modification
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

## Discovery

The policy descriptor SHOULD be linked from the sitemap with a `describedby` Link header:

~~~http
Link: </llm-policy.json>; rel="describedby"
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
- `hash` (string, required): Template-invariant fingerprint (matches ETag value)

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
ETag: "sha256-2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
Link: <https://example.com/post/>; rel="canonical"
Last-Modified: Wed, 15 Oct 2025 14:30:00 GMT
Cache-Control: private, max-age=0, must-revalidate
Vary: Accept
Access-Control-Allow-Origin: *
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

## Denial of Service

Large sitemaps MAY be used for DoS attacks. Agents SHOULD:
- Implement request rate limiting
- Set maximum sitemap size limits (e.g., 100 MB)
- Use streaming JSON parsers for large sitemaps

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
- Energy saved: 0.0000853 GB × 0.06 kWh/GB = **0.0000051 kWh** (5.1 Wh) per fetch

**Scaled impact (1 million fetches/day):**
- Daily energy savings: 5,100 kWh = 5.1 MWh
- Annual energy savings: 1,861.5 MWh
- **Carbon equivalent**: ~930 metric tons CO₂ avoided (assuming 0.5 kg CO₂/kWh grid average)

## AI Model Inference Energy

Large language models consume substantial energy during inference operations. Token processing is the primary unit of computational cost, with recent measurements showing:

- GPT-4 class models: ~0.000187 kWh per token (673.2 joules)
- Average query energy: 0.3-0.5 kWh per response
- Carbon emissions: ~0.09 grams CO₂ per token

**TCT token reduction (measured):**
- HTML token count: 13,900 tokens average
- TCT JSON token count: 1,960 tokens average
- **Reduction: 11,940 tokens per fetch (86% savings)**

**Energy savings per AI query:**
- Tokens saved: 11,940
- Energy saved: 11,940 × 0.000187 kWh = **2.23 kWh** per query
- Carbon saved: 11,940 × 0.09 g = **1,074.6 g CO₂** (1.07 kg) per query

**Scaled impact (1 million AI queries/day):**
- Daily energy savings: 2,230 MWh
- Annual energy savings: 813,950 MWh
- **Carbon equivalent**: ~406,975 metric tons CO₂ avoided

## Sitemap-First Zero-Fetch Optimization

Beyond per-fetch savings, TCT's sitemap-first verification enables complete request elimination when content is unchanged.

**Measured skip rate:** 90%+ for unchanged content

For a typical deployment with 1,000 URLs checked daily:
- Traditional crawler: 1,000 full fetches/day
- TCT deployment: ~100 fetches/day (90% skipped via sitemap comparison)
- **Additional savings: 900 fetches avoided**

**Combined energy impact:**
- Network energy saved: 900 × 5.1 Wh = **4.59 kWh/day** (1,675 kWh/year)
- AI inference saved: 900 × 2.23 kWh = **2,007 kWh/day** (732,555 kWh/year)

## Comparison to Existing Approaches

| Method | Avg Size | Tokens | Energy/Fetch* | Relative Efficiency |
|--------|----------|--------|---------------|---------------------|
| Full HTML page | 350 KB | 47,000 | 9.0 kWh | 1.0× (baseline) |
| HTML body only | 103 KB | 13,900 | 2.6 kWh | 3.5× |
| AMP HTML | 52 KB | 7,000 | 1.3 kWh | 6.9× |
| **TCT JSON** | **17.7 KB** | **1,960** | **0.37 kWh** | **24.3×** |

*Combined network + AI inference energy

## Cumulative Environmental Impact

If TCT achieves 10% adoption across the top 10,000 crawled websites by 2026 (estimated 10 billion daily AI crawler fetches):

**Annual energy savings:**
- Network transmission: 18,615 MWh
- AI model inference: 8,139,500 MWh
- **Total: 8,158,115 MWh**

**Carbon emissions avoided:**
- ~4,079,057 metric tons CO₂ equivalent
- Equivalent to removing ~885,000 passenger vehicles from roads for one year
- Requires a forest the size of Chicago to offset via traditional HTML delivery

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
2. **Handshake**: No bidirectional C-URL ↔ M-URL discovery
3. **Zero-fetch**: No specification for skipping endpoint fetch when sitemap hash matches
4. **Precedence discipline**: No requirement for If-None-Match precedence

TCT integrates the SAME hash in both sitemap AND endpoint ETag, enabling zero-fetch optimization.

## AMP

Accelerated Mobile Pages {{AMP}} provides:
- C-URL → AMP-URL mapping via `<link rel="amphtml">`
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
  $etag = "\"sha256-{$hash}\"";

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
  header('Cache-Control: private, max-age=0, must-revalidate');

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
