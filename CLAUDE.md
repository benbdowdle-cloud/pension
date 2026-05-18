# illdoitlater — Pension Fee Comparison

Single-file static site (`index.html`). Compares UK workplace pension providers
on fees and shows the long-term opportunity cost of staying with a high-fee
provider.

## Architecture

- Everything lives in `index.html`: CSS in `<style>`, JS in `<script>` at the end.
- Provider fee data loaded at runtime from a Google Sheet CSV via
  `loadSheetData()` into the `PFEES` object.
- Three SPA "pages" (`form-page`, `switch-page`, `fees-page`) toggled by
  `showPage()`.

## Projection engine (index.html ~lines 999-1140)

Single `runProjection(start, monthly0, years, fees)` function does
month-by-month compounding for both the fee and zero-fee baselines via the
`NO_FEES` constant. `calcFees` returns `{pot, total}` where `total` is
`max(0, noFeePot - feePot)`.

Globals: `GROWTH` (default 9.1%), `INFLATION` (default 2%),
`CONTRIB_GROWTH` (default 2%). All wired to sliders in the assumptions panel.

Key behaviours:
- Contributions are uprated yearly by `CONTRIB_GROWTH`.
- AMC is recalculated each month from the current pot (handles tier
  boundaries correctly).
- Tier helper `tieredAmcAnnual(pot, amcPct, tiers)` applies tiers marginally.
- Final pot is discounted to today's money via `INFLATION`.

## Copy framing rule (important)

The "saving" figure is **opportunity cost** (the compounded growth lost to
fees), not the fees actually charged. Copy must reflect this:

- "extra at retirement" / "more at retirement" / "less lost to fees"
- NEVER "saved", "fees avoided", "save up to"

Only `EXPECTED FEES` (in red on the cards) is an actual fee projection.

## Style rules

- **No em dashes** anywhere in copy. Use commas, colons, or rephrase.
- Form inputs need `font-size: 16px` under 600px so iOS Safari doesn't
  zoom-and-shift on focus. Already in the `@media(max-width:600px)` block.
- `html,body { overflow-x: hidden }` as a viewport guard.

## Google Sheet columns

`Provider`, `Annual Management Charge %`, `Fund Charge (%)`,
`Platform Fixed Charge (Annual £)`, `Contribution Charge (%)`,
`Assumed Fund`, `Notes`, `Tiers` (JSON), `url`, `last_verified`, `source`.

## Comparison card fee row

Five columns: Annual fee · Fund fee · Contribution fee · Projected fees by
retirement · Projected pot at retirement. Fixed yearly fee is **not**
displayed on the card but is still factored into the projection.

Switch button renders as `<a target="_blank" rel="noopener noreferrer">` when
`PFEES[name].url` starts with `http(s)://`, otherwise an inert `<button>`.
Notes render below the fee row only when `notes` is truthy.

## Git / deployment

- Designated dev branch: `claude/check-github-access-Dfu3a`.
- User has authorised pushing directly to `main` when asked.
- Container is ephemeral, so everything must be committed and pushed to persist.
