---
title: "Stock Correlation Heatmap"
description: "Quantitative pipeline that computes dual Pearson/Spearman correlation matrices, reorders assets via Ward hierarchical clustering, and outputs both an interactive Plotly dashboard and a static clustermap."
tech: ["Python", "Pandas", "Plotly", "SciPy"]
github_link: "https://github.com/articwnd/Heat-Map"
year: 2026
featured: true
---

## Problem

Cross-sectional momentum and other multi-asset strategies need a fast, reliable
way to see how a universe of equities co-moves before any portfolio construction
or risk management decision gets made. A naive approach (plot one correlation
number per pair, eyeball it) does not scale past a handful of tickers and hides
two things that matter most: which assets cluster into the same effective bet,
and whether the linear correlation estimate itself can be trusted.

A correlation heatmap solves the visualization problem on the surface, but most
implementations stop at `seaborn.heatmap(df.corr())` and call it done. That
naive version has real failure modes that go undetected until they corrupt
downstream work: silent shape bugs at small N, inconsistent NaN handling
between estimators, and treating the raw Pearson matrix as production-ready
input to a portfolio optimizer when it may not even be positive semi-definite.

## Approach

Built a Python pipeline (`correlation_heatmap.py`) that goes beyond a single
static plot:

- **Data layer:** `yfinance` ingestion with a documented swap-in path to
  OpenBB or Polygon.io for production use, plus an explicit data-quality
  diagnostic step that reports missing observations per ticker before any
  computation runs.
- **Returns, not prices:** daily log returns, since price-level correlation
  is confounded by shared trend and is not the quantity that matters for risk.
- **Dual correlation estimators:** Pearson (linear) and Spearman (rank-based,
  robust to fat tails) computed side by side, with both built on pandas'
  pairwise-complete convention so the two estimators are comparable on a
  consistent sample.
- **Hierarchical clustering reorder:** assets reordered via Ward linkage on a
  correlation-distance metric (`d = sqrt(0.5 * (1 - rho))`), so the heatmap
  reveals block structure (sector clusters, risk-on/risk-off groupings)
  instead of showing tickers in arbitrary input order.
- **Dual output:** an interactive Plotly dashboard (hover reveals both
  correlation estimates plus per-asset annualized volatility) for
  exploration, and a static seaborn clustermap with dendrogram for reports.
- **Automated divergence flagging:** any pair where Pearson and Spearman
  disagree by more than 0.10 is surfaced in a console summary, since large
  divergence signals non-linearity or outlier distortion the Pearson number
  alone would hide.

The project also went through an explicit review pass that caught and fixed
two silent correctness bugs (detailed in Tradeoffs below), which shaped the
final implementation more than the original design did.

## Tradeoffs

**scipy vs. pandas for Spearman correlation.** The first implementation used
`scipy.stats.spearmanr`, which returns a bare scalar (not a matrix) when given
exactly two columns and defaults to `nan_policy="propagate"`, meaning a single
missing observation nulls out an entire asset's row. Both bugs were silent: no
crash, no warning, just wrong or missing numbers downstream. Switched to
`pandas.DataFrame.corr(method="spearman")`, which is shape-correct at any N
and pairwise-complete by default, consistent with the Pearson estimator.
Tradeoff: pandas' pairwise-complete convention means each pairwise entry can
be estimated on a different effective sample if tickers have different data
availability, so the resulting matrix is not guaranteed to be positive
semi-definite as a whole. That is acceptable for visualization and clustering
(the distance metric is computed entrywise) but is documented as a hard
blocker for direct use in mean-variance optimization without an additional
PSD-projection step.

**Pearson as the clustering driver, Spearman as a cross-check, not a second
heatmap.** Showing both as full heatmaps would double the visual real estate
for marginal benefit. Surfacing Spearman in the hover tooltip keeps the
primary view clean while still making the cross-check one hover away.

**Ward linkage over single linkage.** Ward minimizes within-cluster variance
at each merge and produces more balanced, interpretable clusters for
correlation-distance data. Single linkage is more sensitive to chaining
(stringing together loosely related assets through intermediate
nearest-neighbor links) and was rejected for that reason, at the cost of being
a slightly more expensive computation.

**yfinance over a paid data vendor.** Free and sufficient for research and
prototyping, but unofficial and not licensed for commercial use. The
`fetch_prices()` function is isolated specifically so this is a one-function
swap to OpenBB or Polygon.io rather than a rewrite.

## Outcome

A reusable, documented pipeline that produces both an interactive
exploration tool and a publication-ready static figure from a single script
run, with pinned dependencies (`requirements.txt`) for reproducibility and a
README covering methodology, configuration, and data-source swap instructions.

Beyond the artifact itself, the project surfaced two genuine correctness bugs
through systematic verification (testing against actual library behavior at
edge cases like N=2 and injected NaNs, not just happy-path runs), which is the
habit that matters more than the script. The codebase is structured as the
base layer for further work already scoped in the README: rolling
correlations for regime detection, feeding the (PSD-corrected) covariance
matrix into a Markowitz optimizer, and Hierarchical Risk Parity using the same
Ward linkage already computed here.
