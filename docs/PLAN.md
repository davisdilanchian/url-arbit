# url-arbit — Master Plan

A living document. Each recursive planning pass deepens one or more sections and records the
change in [`ITERATION_LOG.md`](ITERATION_LOG.md). Newest summary at the top.

---

## 0. TL;DR of the current plan

Source expiring/aftermarket domains on gname.com → LLM-score them for *generic commercial value*
→ for the good ones, find **multiple** organizations that plausibly want the name for its plain
descriptive meaning → reach out and get a **binding, deposit-backed commitment at a fixed price
*before* we bid** → win the auction with a hard max-bid below that price → transfer via escrow →
keep the spread. The sales-crm is the spine: every domain is a deal, every prospect is a contact,
every stage transition is automated and measured.

The two ideas that most improve the original concept:

1. **Legal reframe (existential):** never target a domain because it matches one company's brand.
   Target domains whose *dictionary/descriptive meaning* makes them valuable to a whole category
   of buyers. This is both legal (legitimate interest, no single mark holder) and better business
   (multiple bidders for the same asset = leverage and a fallback buyer).
2. **Commitment-before-capital (de-risks the whole thing):** the original plan bids/buys and
   *hopes* a buyer agrees. Invert it. Get a signed, deposit-backed buyer commitment first, then
   bid with a max below the agreed price. You almost never hold uncovered inventory, and your
   downside on any single deal is capped at the auction deposit.

---

## 1. Thesis & one-line model

**Thesis:** gname.com surfaces a high volume of expiring/aftermarket domains, many priced on
auto-pilot by sellers/backorderers who don't know which end-users would value them. There is a
spread between "auction clearing price" and "what a motivated category buyer will pay for a clean
descriptive .com/.io/etc." An LLM + CRM pipeline can find and capture that spread at scale that
manual brokers can't.

**Model (one line):** *productized domain brokerage with pre-committed buyers, sourced from
gname auctions, run on rails by sales-crm.*

---

## 2. The reframe that makes this legal and durable

Full detail in [`docs/legal-constraints.md`](legal-constraints.md). The short version drives the
entire product:

- **Hard exclusion filter (first thing in the pipeline, before any outreach):** drop any domain
  that is identical/confusingly similar to an existing trademark, brand, or personal name. Check
  against USPTO/EUIPO trademark data, a known-brands list, and an LLM "is this a brand?" pass.
  When in doubt, drop it. The cost of a false negative (one ACPA suit) dwarfs the value of any
  single domain.
- **Target only names with inherent descriptive/generic value:** `solarfinancing.com`,
  `coldbrew.io`, `clinicalstaffing.com`, two-word category names, strong generic keywords. These
  have *many* legitimate buyers and *no* single rightful owner, which is exactly what keeps us
  clear of bad-faith findings and gives us pricing leverage.
- **Outreach framing matters legally:** we approach buyers as a domain *seller/broker* offering an
  asset on its descriptive merits — never "we noticed this matches your company, pay us." Keep
  templates auditable (see CRM). This framing is also the difference between a 2% and a 0.2%
  reply rate.
- **This reframe reshapes "identify high-interest organizations":** it's no longer "whose brand is
  this?" It's "which companies operate in the category this generic phrase describes?" That's a
  fan-out (many prospects per domain), which the CRM is built to run.

---

## 3. Unit economics & the core risk

### The spread
- **Cost side:** winning auction price + gname deposit/fees + escrow fee (~1–3%) + transfer/renewal
  + our time (LLM + outreach compute is cheap; human review is the real cost).
- **Revenue side:** the pre-agreed price from the committed buyer.
- **Target:** only pursue deals where (agreed price) ≥ (max viable bid) × **margin multiple**.
  Start conservative (e.g. require ≥2–3× headroom) because early-stage hit rates and price
  discovery are noisy.

### The core risk: *commit-before-bid* timing
The original plan has a fatal ordering bug: you propose a price, then go win the auction. Two ways
to lose:
- **Auction overshoots your buyer's price** → you either overpay (negative spread) or back out and
  burn the buyer relationship + your credibility.
- **You win, buyer ghosts** → you're holding inventory you bought on spec (capital tied up,
  renewal costs, may never sell).

**Mitigations (these define the operating model in §4):**
- Get the buyer commitment **first**, with a **deposit/LOI** and a fixed price, *then* bid.
- Bid via gname's **proxy/max-bid** with a hard ceiling = (agreed price) − (target margin) − fees.
  If the auction exceeds the ceiling, walk — you lose only the (refundable-on-loss) backorder
  deposit, never margin.
- Use **escrow.com / Dan / Sedo TransferCenter** so neither side can stiff the other; funds
  release only on verified transfer.

---

## 4. Operating models compared (pick per-deal, default to the safest)

| Model | We own the domain? | Capital at risk | Legal exposure | Upside | When to use |
|---|---|---|---|---|---|
| **A. Pure broker** | No (we never hold it) | ~0 | Lowest | 10–20% commission | Domain already has an identifiable seller; we just connect + close |
| **B. Options dealer (DEFAULT)** | Only after a committed buyer exists | Backorder deposit only (refunded if we lose/walk) | Low (generic names + commitment first) | Full spread | gname auction with pre-committed buyer |
| **C. Inventory arbitrage** | Yes, on spec | Full purchase + carry | Higher (holding undeveloped generic domains can itself look speculative) | Highest, lumpiest | Only exceptional, clearly-generic, liquid names with proven repeat demand |

**Recommendation:** run **B** as the default. Graduate a name to **C** only when (a) it's
unambiguously generic, (b) you've seen ≥2 independent category buyers, and (c) you have a tested
resale channel (Dan/Afternic/Sedo) as a fallback exit so it's never dead inventory. Use **A**
opportunistically when sourcing surfaces a willing seller outside the auction flow.

**gname mechanics that shape this** (from their help docs): domains are backordered for a deposit
before deletion; contested ones go to a **proxy auction** (set your psychological max, system bids
incrementally); **deposit is fully refunded if the backorder is unsuccessful.** So Model B's
"walk-away cost" is genuinely near-zero on lost auctions — the deposit only converts to spend when
we actually win. Confirm current fee schedule and whether deposit is forfeited on *winning then
not paying* (almost certainly yes — never win without a committed buyer).

---

## 5. End-to-end pipeline — stage by stage, with failure modes & mitigations

> Each stage maps to a CRM pipeline stage (§6) and emits metrics (§8).

**S1 — Source / ingest.** Poll gname auction + backorder + pending-delete lists. Capture domain,
TLD, current price, bid count, **time-to-close** (prefer long runway — gives outreach time),
registry, and traffic/age signals.
- *Failures:* missing listings, rate-limiting/scraping breakage, stale prices.
- *Mitigations:* prefer official API/feeds over scraping; idempotent ingest; re-poll hot deals as
  close approaches; alert on feed gaps.

**S2 — Trademark/brand exclusion (HARD GATE).** Before anything else, drop brandable/trademarked
names. (See §2 / legal doc.)
- *Failures:* false negative → legal exposure; false positive → lost generic opportunity.
- *Mitigations:* bias toward dropping; log every decision with rationale for audit; periodically
  sample-review the dropped pile to tune.

**S3 — LLM value scoring.** Score survivors on commercial generic value: keyword strength,
category breadth (how many buyer types?), TLD quality, length/brandability, comparable sales.
Output a score + the **buyer categories** this name serves + a price band.
- *Failures:* LLM over-values junk; hallucinated comps.
- *Mitigations:* calibrate against real comp data (Sedo/NameBio); require category breadth ≥ N
  buyers to proceed; cheap model for triage, strong model for finalists; never auto-bid on LLM
  score alone (human gate before money moves).

**S4 — Prospect identification (fan-out).** For each finalist, find *multiple* real organizations
in the named category — not one brand. Enrich with contact + buying-signal data.
- *Failures:* thin/biased prospect lists; bad contact data; accidentally targeting a single mark
  holder (re-checks §2).
- *Mitigations:* require ≥3 independent prospects before spending; verify emails; re-run the
  brand check on the *prospects* (if every prospect is the same brand, the name was brandable —
  kick back to S2).

**S5 — Outreach + negotiation.** Sequenced, compliant cold outreach (CAN-SPAM/GDPR/CASL):
identify, offer the asset on descriptive merit, propose price, handle replies.
- *Failures:* low reply rate, deliverability/spam, compliance violations, race (domain closes
  before a buyer commits), buyer lowballs below our floor.
- *Mitigations:* warm domains/inboxes, personalization from S3/S4, A/B templates, opt-out + suppression
  list, **timeline-aware sequencing tied to auction close**, hard price floor from §3, parallel
  outreach to multiple prospects so one declining ≠ dead deal.

**S6 — Commitment (the gate before money).** Get a fixed price + **deposit/LOI/escrow hold**
before bidding. No commitment → no bid.
- *Failures:* soft "yes" that evaporates; buyer tries to go around us to the auction directly.
- *Mitigations:* require a real signal (deposit/signed LOI); for auction names the buyer usually
  *can't* easily snipe gname themselves; keep the specific domain confidential until commitment
  where feasible.

**S7 — Acquire (bid / buy).** Place gname proxy bid with hard max = agreed − margin − fees, OR
execute the pre-arranged purchase. Walk if exceeded.
- *Failures:* auction overshoot, sniping, payment/funding hiccup at win, deposit forfeiture rules.
- *Mitigations:* disciplined max-bid (never emotional), pre-funded account, know the exact deposit/
  forfeit rules before bidding, treat a walk as a *success* (margin protected) not a loss.

**S8 — Transfer + settle (escrow).** Move domain to buyer via escrow; funds release on verified
transfer.
- *Failures:* transfer lock (60-day ICANN post-registration lock!), registrar friction, chargeback,
  buyer delays acceptance.
- *Mitigations:* **account for the 60-day transfer lock on freshly-(re)registered domains** in
  timeline + buyer expectations (this is a big one — newly won domains often can't be pushed to
  another registrar immediately; use intra-registrar push or hold-and-park until eligible); escrow
  for funds; clear handoff checklist.

**S9 — Close, learn, recycle.** Record actuals, update comp/price models, feed win/loss back into
S3/S5. Unsold committed-but-fell-through names → resale channel (Dan/Afternic/Sedo) as fallback.
- *Failures:* no learning loop → repeated mistakes; dead inventory.
- *Mitigations:* every deal writes back realized spread + reason codes; fallback listing automation
  so nothing sits idle.

---

## 6. sales-crm — the spine

Full schema in [`docs/crm-data-model.md`](crm-data-model.md). Design principles:
- **A domain is a Deal.** Its lifecycle = the S1–S9 stages above as pipeline stages.
- **Prospect orgs are Contacts/Accounts**, many-to-one against a Deal (fan-out from S4).
- **Every outreach is logged** (template id, variant, timestamps, replies) — needed for both
  optimization *and* legal audit trail.
- **Hard gates are enforced in data:** a Deal cannot move to "Bidding" without a linked
  Commitment record (deposit/LOI). The CRM makes the safe path the only path.
- **Automations** drive timeline-aware sequencing, auction-close reminders, escrow checklists, and
  fallback-listing on fall-through.
- **Guardrail metrics** (§8) are first-class CRM reports, not afterthoughts.

---

## 7. System architecture (components)

1. **Sourcing service** — gname feed/API ingest → normalized listings store.
2. **Screening pipeline** — trademark gate (S2) → LLM scorer (S3) → prospect finder (S4).
   Cheap-model triage, strong-model finalists; human review gate before any spend.
3. **sales-crm** — deals, contacts, commitments, outreach logs, automations, dashboards.
4. **Outreach engine** — sequenced compliant email (warmed inboxes, suppression list, A/B), reply
   ingestion back into CRM.
5. **Acquisition service** — gname proxy-bid with enforced max; escrow + transfer orchestration.
6. **Analytics/learning** — comps, price models, win/loss feedback, guardrail dashboards.

Build the LLM-facing parts on the latest Claude models; keep a cheap-model/strong-model split for
cost. Keep humans in the loop at every point money moves until the metrics earn more automation.

---

## 8. KPIs / guardrail metrics (to minimize failures & lost opportunities)

**Throughput / opportunity capture:** listings ingested, % surviving trademark gate, % scored
"pursue," prospects per finalist, outreach sent, reply rate, commitment rate, auctions entered,
**win rate**, **opportunities lost to auction-close-before-commit** (the key "lost opportunity"
metric to drive down via timeline-aware sequencing).

**Risk / failure:** negative-spread deals (target 0), uncovered inventory count + carry cost,
buyer fall-through rate, transfer failures, **brand-gate escapes caught downstream** (target 0),
compliance incidents/complaints/spam rate.

**Money:** realized spread per deal, gross margin %, deposit-at-risk, cash conversion cycle.

Each has an owner stage in §5 and a CRM report in §6.

---

## 9. Phased build roadmap

- **Phase 0 — Validate by hand (no code).** Manually pull ~20 gname auctions, run the trademark
  gate + value scoring + prospecting by hand, do real outreach, try to close *one* deal end-to-end
  with escrow. Goal: prove the spread and the legal framing exist in reality before building.
- **Phase 1 — CRM + manual pipeline.** Stand up sales-crm (schema in §6) and run Phase-0 by hand
  *through the CRM*. Instrument the metrics. The CRM earns its keep before automation.
- **Phase 2 — Automate sourcing + screening.** gname ingest, trademark gate, LLM scorer, prospect
  finder feed the CRM. Humans still gate outreach + all spend.
- **Phase 3 — Automate outreach + sequencing.** Compliant warmed outreach with reply handling;
  timeline-aware sequencing to kill the "auction closed before commit" loss.
- **Phase 4 — Automate acquisition + transfer + fallback.** Proxy-bid with enforced max, escrow
  orchestration, fallback resale listing. Scale volume; widen automation only where metrics allow.

---

## 10. Open questions to resolve in later iterations

1. **gname specifics:** exact fee/deposit schedule, deposit forfeiture rules on win-then-no-pay,
   API availability vs. scraping, payout/payment rails, whether non-Chinese buyers/sellers hit
   friction. (Heavy Chinese-market platform.)
2. **Prospect data sourcing:** best legal source for category-company lists + verified contacts at
   scale; CAN-SPAM/GDPR/CASL playbook per geography.
3. **Trademark data:** which trademark datasets/APIs to wire into S2; how to score "confusingly
   similar," not just exact match.
4. **Price model:** which comp source (NameBio/Sedo) and how to turn it into a defensible price
   band per category.
5. **Escrow/transfer:** confirm the 60-day post-registration transfer-lock handling for
   freshly-won domains; choose escrow + transfer partners.
6. **Capital + structure:** how much working capital for deposits/inventory; entity/tax setup;
   what max-bid headroom multiple the data actually supports.

---

### Sources informing this plan
- gname auction & backorder docs: https://www.gname.com/auction , https://www.gname.com/us/help/domain-backorder
- ACPA / cybersquatting: https://en.wikipedia.org/wiki/Anticybersquatting_Consumer_Protection_Act , https://www.justia.com/intellectual-property/trademarks/cybersquatting/
- Brokerage/escrow norms: https://www.escrow.com/learn-more/partners/domain-brokers , https://corg.com/how-domain-aftermarket-platforms-work/
