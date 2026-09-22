# Paper-Style Report: Supervised Metadata Pair Compatibility for Multimodal Retrieval

## Abstract

This project studies how training-pair construction affects multimodal contrastive retrieval in a controlled synthetic benchmark. Four pairing conditions are compared: ground-truth pairs, random pairs, metadata-similarity pairs, and a supervised metadata pair-compatibility model. The compatibility model is trained using known true pairs as positive examples and sampled non-pairs as negative examples, then scores candidate pairs from metadata-difference features. The benchmark evaluates pair quality and downstream retrieval under clean, moderate-noise, and high-noise conditions. Results show that supervision quality strongly controls retrieval behavior: ground-truth pairs provide the strongest reference condition, random pairing performs near the lower bound, and metadata-based methods recover useful signal when metadata remains informative. Under high noise, metadata-based methods approach random behavior. The supervised compatibility model should not be interpreted as classical causal propensity-score matching or as a method that works without any labeled pair supervision.

## 1. Motivation

Multimodal contrastive learning depends on the quality of positive pairs used during training. Weak or incorrect pairs can distort the representation space even when the encoder and contrastive objective are implemented correctly.

This project therefore treats pair construction as a separate experimental variable rather than assuming that positive pairs are always trustworthy.

## 2. Research Question

The benchmark asks:

> How strongly does pair-construction quality affect downstream retrieval, and can a supervised metadata-based pair-compatibility model improve over simple metadata similarity under controlled noise?

The experiment varies:

- pairing strategy
- metadata reliability
- feature noise
- sample size
- exact-pair precision
- same-group pair precision

## 3. Methods

The benchmark compares four pairing strategies.

| Strategy | Role |
|---|---|
| Ground-truth pair | Uses known exact pairs and serves as an oracle-style reference condition |
| Random pair | Randomly permutes modality-B IDs and serves as a lower-bound-style condition |
| Metadata similarity | Matches using cosine similarity in standardized metadata space |
| Supervised pair compatibility | Trains logistic regression on labeled true pairs vs sampled non-pairs, then scores candidate pairs |

### 3.1 Supervised pair-compatibility model

For the supervised compatibility condition, the implementation:

1. Uses known true A–B pairs as positive examples.
2. Generates shuffled A–B non-pairs as negative examples.
3. Constructs pairwise features from metadata differences and matches, including age difference, severity difference, condition matches, and sex match.
4. Trains logistic regression with balanced class weights.
5. Uses metadata similarity to create a top-k candidate set for each modality-A sample.
6. Applies the classifier to candidate pairs and interprets the predicted positive probability as a pair-compatibility score.
7. Keeps the highest-scoring candidate for each modality-A sample.

This procedure is supervised because true-pair labels are used to train the classifier.

## 4. Terminology Clarification

Earlier versions of this project described the classifier output as a “propensity score.” That wording was too broad.

In classical causal inference, a propensity score is the probability of treatment assignment conditional on observed covariates:

```text
e(X) = P(T = 1 | X)
```

The implementation in this repository instead estimates a probability closer to:

```text
P(pair = 1 | pairwise metadata features)
```

Therefore, the current method is best described as **supervised metadata pair compatibility**, not as classical Rosenbaum–Rubin propensity-score matching.

The repository name and the CLI strategy label `propensity_weighted` are retained for continuity with earlier experiments, but the methodological interpretation above is the authoritative one.

## 5. Dataset and Experimental Setup

The benchmark uses controlled synthetic multimodal data containing:

| Component | Description |
|---|---|
| Modality A | Synthetic feature vector |
| Modality B | Synthetic feature vector |
| Metadata | Age, severity score, binary conditions, sex |
| Group label | Latent semantic group used for approximate pair-quality evaluation |
| True pair ID | Known exact cross-modal pair |

Three conditions are tested:

| Condition | Meaning |
|---|---|
| Clean | Metadata and modality features are relatively aligned |
| Moderate noise | Metadata and features become less reliable |
| High noise | Metadata becomes weak and pair ambiguity increases |

Two sample sizes are used:

| Total samples | Held-out retrieval pool |
|---:|---:|
| 8,000 | 1,600 |
| 24,000 | 4,800 |

All reported runs use 50 training epochs.

## 6. Evaluation Metrics

The project separates pair-construction quality from downstream retrieval quality.

### Pair quality

| Metric | Meaning |
|---|---|
| Exact-pair precision | Fraction of constructed pairs that are the known true pair |
| Same-group precision | Fraction of constructed pairs from the same latent group |
| Pair score statistics | Distribution of similarity or learned compatibility scores |

### Retrieval quality

| Metric | Meaning |
|---|---|
| Recall@K | Whether the correct match appears in the top K retrieved candidates |
| Lift@K | Improvement over random retrieval at K |
| Positive-pair similarity | Cosine similarity of known true pairs in the learned embedding space |
| Training loss | Contrastive optimization objective |

Separating pair quality from retrieval quality is important because weak training pairs can limit downstream performance independently of model architecture.

## 7. Experiment Matrix

```text
3 data conditions × 2 sample sizes × 4 pairing strategies = 24 runs
```

The benchmark is intended as a method-behavior study rather than a universal comparison of matching algorithms.

## 8. Main Findings

### Ground-truth pair condition

Known exact pairs provide the strongest reference condition in the easier settings, demonstrating the benefit of reliable supervision.

### Random condition

Random pair construction stays close to the lower-bound behavior, showing that arbitrary pairing does not provide useful cross-modal supervision.

### Metadata similarity

Metadata similarity recovers useful group-level signal when metadata remains informative, but degrades as metadata reliability falls.

### Supervised pair compatibility

The supervised classifier can improve over raw metadata similarity in some harder conditions because it learns which pairwise metadata patterns are associated with known true pairs. This advantage depends on access to labeled pair supervision and should not be generalized to fully unpaired settings.

### High-noise condition

When metadata becomes weak, both metadata-based approaches approach random behavior. This exposes a practical failure boundary for pair construction from unreliable covariates.

## 9. Limitations

Important limitations include:

- synthetic data only
- known true pairs are used to train the compatibility classifier
- the current benchmark is effectively single-seed for each configuration
- no uncertainty intervals are reported
- the method is not a classical causal propensity-score estimator
- the supervised compatibility condition cannot be claimed as a solution for settings with zero labeled pair information
- the set of matching baselines is limited

These limitations mean that the results should be interpreted as controlled evidence about supervision quality and pair-construction behavior, not as a universal matching result.

## 10. Future Work

A stronger follow-up would include:

- repeated random seeds with mean and uncertainty estimates
- confidence-thresholded pair filtering
- stronger supervised matching baselines
- weakly supervised or self-supervised pair construction without exact-pair labels
- optimal-transport baselines
- evaluation on authorized real multimodal datasets
- explicit calibration analysis for pair-compatibility probabilities
- embedding-geometry analysis after different pairing strategies

## 11. Key Lesson

The central lesson from this benchmark is:

> **Before asking whether a multimodal retrieval model is good, first ask whether its positive-pair supervision is trustworthy.**

Model architecture and loss design cannot fully compensate for incorrect or ambiguous training pairs.

## 12. Related Work

This project was motivated by classical matching literature and modern work on unpaired multimodal alignment, but it does not reproduce classical propensity-score matching.

Relevant references include:

- Paul R. Rosenbaum and Donald B. Rubin. *The Central Role of the Propensity Score in Observational Studies for Causal Effects*. Biometrika, 1983.
- Peter C. Austin. *An Introduction to Propensity Score Methods for Reducing the Effects of Confounding in Observational Studies*. Multivariate Behavioral Research, 2011.
- Johnny Xi, Jana Osea, Zuheng Xu, and Jason Hartford. *Propensity Score Alignment of Unpaired Multimodal Data*. NeurIPS, 2024.

These works provide useful conceptual background, but the implementation in this repository should be evaluated on its own terms as a supervised pair-compatibility benchmark.
