# Expert Feedback Summary - Quick Reference

**Date:** 2025-10-26  
**Draft:** draft-jurkovikj-collab-tunnel-00.md

---

## 15 Required Changes

### Critical (Must Fix Before Submission)

1. **Add RFC2119 + RFC8174** to normative references (BCP14)
2. **Fix sitemap reference** - RFC6926 is WRONG, use sitemaps.org
3. **Add Requirements section** - Clear MUST/SHOULD/MAY list
4. **Update Cache-Control** (4 locations) - Add stale-while-revalidate + stale-if-error
5. **Update Vary** (3 locations) - Change Accept to Accept-Encoding
6. **Add weak ETag guidance** - Prefer W/"sha256-..." for semantic fingerprints
7. **Add HEAD parity rule** - HEAD must mirror GET headers
8. **Add sitemap parity rule** - contentHash = ETag (sans quotes/W/)
9. **Update WordPress example** - Match live implementation

### Important (Should Add)

10. **Add HEAD support section** - Requirement for HEAD requests
11. **Add sitemap ETag example** - Show conditional sitemap fetch
12. **Clarify If-None-Match precedence** - More explicit per RFC 9110
13. **Enhance Security section** - Add DoS, injection, privacy notes
14. **Add operational notes** - Cache pitfalls, CDN compatibility

### Nice to Have

15. **Move normalization to appendix** - Make it informative

---

## Key Alignments Needed

### Cache-Control (Current vs Recommended)

**Current:**
```
Cache-Control: private, max-age=0, must-revalidate
```

**Should be:**
```
Cache-Control: max-age=0, must-revalidate, stale-while-revalidate=60, stale-if-error=86400
```

**Why:** Remove private (M-URLs cacheable), add graceful degradation

---

### Vary (Current vs Recommended)

**Current:**
```
Vary: Accept
```

**Should be:**
```
Vary: Accept-Encoding
```

**Why:** Content varies by compression, not content-type

---

### ETag (Current vs Recommended)

**Current:**
```
ETag: "sha256-abc..."
```
(Strong)

**Should be:**
```
ETag: W/"sha256-abc..."
```
(Weak)

**Why:** Semantic fingerprint = weak validator

---

## Specification vs Live Implementation

| Feature | Current Spec | Live Code | Status |
|---------|-------------|-----------|--------|
| Cache-Control | private, max-age=0 | max-age=0 + SWR + SIE | OUTDATED |
| Vary | Accept | Accept-Encoding | WRONG |
| ETag | Strong only | Weak | INCOMPLETE |
| Requirements | Missing | Implemented | MISSING |

**Conclusion:** Specification is behind live WordPress/Worker implementations!

---

## Recommended Action

Two options:

1. **Accept expert's PR offer** - They draft requirements + update examples
2. **Implement ourselves** - Use checklist to make all 15 changes

**Expert's offer:**
> "If you want, I can draft the small 'Requirements' subsection and update 
> the example headers/code blocks in the draft to exactly match your stable 
> behavior and submit as a PR against collab-tunnel-spec."

---

## Files Created

1. **EXPERT_FEEDBACK_ANALYSIS.md** - Detailed analysis
2. **EXPERT_FEEDBACK_SUMMARY.md** - This quick reference

---

## Next Steps

1. Review analysis
2. Decide on expert PR vs self-implementation
3. Make all changes
4. Verify alignment with live code
5. Ready for IETF submission

**Est. time:** 2-3 hours for complete implementation
