# IETF GREEN Working Group - Status Update (October 2025)

## Current Status (As of October 18, 2025)

### Leadership
- **Chairs:** Diego Lopez and Robert Wilton
- **Area Director:** Mahesh Jethanandani
- **Area:** Operations and Management (OPS)

### Formation Timeline
- **First BoF:** November 2024 (IETF 121, Dublin)
- **Working Group formed:** November 2024
- **Most recent meeting:** IETF 123, July 21-22, 2025 (Madrid, Spain)
- **Next meeting:** IETF 124, November 1-7, 2025 (Montreal, Canada)

**Key takeaway:** GREEN WG is **less than 1 year old** - still establishing scope and adopting foundational documents.

---

## Charter & Mission

### Focus Areas

The GREEN Working Group is chartered to:

1. **Explore use cases** for energy-efficient networking
2. **Derive requirements** for energy efficiency metrics
3. **Provide solutions** for identifying and characterizing energy efficiency across network infrastructure

### Three Main Deliverables

1. **Terminology Document (Informational)**
   - Common terminology and metric definitions
   - Categorization of metrics at component, device, and network levels
   - **Status:** draft-bclp-green-terminology (strong support: 36 yes/0 no/7 abstain)

2. **YANG Data Models (Standards Track)**
   - Component-level, device-level, and network-level models
   - Energy usage monitoring and consumption management
   - **Status:** Working on parallel development (voted 26 yes/0 no)

3. **Architecture Document (Informational)**
   - Architectural components for managing Energy Efficient Networks
   - Incremental deployment considerations
   - **Status:** draft-belmq-green-framework (adoption premature, but relevant)

### Milestones (from Charter)

- **June 2025:** Adopt standardized terminology ✅ (adopted draft-bclp-green-terminology)
- **December 2025:** Adopt YANG models for energy usage management ⏳ (in progress)
- **June 2026:** Adopt framework for energy-efficient monitoring ⏳ (upcoming)

---

## Recent Activity (IETF 123, July 2025)

### Documents Discussed

**Adopted/Near Adoption:**
- ✅ **draft-bclp-green-terminology** - Strong consensus (36 yes/0 no/7 abstain)
- 🟡 **draft-stephan-green-use-cases** - Adoption poll started (31 yes/25 no on readership)
- 🔴 **draft-belmq-green-framework** - Adoption premature (only 17/49 read it)

**Under Consideration (Outside Current Scope):**
- **draft-sofia-green-energy-aware-diffserv** - 33 yes/1 no on relevance, but outside charter
- **draft-jadoon-green-isac-utilization** - Augmenting base model for ISAC
- **draft-petra-green-api** - Radio-specific APIs (scope questioned)
- **draft-chen-green-transport-network-ucs** - Should merge with use cases doc

### Key Decisions

1. **Parallel development:** Device, component, and network-level models (26 yes/0 no)
2. **Usage guidance:** Develop guidance documentation (15 yes/2 no/3 abstain)
3. **Terminology adoption:** Moving forward with draft-bclp-green-terminology

### Liaisons & Collaboration

GREEN WG is coordinating with:
- **IETF WGs:** IVY, OPSAWG, NETMOD, TVR
- **External:** W3C, ITU-T, ETSI, 3GPP

---

## Current Scope & Focus

### What GREEN WG IS Working On

✅ **Infrastructure-level energy optimization:**
- Router and switch energy consumption
- Network device power monitoring (YANG models)
- Network-wide traffic flow optimization
- Measurement and reporting interfaces
- Component, device, and network-level metrics

### What GREEN WG Is NOT Yet Addressing

❌ **Application-layer energy efficiency:**
- Content delivery optimization
- Application protocol efficiency
- Fetch elimination strategies
- Token/computational cost reduction

**This is where TCT fits!** 🎯

---

## Strategic Opportunity for TCT

### Perfect Timing

1. **Young WG (< 1 year old):** Still defining scope, open to new ideas
2. **Charter gap:** No application-layer energy work yet
3. **Recent meeting (July 2025):** Active discussions, momentum building
4. **Next meeting (Nov 2025):** Opportunity to present before IETF 124

### Complementary Positioning

| GREEN WG Focus | TCT Focus | Relationship |
|----------------|-----------|--------------|
| Infrastructure (routers, switches) | Application layer (content delivery) | Complementary |
| YANG data models | Protocol specification | Different layers |
| Network device monitoring | End-to-end optimization | Additive |
| Terminology & metrics | Real-world measurements | TCT provides data |

### Value Proposition to GREEN WG

1. **Fills charter gap:** Application-layer energy efficiency (not currently addressed)
2. **Real measurements:** 970 URLs, production data (not theoretical)
3. **Massive scale potential:** 8,158 GWh/year savings (bigger than most infrastructure optimizations)
4. **Standards alignment:** TCT can use GREEN's terminology framework
5. **Cross-layer synergy:** Application + infrastructure = comprehensive solution

---

## Recommended Approach for Outreach

### Email Timing

**Best time:** Late October/Early November 2025

**Why:**
- After IETF 123 (July), WG has had time to digest
- Before IETF 124 (Nov 1-7), so there's opportunity to present
- Aligns with December 2025 milestone (YANG model adoption)

### Email Strategy

**Subject line:** Application-Layer Energy Efficiency: The Collaboration Tunnel Protocol

**Key talking points:**
1. **Acknowledge their work:** "I've been following GREEN's progress on infrastructure-level energy optimization..."
2. **Position as complementary:** "TCT addresses application-layer efficiency, which complements GREEN's infrastructure focus..."
3. **Cite recent meeting:** "I noticed at IETF 123 the WG is working on terminology and use cases. TCT has production measurements that could inform this work..."
4. **Offer specific value:** "TCT measurements show 24.3× energy efficiency - these real-world numbers could help validate GREEN's frameworks..."
5. **Propose presentation:** "Would it be valuable to present TCT at IETF 124 in Montreal?"

### Engagement Levels

**Option 1 (Conservative):** Email mailing list, gauge interest
**Option 2 (Moderate):** Email + offer to present at IETF 124 side meeting
**Option 3 (Aggressive):** Submit individual draft referencing GREEN + request WG adoption

**Recommendation:** Start with Option 1, escalate based on response.

---

## Key Contacts

### Mailing List
- **List:** green@ietf.org
- **Subscribe:** https://mailman3.ietf.org/mailman3/lists/green.ietf.org/
- **Archive:** https://mailarchive.ietf.org/arch/browse/green/

### Chairs
- **Diego Lopez** (likely diego.r.lopez@telefonica.com or similar)
- **Robert Wilton** (likely rwilton@cisco.com or similar)

*(Check datatracker.ietf.org/group/green/about/ for exact contacts)*

### Area Director
- **Mahesh Jethanandani** (likely mjethanandani@gmail.com or similar)

---

## Documents to Reference in Email

### GREEN WG Documents (show you've done homework)

1. **draft-bclp-green-terminology** - Being adopted
2. **draft-stephan-green-use-cases** - Under discussion
3. **draft-belmq-green-framework** - Framework document

### TCT Documents (what you're bringing)

1. **Specification:** https://github.com/antunjurkovic-collab/collab-tunnel-spec
2. **Energy section:** Section added to spec (lines 431-553)
3. **Python library:** https://pypi.org/project/collab-tunnel/
4. **Public validator:** https://llmpages.org/validator/

---

## Potential Questions/Objections (Be Prepared)

### Q1: "Is application-layer energy efficiency in GREEN's charter?"

**Answer:** Not explicitly, but charter says "energy efficiency across the network" - this could include cross-layer optimization. TCT complements infrastructure work by reducing traffic volume, which directly impacts device energy consumption.

### Q2: "Shouldn't this go to HTTP WG instead?"

**Answer:** TCT is relevant to both. HTTP WG focuses on protocol mechanics, GREEN focuses on energy impact. I'm already engaged with HTTP WG (httpwg@ietf.org subscriber) and can coordinate between groups.

### Q3: "Your measurements seem very optimistic. How can we trust them?"

**Answer:**
- 970 URLs deployed in production (not lab)
- 100% protocol compliance verified by public validator
- Conservative energy estimates (0.06 kWh/GB is industry standard, not best-case)
- Methodology fully documented in spec
- Open to third-party validation

### Q4: "There's a patent. Doesn't this conflict with IETF IPR policy?"

**Answer:**
- Provisional patent (placeholder, not granted)
- Reference implementation is open source (GPL v2+)
- Large-scale commercial users may need license (similar to many IETF standards)
- IETF IPR disclosure will be filed if spec advances
- This is disclosed upfront for transparency

### Q5: "Why should we care about AI inference energy?"

**Answer:**
- AI crawlers now dominate web traffic (2025 reality)
- GPT-4 alone: 391-463 TWh/year projected
- Data centers powering AI are 2-3% of global electricity
- This is a massive and growing energy consumer
- TCT addresses both network AND computational energy

---

## Success Metrics

### Short-term (1-2 months)
- ✅ Email sent to green@ietf.org
- ✅ At least 5 responses from WG members
- ✅ Acknowledgment from chairs

### Medium-term (3-6 months)
- 🎯 Present at IETF 124 (Nov 2025) or IETF 125 (Mar 2026)
- 🎯 TCT referenced in GREEN WG use cases document
- 🎯 Collaboration on metric standardization

### Long-term (6-12 months)
- 🎯 Submit TCT as individual draft with GREEN WG coordination
- 🎯 Cross-reference between TCT spec and GREEN documents
- 🎯 Joint work on application-layer energy metrics

---

## Updated Email Recommendations

### Add These Elements Based on Fresh Research

1. **Acknowledge IETF 123:**
   > "I've been following GREEN's recent progress, including the productive discussions at IETF 123 in Madrid. I noticed the WG is focusing on terminology (draft-bclp-green-terminology) and use cases (draft-stephan-green-use-cases)..."

2. **Reference terminology work:**
   > "TCT measurements could provide real-world validation for the terminology framework being developed. For example, we measure energy savings at both the network transmission layer (5.1 Wh per fetch) and application processing layer (2.23 kWh per AI query)..."

3. **Offer specific contribution:**
   > "I noticed there was discussion about whether energy-aware DiffServ (draft-sofia-green-energy-aware-diffserv) fits within charter. TCT takes a different approach - reducing traffic volume rather than prioritizing it - which might align better with GREEN's current scope..."

4. **Propose IETF 124 presentation:**
   > "Would it be valuable to present TCT's application-layer energy efficiency approach at IETF 124 in Montreal (November 1-7)? I could provide a 10-15 minute overview with production measurements..."

---

## Bottom Line

✅ **GREEN WG is active and growing** (< 1 year old)
✅ **Current focus is infrastructure**, not application layer
✅ **Perfect timing** to introduce complementary work
✅ **Next meeting is Nov 1-7** (IETF 124, Montreal)
✅ **Strong opportunity** to position TCT as cross-layer solution

**Action:** Send email **this week** (October 21-23, 2025) to land before IETF 124 planning.

---

**Document prepared:** October 18, 2025
**Sources:** IETF Datatracker, IETF 123 minutes, GREEN WG charter, mailing list archives
