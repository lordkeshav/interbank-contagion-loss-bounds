# interbank-contagion-loss-bounds
Payment-conservation loss bounds and graph-learned network reconstruction for interbank contagion under partial disclosure



Code, fitted models and simulation results for the manuscript



## Overview

When supervisors see only part of the bilateral interbank exposure matrix but know each bank's exact totals, many networks are consistent with the same data. This project:

- computes a **certified interval** for the stress loss passed to creditors outside the banking system under Eisenberg–Noe proportional clearing, combining an observed-only lower bound, a creditor-by-creditor upper bound and a new **payment-conservation cap**;
- audits how tight the bounds are, **exactly** on small networks (two hidden borrower rows) and with **feasible adverse completions** on 32-bank networks;
- compares point reconstructions that respect all disclosed totals: uniform RAS, a pairwise scorer, two MLPs without message passing (one parameter-matched), a **directed graph neural network (GNN-RAS)** and a reciprocal sparse reconstruction;
- runs market-calibrated experiments based on published Argentine, Mexican and Turkish interbank statistics, plus a 65-bank factorial design.

All bilateral exposures, balance sheets and stress scenarios are **synthetic**; no confidential supervisory data are used or required.

## Requirements

- Python 3.12
- Packages pinned in `requirements.txt` (NumPy, pandas, SciPy, scikit-learn, Matplotlib)

```sh
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## What the study uses

Every bilateral exposure, bank balance sheet and stress scenario in the simulations is generated. The modular and core–periphery experiments use network-generation seeds 60431 and 190517. Each run generates 480 training networks, 120 validation networks and 450 unique test networks. A network has nine shock scenarios. Reporting variants reuse the same underlying network and shocks. Exact balance-sheet margins are calculated from each generated network and provided as disclosed inputs. Full bilateral data are used for supervised training and held-out scoring.

The largest-first masks disclose the same exposures in both information sets. In the known-cutoff setting the smallest reported exposure caps each hidden exposure. The unknown-rule case uses the observed entries and margins. The conservation cap and illustrative 10, 25 and 50 bp decision thresholds apply to incremental outside-creditor losses. These thresholds are simulation inputs for the screening exercise.

The cross-market reference uses published summary statistics from Forte (2020) and Batiz-Zuk et al. (2016). The controlled 65-bank experiment varies density, total exposure share and uniform disclosure in a full factorial design with matched seeds. The 48-bank single-market experiment uses the bank count, on-balance-sheet density and range of exposure-to-assets shares reported for Türkiye by Sümer and Özyıldırım (2022). Its exposure levels are 0.90%, 2.00% and 3.10%. The middle value is the midpoint of the published range. Tables 3 and 4 and Section 2 in that article supply the targets. Reciprocity in those networks is generated. Source links and metadata are in `references.json`.

## Reproduce the results

Install the exact versions in `requirements.txt` with Python 3.12. Run the following commands from the extracted archive. The archive already contains the inputs, weights and published numerical results. Timing in the manuscript uses a single-threaded numerical backend on an AMD EPYC 9V74 CPU and excludes feature construction, benchmark truth and bounds. Runtime can vary with hardware and library configuration.

Set the numerical libraries to one thread before rerunning the estimates.

```sh
export OPENBLAS_NUM_THREADS=1 MKL_NUM_THREADS=1 OMP_NUM_THREADS=1
python verify.py --extension extension results replication
python verify_review.py
python derived_statistics.py --root .
```

The next commands recreate the analysis in a separate directory. The initial models and exact two-debtor audit precede the expanded comparisons. The complete runs use the seeds in the released configuration, and may take several minutes.

```sh
python study.py --network-seed 60431 --output rerun/results
python study.py --network-seed 190517 --output rerun/replication
python reconstruction_extension.py --root rerun --output rerun/extension
python bound_audit.py --output rerun/extension
python review_experiments.py --root rerun --output rerun/review_results
python adverse_audit.py --root rerun --output rerun/review_results
python verify_review.py --review rerun/review_results
python figures.py --results rerun/results --review rerun/review_results --output rerun/figures
python derived_statistics.py --root rerun
```

## Files

| Path | Contents |
|---|---|
| `study.py`, `reconstruction_extension.py` | Core model fitting, clearing, margin and cutoff projections, and reciprocal support reconstruction |
| `bound_audit.py`, `adverse_audit.py` | Exact extrema for two-row hidden blocks, payment-conservation checks, and feasible adverse searches on 32-bank masks |
| `review_experiments.py` | Capacity-matched comparator, cutoff conditions, paired intervals, runtime, screening and sensitivity experiments |
| `results/`, `replication/`, `extension/` | Initial weights, scenarios, support benchmark and original exact small-network audit |
| `review_results/results/`, `review_results/replication/` | Seven disclosure regimes per run, matched model weights, support model coefficients, metrics and scenario records |
| `review_results/validation/` | Cross-market and single-market calibrations and eight factorial cells |
| `review_results/small_audit.csv`, `review_results/large_audit.csv` | Recalculated exact-mask caps and feasible adverse losses on larger masks |
| `verify.py`, `verify_review.py`, `tables.py` | Consistency checks and the numerical tables and manuscript values |
| `derived_statistics.py` | Interval validity, conservation-cap binding rates, errors by shock type and audit tightness shares reported in Sections 7.2, 7.4 and 7.5 |
| `figures.py`, `figures/` | Figure generation and seven editable vector EPS files |
| `references.json` | Literature metadata, calibration sources and their associated claims |

In the revised scenario files, `true_bps` uses the full generated network, and `predicted_bps` uses the specified estimator. `lower_bps` is observed-only clearing, `column_bps` is the columnwise upper bound, `conservation_bps` is the conservation cap and `upper_bps` is the smaller of the two upper bounds. All are incremental outside-creditor losses scaled by nominal outside liabilities. Each summary reports the count of projections that switch to the L1 feasibility repair. There are six member allocations per network for the reciprocal sparse model.

In `small_audit.csv`, `minimum_feasible` and `maximum_feasible` are the attained extrema. In `large_audit.csv`, `adverse` is the largest loss found by the feasible search. Its gap to the upper bound includes possible search slack. The search uses disclosed inputs and shocks. The full exposure matrix enters the larger-mask audit only as an evaluation benchmark.
