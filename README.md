# Collaboration Tunnel Protocol Specification

**Status:** Individual Draft (Not yet submitted to IETF)
**Version:** draft-jurkovikj-collab-tunnel-00
**Date:** October 2025
**Author:** Antun Jurkovikj

## Abstract

The Collaboration Tunnel Protocol enables efficient, verifiable content delivery between web publishers and automated agents, achieving up to 90% bandwidth reduction through bidirectional URL discovery, template-invariant content fingerprinting, sitemap-first verification, and strict conditional request discipline.

## Specification

📄 **[View Specification](./draft-jurkovikj-collab-tunnel-00.md)**

## Implementation Status

### Publisher Implementations
- **[WordPress Plugin](https://github.com/antunjurkovic-collab/trusted-collab-tunnel)** - 491 URLs across 4 production sites, 100% RFC compliance
- **[Cloudflare Worker](https://github.com/antunjurkovic-collab/trusted-collab-worker)** - Edge-based header normalization

### Consumer Implementations
- **[Python Client Library](https://github.com/antunjurkovic-collab/collab-tunnel-python)** - Available on PyPI: `pip install collab-tunnel`

### Live Demonstrations
Test the protocol on these production sites:
- https://bestdemotivationalposters.com/llm-sitemap.json (200 URLs)
- https://wellbeing-support.com/llm-sitemap.json (200 URLs)
- https://omacedonii.com/llm-sitemap.json (65 URLs)
- https://llmpages.org/llm-sitemap.json (26 URLs)

### Validator Tool
- https://llmpages.org/validator/

## Measured Results

Based on 491 URLs across 4 production sites:
- **Bandwidth savings:** 83% vs HTML-only crawling
  (103 KB average HTML → 17.7 KB average JSON)
- **Token savings:** 86% for AI processing
  (13,900 tokens → 1,960 tokens)
- **Skip rate:** 90%+ for unchanged content using sitemap-first verification
- **304 response rate:** 100% after initial fetch (ETag validation working)
- **RFC compliance:** 100% of endpoints (RFC 9110, 9111, 8288)

## Protocol Overview

The Collaboration Tunnel Protocol consists of four coordinated mechanisms:

### 1. Bidirectional Discovery
- C-URL (HTML page) → M-URL (JSON endpoint) via `<link rel="alternate">`
- M-URL → C-URL via `Link: <C-URL>; rel="canonical"` header
- Prevents SEO conflicts

### 2. Template-Invariant Fingerprinting
- Content normalized through 6-step pipeline (operates on JSON `content` field, not HTML):
  1. Decode HTML entities (&amp; → &, &#x2014; → —)
  2. Apply Unicode NFKC normalization
  3. Apply Unicode case folding (locale-independent lowercase)
  4. Remove control characters (Unicode category Cc), except TAB, LF, CR
  5. Collapse ASCII whitespace (SPACE, TAB, LF, CR) to single space
  6. Trim leading/trailing whitespace
- Then compute SHA-256 hash over normalized UTF-8 bytes
- Weak ETag format: `W/"sha256-..."`
- Stable across theme/template changes

### 3. Conditional Request Discipline
- If-None-Match takes precedence
- 304 Not Modified for unchanged content
- Strict caching requirements

### 4. Sitemap-First Verification
- JSON sitemap lists (cUrl, mUrl, contentHash)
- Skip fetch if hash unchanged (zero-fetch optimization)
- 90%+ skip rate in production

## Response Format

### M-URL JSON Payload
```json
{
  "profile": "tct-1",
  "llm_url": "https://example.com/post/llm/",
  "canonical_url": "https://example.com/post/",
  "hash": "sha256-e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "title": "Article Title",
  "content": { "text": "Article content..." },
  "modified": "2025-10-23T18:00:00Z"
}
```

### M-URL HTTP Headers
```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
Link: <https://example.com/post/>; rel="canonical"
ETag: W/"sha256-e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60, stale-if-error=86400
Vary: Accept-Encoding
```

### Sitemap JSON Format
```json
{
  "version": 1,
  "profile": "tct-1",
  "items": [
    {
      "cUrl": "https://example.com/post/",
      "mUrl": "https://example.com/post/llm/",
      "modified": "2025-10-23T18:00:00Z",
      "contentHash": "sha256-e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    }
  ]
}
```

**Profile Field**: The `"profile": "tct-1"` field enables protocol versioning. Future versions (e.g., `tct-2`) can introduce new fields while maintaining backward compatibility.

## Quick Start

### For Publishers (Website Owners)

**WordPress:**
```bash
# Install plugin
wp plugin install trusted-collab-tunnel --activate

# Or download from: https://wordpress.org/plugins/trusted-collab-tunnel/
```

**Cloudflare Worker:**
```javascript
// Deploy worker from: https://github.com/antunjurkovic-collab/trusted-collab-worker
```

### For Consumers (AI Crawlers)

**Python:**
```bash
pip install collab-tunnel
```

```python
from collab_tunnel import CollabTunnelCrawler

crawler = CollabTunnelCrawler(user_agent="MyBot/1.0")
sitemap = crawler.fetch_sitemap("https://example.com/llm-sitemap.json")

for item in sitemap.items:
    if crawler.should_fetch(item):  # Zero-fetch optimization
        content = crawler.fetch_content(item['mUrl'])
        print(content['title'])

stats = crawler.get_stats()
print(f"Bandwidth saved: {stats['savings_percentage']}%")
```

## Feedback

This is an **individual draft** seeking community feedback before potential IETF submission.

- **Issues:** https://github.com/antunjurkovic-collab/collab-tunnel-spec/issues
- **Email:** antunjurkovic@gmail.com
- **IETF HTTP Working Group:** Joined mailing list, observing (not yet formally presenting)

## Patent Notice

This specification implements methods covered by **US Patent Application 63/895,763** (filed October 8, 2025, status: Patent Pending).

**For Implementers:**
- ✅ Website owners: FREE to implement under GPL for your own sites
- ✅ Individual developers: FREE to build clients/libraries
- ⚠️ Large-scale commercial services: May require licensing for implementations serving >10,000 URLs/month
- Contact: licensing@llmpages.org

The specification is published to encourage adoption and enable interoperability. Patent rights are reserved to protect against large-scale commercial exploitation without appropriate licensing.

## License

This specification document is provided under the [IETF Trust Legal Provisions](https://trustee.ietf.org/documents/trust-legal-provisions/) for IETF Documents.

Implementations of the protocol are subject to the patent notice above.

## Links

- **Website:** https://llmpages.org
- **Validator:** https://llmpages.org/validator/
- **Documentation:** https://llmpages.org/docs/
- **PyPI Package:** https://pypi.org/project/collab-tunnel/
- **WordPress Plugin:** https://wordpress.org/plugins/trusted-collab-tunnel/

## Contributing

Contributions, suggestions, and feedback are welcome!

Please open an issue or submit a pull request for:
- Specification improvements
- Clarifications
- Additional examples
- Error corrections

## Acknowledgments

Thanks to:
- WordPress and Cloudflare communities for implementation platforms
- IETF HTTP Working Group for foundational HTTP standards (RFC 9110, 9111, 8288, 6596)
- Open Archives Initiative for ResourceSync inspiration
- Early adopters testing the protocol on production sites
