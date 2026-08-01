# CLAUDE.md

A single public page for a local dog-walking business, built around an instant quote calculator used by both the owner and her customers.

## Principles
- The quote is the product: a customer and the owner entering the same details must get the same number, instantly and visibly.
- Keep it plain and legible — the owner should be able to open the file and adjust her own prices.

## Constraints
- Single-file web app: one HTML file with embedded CSS and JS. No backend, no database, no login, no external API calls, no frameworks.
- Keep all pricing rules in one clearly-labelled place so rates are trivial to change.

## Definition of done
- Opens by double-clicking the HTML file in any modern browser and works offline; readable on phone and desktop.
- Changing an input updates the correct quote immediately, and the price rules are easy to find and edit.
