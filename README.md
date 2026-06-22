# Delay-Vector Clustering of SPY Daily Returns

Empirical study testing whether clustering delay vectors of daily log-returns
of the SPY ETF can identify local market states with predictive power,
under a strict walk-forward protocol with transaction costs and
multiple-testing correction.

**Result: negative.** The method extracts an interpretable and reproducible
structure (a "rebound after decline" state appears in 64% of walk-forward
windows), but this structure does not translate into out-of-sample trading
advantage. The strategy underperforms passive Buy-and-Hold, fails the
Deflated Sharpe Ratio test (DSR = 0.0092), and is statistically
indistinguishable from random trading (permutation p-value = 0.306).

## Method

Daily SPY log-returns are embedded into delay vectors
v[t] = (r[t], r[t−τ], ..., r[t−(m−1)τ]). On each walk-forward window,
k-means clusters the training delay vectors; each cluster receives a
statistical passport (mean future return μ̂, standard deviation,
t-statistic, sample size). Clusters passing informativeness filters
generate long signals; otherwise the strategy holds cash. Hyperparameters
(m, τ, k, prediction horizon h) are selected on a validation window,
then applied to a non-overlapping holdout block. Holdout blocks are
concatenated into a pseudo-out-of-sample (pOOS) series of 3717 days
(2011-01-05 to 2025-10-15).

**Protocol:** train 1260 days / validation 252 / holdout 63 / step 63;
transaction costs 5 bps per side; execution at next adjusted open;
long/flat strategy. Parameter grid: m ∈ {3..8}, τ ∈ {1,2,3,5},
k ∈ {4,6,8,10,12}, h ∈ {1,5,10,20} — 480 candidate configurations
per walk, 59 walks.

## Headline results

| Metric            | Strategy | Buy-and-Hold | Momentum |
|-------------------|---------:|-------------:|---------:|
| Annualized Sharpe |    0.179 |        0.780 |   −0.249 |
| CAGR              |    2.08% |       13.90% |   −2.69% |
| Max drawdown      |   30.58% |       32.05% |   41.68% |
| Exposure          |    39.1% |         100% |    56.2% |

The strategy's structurally low exposure (61% of time in cash) on a
trending market is itself sufficient to explain underperformance,
before any question of signal quality.

## Robustness checks

The negative result is supported by five independent diagnostics:

- **Subperiod analysis** — strategy underperforms Buy-and-Hold in all
  four calendar subperiods; gap smallest in non-trending 2022–2025.
- **Drawdown-period analysis** — on days when Buy-and-Hold is in drawdown,
  the strategy loses less at every threshold (+9 to +16 pp), and in the
  slow 2022 bear market it even turns positive (+4.9% vs −21.6%). But this
  protection is structural (low exposure), not predictive: in the fast
  COVID-2020 crash the strategy lost *more* than Buy-and-Hold (−20.2% vs
  −12.8%), having been caught long.
- **Alternative algorithms** — k-means, z-standardized k-means, and
  GMM (diagonal) all give Sharpe 0.06–0.18; weakness is in the approach,
  not the algorithm.
- **Deflated Sharpe Ratio = 0.0092** — after correcting for selection
  of the best of 480 configurations per walk and for heavy left tails
  (skewness −1.7, kurtosis 30.9).
- **Permutation test, p-value = 0.306** — real Sharpe lies at the 69th
  percentile of 1000 random strategies with matched trade counts and
  holding horizons. Statistically indistinguishable from random trading.

## Repository structure
spy-delay-clustering/

├── spy_clustering_fedorov.ipynb   # main Colab notebook

├── data/

│   └── SPY_raw_yahoo_2005_2025.csv

├── requirements.txt

└── README.md

## Data and reproducibility

Daily SPY OHLCV from Yahoo Finance, **frozen** in
`data/SPY_raw_yahoo_2005_2025.csv` and read only from disk. Yahoo
retrospectively adjusts historical prices for dividends, so a fresh
download produces slightly different values on each run; freezing the
data ensures exact reproducibility.

With `random_state = 42` and the frozen file, the pipeline is
deterministic — independent clean runs produce identical numbers
(verified by running the full walk-forward twice on a clean kernel).

## Running in Google Colab

The notebook ships with `USE_GOOGLE_DRIVE = False` — Drive is not
mounted, no folders are created on the runner's Drive.

1. Open `spy_clustering_fedorov.ipynb` in Google Colab.
2. Run the first cell — it creates `/content/data/` in the session.
3. Upload `SPY_raw_yahoo_2005_2025.csv` into `/content/data/`
   (drag from this repo's `data/` folder into Colab's left panel).
4. Runtime → Run all.

Full run ~30 min. Results saved to
`/content/SPY_clustering_project/{figures,tables,metrics}/` in the
session. For reproducibility, always start from a clean kernel
(Runtime → Restart session) before Run all.

## Dependencies

See `requirements.txt`: numpy, pandas, scikit-learn, scipy, matplotlib.
Tested on Python 3.10+ in Google Colab.

## Scope

Results apply to one instrument (SPY), daily frequency, 2005–2025, and
the specific protocol above. The work is best read as a methodological
template for testing predictability hypotheses under strict conditions,
and as evidence that interpretability and in-sample statistical
separation of clusters are not sufficient grounds for a trading decision.
