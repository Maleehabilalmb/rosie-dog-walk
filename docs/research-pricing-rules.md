# Findings: Rule-Based Pricing Calculators (2026)

Research notes for the dog-walking quote calculator. Scope: discounts, surcharges, and rules that can conflict. No design or code proposed here — this is background before we decide anything.

## 1. How tiered/conditional pricing logic is usually structured

The consistent 2026 pattern is a **base price plus an ordered list of adjustments** rather than a tangle of `if` statements. A quote is built in layers:

- **Base rate** (e.g. per walk, per dog, per duration tier).
- **Adjustments** — each discount or surcharge is a *rule*: a condition ("2+ dogs", "weekend", "holiday", "regular client") paired with an effect (percentage or fixed amount).
- **Rules are data, not code.** Even simple engines keep rates and thresholds in one editable place so pricing can change without touching logic — the system just recalculates when inputs change. This matches the constitution's "keep all pricing rules in one clearly-labelled place."
- Rules commonly carry a **scope/level** (global → category → specific) and a **priority**, so the engine knows what to evaluate and in what order.

## 2. Evaluating multiple rules without silent conflicts

Three named strategies dominate — pick one deliberately per rule *type*:

- **Best deal** — apply only the single largest-benefit rule (customer-friendly for discounts).
- **First match** — walk rules in priority order, apply the first that qualifies, stop.
- **Additive stacking** — apply all eligible rules, capped at a ceiling (e.g. max total discount %).

To stop stacking chaos, the standard device is a **promotion/exclusivity group**: rules in the same group can't combine with each other, but a rule from one group can combine with a rule from another. Compatibility is treated as **bilateral** — both rules must opt into combining. Production systems typically cap concurrent promotions at **2–3** and evaluate in **descending priority**, where a higher-priority non-stackable rule causes the rest to be skipped.

## 3. Typical edge cases

- **Order of operations.** Discounts apply *before* tax in standard US retail; tax is charged on the reduced price. (Likely N/A for a simple quote, but decide it explicitly.) For surcharges vs. discounts, the order changes the answer and must be fixed, not incidental.
- **Stacked percentages compound.** 15% + 20% is **~32% off, not 35%** — each applies to the already-reduced amount. "Which base does this rule apply to?" must be answered for every rule.
- **Rounding.** Carry full precision through the whole calculation and round **once at the end** to two decimals. Rounding mid-chain (per line) gives different totals and causes "the receipt doesn't match" complaints.
- **Which rule wins.** When two rules match, tie-breakers used in practice: explicit **priority/salience** (an integer), **specificity** (more conditions = more specific = wins, so special cases beat defaults), or **best-deal**. Whatever the choice, it must be explicit — the owner and the customer both need to see the same number.

## 4. Failure modes to watch for

- **Double discounting** — the same reduction applied twice via overlapping rules or mixed pricing paths.
- **Invisible sequencing** — several rules fire in an order the user can't see, and the final total matches no single quoted offer. Erodes the "instant, visible, trustworthy" quote.
- **Runaway / margin blowout** — uncapped stacking or a surcharge/discount interaction driving price to an unintended value; guard with a floor and a discount ceiling.
- **Negative or nonsensical totals** — a fixed discount larger than the (already reduced) subtotal. Clamp at a sensible minimum.
- **Silent conflict** — two matching rules with no defined tie-breaker producing contradictory results; cited as one of the most common rule-engine production bugs.
- **Non-reproducibility** — same inputs, different output because rounding or rule order isn't deterministic. Fatal for a quote two people must agree on.

## Sources
- [Pricing Engine Software — Complete Guide (2026), Price Engine](https://manageprices.com/knowledge-base/what-is-pricing-engine/)
- [Top 2026 Pricing Trends Reshaping Business Services, Revenue Management Labs](https://revenueml.com/insights/articles/top-2026-pricing-trends-reshaping-business-services/)
- [Prioritize and Stack Rules, Bold Commerce Developer Docs](https://developer.boldcommerce.com/guides/price-rules/working-with-rulesets/prioritization-and-stacking)
- [What is promotion stacking? Voucherify](https://www.voucherify.io/glossary/promotion-stacking)
- [Coupon & Discount Engine Development, Codesol](https://www.codesoltech.com/blog/coupon-discount-engine-development/)
- [How to Calculate Discounts — Stacked Discounts & Common Mistakes, Basic Free Tools](https://basicfreetools.com/blog/how-to-calculate-discounts/)
- [Rounding and Tax Calculations, Evolution X Help Center](https://docs.evolutionx.io/en/articles/2117376-rounding-and-tax-calculations)
- [6 Shopify Discount Stacking Problems, Skailama](https://www.skailama.com/blog/shopify-discount-stacking-issues)
- [Discount Combinations & Catching Conflicts, PromoOS](https://promly.app/blog/shopify-discount-combinations/)
- [What Happens When Two Rules Conflict in a Rule Engine? Nected](https://www.nected.ai/blog/what-happens-when-rules-conflict-rule-engine)
- [Conflict Resolution Strategy, Wikipedia](https://en.wikipedia.org/wiki/Conflict_resolution_strategy)
