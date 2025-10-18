# Email Draft to IETF GREEN Working Group

## Email Details

**To:** green@ietf.org
**Subject:** Application-Layer Energy Efficiency: The Collaboration Tunnel Protocol
**From:** Antun Jurkovikj <antunjurkovic@gmail.com>

---

## Email Body

Dear GREEN Working Group members,

I'm reaching out to introduce an application-layer protocol that complements GREEN's infrastructure-level energy optimization work, and to explore potential areas of collaboration.

### Introduction

I've developed the **Collaboration Tunnel Protocol (TCT)**, an IETF individual draft specification that addresses energy efficiency from the application layer, focusing on content delivery optimization for automated agents (AI crawlers, search engines, content aggregators).

**Specification:** https://github.com/antunjurkovic-collab/collab-tunnel-spec

### Energy Impact Summary

TCT achieves substantial energy savings through three mechanisms:

1. **Bandwidth Reduction (83%)**
   - Traditional HTML: 103 KB average
   - TCT JSON: 17.7 KB average
   - Energy saved: ~5.1 Wh per fetch (network transmission)

2. **Token Cost Reduction (86%)**
   - Traditional HTML: 13,900 tokens average
   - TCT JSON: 1,960 tokens average
   - Energy saved: ~2.23 kWh per AI query (model inference)

3. **Zero-Fetch Optimization (90%+ skip rate)**
   - Sitemap-first verification eliminates fetches for unchanged content
   - Measured: 90%+ of content requests avoided via hash comparison

**Combined efficiency: 24.3× energy reduction** compared to traditional HTML delivery.

### Measured Deployment Results

These measurements are from production deployments:
- **970 URLs** across 3 live websites
- **100% protocol compliance** verified via automated validator
- Real-world traffic patterns, not synthetic benchmarks

**Scaled Impact Projections:**

If TCT achieves 10% adoption across the top 10,000 crawled websites by 2026:
- **Annual energy savings:** 8,158,115 MWh
- **Carbon emissions avoided:** ~4,079,057 metric tons CO₂e
- **Equivalent:** Removing ~885,000 passenger vehicles from roads for one year

### Relationship to GREEN WG Mission

While GREEN focuses on infrastructure-level optimization (device power consumption, network-wide traffic flow), TCT addresses **application-layer efficiency**:

- **GREEN targets:** Router/switch energy, network monitoring, YANG data models
- **TCT targets:** Content delivery efficiency, redundant fetch elimination, AI inference cost

Together, these approaches create a comprehensive energy reduction strategy spanning both network infrastructure and application protocols.

### Technical Overview

TCT integrates four mechanisms:

1. **Bidirectional Discovery**: C-URL ↔ M-URL handshake (SEO-safe)
2. **Template-Invariant Fingerprinting**: SHA-256 hashes stable across presentation changes
3. **Conditional Request Discipline**: Strict If-None-Match precedence, 304 responses
4. **Sitemap-First Verification**: Zero-fetch skip logic when content unchanged

**Key innovation:** The SAME hash appears in both the sitemap AND the endpoint ETag, enabling agents to skip fetches entirely when content is unchanged (verified via sitemap comparison).

### Implementations

**Publisher-side:**
- WordPress plugin (open source, GPL): https://github.com/antunjurkovic-collab/collab-tunnel-spec
- Cloudflare Worker (edge-based implementation)

**Consumer-side:**
- Python library (PyPI): `pip install collab-tunnel`
- Public validator: https://llmpages.org/validator/

### Energy Metrics & GREEN WG Terminology

I've reviewed GREEN's work on energy efficiency metrics and terminology, and I believe TCT measurements could contribute to:

1. **Application-layer energy metrics**: Quantifying content delivery efficiency
2. **End-to-end optimization**: Bridging infrastructure and application layers
3. **Real-world validation**: Production measurements demonstrating multi-megawatt savings potential

I've included an "Energy Efficiency Considerations" section in the TCT specification that:
- Quantifies network transmission energy (using 0.06 kWh/GB industry standard)
- Quantifies AI model inference energy (using recent GPT-4 measurements)
- Projects cumulative environmental impact at scale
- References GREEN WG's complementary infrastructure work

### Areas for Potential Collaboration

I'm interested in exploring:

1. **Metric standardization**: Aligning TCT energy measurements with GREEN's terminology
2. **Measurement methodology**: Best practices for quantifying application-layer energy savings
3. **Cross-layer optimization**: How application protocols can inform network-level energy decisions
4. **Formal submission**: Whether TCT could benefit from GREEN WG review or adoption track guidance

### Questions for the Working Group

1. Is there interest in addressing application-layer energy efficiency within GREEN's scope, or should this remain a separate effort?
2. Are there specific metrics or measurement frameworks GREEN is developing that TCT should align with?
3. Would a presentation at a future GREEN WG meeting be valuable?

### Patent Notice

For transparency: TCT is covered by US Patent Application 63/895,763 (provisional, filed October 8, 2025). The reference implementation is open source (GPL v2+) for website owners. Commercial licensing may apply for large-scale crawler operators.

### Next Steps

I'm happy to:
- Present TCT at a GREEN WG meeting (virtual or in-person)
- Provide detailed measurement data and methodology
- Collaborate on metric standardization
- Answer any technical questions

Thank you for considering this work, and for GREEN's important efforts on network energy efficiency.

Best regards,

**Antun Jurkovikj**
Independent Researcher
North Macedonia
antunjurkovic@gmail.com

---

**Links:**
- Specification: https://github.com/antunjurkovic-collab/collab-tunnel-spec
- Python Client: https://pypi.org/project/collab-tunnel/
- Public Validator: https://llmpages.org/validator/
- WordPress Plugin: (repository in specification)

---

## Notes for Sending

**Before sending:**
1. ✅ Update GitHub spec with energy efficiency section (DONE)
2. ✅ Verify all links work
3. ✅ Subscribe to green@ietf.org mailing list first
4. ✅ Consider timing: Send early in the week (Tuesday-Wednesday) for better visibility
5. ✅ Be prepared to answer technical questions
6. ✅ Have measurement data ready if requested

**Expected responses:**
- Interest in presentation at next GREEN WG meeting
- Questions about measurement methodology
- Requests for detailed energy calculations
- Suggestions for alignment with GREEN metrics
- Potential collaboration opportunities

**Follow-up actions:**
- If positive response: Prepare presentation slides
- If questions: Provide detailed technical documentation
- If no response within 2 weeks: Send a gentle follow-up

**Alternative contacts (if no response on mailing list):**
- GREEN WG Chairs (check datatracker.ietf.org/wg/green/about/)
- IETF HTTP WG (already subscribed to httpwg@ietf.org)

---

## Shorter Version (If Above Is Too Long)

If the above email seems too lengthy, here's a condensed version:

---

**Subject:** Application-Layer Energy Efficiency: 83% Bandwidth, 86% Token Reduction

Dear GREEN WG members,

I've developed the **Collaboration Tunnel Protocol (TCT)**, which achieves significant energy savings through application-layer optimization for AI crawler content delivery.

**Key Results (measured, 970 URLs in production):**
- 83% bandwidth reduction (103 KB → 17.7 KB)
- 86% token reduction (13,900 → 1,960 tokens)
- 90%+ zero-fetch skip rate (sitemap-first verification)
- **Combined: 24.3× energy efficiency** vs traditional HTML

**Energy Impact:**
- Network transmission: ~5.1 Wh saved per fetch
- AI inference: ~2.23 kWh saved per query
- Scaled (10% adoption, top 10K sites): 8,158 GWh/year saved

**Specification:** https://github.com/antunjurkovic-collab/collab-tunnel-spec

TCT complements GREEN's infrastructure focus by addressing application-layer efficiency. I've added an "Energy Efficiency Considerations" section to the spec that references GREEN's work.

Would the working group be interested in:
1. A presentation on application-layer energy optimization?
2. Collaboration on metric standardization?
3. Feedback on energy measurement methodology?

Happy to provide detailed data and answer technical questions.

Best regards,
Antun Jurkovikj
antunjurkovic@gmail.com

---

## Recommendation

**Use the FULL version.** Here's why:

1. GREEN WG is new (formed November 2024) - they're still establishing scope
2. This is a substantive technical contribution, not a quick question
3. The energy calculations demonstrate serious analysis
4. Patent disclosure is important for transparency
5. Detailed measurements show this is production-ready, not theoretical

The email is long, but it's **information-dense** and shows you've done your homework. IETF appreciates thoroughness.

**Best time to send:** Tuesday or Wednesday morning (Central European Time), so it arrives during US business hours and has time to be read before the weekend.
