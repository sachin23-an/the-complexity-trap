# Addendum (v1.1)

The published paper (v1, PDF) and the initial version of the notebook contained
two issues. Both are corrected in the notebook going forward; the PDF itself is
left unchanged as the original record.

## 1. Framing — the Complexity Penalty is not a "which strategy wins" metric

The Abstract and Conclusion emphasize SMA Crossover's cost-robustness but do
not equally state that, on **raw net Sharpe ratio**, Random Forest (5.65) and
XGBoost (3.16) still outperform SMA Crossover (0.78) and Bollinger Bands
(1.60) over this test window.

The Complexity Penalty (Gross Sharpe − Net Sharpe) and the break-even cost
threshold measure how much of a strategy's edge is **fragile to transaction
costs**. They do not measure which strategy has the better absolute edge.
A strategy can have a small Complexity Penalty and still be a worse trade
than one with a larger penalty, if its raw edge is small enough. That is the
case here: SMA and Bollinger Bands are more cost-robust, but Random Forest
and XGBoost still win on raw net Sharpe over this specific test window.

**Revised takeaway:** transaction-cost sensitivity is a necessary axis of
strategy evaluation that the literature under-reports — not a claim that
simple strategies are the better trade. Both axes (net Sharpe and cost
robustness) matter, and reporting only one is what the paper originally
argued against; the same standard now applies to its own conclusion.

## 2. Arithmetic error — the "43x" comparison

The paper states that SMA Crossover's break-even cost (5.977%) is
**"43 times higher"** than LSTM's break-even cost (0.291%).

The correct ratio is:
```
5.977 / 0.291 ≈ 20.5x
```

The 43x figure is actually the ratio of SMA's break-even cost to the
**baseline round-trip cost** (0.14%):
```
5.977 / 0.14 ≈ 42.7x ≈ 43x
```

These are two different comparisons that were conflated in the Abstract and
Conclusion. Both figures are individually correct; they should not have been
presented as the same ratio.

| Comparison | Ratio |
|---|---|
| SMA break-even ÷ LSTM break-even | ~20.5x |
| SMA break-even ÷ baseline cost (0.14%) | ~43x |
| SMA break-even ÷ XGBoost break-even | ~12x |
| XGBoost break-even ÷ baseline cost (0.14%) | ~3.5x |

## Status

- **PDF (v1):** unchanged, left as the original published record.
- **Notebook:** updated with a revision note and corrected summary output.
- **README:** updated wording, see repo.

Thanks to Avi Khobragade for the review that prompted this correction.
