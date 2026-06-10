# Iteration log

Each recursive `/loop` planning pass records here what it changed and why. Newest first.
Next passes should pick an item from PLAN §10 (open questions), resolve it with research, deepen
the relevant section, and log it below.

## v3 — 2026-06-10 — daily-scrape cadence, winnability filter, ≥2-buyer outreach trigger
- Added the explicit **daily operating loop** to `PLAN.md` §5: scrape gname daily → keep only
  *winnable* + single-buyer-clear candidates → find buyers → outreach only when ≥2 reliable
  independent buyers exist → commitment → bid → transfer. Cheapest filters first so expensive
  prospecting/LLM runs only on the small surviving set.
- Added a **winnability** score (S3 + CRM `winnability` field): P(we win the lot ≤ max_bid) from bid
  count, price trajectory, time-to-close, comps — deprioritize likely-overbid lots before spending
  prospecting effort. New CRM transition gate `Scored → Prospected` on winnability.
- **Reconciled the buyer-count threshold with the user's instruction:** lowered the outreach trigger
  from ≥3 to **≥2 genuine, independent, comparable buyers** (3+ still preferred for margin) across
  `PLAN.md`, `legal-constraints.md`, and `crm-data-model.md`. Added the binding safety qualifier: a
  dominant/famous mark never counts toward the two (else it's the single-buyer case in disguise).
  CRM `Prospected → In-outreach` is a **hard block** below 2.
- Added **daily-funnel metrics** (§8) including the two-reliable-buyers yield rate, plus
  `scraped_date`/`daily_batch_id` on the Deal for funnel tracking.
- *Tradeoff noted:* 2 is the floor that establishes non-exclusivity; it carries less legal cushion
  than 3, mitigated by the independence/comparability quality rule.

## v2 — 2026-06-10 — encode the single-buyer test as the headline filter
- Promoted the **single-buyer test** to the top of the legal reframe in `PLAN.md` §2 and made it the
  primary operational filter: *how many unrelated, comparable entities would plausibly want this
  name for its own meaning?* 1 → drop; ≥3 genuine independent buyers → pursue.
- Added **worked pursue-vs-drop examples** to `legal-constraints.md` (`emule.com`/`apple.com` = drop;
  `youngsister.com`/`clinicalstaffing.com`/`coldbrew.io` = pursue), the three-prong UDRP test, and
  **Reverse Domain Name Hijacking** as a protection for legitimate generic owners.
- Added a **"defenses that do NOT work"** section (manufactured ignorance, pretextual/"flimsy" use,
  "finder's fee" labeling) and the **inbound-vs-outbound** distinction, so nobody later builds the
  business on an evasion that backfires.
- Added the **outreach framing rule**: market to the *whole category* with the *same generic pitch*
  to every prospect; never single out one mark holder. (Carried into `PLAN.md` S5.)
- Hardened the CRM (`crm-data-model.md`): new `independent_buyer_count` field; auto-drop on
  single-buyer; **hard block** on advancing below 3 independent comparable buyers; same-template
  outreach constraint with a flag for brand-referencing copy.
- *Origin:* refined live with the user while pressure-testing the legal line (emule.com vs.
  youngsister.com, "what if I reach out to many buyers at once").

## v1 — 2026-06-10 — initial plan
- Established the master plan, legal guardrail, and CRM data model from the original idea.
- **Biggest changes vs. the original idea:**
  1. **Legal reframe:** retargeted from brand-matching domains (cybersquatting under ACPA/UDRP) to
     generic/descriptive, multi-buyer domains. Made the trademark check a hard *first* gate.
  2. **Commitment-before-bid:** inverted the order so we secure a deposit/LOI-backed buyer at a
     fixed price *before* bidding, with an enforced max-bid ceiling — capping per-deal downside at
     the (loss-refundable) gname backorder deposit.
  3. **Operating-model framing:** default to "options dealer" (Model B), reserve speculative
     inventory (Model C) for proven-liquid generics with a fallback resale exit.
  4. **CRM as enforcement layer:** gates enforced as data transitions; guardrail metrics as
     first-class reports; timeline-aware sequencing to kill "auction closed before commit" losses.
  5. **Surfaced concrete operational landmines:** 60-day ICANN post-registration transfer lock;
     gname deposit/forfeit rules; CAN-SPAM/GDPR/CASL on outreach.
- **Research done:** gname backorder/proxy-auction mechanics; ACPA/UDRP bad-faith standard;
  brokerage commission (10–20%) and escrow norms.
- **Next suggested passes (from PLAN §10):** (a) gname fee/deposit/forfeit + API specifics;
  (b) trademark dataset/API choice for S2; (c) price-comp model source (NameBio/Sedo); (d) prospect
  data sourcing + per-geography outreach compliance playbook.
