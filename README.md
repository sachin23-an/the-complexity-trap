# The Complexity Trap
## A Comparative Analysis of Classical and Machine Learning Trading Strategies on NIFTY 50

**Sachin Kumar | IIT Madras | 2026**
Paper: [The_Complexity_Trap.pdf](paper/The_Complexity_Trap.pdf)
Code: [complexity_trap.ipynb](complexity_trap.ipynb)

**Status:** v1.1 — see [ADDENDUM.md](ADDENDUM.md) for a correction to the
break-even methodology's framing and an arithmetic fix, made after peer
review. The PDF is left unchanged as the original v1 record.

---

## The Central Finding

Machine learning strategies suffer disproportionate performance erosion
from transaction costs relative to their gross edge — the Complexity
Penalty, defined as the difference between gross and net Sharpe ratio,
increases monotonically with model sophistication.

**Note:** on raw net Sharpe over this test window, Random Forest (5.65) and
XGBoost (3.16) still outperform SMA Crossover (0.78) and Bollinger Bands
(1.60) in absolute terms. The Complexity Penalty measures how cost-fragile
a strategy's edge is — it is not a claim that lower-complexity strategies
are the better trade. Both axes matter; see the Addendum for the fuller
picture.

**SMA Crossover remains profitable at costs up to 5.977% per trade.
XGBoost fails at 0.490%. LSTM is already losing money at current
Indian market cost levels (0.14%).**

SMA's break-even cost is 12x XGBoost's and ~20x LSTM's. Measured against
the current baseline cost (0.14%), SMA can absorb costs ~43x higher before
failing — XGBoost only ~3.5x.

This is not because ML models are wrong about direction.
It is because the cost of acting on their predictions erodes
more of the edge they identify, trade for trade, than it does
for lower-frequency strategies. The model is not "wrong" —
its edge is simply more exposed to cost assumptions.

---

## Results (NIFTY 50 | Test Period: Nov 2023 – Dec 2024 | 282 days)

### Gross vs Net Performance

| Strategy | Gross Sharpe | Net Sharpe | Complexity Penalty | Penalty % |
|---|---|---|---|---|
| Buy and Hold | 0.91 | 0.91 | 0.00 | 0.0% |
| SMA Crossover | 0.79 | 0.78 | 0.02 | **1.9%** |
| Bollinger Bands | 1.64 | 1.60 | 0.04 | **2.7%** |
| Random Forest | 6.24 | 5.65 | 0.59 | **9.4%** |
| XGBoost | 3.73 | 3.16 | 0.57 | **15.3%** |
| LSTM | -0.09 | -0.16 | 0.07 | N/A |

**Reading this table correctly:** Random Forest and XGBoost have larger
Complexity Penalties (they lose more of their edge to costs) but still post
the highest *net* Sharpe ratios in this table. Penalty % and net Sharpe
rank differently — that's the point of reporting both, not a contradiction.

### Sensitivity Analysis — Net Sharpe at Different Cost Levels

| Strategy | 0.05% | 0.10% | 0.14% | 0.20% | Break-Even |
|---|---|---|---|---|---|
| Buy and Hold | 0.91 | 0.91 | 0.91 | 0.91 | N/A |
| SMA Crossover | 0.78 | 0.77 | 0.76 | 0.75 | **5.977%** |
| Bollinger Bands | 1.61 | 1.58 | 1.55 | 1.52 | **3.742%** |
| Random Forest | 5.83 | 5.39 | 5.03 | 4.46 | **0.719%** |
| XGBoost | 3.33 | 2.90 | 2.53 | 1.95 | **0.490%** |
| LSTM | -0.22 | -0.28 | -0.33 | -0.40 | **0.291%** |

**Indian market round-trip cost baseline: 0.14%**
At this cost level, LSTM is already unprofitable.
XGBoost survives only ~3.5x current baseline cost before failing.
SMA survives ~43x current baseline cost before failing.

---

## What Is the Complexity Penalty?

```
Complexity Penalty = Gross Sharpe − Net Sharpe
```

A metric introduced in this paper to quantify how much of a strategy's
apparent performance is destroyed by transaction costs, given its actual
trading frequency. A high penalty means a strategy's *edge* is fragile to
cost assumptions — it does **not** by itself mean the strategy is a worse
trade than one with a lower penalty. Compare penalty and net Sharpe
together, not either in isolation.

The penalty rises with trading frequency because each trade incurs a fixed
cost regardless of the size of the edge captured. On daily NIFTY 50 data
with a signal-to-noise ratio of roughly 0.02, that fixed per-trade cost
consumes a larger share of a high-frequency strategy's edge — even when
that strategy's total, uneroded edge remains larger in absolute terms.

---

## Strategy Overview

| # | Strategy | Complexity | Trades/Year | Description |
|---|---|---|---|---|
| 1 | Buy and Hold | Zero | 1 | Passive benchmark |
| 2 | SMA Crossover (20/50) | Low | ~3 | Golden/death cross |
| 3 | Bollinger Bands (20, 2σ) | Low | ~5 | Mean reversion |
| 4 | Random Forest | Medium | ~78 | 15 features, depth-5 |
| 5 | XGBoost | High | ~56 | Gradient boosting, early stopping |
| 6 | LSTM | Very High | ~12 | 60-day sequence, 33k parameters |

---

## Leakage Audit (Appendix B of paper)

Every strategy passed all 5 checks before results were reported:

- Target uses `.shift(-1)` only — no overlapping windows
- All features computed from past data only
- Chronological train/test split — no shuffle
- StandardScaler fitted on training data only
- All signals shifted by 1 day before execution

---

## Key Technical Decisions

**Why depth-5 for Random Forest?**
Financial daily return data has signal-to-noise ratio ~0.02. Unlimited
depth finds noise. Depth-5 with min_samples_leaf=20 forces the model to
learn only robust patterns. Note: this is a specific configuration choice,
not an inherent property of Random Forests — a shallower/deeper tree or a
different probability threshold would change both the trade count and the
resulting Complexity Penalty. See Addendum for more on this.

**Why early stopping for XGBoost?**
Prevents memorisation of training data. Stops when validation logloss
stops improving for 20 consecutive rounds.

**Why does LSTM underperform despite complexity?**
33,451 parameters trained on 1,077 sequences. This is severe data
starvation. Daily equity data provides insufficient examples for a model
this complex to generalise on a 282-day test window.

**Why 0.14% round-trip cost?**
Brokerage (0.03%) + STT (0.025%) + Stamp Duty (0.005%) + estimated
slippage (0.01%) = 0.07% one-way = 0.14% round-trip. Indian transaction
costs are 3-5x higher than US zero-commission brokers, which amplifies
the Complexity Penalty. This is a fixed cost model — see Limitations.

---

## Honest Limitations

- Test window is 282 trading days — short for definitive conclusions
- Single asset (NIFTY 50 index) — not a diversified portfolio
- ML gross Sharpe ratios on short test window may reflect overfitting
  rather than genuine alpha
- Fixed cost model — real costs vary with volatility and order size
- No shorting — long or flat positions only
- **The break-even cost metric is mechanically biased toward
  low-trade-count strategies** — the same total edge divided by fewer
  trades yields a larger per-trade break-even threshold, independent of
  whether the underlying signal is actually more robust. It measures
  cost-elasticity given a strategy's trade count, not signal quality in
  isolation. See [ADDENDUM.md](ADDENDUM.md).
- RF/XGBoost hyperparameters (depth, threshold) are specific configuration
  choices that directly determine trade frequency and therefore the
  reported Complexity Penalty — not an inherent property of "more complex
  models trade more."

---

## How to Reproduce

Open `complexity_trap.ipynb` in Google Colab and run all cells.

```python
# All dependencies installed in first cell
!pip install yfinance xgboost tensorflow
```

Total runtime: approximately 3-5 minutes including LSTM training.

---

## Files

```
the-complexity-trap/
├── README.md
├── ADDENDUM.md              ← correction notes, added after peer review
├── complexity_trap.ipynb    ← complete reproducible code
├── paper/
│   └── The_Complexity_Trap.pdf   ← original v1 paper, unchanged
└── results/
    ├── complexity_trap_main.png
    └── cumulative_returns_sensitivity.png
```

---

## Tech Stack

Python · Pandas · NumPy · Matplotlib · Scikit-learn ·
XGBoost · TensorFlow/Keras · yfinance

---

## Citation

If you use this work, please cite:

Kumar, S. (2026). The Complexity Trap: A Comparative Analysis
of Classical and Machine Learning Trading Strategies on the
NIFTY 50 Index. IIT Madras. github.com/sachin23-an

---

*Before you build a neural network, build a cost model.
Before you optimize for accuracy, optimize for net Sharpe.
And before you celebrate a gross backtest, ask what remains
after the market takes its cut.*

— The Complexity Trap, Conclusion
