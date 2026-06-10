# url-arbit

A system for sourcing expiring/aftermarket domains on [gname.com](https://www.gname.com/auction),
scoring them with an LLM, matching each to organizations that have a *legitimate descriptive
interest* in the name, and brokering a sale — coordinated end-to-end through an in-house
**sales-crm**.

> **Read this first:** the naive version of this idea ("buy a company's brand domain, then sell
> it back to them at a markup") is textbook cybersquatting under the U.S. ACPA and the UDRP. This
> project is deliberately designed to stay on the *generic / descriptive / multi-buyer* side of
> that line. See [`docs/legal-constraints.md`](docs/legal-constraints.md) — it is the load-bearing
> constraint, not an afterthought.

## Documents
- [`docs/PLAN.md`](docs/PLAN.md) — the master plan (recursively improved; see its iteration log)
- [`docs/legal-constraints.md`](docs/legal-constraints.md) — the trademark/cybersquatting guardrail
- [`docs/crm-data-model.md`](docs/crm-data-model.md) — sales-crm schema, pipeline stages, automations
- [`docs/ITERATION_LOG.md`](docs/ITERATION_LOG.md) — what each planning pass changed and why

## Status
Planning. No code yet. The plan is being refined on a recurring loop.
