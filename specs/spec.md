# spec.md — Rosie's Dog Walking Quote Calculator

Behaviour specification for the single page. No code here. Every requirement is written so that a build ignoring it would produce a visibly wrong or missing result. All open questions from the client brief were resolved in an interview; the resolutions are baked in below (see **Decisions from interview**).

## Goal

Give an **instant, self-serve price for one dog walk** so Rosie stops working quotes out by hand. The page is public and used two ways from the *same* URL: a customer prices their own walk, and Rosie prices a walk when someone texts her. Both must get the **identical number for identical inputs** — the quote is the product.

## User scenarios

**S1 — Customer, self-serve (phone).** A prospective customer opens the link on their phone. They set the number of dogs, tick whether it's a same-day or holiday walk, tick whether any dog in the booking is aggressive or reactive, and immediately see the price. Nothing to submit, no sign-up. Below the quote they see a "Text to book" prompt and tap it to message Rosie.

**S2 — Rosie, answering a text (desktop or phone).** Someone texts "how much for 2 dogs on Christmas Day?" Rosie opens the same page, enters 2 dogs and ticks Holiday, reads the number, and texts it back. She entered the same details the customer would have, so she quotes the same figure the page would have shown them.

**S3 — Rosie changing her prices.** Rosie's rates change. She opens the single HTML file, finds one clearly-labelled block of pricing values, edits the base rate, same-day surcharge, and her phone number, saves, and every quote and the contact prompt update. She does not touch any calculation logic.

## Functional requirements

1. **Single page, no build, works offline.** One HTML file with embedded CSS and JS. It runs by double-clicking the file or opening the URL, with no backend, database, login, external API call, network request, or framework. Opened with no internet connection, it still calculates. *(A build that fetches anything over the network fails this.)*
2. **Inputs.** The page exposes exactly four inputs: (a) number of dogs, (b) a same-day toggle, (c) a holiday toggle, (d) an aggressive/reactive-dog toggle. Number of dogs accepts whole numbers ≥ 1 and defaults to 1, so a valid quote is always on screen. The aggressive/reactive toggle is a single booking-level toggle — it is *not* per dog, because the surcharge it triggers is flat per booking (see Pricing model, step 4).
3. **Live output.** Changing any input updates the displayed quote immediately, with no submit button and no page reload. *(A build with a "Calculate" button fails this.)*
4. **One visible quote.** The page shows a single, unambiguous total price in USD (`$`) to two decimal places (e.g. $15.00). The current inputs it reflects are visible on screen at the same time, so Rosie and the customer can confirm they entered the same thing.
5. **Deterministic.** Identical inputs always produce an identical displayed total. No randomness and no dependence on the current date/time — the same-day and holiday conditions are set by the user via the toggles, never inferred from the system clock. *(If the page reads the clock to auto-decide "holiday," it fails this.)*
6. **Single pricing block.** All editable values — base rate per dog, same-day surcharge amount, aggressive/reactive surcharge amount, currency symbol, and Rosie's phone number — live together in one clearly-labelled place in the file, separated from the calculation, so they are trivial to change without editing logic.
7. **Business name + text-to-book prompt.** The page shows "Rosie's Dog Walking" as a heading and, near the quote, a "Text to book: [number]" line that is a tap-to-text link on mobile. This is static text only — no form, no data captured, nothing sent anywhere.
8. **Readable on phone and desktop.** Inputs are tappable and the quote is legible on a narrow phone screen and on a desktop, without horizontal scrolling.

## Pricing model, edge cases & rules

The quote is computed for **one walk** in this fixed order. The order is deliberate and must be reproducible.

**Fixed factors (from the brief, not editable behaviour):** second dog = 50% of the first dog's rate; holiday multiplier = ×2.
**Editable values (Rosie's, in the pricing block):** `BASE` = rate for one dog; `SAME_DAY` = flat same-day surcharge; `AGGRESSIVE` = flat aggressive/reactive-dog surcharge (ships at `$10`).

Calculation, in order:
1. **Dog charges.** First dog = `BASE`. Second dog = `0.5 × BASE` (half price — **not free, not full price**). Each dog beyond the second = `BASE` (full price — only the second dog is ever discounted).
2. **Holiday.** If the holiday toggle is on, multiply the dog charges (discount included) by 2. Otherwise leave them unchanged.
3. **Same-day.** If the same-day toggle is on, add `SAME_DAY` **once** to the running total. The surcharge is a flat fee per walk, added *after* any holiday doubling, and is **not** itself doubled.
4. **Aggressive / reactive dog.** If the aggressive-dog toggle is on, add `AGGRESSIVE` **once** to the running total. Per issue #1: *"Can we add a $10 surcharge whenever at least one dog in the booking is marked aggressive or reactive?"* and *"It's a flat $10 per booking no matter how many aggressive dogs are in it, not per dog — and it doesn't double on holidays, it just gets added on top after the holiday doubling."* So the surcharge is added at face value, once per booking, **after** the holiday doubling, and is **not** multiplied by dog count and **not** doubled. Steps 3 and 4 are both flat additions after doubling, so their relative order does not change the total.
5. **Round once, at the end,** to two decimal places (nearest cent). No intermediate rounding.

Edge cases:
- **No stacking discounts.** The 2nd-dog half-price reduction is the only discount in the system and is applied at most once (to the second dog only). No discount ever compounds with another. Holiday (×2), same-day (surcharge) and aggressive/reactive (surcharge) are **increases**, not discounts, and always apply when toggled.
- **Aggressive surcharge is flat and count-independent.** One aggressive dog and five aggressive dogs both add `AGGRESSIVE` exactly once. A 1-dog booking with the toggle on gets it too — the toggle means "at least one dog in this booking is aggressive or reactive".
- **Aggressive surcharge never doubles.** On a holiday walk it is added at face value after the ×2, exactly like the same-day surcharge. All three toggles can be on at once: dog charges double, then `SAME_DAY` and `AGGRESSIVE` are each added once on top.
- **Discount survives the holiday rate.** On a holiday walk the 2nd dog stays half price and then the whole walk doubles — the holiday rate does not cancel the dog discount.
- **Both toggles on.** A walk can be both holiday and same-day (a holiday walk booked that morning). Dog charges double, then the flat same-day surcharge is added on top at face value.
- **One dog.** No second-dog discount applies; quote is `BASE` (× holiday, + same-day as applicable).
- **No minimum charge.** The quote is exactly what the rules produce; there is no floor or call-out fee.
- **Invalid input.** Fewer than 1 dog is not a valid walk. Because the dog count defaults to 1 and cannot go below 1, the page never shows `$0.00` as a real quote and never a negative number.

## Decisions from interview

Resolutions of the gaps the brief left open — now settled, recorded here so the intent is traceable:
- **One flat rate per walk.** Price depends only on dog count + toggles; no walk-length or walk-type variable.
- **Only the second dog is discounted;** a third or later dog is full price.
- **The 2nd-dog discount still applies on holidays;** the whole walk (discount included) is what doubles.
- **Same-day surcharge is a flat fee per walk,** added after holiday doubling and not doubled.
- **Aggressive/reactive surcharge is a flat fee per booking** (issue #1), added after holiday doubling, not doubled, and not multiplied by the number of aggressive dogs. It is captured by one booking-level toggle rather than per-dog marking, since the price cannot depend on how many dogs are marked.
- **No minimum charge.**
- **Currency is USD (`$`).**
- **The page shows the business name and a tap-to-text "book" prompt.**
- **Rates ship as clearly-marked placeholders** (`BASE = $10`, `SAME_DAY = $5`, phone = placeholder) for Rosie to replace before sharing the link. `AGGRESSIVE = $10` is not a placeholder — it is the rate Rosie asked for in issue #1 — but it lives in the same editable block so she can change it the same way. The examples below use these values.

## Out of scope

- Booking, scheduling, calendar, or availability.
- Taking payment, deposits, or card details.
- Saving quotes, customer records, or any persistence between visits.
- Accounts, login, or an admin area (Rosie edits rates by editing the file).
- Any form, contact capture, or sending the quote/message anywhere. (The "text to book" prompt is a static link only.)
- Multiple services, recurring-walk packages, distance/travel pricing, or per-breed pricing.
- Any backend, database, analytics, or third-party script.

## Acceptance criteria

Using the values `BASE = $10`, `SAME_DAY = $5`, `AGGRESSIVE = $10` (values only; the *relationships* are what is being tested). Criteria 1–13 assume the aggressive/reactive toggle is **off**:

1. **1 dog, no toggles →** $10.00.
2. **2 dogs, no toggles →** $15.00 (10 + 5). Second dog is half price: the total is 1.5 × `BASE`, never $20 (full) and never $10 (free).
3. **3 dogs, no toggles →** $25.00 (10 + 5 + 10). Third dog is full price.
4. **1 dog, holiday →** $20.00 (10 × 2).
5. **2 dogs, holiday →** $30.00 ((10 + 5) × 2). Discount survives, then the walk doubles.
6. **1 dog, same-day →** $15.00 (10 + 5).
7. **2 dogs, holiday + same-day →** $35.00 ((10 + 5) × 2, then + 5). Surcharge is added after doubling and is not itself doubled.
8. **Same inputs, twice →** identical total both times (determinism); no dependence on today's date.
9. **Dog count cannot drop below 1 →** no $0.00-as-quote, no negative total.
10. **Live update →** changing any input changes the displayed total with no button press and no reload.
11. **Rosie and customer parity →** the same three inputs on the same page yield the same number regardless of who enters them or what device they use.
12. **Offline →** with the network disabled, every criterion above still holds.
13. **Rate edit →** changing `BASE` in the pricing block to $12 makes criterion 1 show $12.00 with no change to calculation logic; changing the phone number updates the text-to-book prompt.
14. **1 dog, aggressive →** $20.00 (10 + 10). The surcharge applies to a single-dog booking whenever that dog is marked aggressive or reactive.
15. **3 dogs, aggressive →** $35.00 (10 + 5 + 10 = 25, then + 10). Flat per booking — it does not scale with dog count.
16. **2 dogs, holiday + aggressive →** $40.00 ((10 + 5) × 2 = 30, then + 10). The surcharge is added after doubling and is not itself doubled — never $50.00.
17. **2 dogs, holiday + same-day + aggressive →** $45.00 ((10 + 5) × 2 = 30, then + 5 + 10). Both flat surcharges are added once each, at face value.
18. **Aggressive toggle off →** every total in criteria 1–7 is unchanged by this feature; the toggle adds nothing when unticked.
19. **Aggressive rate edit →** changing `AGGRESSIVE` in the pricing block to $15 makes criterion 14 show $25.00, with no change to calculation logic.
