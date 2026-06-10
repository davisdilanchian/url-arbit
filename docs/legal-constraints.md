# Legal constraints — the load-bearing guardrail

> This is not legal advice. It is an engineering specification for staying clear of the failure
> mode that can shut the whole business down. Get a real IP attorney before operating.

## Why this document exists

The original idea — *"find organizations likely to be interested in a domain, buy/win it, then sell
it to them at a markup"* — describes, almost word for word, the conduct that the **Anticybersquatting
Consumer Protection Act (ACPA, 15 U.S.C. §1125(d))** and the **UDRP** were created to punish, *when
the domain is identical or confusingly similar to that organization's trademark*.

- **ACPA penalties:** statutory damages **$1,000–$100,000 per domain**, plus loss of the domain.
- **UDRP:** loss/transfer of the domain (no damages, but fast and cheap for the complainant).
- **Bad-faith factors courts weigh** include: the registrant has *no* IP rights/legitimate interest
  in the name; the name isn't the registrant's; and **the domain was offered for sale to the mark
  owner at an above-cost price.** Our naive flow hits several of these at once.

One adverse finding isn't a bad month — it's an existential and reputational event. So the legal
constraint is the *first* filter in the pipeline, not a compliance checkbox at the end.

## The design rule

**Only ever pursue domains with inherent generic/descriptive value and many potential buyers.
Never pursue a domain because it matches one company's brand.**

This single rule simultaneously:
- **Removes bad-faith intent:** a generic descriptive term has no single rightful owner; holding/
  selling `coldbrew.io` or `clinicalstaffing.com` is legitimate domain commerce.
- **Creates legitimate interest:** descriptive value *is* a recognized legitimate interest.
- **Improves the business:** many buyers = leverage, competitive tension, and a fallback buyer if
  one walks. (Brand resale has exactly one possible buyer — bad business *and* illegal.)

## The decision rule: generic value vs. brand-specific value

The single question that sorts every domain is: **does the domain's value come from a generic/
descriptive meaning (many possible users), or from one party's distinctive identity (one user)?**

**The single-buyer test (the fastest operational filter).** Ask: *how many unrelated, comparable
entities would plausibly want this name for its own meaning?*
- **Exactly one → DROP.** One realistic buyer means the value *is* that party's identity. Selling
  to them is the bad-faith pattern (ACPA factor #6). This is true no matter how you frame the fee.
- **Several genuine, independent, comparable buyers → PURSUE.** Multiplicity is the *proof* the name
  is generic and that you hold a market asset, not someone's brand. Require **≥3** before spending.

This is why "reach out to many buyers" is legitimate *only when the multiplicity is real*. The
protection comes from the name genuinely having many interested parties — not from the act of
mass-emailing. Spamming a single-buyer brand name (e.g. `emule.com`) to 50 companies does not
launder it: 49 have no reason to want it, the value is still the one mark holder's identity, and a
panel sees through it. Multiplicity is a *symptom* of generic value, never a manufactured cover.

### Worked examples (pursue vs. drop)

| Domain | Possible buyers | Why | Verdict |
|---|---|---|---|
| `emule.com` | one (the eMule project) | coined name = one party's identity | **DROP** |
| `apple.com` → Apple | one (famous mark) | distinctive brand | **DROP** |
| `youngsister.com` | many (a band, a clothing co., a nonprofit, a film…) | ordinary two-word phrase, no one owns it | **PURSUE** |
| `clinicalstaffing.com` | many (dozens of staffing firms) | descriptive category | **PURSUE** |
| `coldbrew.io` | many (any coffee brand) | generic product term | **PURSUE** |

Note on the multi-buyer case: market the asset to the **whole category** with the **same generic
pitch** ("premium two-word .com that may fit your brand"), and let interested parties — including any
entity that happens to share the name — come to you or compete. The existence of one company called
"Young Sister" does **not** give them ownership of a common English phrase; trademark law does not
let anyone monopolize ordinary descriptive words. A complainant must prove all three UDRP prongs
(confusingly similar to *their* mark; you have *no* legitimate interest; registered *and* used in
bad faith), and a genuine generic fails them on prongs 2 and 3. Overreaching trademark holders who
try to grab legitimate generics can even be sanctioned for **Reverse Domain Name Hijacking** — the
process protects the generic owner.

### Defenses that do NOT work (do not build the business on these)

- **"I didn't know about the trademark."** Not credible for a distinctive name (a "knew or should
  have known" standard applies; willful blindness counts as knowledge), and irrelevant the moment
  you *find the holder and solicit them* — that outbound act forms the bad-faith intent regardless
  of what you knew at purchase. Clean sales are **inbound** (the mark owner approaches you about a
  domain you hold for legitimate reasons), not **outbound** (you hunt the mark owner because of
  their name).
- **"I'll put up a flimsy use, then sell."** Pretextual/sham use is the most-litigated issue in
  UDRP and panels routinely see through it (site stood up only before a sale, thin/placeholder
  content, or use that trades on the mark's meaning). A sham use can *aggravate* the case —
  evidence of bad faith, pushing an ACPA finding toward *willful* and damages toward the $100k end.
  A **genuine, substantial** business use creates a real interest, but then you're an operator who
  spent real time/money, not a flipper.
- **"It's a finder's fee, not a ransom."** The label on the payment is irrelevant; factor #6 turns
  on financial gain from the mark owner without a legitimate interest of your own.

## Pipeline implications (enforced in code/CRM, not left to judgment)

1. **S2 hard gate — drop if any of:**
   - exact/confusingly-similar match to a registered trademark (USPTO/EUIPO/etc.),
   - matches a known brand/company name or personal name,
   - LLM "is this a brand/coined term?" returns yes,
   - the *only* plausible buyers found in S4 are a single brand and its competitors-by-name (i.e.
     it's really a brand term in disguise).
   When uncertain → **drop.** False negatives are catastrophically asymmetric.

2. **Audit trail:** every domain's gate decision (keep/drop + reason) is logged and retained. If a
   dispute ever arises, we can show a documented, good-faith, generic-only sourcing policy.

3. **Outreach framing:** we are a domain **seller/broker** presenting an asset on its descriptive
   merits to category participants. We never write "this matches your trademark/brand, buy it from
   us." Templates are reviewed and stored in the CRM. (This framing also materially lifts reply
   rates — buyers respond to "valuable generic asset," not to a perceived shakedown.)

4. **Multi-buyer requirement:** require **≥3 independent prospective buyers** before spending. If a
   name only has one realistic buyer, that's a signal it's a brand term → kick back to S2.

## Adjacent compliance (not optional, lower severity)

- **Cold outreach:** CAN-SPAM (US), CASL (Canada), GDPR/ePrivacy (EU) — accurate headers, physical
  address, working opt-out, honored suppression list, lawful basis per geography.
- **Tax/entity:** domain trading is taxable; set up the entity and bookkeeping early.
- **Platform ToS:** gname, escrow, and resale-marketplace terms — read and comply, especially
  around automated bidding/scraping and payout rules.

## Open legal questions (for an attorney + later iterations)

- How to operationalize "confusingly similar" beyond exact match (phonetic/visual/typo variants).
- Cross-border exposure given gname's Chinese-market focus and international buyers.
- Whether speculative **inventory** holding (Model C) raises the bad-faith profile vs. the
  commitment-first **options** model (Model B) — lean B until advised otherwise.
