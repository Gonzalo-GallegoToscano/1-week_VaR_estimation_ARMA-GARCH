# 1-week VaR for a SPX and DAX portfolio: ARMA-GARCH marginals and a copula

Group coursework. This was joint work by six students. authorship notes are at the bottom. I host the repo, the
work is the group's.

We estimate 1-week 95% and 99% Value at Risk for an equally weighted,
weekly rebalanced portfolio of the S&P 500 and the DAX, using weekly data
from 2000 to 2025. The point of the exercise is that classical VaR leans
on joint normality and linear correlation, and equity returns break both:
heavy tails, volatility clustering, and dependence that strengthens
exactly in the crashes. So the dependence gets its own model, a copula,
fitted separately from the marginals.

## What is in here

| file | what it is |
|---|---|
| `var_spx_dax.Rmd` | the full analysis: diagnostics, marginals, copula, VaR, backtests |
| `references.bib` | bibliography |
| `report.pdf` | the knitted report |
| `data/` | gitignored, see below for how to get the two series |

## Data

Two weekly close-price series, 2000 to 2025: the S&P 500 and the DAX. We
used the free downloads from stooq.com (tickers `^spx` and `^dax`, weekly
interval), which arrive named `^spx_w.csv` and `^dax_w.csv`. Place both
files in `data/` and the Rmd finds them.

## The model in short

The workflow follows the logic that a static copula needs approximately
i.i.d. inputs, and raw returns are not that. So: exploratory diagnostics
first (heavy tails, autocorrelation, volatility clustering, all present),
then ARMA-GARCH marginals per index to soak up the dynamics, AR(1) for
the S&P and AR(3) for the DAX by information criteria, each with a
GARCH(1,1) variance and skewed Student-t innovations. The standardised
residuals go through the probability integral transform, and the copula
is chosen on the resulting pseudo-observations by AIC among Gaussian, t,
Clayton, Gumbel and BB1 families. The winner is a Survival BB1 with
(theta, delta) = (0.24, 1.85) and Kendall's tau = 0.52, which is the
family that allows the asymmetric tail dependence the data shows: the two
indices crash together more strongly than they rally together. VaR then
comes from Monte Carlo, 50,000 joint draws through the fitted copula and
marginals.

## Results

For the equal-weight weekly rebalanced portfolio, the 1-week estimates
are VaR at 95% of 2.89% and at 99% of 4.98%. Kupiec's unconditional
coverage test and Christoffersen's independence test both pass at both
levels, so the exceedances are as frequent as they should be and do not
cluster.

Three limitations we state rather than hide. The uniformity diagnostics
show residual misspecification in the S&P marginal, which feeds into the
copula. The model-based VaR is noticeably less conservative than the
historical empirical quantiles, because it conditions on current GARCH
volatility instead of averaging over all regimes, worth understanding
before using numbers like these. And copula selection relied on AIC and
visual checks, with no formal goodness-of-fit test.

## Running it

R with `tseries`, `fGarch`, `KScorrect`, `VineCopula`, `goftest` and
`rugarch`, then knit `var_spx_dax.Rmd` (RStudio knit button, or
`rmarkdown::render("var_spx_dax.Rmd")`).

## Authorship

This was a six-person group project at UCL. The analysis and code
were produced jointly by 2 coursemates and me (Gonzalo Gallego
Toscano). the group split roughly between code and report writing, and
both halves are joint work. I prepared this public version of the repo,
the credit is shared.
