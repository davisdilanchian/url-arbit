# sales-crm — data model, pipeline, automations

The CRM is the spine of url-arbit. Its job is to make the *safe, high-yield* path the *only* path:
hard gates are enforced as data constraints, and the metrics that minimize failures/lost
opportunities are first-class reports.

## Core principle
- A **domain = a Deal** moving through the S1–S9 stages from [`PLAN.md`](PLAN.md) §5.
- The brand gate (S2) and the commitment gate (S6) are **enforced transitions**: a Deal physically
  cannot advance without the required linked record. The CRM removes the ability to skip a gate.

## Entities

### Deal (one per domain)
| Field | Notes |
|---|---|
| `id` | |
| `domain`, `tld` | |
| `stage` | enum mirroring S1–S9 (see Pipeline below) |
| `source` | gname auction / backorder / pending-delete / off-market |
| `auction_close_at` | drives timeline-aware sequencing; **prefer long runway** |
| `current_price`, `bid_count` | refreshed near close |
| `brand_gate` | `pending` / `passed` / `dropped` + `reason` (required to leave S2). Auto-`dropped` with reason `single-buyer` when `independent_buyer_count` < 2 |
| `independent_buyer_count` | # of unrelated, comparable entities that plausibly want the name for its own meaning. **The single-buyer test:** 1 → drop; ≥3 → eligible to pursue |
| `llm_score`, `buyer_categories[]`, `price_band` | from S3 |
| `max_bid` | computed = agreed_price − margin − fees; **enforced ceiling at bid time** |
| `agreed_price` | from the winning Commitment |
| `realized_spread`, `outcome`, `reason_code` | written at S9 (won/lost/walked/fell-through) |

### Account / Contact (prospect orgs — fan-out, many per Deal)
| Field | Notes |
|---|---|
| `org_name`, `category` | category company, **not** a single brand owner |
| `contacts[]` (name, role, verified_email) | |
| `buying_signals` | why they plausibly want this generic name |
| `suppression` | opt-out / do-not-contact (compliance) |
| `brand_conflict_flag` | if this prospect *is* the brand → kicks Deal back to S2 |

### Outreach (one per touch; full log)
`deal_id`, `contact_id`, `template_id`, `variant` (A/B), `sent_at`, `channel`, `reply_at`,
`disposition` (interested / negotiating / declined / no-reply), `notes`. Retained for optimization
**and** legal audit trail.

### Commitment (the gate before money) — required to enter "Bidding"
`deal_id`, `account_id`, `agreed_price`, `commitment_type` (deposit / signed LOI / escrow hold),
`evidence_ref`, `expires_at`. **No Commitment record → Deal cannot enter S7.**

### Acquisition / Transfer
`deal_id`, `bid_max`, `win_price`, `deposit_state`, `escrow_id`, `transfer_method`
(intra-registrar push / inter-registrar), `transfer_lock_until` (track the **60-day post-registration
ICANN lock**), `settled_at`.

## Pipeline (stages = S1–S9)
`Sourced → Brand-screened → Scored → Prospected → In-outreach → Committed → Bidding/Acquiring →
Transferring → Closed`, plus terminal `Dropped` (failed brand gate / low value) and `Lost`
(auction overshoot-walk / buyer fell through → route to fallback resale).

### Enforced transitions (the safety rails)
- `Brand-screened → Scored`: requires `brand_gate = passed`.
- `Brand-screened → Scored`: also auto-drops if `independent_buyer_count` < 2 (single-buyer test).
- `Scored → Prospected`: requires `independent_buyer_count` ≥ 3 (genuine, *independent, comparable*
  buyers — not one famous mark plus noise). **Hard block** below 3; this is the multi-buyer rule
  that keeps us in the generic/legal lane.
- *Outreach constraint:* all prospects on a Deal get the **same generic pitch template**; the CRM
  flags any per-contact copy that references a specific prospect's trademark/brand (red-zone framing).
- `Committed → Bidding`: requires a linked **Commitment** record. **Hard block.**
- `Bidding`: `bid_max` is enforced; system walks if auction price > `bid_max` and sets `Lost/walked`
  (recorded as margin-protected, **not** a failure).
- `Transferring → Closed`: requires escrow `released` + transfer `verified`.

## Automations
- **Timeline-aware sequencing:** schedule/escalate outreach against `auction_close_at`; auto-flag
  deals at risk of closing before a commitment (drives down the top "lost opportunity" metric).
- **Auction-close watcher:** refresh price/bid near close; alert if approaching `max_bid`.
- **Compliance:** enforce suppression list, attach opt-out, throttle for deliverability.
- **Fall-through recovery:** if a Committed buyer evaporates after a win, auto-create a fallback
  resale listing (Dan/Afternic/Sedo) so nothing becomes dead inventory.
- **Learning loop:** on Close, write realized spread + reason code back to the price/score models.

## Reports (the guardrail metrics from PLAN §8 as dashboards)
Opportunity funnel (ingested → passed gate → scored → prospected → replied → committed → won);
**lost-to-close-timing** count; negative-spread count (target 0); **brand-gate escapes caught
downstream** (target 0); uncovered-inventory + carry; fall-through rate; realized spread + gross
margin; cash conversion cycle; compliance complaints/spam rate.

## Build note
Start as the simplest thing that enforces the two gates and logs outreach (even a Postgres schema +
thin app). The CRM must exist and capture metrics in Phase 1 *before* automation — it's how we learn
whether the spread and the legal framing hold in reality.
