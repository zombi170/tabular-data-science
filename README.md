# Network Traffic Anomaly Detection

This repository contains a comprehensive data science pipeline designed to classify and analyze network intrusion attempts. It emphasizes statistical rigor, feature dimensionality reduction, and multi-class model evaluation.

## Methodology

### 1. Data Engineering & Consolidation
*   **Dataset:** Processed the `MachineLearningCVE` dataset, aggregating over 2.8 million flow records and 79 distinct features.
*   **Target Mapping:** Engineered a `dynamic_grouping` function to map 15 highly specific sub-attacks into 8 generalized `Attack Family` categories (e.g., `DDoS`, `PortScan`, `Brute Force`, `Botnet`) to mitigate class imbalance.

### 2. Feature Compression
*   **Dimensionality Reduction:** Executed Principal Component Analysis (PCA) on the scaled dataset.
*   **Variance Retention:** Successfully reduced the feature space from 78 original features to 25 Principal Components while retaining 95.4% of the cumulative explained variance.

### 3. Model Evaluation
*   **Architectures:** Evaluated multiple classifier architectures against the compressed dataset, including `DecisionTreeClassifier`, `GaussianNB`, and `KNeighborsClassifier`.
*   **Results:** The Decision Tree architecture achieved the highest Macro F1 score (0.955) and an accuracy of 99.8%, utilizing 100-run Monte-Carlo Cross-Validation (MCCV) to ensure statistical validity.
