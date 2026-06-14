# Delay-Vector Clustering of SPY Daily Returns

Empirical study testing whether clustering delay vectors of daily log-returns
of the SPY ETF can identify local market states with predictive power, under
a strict walk-forward protocol with transaction costs and multiple-testing
correction.

**Result: negative.** The clustering method extracts an interpretable and
reproducible structure (a "rebound after decline" state appears in 64% of
walk-forward windows), but this structure does not translate into out-of-sample
trading advantage. The strategy underperforms passive Buy-and-Hold across all
return metrics, fails the Deflated Sharpe Ratio test (DSR = 0.0092), and is
statistically indistinguishable from random trading (permutation p-value = 0.306).

This is the empirical companion to a graduate course project at HSE
(Higher School of Economics), Faculty of Computer Science. The work
demonstrates rigorous methodology for testing market-predictability hypotheses
rather than a profitable trading system.

## Why this project

The hypothesis is grounded in nonlinear dynamics: if a financial market has
local predictability, it should appear as regions of the delay-vector phase
space with statistically distinct future returns. Such regions can in principle
be identified by clustering. The question is whether this approach yields a
*tradeable* signal on a liquid instrument under a rigorous evaluation protocol.

Many studies in algorithmic trading suffer from overfitting, look-ahead bias,
and absence of multiple-testing correction, leading to inflated and
non-reproducible results. This project applies a deliberately strict protocol
to a single liquid instrument (SPY, 2005–2025) and reports an honest negative
result, which is itself a methodological contribution.

## Method

Daily SPY log-returns are embedded into delay vectors v[t] = (r[t], r[t−τ],
..., r[t−(m−1)τ]). On each walk-forward window, k-means clusters the training
delay vectors; each cluster receives a statistical "passport" (mean future
return μ̂, standard deviation, t-statistic, sample size). Clusters passing
informativeness filters generate long signals; otherwise the strategy holds
cash. Hyperparameters (m, τ, k, prediction horizon h) are selected on a
validation window, then applied to a non-overlapping holdout block. Holdout
blocks are concatenated into a pseudo-out-of-sample (pOOS) series.

**Protocol parameters:**
- Train window: 1260 trading days (~5 years)
- Validation window: 252 days (~1 year)
- Holdout window: 63 days (~1 quarter)
- Step: 63 days (non-overlapping holdouts)
- Transaction costs: 5 bps per side
- Execution: next adjusted open
- Strategy type: long/flat
- Parameter grid: m ∈ {3..8}, τ ∈ {1,2,3,5}, k ∈ {4,6,8,10,12}, h ∈ {1,5,10,20}

**Number of walks:** 59. **pOOS length:** 3717 trading days (2011-01-05 to 2025-10-15).

## Robustness checks

The negative result is supported by four independent diagnostics:

1. **Subperiod analysis** — strategy underperforms Buy-and-Hold across all four
   calendar subperiods; the gap is smallest in non-trending periods (2022–2025).
2. **Alternative clustering algorithms** — replacing k-means with GMM
   (diagonal covariances) or z-standardized k-means yields equally weak results
   (Sharpe 0.06–0.18); the weakness is in the approach, not the algorithm.
3. **Deflated Sharpe Ratio** — after correcting the in-sample selection of the
   best of 480 configurations per walk and accounting for heavy left tails
   (skewness −1.7, kurtosis 30.9), DSR = 0.0092. The observed daily Sharpe is
   below the null-hypothesis expected maximum.
4. **Permutation test (placebo)** — the real strategy's Sharpe (0.179) lies at
   the 69th percentile of 1000 random strategies with the same number of trades
   and the same distribution of holding horizons, giving a p-value of 0.306.
   The signal is statistically indistinguishable from random trading.

## Headline results

| Metric              | Strategy | Buy-and-Hold | Momentum |
|---------------------|---------:|-------------:|---------:|
| Annualized Sharpe   | 0.179    | 0.780        | −0.249   |
| CAGR                | 2.08%    | 13.90%       | −2.69%   |
| Annualized vol      | 11.51%   | 16.68%       | 10.94%   |
| Max drawdown        | 30.58%   | 32.05%       | 41.68%   |
| Exposure            | 39.1%    | 100%         | 56.2%    |

The strategy's structurally low exposure (61% of time in cash) on a strongly
trending market is itself enough to explain the underperformance, before any
question of signal quality.

## Repository structure
spy_clustering_fedorov.ipynb        Main Google Colab notebook with all calculations

data/

SPY_raw_yahoo_2005_2025.csv       Fixed daily SPY data used in all runs

requirements.txt                    Python dependencies

README.md


## Data

Daily SPY OHLCV from Yahoo Finance, 2005–2025. The data are **frozen in a CSV
file** and read only from disk — Yahoo Finance is not queried at runtime.
This is critical for reproducibility: Yahoo retrospectively adjusts historical
prices for dividends, so a fresh download produces slightly different values
on each run, which destabilizes the entire pipeline. The frozen file ensures
that any future re-run reproduces the reported numbers exactly.

## Running the notebook (Google Colab)

The notebook ships with `USE_GOOGLE_DRIVE = False`, so Google Drive is **not**
mounted and no folders are created in the runner's Drive.

1. Open `spy_clustering_fedorov.ipynb` in Google Colab.
2. Run the first cell — it creates `/content/data/` inside the Colab session.
3. Upload `SPY_raw_yahoo_2005_2025.csv` into `/content/data/`
   (drag from `data/` of this repo into the Colab file browser on the left,
   inside the `data` folder).
4. Runtime → Run all.

Full run takes ~10 minutes (the main walk-forward loop is ~9 min; alternative
algorithms ~30 sec each; permutation test ~10 sec). Results are written to
`/content/SPY_clustering_project/{figures,tables,metrics}/` inside the Colab
session and cleared on session restart — fine for one-off verification.

**For reproducibility:** start from a clean kernel (Runtime → Restart session)
and execute Runtime → Run all from top to bottom. Running cells out of order
on top of stale state can yield mixed results.

## Reproducibility

With `random_state = 42` and the frozen data file, the pipeline is
deterministic: independent clean runs produce identical numbers (Sharpe 0.179,
DSR 0.0092, permutation p-value 0.306, representative walk 48, etc.).
Floating-point determinism was verified by running the full walk-forward
twice in succession on a clean kernel.

## Dependencies

See `requirements.txt`. Core packages: numpy, pandas, scikit-learn, scipy,
matplotlib. Tested on Python 3.10+ in Google Colab.

## Honest scope

The result applies to **one instrument** (SPY), **daily frequency**,
**2005–2025**, and the **specific protocol** described above. Stronger
or weaker conclusions on other instruments, frequencies, or periods would
require separate validation. The work is best read as a methodological
template for testing market-predictability hypotheses under strict conditions,
and as evidence that interpretability and in-sample statistical separation
of clusters are not sufficient grounds for a trading decision.

## License

Academic / non-commercial use. Please cite the course project if reused.
