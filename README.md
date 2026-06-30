# The Complexity Trap
## A Comparative Analysis of Classical and ML Trading Strategies on NIFTY 50

**Sachin Kumar | IIT Madras | 2026**  
Paper: [The_Complexity_Trap.pdf](/The_Complexity_Trap_Paper.pdf)  
Code: [complexity_trap.ipynb](/The_Complexity_Trap_NIFTY50_Research.ipynb)

---

## The Central Finding

Machine learning strategies suffer disproportionate performance 
erosion from transaction costs. The Complexity Penalty — defined 
as the difference between gross and net Sharpe ratio — increases 
monotonically with model sophistication..

**SMA Crossover remains profitable at costs up to 5.977% per trade.  
XGBoost fails at 0.490%. LSTM is already losing money at current 
Indian market cost levels (0.14%).**

SMA is 12x more robust to cost variation than XGBoost.  
SMA is 21x more robust to cost variation than LSTM.

This is not because ML models are wrong about direction.  
It is because the cost of acting on their predictions exceeds  
the edge they identify. The model is not the problem.  
The frequency is.

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
XGBoost survives only 3.5x before failing.  
SMA survives 43x before failing.

---

## What Is the Complexity Penalty?
A novel metric introduced in this paper to quantify how much of 
a strategy's apparent performance is destroyed by transaction 
costs. Strategies with low penalties are robust. Strategies 
with high penalties are paper tigers — impressive in backtests, 
dangerous in live trading.

The penalty increases because complex models trade more 
frequently. More trades means more costs. On daily NIFTY 50 
data with signal-to-noise ratio of approximately 0.02, the 
cost of acting on ML signals exceeds the edge those signals 
identify.

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
Financial daily return data has signal-to-noise ratio ~0.02. 
Unlimited depth finds noise. Depth-5 with min_samples_leaf=20 
forces the model to learn only robust patterns.

**Why early stopping for XGBoost?**  
Prevents memorisation of training data. Stops when validation 
logloss stops improving for 20 consecutive rounds.

**Why does LSTM underperform despite complexity?**  
33,451 parameters trained on 1,077 sequences. This is severe 
data starvation. Daily equity data provides insufficient 
examples for a model this complex to generalise.

**Why 0.14% round-trip cost?**  
Brokerage (0.03%) + STT (0.025%) + Stamp Duty (0.005%) + 
estimated slippage (0.01%) = 0.07% one-way = 0.14% round-trip. 
Indian transaction costs are 3-5x higher than US zero-commission 
brokers, which amplifies the Complexity Trap.

---

## Honest Limitations

- Test window is 282 trading days — short for definitive conclusions
- Single asset (NIFTY 50 index) — not a diversified portfolio
- ML gross Sharpe ratios on short test window may reflect 
  overfitting rather than genuine alpha
- Fixed cost model — real costs vary with volatility and order size
- No shorting — long or flat positions only

---

## How to Reproduce

Open `complexity_trap.ipynb` in Google Colab and run all cells.

```python
# All dependencies installed in first cell
!pip install yfinance xgboost tensorflow
```

Total runtime: approximately 3-5 minutes including LSTM training.

---

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
