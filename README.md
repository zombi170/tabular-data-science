# Tabular Data Science Portfolio

This repository contains a collection of robust data science pipelines and exploratory data analysis (EDA) projects focusing on structured tabular data. It demonstrates advanced data wrangling, feature engineering, dimensionality reduction, and rigorous statistical model evaluation across classification tasks.

## 1. Network Traffic Anomaly Detection

An end-to-end pipeline designed to classify and analyze network intrusion attempts from a massive, high-variance cybersecurity dataset.

*   **Data Ingestion & Cleaning:** Loaded and concatenated the `MachineLearningCVE` dataset containing 2,830,743 flow records and 79 network traffic features. 
*   **Target Mapping:** Engineered a `dynamic_grouping` function to consolidate highly specific and imbalanced sub-attacks into 8 core `Attack Family` labels: Benign, Botnet, Brute Force, DDoS, DoS, Infiltration, PortScan, and Web Attack.
*   **Exploratory Data Analysis (EDA):** Visualized feature distributions such as `Flow Duration` across attack families, identifying that PortScan traffic consists of rapid packets, while Infiltration and DoS attacks exhibit the highest median durations.
*   **Feature Engineering & Selection:**
    *   **Sequential Feature Selection (SFS):** Deployed Forward, Backward, and Bidirectional SFS utilizing a k-NN estimator to reduce the feature space from 79 down to an optimal set of 28 highly predictive features.
    *   **Dimensionality Reduction:** Executed Principal Component Analysis (PCA) on the scaled dataset, successfully capturing 95.4% of the cumulative explained variance using only 25 principal components.
*   **Model Evaluation:** 
    *   Trained and evaluated multiple multi-class classifiers, including Decision Tree, Naive Bayes, and k-NN.
    *   The Decision Tree architecture dominated the benchmark, achieving an accuracy of 99.86% and a Macro F1 score of 0.955. It successfully captured the hardest class (Infiltration) and vastly outperformed Naive Bayes, which failed with a 4.6% accuracy due to massive false positives on Benign traffic.

## 2. Titanic Survival Prediction

A rigorous machine learning pipeline predicting passenger survival based on demographic and socio-economic indicators, moving beyond standard out-of-the-box model fitting.

*   **Missing Value Imputation:** Applied targeted mean imputation for continuous variables like passenger age to preserve underlying demographic distributions without heavy distortion.
*   **Latent Feature Extraction:** Programmatically parsed the highly sparse `Cabin` text column to extract actionable variables such as `CabinCount` (number of booked cabins) and applied one-hot encoding to extract specific `Deck` levels.
*   **Statistical Validation:** Implemented 5-fold cross-validation across the training set to prevent random chance anomalies and acquire a robust estimate of model generalization.
*   **Hyperparameter Optimization:** Utilized `GridSearchCV` to systematically tune hyperparameters (`max_depth`, `min_samples_leaf`, `learning_rate`) across tree-based classifiers (Decision Trees, Random Forests, Gradient Boosting) to maximize peak performance.

## Tech Stack

*   **Machine Learning:** Scikit-Learn (Decision Trees, Random Forests, Gradient Boosting, k-NN, Naive Bayes, PCA, SequentialFeatureSelector)
*   **Data Manipulation:** Pandas, NumPy
*   **Visualization:** Matplotlib, Seaborn
