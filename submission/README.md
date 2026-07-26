# submission/

# FlyRank Search Intelligence Capstone — Refresh / Content Opportunity Scoring

## Lane
**Refresh / Content Opportunity Scoring.** The goal: score every page as one of
`growing / declining / recovering / stagnant / review`, then turn that into a
ranked action list with reason codes (e.g. "declining + high impressions +
falling CTR → rewrite candidate").

Chosen over the other lanes because the label is naturally definable from
time-windowed performance deltas (no need to invent a subjective archetype
taxonomy), and the output — a ranked, reason-coded action list — is the most
directly usable deliverable of the four.

## Status

This repo is **scaffolded and ready to run, not yet executed against real
data.** The FlyRank warehouse (`FlyRank/internship-warehouse` on Hugging
Face) is gated, and this environment has no network path to
`huggingface.co` — with or without a token. Every notebook below is fully
written — point `HF_TOKEN` at a valid token and run `work/01_data_aggregation.py`
first; everything downstream consumes its output.

The real table architecture is now confirmed from the public dataset card
(star schema: `dim_clients`, `dim_content`, `fact_content_daily_performance`,
`fact_content_query_90d`; 78.8M daily-fact rows, monthly-partitioned, with a
single-month `_sample.parquet` to start from). Exact column names inside
each table are still a grounded best-guess — see
`work/00_data_card_notes.md` for the one confirmation query to run once you
have a token, before trusting `01_data_aggregation.py`'s output.

## Plan / structure

| File | Purpose |
|---|---|
| `work/00_data_card_notes.md` | Assumptions about the warehouse schema — **verify against the actual dataset card before running anything**, then delete this caveat |
| `work/01_data_aggregation.py` | DuckDB aggregation over `hf://...` into a page-week panel |
| `work/02_labeling.py` | Time-aware label construction (growing/declining/recovering/stagnant), leakage-safe |
| `work/03_features.py` | Feature engineering from trailing windows only (no future leakage) |
| `work/04_modeling.py` | Baseline (rule-based / logistic) vs. model (gradient boosted trees), grouped time-aware split |
| `work/05_validation.py` | Evaluation, leakage checks, calibration, error analysis |
| `work/06_ranked_recommendations.py` | Turns model output into the ranked action engine with reason codes |
| `capstone_notebook.py` | Ties 01–06 together end to end — this is the capstone deliverable notebook |
| `paper/index.html` | The deployed research paper (template — fill in Results once the model runs) |
| `submission/paper_url.txt` | **Mandatory** — one line, the deployed paper URL |

## Public-safety rules honored in code

- No client names, domains, raw URLs, or credentials ever printed, logged, or
  committed — all aggregation happens above the per-URL level before any
  output leaves `work/01_data_aggregation.py`.
- No claims of causal Google-algorithm impact — language is restricted to
  "observed association" / "directional" / "decision-support" throughout,
  enforced as a lint check in `05_validation.py` (`FORBIDDEN_CAUSAL_PHRASES`).

## To actually run this

```bash
export HF_TOKEN=hf_xxx           # your gated read token
pip install duckdb scikit-learn pandas numpy huggingface_hub
python work/01_data_aggregation.py
python work/02_labeling.py
python work/03_features.py
python work/04_modeling.py
python work/05_validation.py
python work/06_ranked_recommendations.py
```

Then fill the Results section of `paper/index.html` with the real numbers
`05_validation.py` prints, deploy it (GitHub Pages is easiest — see below),
and put the live URL in `submission/paper_url.txt`.

## Deploying the paper (GitHub Pages)

```bash
git checkout -b gh-pages
git add paper/index.html
git commit -m "Deploy capstone paper"
git subtree push --prefix paper origin gh-pages
# paper goes live at https://<you>.github.io/<repo>/
```
