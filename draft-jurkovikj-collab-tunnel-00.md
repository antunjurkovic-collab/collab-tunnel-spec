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
- `items` (array): List of URL pairs
  - `cUrl` (string, required): Canonical URL
  - `mUrl` (string, required): Machine URL
  - `modified` (string, ISO 8601): Last modification timestamp
  - `contentHash` (string, required): Template-invariant fingerprint (same as M-URL ETag)

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
  "canonical_url": "https://example.com/post/",
  "title": "Article Title",
  "content": "Core article content...",
  "hash": "2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae"
}
~~~

**Optional fields:**

~~~json
{
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
