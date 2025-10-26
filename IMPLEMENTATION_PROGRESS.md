# Specification Update Progress

**Date:** 2025-10-26  
**Target:** draft-jurkovikj-collab-tunnel-00.md  
**Expert Feedback:** Implementing 15 recommended changes

---

## Progress: 6/12 Tasks Complete (50%)

### ✅ Completed (6)

1. **RFC2119 + RFC8174 References** - Added BCP14 RFCs to normative section
2. **XML Sitemap Reference** - Fixed RFC6926 error, added sitemaps.org reference  
3. **Requirements Section** - Added comprehensive MUST/SHOULD/MAY section (72 lines)
4. **Cache-Control Updates** - Updated all 4 locations with SWR + SIE directives
5. **Vary Header Updates** - Changed Accept to Accept-Encoding (3 locations)
6. **WordPress Example** - Updated with correct Cache-Control and Vary

### 🔄 In Progress (1)

7. **Weak ETag Guidance** - Adding W/"sha256-..." examples and guidance

### ⏳ Pending (5)

8. **HEAD Support Section** - Document HEAD parity requirement
9. **Sitemap Parity Rule** - contentHash = ETag (sans quotes/W/)
10. **Sitemap ETag Example** - Show conditional sitemap fetch
11. **If-None-Match Precedence** - Clarify RFC 9110 precedence
12. **Security Enhancements** - Add DoS, injection, privacy notes

---

## Changes Made So Far

### Lines Added: ~130 lines
- Requirements section: 72 lines
- Cache-Control explanations: 10 lines  
- Vary explanations: 5 lines
- Reference additions: 10 lines

### Lines Modified: ~15 locations
- 4× Cache-Control headers
- 3× Vary headers
- 1× XML Sitemap reference
- Multiple section rewrites

---

## Time Spent

- Start: ~30 minutes ago
- Tasks 1-6: Complete
- Remaining: ~45-60 minutes estimated

---

## Next Steps

1. Complete weak ETag guidance
2. Add HEAD support section
3. Add sitemap parity rule
4. Add examples
5. Final review
6. Commit and push

**Total time estimate:** 2-3 hours for all changes
**Current pace:** On track!
