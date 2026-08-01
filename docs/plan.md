# plan.md — Technical plan for the quote calculator

How the single page will be built. Aligned to the constitution (one HTML file, embedded CSS + JS, no backend, no deps, no frameworks) and `spec.md`. This is for review — no code written yet.

## Shape of the file

One `index.html`, structured top-to-bottom as:

1. `<style>` — all CSS, inline.
2. Markup — heading, three inputs, the quote display, the text-to-book line.
3. `<script>` — a **config block**, a **pure pricing function**, and **DOM wiring**, in that order.

Nothing is fetched. No fonts, no CDN, no icons-over-network — anything visual is CSS or a Unicode character. Opening the file offline is the primary test.

## Key decisions (and why)

**D1 — One pure function computes the quote; the DOM never does math.**
`quote(dogs, sameDay, holiday, config) → number`. It takes plain values and returns a number, touching no DOM and no globals. Why: it makes the acceptance criteria checkable literally (each one is a call with a known answer), keeps all pricing logic in one place per the constitution, and stops display code and pricing code from drifting apart.

**D2 — Fixed evaluation order: dogs → holiday → same-day → round.**
1. dog charges: `BASE` + `0.5·BASE` (2nd only) + `BASE`·(each dog past the 2nd)
2. if holiday, `× 2`
3. if same-day, `+ SAME_DAY` (once)
4. round to cents, once
Why: this is the exact order the interview settled and the acceptance criteria encode (e.g. 2 dogs holiday+same-day = $35, not $40 or $30). The research flagged order-of-operations as the top source of silent conflict, so it's pinned in one function, not scattered across `if`s. Only the 2nd dog is ever discounted, so "no stacking" is structural — there's only one discount and it's applied once.

**D3 — Round exactly once, at the very end.**
Carry full numeric precision through steps 1–3; apply a single round-to-two-decimals at step 4 only. Why: mid-chain rounding is the "receipt doesn't match" bug from the research. The half-price 2nd dog can produce a fractional cent (odd base rates); one final round is the only place that's resolved, so Rosie and the customer always land on the same figure.

**D4 — The calculation never reads the clock.**
"Same-day" and "holiday" come *only* from the user's toggles; no `Date`, no timezone, no "is today a holiday" logic. Why: determinism (spec FR5) — the same inputs must give the same number for anyone, any day, on any device. Auto-detecting the date would break parity between Rosie and the customer.

**D5 — A single config object holds everything editable.**
One clearly-labelled object at the top of the script: base rate, same-day surcharge, currency symbol, phone number. The 50% and ×2 factors stay in the function as fixed rules (they're policy from the brief, not knobs). Why: the constitution promises Rosie can open the file and change her prices without touching logic; the config/logic split makes that safe.

**D6 — Live recalculation on every input event.**
One `recalc()` reads the three inputs, calls `quote()`, and writes the formatted result plus an echo of the current inputs. Bound to `input`/`change` on all three controls. No submit button, no form submission. Why: spec FR3/FR4 — the number updates as you type/toggle, and the inputs it reflects are shown alongside it so both users can confirm they entered the same thing.

**D7 — Dog count is an integer input, floored at 1, default 1.**
`type="number"`, `min="1"`, `step="1"`, starts at 1; the code also clamps/ignores anything below 1 or non-integer rather than trusting the field. Why: a valid quote is always on screen, and it structurally prevents the $0.00-as-quote and negative-total failure modes.

**D8 — Currency formatting is symbol + `toFixed(2)`.**
Prefix the configured symbol and force two decimals (`$15.00`). Not `Intl.NumberFormat`, since the symbol is a config value and this keeps it trivial and locale-independent. Why: simplest thing that always shows two decimals; nothing to go wrong offline.

**D9 — Contact is a static `tel:`/`sms:` link.**
The "Text to book" line links to the configured number; on a phone it opens the messaging app, on desktop it's just visible text. No form, nothing submitted. Why: spec FR7 and out-of-scope — reach Rosie without capturing data or needing a backend.

**D10 — Mobile-first, single-column, system font stack.**
A narrow centered column (`max-width`), fluid width, large tap targets, `system-ui` fonts, no media queries beyond what a phone-and-desktop layout needs. Why: spec FR8 (legible on phone and desktop, no horizontal scroll) without pulling in a CSS framework the constitution forbids.

## Verification approach

Lightweight, matching a weekend build — no test framework. Two things:
- An optional block of `console.assert` calls encoding the 13 acceptance criteria (1 dog = $10, 2 dogs = $15, 2 dogs holiday+same-day = $35, determinism, etc.), runnable in the browser console and easy to delete. It exists to catch a broken edit to `quote()`, not as process.
- A manual pass of the same criteria in the page itself, plus the offline check (disable network, confirm it still calculates) and the rate-edit check (change `BASE`, confirm the quote and no logic changed).

## Explicitly not doing

- No build step, bundler, transpiler, or package.json.
- No state persistence, URL params, or storage.
- No date/holiday detection, no calendar.
- No analytics, service worker, or any network call.

## Nothing blocking

Every value the code needs either has a fixed rule (50%, ×2) or a placeholder in the config (base, surcharge, phone, `$`) that Rosie replaces later. Ready to implement on your go-ahead.
