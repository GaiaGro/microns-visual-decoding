# Neural Decoding of Visual Stimuli in Mouse Visual Cortex

Machine learning analysis of visual stimulus decoding from two-photon population activity across four mouse visual cortical areas (V1, LM, AL, RL), using the MICrONS Phase 3 functional dataset.

The project investigates whether visual stimulus categories can be decoded from neural population activity and whether decoding performance varies systematically across cortical areas, while controlling for population size and behavioural state.

## Main result

For the most fine-grained contrast — discrimination between three natural video categories (Cinematic, Sports1M, and Rendered) — LM achieved the highest decoding performance in 9 out of 10 sessions.

LM exceeded V1 by Δ = +0.031 and AL/RL by approximately Δ = +0.05, with statistical significance assessed using Bonferroni-corrected paired comparisons.

Coarser contrasts, including natural vs. parametric stimuli and Monet2 vs. Trippy, reached near-ceiling performance across all areas and therefore provided little separation between cortical regions.

![Pairwise area differences in balanced accuracy](figures/fig2_pairwise_area_differences.png)

*Pairwise differences in balanced accuracy across cortical areas, averaged across 10 sessions. The fine natural-category contrast shows the clearest cross-area differences.*

**[Full report (PDF)](documents/report.pdf)** — 11 pages, 14 figures.

---

## Repository layout

| Path | Description |
|---|---|
| [`01_category_decoding/`](01_category_decoding) | Trial-mean decoding across cortical areas for natural vs. parametric stimuli, Monet2 vs. Trippy, and the three-way natural-category contrast. Includes neuron-count matching, permutation tests, behavioural regression, and paired area comparisons. |
| [`02_time_resolved_decoding/`](02_time_resolved_decoding) | Time-resolved decoding of natural video categories using logistic regression and linear SVM, including temporal averaging analyses. |
| [`extra/`](extra) | Earlier single-session analysis pipeline, retained for reference but not included in the final report. |
| [`utils/`](utils) and [`reader.py`](reader.py) | Shared utilities and data-access functions. |
| [`docs/DATASET.md`](docs/DATASET.md) | Description of the MICrONS HDF5 dataset structure and `MicronsReader` API. |
| [`documents/`](documents) | Final report and related documents. |

## Research questions

| Question | Decoding task | Chance level | Analysis |
|---|---|---:|---|
| **Q1a** | Natural vs. parametric stimuli | 0.50 | `01_category_decoding/` |
| **Q1b** | Parametric discrimination: Monet2 vs. Trippy | 0.50 | `01_category_decoding/` |
| **Q1c** | Natural-category discrimination: Cinematic vs. Sports1M vs. Rendered | 0.33 | `01_category_decoding/` |
| **Q2** | Time-resolved per-frame natural-category decoding | 0.33 | `02_time_resolved_decoding/` |

## Results

### Trial-mean decoding

Balanced accuracy at matched neuron count, reported as mean ± SD across 10 sessions:

| Task | V1 | LM | AL | RL |
|---|---:|---:|---:|---:|
| **Q1a** — Natural vs. parametric *(chance = 0.50)* | **0.959 ± 0.016** | 0.951 ± 0.015 | 0.918 ± 0.020 | 0.936 ± 0.022 |
| **Q1b** — Monet2 vs. Trippy *(chance = 0.50)* | **0.994 ± 0.006** | 0.988 ± 0.009 | 0.979 ± 0.019 | 0.980 ± 0.015 |
| **Q1c** — Natural categories *(chance = 0.33)* | 0.647 ± 0.033 | **0.677 ± 0.036** | 0.621 ± 0.061 | 0.622 ± 0.058 |

All 120 session × area × question combinations decoded significantly above the shuffle-label null.

Q1a and Q1b showed near-ceiling performance across areas. Q1c provided the clearest separation between cortical regions, with LM achieving the highest mean decoding accuracy.

### Time-resolved decoding

Peak balanced accuracy for Session `5_6` *(chance = 33.3%)*:

| Window | Classifier | V1 | LM | AL | RL | Average |
|---|---|---:|---:|---:|---:|---:|
| *w* = 1 | Logistic Regression | 41.4% | 44.2% | **47.4%** | 42.6% | 43.9% |
| *w* = 1 | Linear SVM | 40.2% | 43.5% | 44.6% | 41.2% | 42.5% |
| *w* = 5 | Logistic Regression | 46.5% | 48.4% | **50.2%** | 48.5% | 48.4% |
| *w* = 5 | Linear SVM | 44.2% | 46.6% | 47.5% | 47.2% | 46.4% |

Per-frame decoding was significantly above chance in every area (*p* < 10⁻⁹; 0/50 shuffles exceeded the observed accuracy), although performance remained below 50%.

Five-frame temporal averaging improved decoding by approximately 4.5 percentage points across areas, consistent with category information being distributed over time rather than concentrated at stimulus onset.

### Behavioural confounds

Regressing out pupil-related features and treadmill velocity before trial averaging reduced accuracy by approximately 0.03 for Q1a, with the largest reduction in AL (−0.038).

For Q1b and Q1c, the effect remained below 0.01 and was inconsistent in sign. The LM advantage observed in Q1c therefore persisted after behavioural cleaning.

### Confusion structure

For Q1c, Cinematic ↔ Rendered was the dominant source of confusion across all areas, with off-diagonal values of approximately 0.19–0.23.

Sports1M was the most consistently classified category, with diagonal values of approximately 0.64–0.71. The LM advantage was distributed across the three classes rather than being driven by a single category.

## Methods

- **Data.** Ten MICrONS sessions acquired at a matched imaging rate (~6.30 Hz) were included. Sessions `9_3`, `9_4`, and `9_6` were excluded because they were acquired at a higher imaging rate (8.62–9.62 Hz), while session `7_4` was excluded because it was experimentally corrupted. Each included session contained 464 trials: 128 Cinematic, 128 Sports1M, 128 Rendered, 40 Monet2, and 40 Trippy.

- **Preprocessing.** The first three frames (~475 ms) were discarded to account for response-onset lag. Trial-mean neural responses were then computed separately for each cortical area. Behaviour-cleaned features were obtained by regressing each neuron's activity on four pupil-related variables and treadmill velocity.

- **Decoder.** Neural activity was decoded using a `StandardScaler → LogisticRegression` pipeline with ℓ2 regularization, `C = 1`, balanced class weights, and 5-fold stratified cross-validation. Performance was measured using balanced accuracy.

- **Population-size control.** Because cortical areas contained different numbers of recorded neurons, cross-area comparisons were performed after matching population size to the minimum neuron count available within each session (`Nmin`, ranging from 287 to 468 neurons). Fifty random neuron subsamples were evaluated for each comparison.

- **Statistics.** Statistical significance against chance was assessed using shuffle-label null distributions with 100 permutations per session, area, and decoding task. Cross-area comparisons used paired Wilcoxon signed-rank tests across sessions, with Bonferroni correction applied within each research question.

- **Time-resolved decoding.** Time-resolved analyses were performed on Session `5_6`, containing 8,592 neurons, 384 natural-video trials, and 72 timepoints. Population size was matched to 468 neurons per area. Each timepoint was decoded independently using logistic regression and a linear SVM. `GroupKFold` splitting by clip hash prevented the same video clip from appearing in both training and test folds. Temporal averaging was additionally evaluated using a five-frame window.

## Limitations

The analyses rely on linear decoders and therefore quantify how linearly accessible stimulus-category information is in each cortical area. Higher decoding accuracy in LM should not be interpreted as evidence that LM intrinsically “represents” visual categories more strongly than V1.

The dataset also contains imbalanced class counts, with 384 natural-video trials and 80 parametric-stimulus trials per session. In addition, the matched population size (`Nmin`) varies across sessions, meaning that comparisons are controlled within each session but are not based on an identical neuron count across the full dataset.

## Reproducing

```bash
git clone https://github.com/GaiaGro/microns-visual-decoding.git
cd microns-visual-decoding

uv sync                      # or: pip install -r requirements.txt

cp .env.example .env         # point DATA_PATH at microns.h5

python main_runner.py --question 1
```

Data is pulled from [`NeuroBLab/MICrONS`](https://huggingface.co/datasets/NeuroBLab/MICrONS) on first run. See [`docs/DATASET.md`](docs/DATASET.md) for the HDF5 schema and reader API.

## Authors

Research project supervised by **Prof. Alessandro Sanzeni**, Bocconi University, March–April 2026.

**Gaia Grossi** · Max David · Leo Arthur Morvan · Anna Notaro · Beatrice Porta

*Gaia Grossi: `02_time_resolved_decoding/` — time-resolved per-frame decoding, logistic regression and linear SVM analyses, GroupKFold validation, and temporal averaging.*

## References

Stringer et al., *Nature* 2019 · Goltstein et al., *Nature Neuroscience* 2021 · Chen et al., *PLOS Computational Biology* 2024 · Ding et al., *Nature* 2025 (MICrONS functional connectomics)