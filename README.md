# Trivago Filter Architecture & Result States: A Technical Spec
An architectural teardown and interaction specification analyzing systematic logic bugs within Trivago's core hotel search flows, detailing structural solutions for unified input sync, constraint relaxation, and fallback states.

## Core Architectural Disconnect
Rigid discrete selection systems and natural-language "Smart Search" inputs currently operate in isolated silos, causing systemic zero-state friction, uncommunicated constraint relaxation, and loss of user conversion trust.

---

## Technical Specifications Matrix

### Module 1: Unified Query Input & Bidirectional Sync
* **The System Bug:** Free-text natural-language criteria (e.g., `"near Hamburg Hauptbahnhof"`) have no state parity with manual checkbox criteria.
* **The Specification:** Implement a bidirectional compilation parser. Manual checkbox items dynamically inject text parameters into the query string, while text token entries render matching filter chips instantly.
* **Edge-Case Exception Handling:**
  * **Text Override:** Manual input changes flag the text query as primary state. **System Notice:** `"Showing results based on your search text below."`
  * **Divergent Checkbox:** Suppressing a keyword while maintaining a checked filter exposes a manual re-sync CTA. **System Notice:** `"'Breakfast included' is still selected — [Add back to search text]"`
  * **Unparsed Attributes:** Catch and flag unindexed text strings transparently. **System Notice:** `"We've included 'near the harbour' as a search term — results may not be limited to this."`

### Module 2: Threshold Quality Floors
* **The UI Clutter Bug:** Forcing users through 6 separate star checkboxes ($0–1, 2, 3, 4, 5$) violates cognitive load limits.
* **The Specification:** Replace discrete value checkboxes with threshold quality floors (`3★+`, `4★+`, and `5★`) matching baseline user expectations. Route sub-3 star traffic into defensive price filter logic.

### Module 3: Tiered Result Fallthrough & Spatial Variance
* **The Silent Relaxation Bug:** When strict boundaries (e.g., `"within 2km of airport"`) are unfulfilled, non-compliant listings (e.g., $6.9\text{ km}$ away) are blended silently into exact results without warning.
* **The Specification:** Eradicate flattened result sets via a 3-Tier mathematical hierarchy:
  1. **Tier 1 (Exact Match):** $100\%$ parameter compliance (e.g., $\le 2.0\text{ km}$).
  2. **Tier 2 (Near Miss):** Strict delta boundaries ($\le +15\%$ spatial variance, e.g., $2.1\text{ km} - 2.5\text{ km}$; $\pm 0.2\text{★}$ rating variance).
  3. **Tier 3 (Related Result):** Triggered only when Tier 1 & 2 inventories are exhausted, isolated by explicit system division tags.
* **Delta Variance Microcopy Controls:**
  * **Section Tag:** `"We ran out of exact matches — here's what else fits."`
  * **Card Distance Badge:** `"3.2 km from the airport — 1.2 km past your limit."`
  * **Rating Variance Whisker:** `"3.9 ★ — just under your 4 cutoff."`

### Module 4: N-1 Proactive Zero-State Recovery
* **The Brittle Flow Bug:** Over-constraining a query drops a dead-end screen with no inventory fallback paths.
* **The Specification:** Automatically isolate and populate inventory satisfying $N-1$ constraints. Highlight missing elements using high-contrast delta flags (e.g., `⚠️ No on-site tennis court • Public court 800m away`) paired with an explicit parameters adjustment CTA.
