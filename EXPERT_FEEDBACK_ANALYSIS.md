# Expert Feedback Analysis & Required Changes

**Date:** 2025-10-26  
**Draft:** draft-jurkovikj-collab-tunnel-00.md  
**Status:** Analyzing expert recommendations

---

## Critical Issues Found

### 1. Missing BCP14 References ❌
- Line 95 references RFC2119 and RFC8174 but they are NOT in normative section
- **Action:** Add both RFCs to normative references

### 2. Incorrect Sitemap Reference ❌
- Line 72: XML Sitemaps {{?RFC6926}} is WRONG
- RFC 6926 is NOT about sitemaps
- **Action:** Replace with sitemaps.org reference

### 3. Cache-Control Outdated ❌
**Current:** Cache-Control: private, max-age=0, must-revalidate  
**Should be:** Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60, stale-if-error=86400

**Locations:**
- Line 273 (304 example)
- Line 285 (Cache-Control section)
- Line 542 (Complete response)
- Line 820 (WordPress example)

### 4. Vary Header Wrong ❌
**Current:** Vary: Accept  
**Should be:** Vary: Accept-Encoding

**Locations:**
- Line 274, 290, 543

### 5. Missing Weak ETag Guidance ❌
**Current:** Only shows strong ETags  
**Should:** Prefer weak ETags for semantic fingerprints

Example: W/sha256-abc123... instead of sha256-abc123...

### 6. No Requirements Section ❌
**Need:** Explicit section with MUST/SHOULD/MAY requirements

### 7. Missing Protocol Rules ❌
- HEAD must mirror GET headers
- Sitemap contentHash must equal M-URL ETag (sans quotes/W/)
- If-None-Match precedence over If-Modified-Since

---

## Summary: 15 Changes Required

**High Priority (9):**
1. Add RFC2119/RFC8174 to normative
2. Fix sitemap reference  
3. Add Requirements section
4. Update Cache-Control (4 locations)
5. Update Vary (3 locations)
6. Add weak ETag guidance
7. Add HEAD parity rule
8. Add sitemap parity rule
9. Update WordPress example

**Medium Priority (5):**
10. Move normalization to Appendix
11. Add Operational Considerations
12. Clarify policy as informative
13. Enhance Security section
14. Add sitemap ETag examples

**Low Priority (1):**
15. Add test vectors

---

## Alignment Check

### Current Spec vs Live Implementation

| Feature | Spec | Live | Status |
|---------|------|------|--------|
| Cache-Control | private, max-age=0 | max-age=0 + SWR + SIE | ❌ Outdated |
| Vary | Accept | Accept-Encoding | ❌ Wrong |
| ETag | Strong | Weak | ❌ Different |
| Requirements | None | Clear MUST/SHOULD | ❌ Missing |

**Conclusion:** Spec is behind live implementation\!

---

## Next Steps

1. Review this analysis with you
2. Decide: Accept expert's PR offer OR implement ourselves
3. Make all changes to align spec with live behavior
4. Verify all 15 items addressed
5. Ready for IETF submission

**Estimated time:** 2-3 hours for all changes
