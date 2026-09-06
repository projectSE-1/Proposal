# AI Perfumery Engine — Project Proposal (Updated)

**Course:** 1305493 Software Engineering Case Studies, 1/2569
**Team / Company name:** 404 Fragrance Not Found

## Problem Statement

Fragrance formulation requires a formulator to track many pieces of information at once —
ingredients, concentrations, quantities, and restrictions — while calculating and checking
compliance by hand, with no reliable way to catch a mistake before physically mixing a formula.

This was confirmed through a real interview with our stakeholder (2026-09-02; full writeup in
`.docs/00-context/project-context.md` §3), who named five concrete pain points:

1. Too much information to track across many ingredients and properties at once.
2. Manual calculation of weight, percentage, and totals, then checking it by hand.
3. Rule/compliance checking that is complicated and easy to get wrong or overlook.
4. A formula's ingredient list, on its own, doesn't show what matters most or what's risky.
5. No way to check a formula before physically mixing it — mistakes surface only afterward.

## Target Users

- **Formulator** (primary persona, evidenced by a real interview): a fragrance formulator who
  needs to understand and evaluate a formula quickly and correctly before committing to it
  physically.
- **Domain Expert**: approves changes to chemistry rules, thresholds, and material groups. Not
  evidenced by the interview itself; required by our own compliance rules (`rule.md`).

## Proposed Solution

A calculation engine that automatically computes a formula's material weights, percentages, and
totals, and highlights the rules and restrictions relevant to that formula — so a formulator can
understand and evaluate it without manual calculation or guesswork. This is a bespoke,
deterministic tool for one real customer's internal use: it does not offer public
formula-generation-and-ordering, and it does not use any generative-AI/LLM feature (confirmed via
stakeholder meetings, `project-context.md` §17).

## Scope for This Build Cycle

One core workflow: log in, open the formula list, open a formula, and see all of its information
(materials, calculated weights/percentages, and highlighted rules/restrictions), with optional
in-place "what-if" recalculation and export. Authoring a new formula from scratch is out of scope
for this cycle — see `.docs/01-requirements/backlog.md` Open Question 1.

## Note on User Validation

Per a lecturer-confirmed exception (2026-09-02), this project's requirements are sourced from one
real customer relationship (the stakeholder above) rather than the course's usual ≥15-interview
panel, because this product is a bespoke tool built for that single real customer rather than a
multi-user consumer product. Recorded here per the recommendation to get this in writing.
