# Liquidity Report

A single, self-contained HTML report for estimating how much of a fund's
portfolio can be liquidated over a range of day-horizons. Everything runs
locally in the browser — no server, no internet, no install. Just open the
HTML file and drag in a holdings export.

## Usage

1. Open **`Liquidity_Report.html`** in any modern browser.
2. Drag an OversightSummary `.xlsx` onto the page (or click to browse).
3. Pick a fund (by description or fund ticker), then adjust parameters — the report
   recalculates live.

The report has three tabs:

- **Report** — the fund selector, parameters, and the *Portfolio Percent Liquid by
  Horizon* table.
- **Data** — the selected fund's securities at row level (sortable), including
  `asset_class`, `liquidity_methodology`, and `liquidation_size`, with a totals
  footer that flags when the security weights don't sum to 100%. Includes a
  **Download CSV** button.
- **Notes** — a plain-language explanation of the methodology.

The report reads the OversightSummary worksheet and expects these columns:

| Column | Meaning |
|---|---|
| `fund` | Fund code |
| `fund_description` | Fund name |
| `market_value` | Market value of the security |
| `market_value__pct` | Security weight in the fund, in percentage points (sums to 100) |
| `days_to_liquidate` | Days to liquidate the position at the base ADV assumption |
| `significant_market_impact` | The ADV assumption used per security when the file was produced (standard = 20, i.e. 20% of ADV/day; may differ per security) |

The report date is parsed from the filename pattern `_MMDDYYYY_`
(e.g. `OversightSummary_DFA_06302026_pivot.xlsx` → June 30, 2026).

## Parameters

- **Significant market impact** — the assumed % of ADV sellable per day; higher =
  faster liquidation. On fund selection the report shows the **initial assumption
  used in the file** and flags when it is **not uniform** across securities. Three
  override modes:
  - **Original (from file)** — each security keeps its own value (reproduces the file).
  - **Set all to a constant %** — every security recomputed at one assumed %.
  - **Scale each by %** — multiply every security's own assumption (100 = unchanged,
    200 = double, 50 = half). Preserves each security's relative liquidity, so it's
    the natural choice for files whose assumptions vary by security.
- **M — market value override** (default = the fund's actual total). A larger M
  represents a hypothetically larger fund and is *harder* to liquidate.
- **7 day-horizons** (default 1, 2, 3, 4, 5, 10, 20).

## Methodology

For each security, effective days-to-liquidate:

```
effective_dtl = days_to_liquidate × (significant_market_impact / Z) × (M / fund_total_market_value)
```

At horizon `X`, a security contributes its full weight if it fully liquidates
within `X` days, otherwise `X / effective_dtl` of its weight:

```
sellable_weight = weight × min(1, X / effective_dtl)      where weight = market_value__pct / 100
```

The percent of the portfolio liquidatable in `X` days is the sum of
`sellable_weight` across all securities.

Because both overrides scale `days_to_liquidate` reciprocally, their effects
compose predictably — e.g. doubling **Z** (twice as fast) is exactly offset by
doubling **M** (twice as large), leaving the result unchanged.

## Notes

- All calculation is client-side (the [SheetJS](https://sheetjs.com) parser is
  inlined), so no data ever leaves your machine.
- Any bundled sample data is fictitious (EXAMPLE CORP, etc.).
