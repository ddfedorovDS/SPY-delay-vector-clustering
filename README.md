# SPY Delay Vector Clustering

This repository contains the empirical code for a course project on generating trading signals from clustering delay vectors of daily log-returns of SPY ETF.

## Project description

The project tests whether delay vectors constructed from daily log-returns of SPY can be used as local market-state representations. The delay vectors are clustered using k-means. Cluster-level statistics are then used to generate long/flat trading signals under a walk-forward validation protocol.

The method is treated as feature engineering rather than strict attractor reconstruction.

## Repository structure

* `course2_structured.ipynb` — main Google Colab notebook with all calculations;
* `data/` — raw and prepared SPY price data;
* `tables/` — exported result tables;
* `figures/` — figures used in the empirical chapter;
* `metrics/` — additional metrics and diagnostic outputs;
* `requirements.txt` — Python package requirements.

## Data

The data are daily SPY prices from Yahoo Finance for the period 2005–2025. The notebook uses `Open`, `Close`, and `Adjusted Close` to construct adjusted open prices and log-returns.

## Method

The experiment uses a walk-forward protocol:

* train window: 1260 trading days;
* validation window: 252 trading days;
* holdout window: 63 trading days;
* step: 63 trading days;
* transaction costs: 5 bps per side;
* execution: next adjusted open;
* strategy type: long/flat.

Model parameters are selected on validation windows. Final performance is computed only on non-overlapping holdout blocks.

## How to run

1. Open `course2_structured.ipynb` in Google Colab.
2. Install required packages from `requirements.txt`, if needed.
3. Run all notebook cells from top to bottom.
4. Output tables are saved to `tables/`, figures to `figures/`, and metrics to `metrics/`.

## Main result

The strategy does not outperform Buy-and-Hold SPY on the pseudo-out-of-sample period. The empirical results are interpreted as a test of whether delay-vector clustering can identify local market states with different future return profiles, not as evidence of a stable trading advantage.
