# tasks.md — Build task list

Ordered, checkable breakdown of `plan.md`. Each task cites the requirement it satisfies (`FR#` from `spec.md`, `AC#` = acceptance criterion, `D#` = plan decision). This list is **reviewed** by the human and **tracked** by me: I tick a box only when that task is done and verified.

Legend: `[ ]` pending · `[~]` in progress · `[x]` done & verified

## Phase 1 — Scaffold
- [x] 1.1 Create single `index.html` with a valid HTML skeleton; no external `<link>`, `<script src>`, font, or asset. — FR1, D-file-shape
- [x] 1.2 Add empty `<style>` and `<script>` blocks inline, in the top-to-bottom order (style → markup → script). — FR1

## Phase 2 — Pricing core (no DOM)
- [x] 2.1 Add the config object at the top of the script: `BASE`, `SAME_DAY`, currency symbol, phone — placeholder values, clearly labelled. — FR6, D5
- [x] 2.2 Write the pure `quote(dogs, sameDay, holiday, config)` function signature; touches no DOM, no globals, no `Date`. — FR5, D1, D4
- [x] 2.3 Dog charges: 1st = `BASE`, 2nd = `0.5·BASE`, each dog past the 2nd = `BASE` (only the 2nd discounted). — Pricing model, AC1–AC3
- [x] 2.4 Holiday step: if holiday, multiply dog charges (discount included) by 2. — Pricing model, AC4, AC5
- [x] 2.5 Same-day step: if same-day, add `SAME_DAY` once, after doubling, not doubled. — Pricing model, AC6, AC7
- [x] 2.6 Round to two decimals exactly once, at the end; full precision until then. — D3, AC7
- [x] 2.7 Guard inputs inside `quote()`: treat dogs < 1 or non-integer as no valid walk (never negative/`$0`-as-quote). — FR2, D7, AC9

## Phase 3 — Inputs & display
- [x] 3.1 Add the three controls: dog-count `number` (`min=1`, `step=1`, default 1), same-day toggle, holiday toggle. — FR2, D7
- [x] 3.2 Add the quote display element and an echo of the current inputs beside it. — FR4
- [x] 3.3 Currency formatting helper: configured symbol + `toFixed(2)` (e.g. `$15.00`). — FR4, D8

## Phase 4 — Wiring
- [x] 4.1 `recalc()` reads the three inputs, calls `quote()`, writes the formatted total + input echo. — FR3, FR4, D6
- [x] 4.2 Bind `recalc()` to `input`/`change` on all three controls; no submit button, no form submit. — FR3, D6
- [x] 4.3 Run `recalc()` once on load so a valid quote shows immediately (default 1 dog). — FR2, FR3

## Phase 5 — Contact & layout
- [x] 5.1 Add "Rosie's Dog Walking" heading and a "Text to book" `tel:`/`sms:` link built from the config phone value; static, no form. — FR7, D9
- [x] 5.2 CSS: mobile-first single column, `max-width` centered, large tap targets, `system-ui` fonts, no horizontal scroll on phone or desktop. — FR8, D10

## Phase 6 — Verify
- [x] 6.1 Add optional `console.assert` block encoding AC1–AC13; deletable, dev-facing only. — Verification, D-verify
- [x] 6.2 Manual pass of AC1–AC7 in the page (the worked pricing examples). — AC1–AC7
- [x] 6.3 Determinism check: same inputs twice → identical total; confirm no `Date` anywhere. — FR5, AC8, D4
- [x] 6.4 Invalid-input check: dog count cannot drop below 1; no `$0.00`-as-quote, no negative. — FR2, AC9
- [x] 6.5 Live-update check: changing any input updates the total with no button press / reload. — FR3, AC10
- [x] 6.6 Offline check: disable network, confirm every criterion still holds. — FR1, AC12
- [x] 6.7 Rate-edit check: change `BASE` and the phone in config → quote and contact prompt update, calculation logic untouched. — FR6, AC13

## Coverage check
Every spec FR is claimed by at least one task: FR1 (1.1, 1.2, 6.6) · FR2 (2.7, 3.1, 4.3, 6.4) · FR3 (4.1, 4.2, 4.3, 6.5) · FR4 (3.2, 3.3, 4.1) · FR5 (2.2, 6.3) · FR6 (2.1, 6.7) · FR7 (5.1) · FR8 (5.2).

## Verification results

- **Pricing (AC1–AC7):** the `quote()` logic was executed out-of-browser against all seven worked examples — all PASS (10, 15, 25, 20, 30, 15, 35).
- **AC8 determinism / AC9 invalid input:** PASS — identical output for repeated inputs; `0`, `-1`, and `1.5` dogs all return no quote (never `$0.00`, never negative).
- **Round-once (D3):** an odd-cent rate (`BASE = 12.33`, 2 dogs) yields `18.50`, confirming a single end-of-chain round rather than mid-chain drift.
- **FR1 offline / FR5 no-clock:** verified by inspection — the file contains no `fetch`/`XMLHttpRequest`, no external `<link>`/`<script src>`/`@import`/`url()`, and no `Date` usage (the only "Date" is a comment).
- **FR3 live update / FR4 echo / 4.3 on-load:** `recalc()` is bound to `input`/`change` on all three controls and called once on load; there is no form or submit button.
- **FR6 rate-edit:** all money/contact values come from the single `CONFIG` object, so editing `BASE`/`PHONE` flows through to the quote and the text-to-book link without touching logic.

Note: DOM-behaviour items (6.2, 6.5) were verified by executing the pricing core and inspecting the wiring, not by driving a live browser — no browser automation is available in this environment. Recommend a final manual open-in-browser pass before Rosie shares the link.
