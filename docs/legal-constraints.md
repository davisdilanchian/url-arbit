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
