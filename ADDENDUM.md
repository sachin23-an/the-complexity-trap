# Addendum (v1.1)

The published paper (v1, PDF) and initial notebook contained two issues, both 
corrected in the notebook going forward:

1. **Framing:** The Abstract and Conclusion emphasize SMA's cost-robustness but 
   do not equally state that, on raw net Sharpe, Random Forest (5.65) and 
   XGBoost (3.16) outperform SMA (0.78) and Bollinger Bands (1.60) over this 
   test window. The Complexity Penalty measures how much of a strategy's edge 
   is cost-fragile — it is not a claim that low-complexity strategies are the 
   better trade.

2. **Arithmetic error:** The paper states SMA's break-even cost (5.977%) is 
   "43 times higher" than LSTM's (0.291%). The correct ratio is ~20.5x. The 
   43x figure is the ratio of SMA's break-even cost to the baseline cost 
   (0.14%), a different comparison, mislabeled in the text.

