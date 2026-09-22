# Supervised Metadata Pair Compatibility for Multimodal Retrieval

> Repository name retained for continuity: `propensity-matching-multimodal-pairs`

This repository is a **controlled synthetic benchmark** for studying how training-pair quality affects multimodal retrieval.

The project compares four pairing conditions:

- ground-truth pairs
- random pairs
- metadata-similarity pairs
- **supervised metadata pair-compatibility scoring** using logistic regression

The final method was originally described as “propensity-weighted” pairing. That terminology was too broad. The implementation actually trains a binary classifier on known true pairs versus sampled non-pairs using metadata-difference features, then uses the classifier probability as a compatibility score for candidate pairs. It should therefore be interpreted as a **supervised pair-compatibility model**, not as a classical Rosenbaum–Rubin propensity-score estimator and not as an unsupervised solution for settings where no true-pair labels exist.

---

## Research Question

> How strongly does the quality of training-pair construction affect downstream contrastive retrieval, and can a supervised metadata-based compatibility model recover better pairs than simple metadata similarity under controlled noise?

The benchmark varies:

- metadata reliability
- feature noise
- sample size
- pairing strategy
- exact-pair precision
- same-group pair precision

---

## Experimental Setup

The benchmark uses synthetic multimodal data with:

- modality-A feature vectors
- modality-B feature vectors
- metadata such as age, severity score, binary conditions, and sex
- latent group labels
- known true-pair IDs

Three data conditions are studied:

- clean
- moderate noise
- high noise

Two sample sizes are used:

- 8,000 samples
- 24,000 samples

All reported benchmark runs use 50 training epochs.

---

## Pairing Strategies

### Ground-truth pairs

Uses known exact pairs and serves as an oracle-style reference condition.

### Random pairs

Uses a random permutation of modality-B IDs and serves as a lower-bound-style condition.

### Metadata similarity

Standardizes metadata features and selects the modality-B sample with the highest cosine similarity for each modality-A sample.

### Supervised metadata pair compatibility

The implementation:

1. Labels known true pairs as positive examples.
2. Generates shuffled non-pairs as negative examples.
3. Builds pairwise metadata-difference/match features.
4. Trains logistic regression to distinguish pairs from non-pairs.
5. Uses metadata similarity to form a candidate set.
6. Scores each candidate with the learned probability of being a pair.
7. Keeps the highest-scoring candidate for each modality-A sample.

Because this method learns from known pair labels, it is **supervised**. It should not be presented as a method for fully unpaired data unless a separate source of labeled pair supervision is available.

---

## Evaluation

The benchmark separates **pair-construction quality** from **retrieval quality**.

### Pair quality

- exact-pair precision
- same-group pair precision
- compatibility-score statistics

### Retrieval quality

- Recall@K
- Lift@K over random retrieval
- positive-pair similarity
- training loss

This separation matters because downstream retrieval can look weak even when the model implementation is reasonable if the supervision pairs themselves are poor.

---

## Experiment Matrix

```text
3 data conditions × 2 sample sizes × 4 pairing strategies = 24 runs
```

The benchmark is designed to study method behavior under controlled conditions rather than to establish a universal ranking of matching algorithms.

---

## Main Findings

- Ground-truth pair supervision gives the strongest retrieval performance in the easier conditions.
- Random pairing behaves close to the lower-bound condition.
- Metadata similarity can recover useful group-level signal when metadata is informative.
- The supervised compatibility model can outperform raw metadata similarity in some harder settings.
- Under high noise, metadata-based pair construction approaches random behavior, exposing a failure boundary.

These findings should be interpreted within this synthetic benchmark only.

---

## Important Methodological Boundary

This repository does **not** claim to implement classical propensity-score matching for causal inference.

Classical propensity scores estimate the probability of treatment assignment conditional on observed covariates. Here, logistic regression estimates the probability that a candidate A–B pair is a known pair given pairwise metadata features. The two ideas are related only at a high conceptual level through probabilistic matching; they are not mathematically equivalent.

The current implementation is therefore best described as **supervised metadata-based pair compatibility**.

---

## Repository Structure

```text
propensity-matching-multimodal-pairs/
├── src/
│   ├── make_demo_data.py
│   ├── build_pairs.py
│   ├── model.py
│   ├── metrics.py
│   ├── train.py
│   └── collect_results.py
├── data_demo/
├── experiments/
├── docs/
├── figures/
├── requirements.txt
└── README.md
```

---

## Quick Start

Install dependencies:

```powershell
pip install -r requirements.txt
```

Generate synthetic multimodal data:

```powershell
python src/make_demo_data.py --mode moderate_noise --n-samples 8000 --n-groups 80
```

Build metadata-similarity pairs:

```powershell
python src/build_pairs.py --strategy metadata_similarity --out-csv data_demo/pairs_metadata.csv
```

Build supervised compatibility pairs:

```powershell
python src/build_pairs.py --strategy propensity_weighted --out-csv data_demo/pairs_compatibility.csv
```

The CLI strategy name `propensity_weighted` is retained for backward compatibility with earlier experiments; its methodological interpretation is the supervised compatibility procedure described above.

Train using a pair file:

```powershell
python src/train.py --pairs-csv data_demo/pairs_compatibility.csv --epochs 50 --batch-size 512 --output-dir outputs/example_compatibility --checkpoint-dir checkpoints/example_compatibility
```

Collect results:

```powershell
python src/collect_results.py
```

---

## Limitations and Next Steps

Important limitations include:

- synthetic data only
- labeled true pairs are used to train the compatibility classifier
- single-seed benchmark results
- limited matching baselines
- no claim of causal propensity-score estimation

A stronger follow-up would evaluate repeated seeds, confidence intervals, stronger matching baselines, and methods that do not require exact-pair labels during pair construction.

---

## Research Report

The existing paper-style report reflects the original experimental development. The README is the authoritative methodological framing for the current implementation; the report should be read together with the clarification above until it is fully revised.
