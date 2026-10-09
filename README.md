# Tabular Data Science: Network Intrusion Detection and Titanic

Two notebooks on structured data. The main one is a multi-class intrusion-detection study on CIC-IDS2017 that puts as much weight on data quality and evaluation design as on the models. The Titanic notebook is a smaller feature-engineering and model-tuning exercise.

| Notebook | Task | Data |
|---|---|---|
| [`network_attack_classification.ipynb`](network_attack_classification.ipynb) | Classify network flows into 8 attack families | CIC-IDS2017, 2.83M flows, 78 features |
| [`titanic.ipynb`](titanic.ipynb) | Predict passenger survival | Kaggle Titanic, 891 passengers |

## 1. Network intrusion detection (CIC-IDS2017)

### Data

The [CIC-IDS2017](https://www.unb.ca/cic/datasets/ids-2017.html) dataset from the Canadian Institute for Cybersecurity, in its `MachineLearningCVE` CSV form: 2.83 million flow records described by 78 numeric CICFlowMeter features. The 15 original labels are grouped into 8 families: Benign, DoS, DDoS, PortScan, Brute Force, Web Attack, Botnet and Infiltration. The data is heavily imbalanced: about 80% of the flows are Benign and Infiltration has 36. Results are therefore reported as **macro F1** and per-class recall rather than accuracy.

Download the CSVs and place them in `./MachineLearningCVE/` before running the notebook.

### Pipeline

1. **Load:** read the CSVs and group the labels into attack families.
2. **Exploratory analysis:** family distribution, flow duration per family, correlation analysis and a 2-D PCA projection.
3. **Data audit and cleaning:** find duplicate rows, feature vectors with conflicting labels, constant columns and duplicated columns. Measure how much a naive random split leaks, then remove all of these, together with `Destination Port`. A port identifies the attacked service rather than the attack's behaviour.
4. **Feature engineering** on a stratified 80/20 split of the cleaned data, with every learned step fitted on the training part only:
   - **Strategy A:** forward sequential feature selection with k-NN on a class-capped sample (up to 1,500 flows per family). It is scored by macro F1 and stops when a feature adds less than 0.001.
   - **Strategy B:** PCA on standardised features, keeping 95% of the variance.
5. **Models:** Decision Tree, Gaussian Naive Bayes and k-NN, with scaling pipelines where needed. Section 5.3 isolates the effect of duplicates, feature choice and the TCP initial-window features.
6. **Monte-Carlo cross-validation:** 100 seeded repetitions on stratified 6.25% subsamples, with at least 5 flows per family.
7. **Hyperparameter tuning:** randomised search over Decision Tree hyperparameters, scored by macro F1.

### Data audit

| Finding | Value |
|---|---|
| Duplicate feature vectors | 307,775 (10.9% of rows): PortScan 43%, Brute Force 34%, DoS 23% |
| Rows whose feature vector appears with another label | 6,666 |
| Constant columns | 8 (`Bwd PSH Flags`, `Bwd URG Flags` and the six `*Bulk*` features) |
| Columns identical to another column | 5 (`Subflow Fwd Packets`, `Subflow Bwd Packets`, `SYN Flag Count`, `CWE Flag Count`, `Fwd Header Length.1`) |
| Test rows with an identical copy in training, under a random 80/20 split of the uncleaned data | 13.4% overall, 59% for PortScan |

After cleaning: 2,519,404 flows and 64 features. The test set holds 503,881 flows, but only 7 of them are Infiltration.

### Results

On the deduplicated test set:

| Model | Features | Macro F1 |
|---|---|---|
| **Decision Tree, tuned** | Strategy A (6 features) | **0.892** |
| Decision Tree | Strategy A | 0.879 |
| k-NN | Strategy A | 0.765 |
| Naive Bayes | Strategy A | 0.246 |
| k-NN | Strategy B (25 components) | 0.882 |
| Decision Tree | Strategy B | 0.870 |
| Naive Bayes | Strategy B | 0.268 |

Monte-Carlo cross-validation on 6.25% subsamples: Decision Tree 0.812 ± 0.051, k-NN 0.725 ± 0.022, Naive Bayes 0.272 ± 0.024 macro F1.

The 6 selected features are `Total Fwd Packets`, `Fwd Packet Length Max`, `Bwd Packet Length Std`, `Flow IAT Max`, `Flow IAT Min` and `Init_Win_bytes_backward`. With the tuned tree, Benign, DoS, DDoS, PortScan and Brute Force all reach a recall of at least 0.97. The hard classes are Web Attack (0.80), Infiltration (5 of 7 flows) and Botnet (0.53).

### What drives the score (section 5.3, Decision Tree)

| Setup | Features | Macro F1 |
|---|---|---|
| Random split, duplicates kept, 28-feature set* | 28 | 0.955 |
| Deduplicated split, the same set without its constant and duplicated columns | 18 | 0.949 |
| Deduplicated split, Strategy A features | 6 | 0.879 |
| 18-feature set without `Init_Win_bytes_*` | 16 | 0.879 |
| Strategy A without `Init_Win_bytes_backward` | 5 | 0.799 |

\*Majority vote of three sequential selectors that were each forced to return 30 features. Ten of the 28 are constant or duplicated columns.

- **Duplicates barely move macro F1** (0.955 → 0.949). The duplicated rows are concentrated in PortScan, Brute Force and DoS, which are easy to classify anyway. Macro F1 is decided by the rare families, which have almost no duplicates.
- **The stopping rule is too aggressive for a tree.** The 6 features chosen with a k-NN wrapper on a balanced sample score 0.879, while the 18-feature set reaches 0.949. A selection made for k-NN does not transfer fully to a Decision Tree. The 18-feature set was selected on data that still contained duplicates, so its score is slightly optimistic.
- **The TCP initial window sizes carry a lot of the signal.** Removing them costs 7–8 points of macro F1 with either feature set. These values reflect the sending machine's operating system and tooling, so part of the score probably depends on the specific attacker machines in the CIC testbed rather than on attack behaviour. This is the main reason to expect lower performance on other networks.
- **Naive Bayes is unusable.** Its Gaussian independence assumption fails on these heavy-tailed, strongly correlated flow features.

Problems with CIC-IDS2017 itself (labelling errors and flow-extraction artefacts) are documented in Engelen et al., *Troubleshooting an Intrusion Detection Dataset: the CICIDS2017 Case Study* (IEEE Security and Privacy Workshops, 2021).

## 2. Titanic survival prediction

A feature-engineering and tuning exercise on the Kaggle Titanic data:

- mean imputation for age, and features parsed from the sparse `Cabin` column (number of cabins, deck);
- Decision Tree, Random Forest, Gradient Boosting, Logistic Regression and SVM, compared with 5-fold cross-validation;
- hyperparameter search with `GridSearchCV` and Optuna, plus model-specific feature selection.

The best models reach about **0.82–0.83 5-fold CV accuracy** (Gradient Boosting and a tuned Random Forest).
