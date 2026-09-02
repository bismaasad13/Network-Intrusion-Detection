# ML-Based Network Intrusion Detection & Threat Classification System

##  Project Overview

This project develops an end-to-end Machine Learning-based Network Intrusion Detection System (NIDS) using the **CIC-IDS2017** dataset.

The system analyzes network traffic characteristics and classifies traffic into multiple categories, including benign traffic and different types of network attacks.

The project covers the complete Machine Learning workflow, from dataset preparation and exploratory data analysis to model training, evaluation, optimization, explainability, and reusable model deployment.

---

##  Objectives

The main objectives of this project are:

* Detect malicious network traffic using Machine Learning.
* Classify network traffic into different attack categories.
* Perform exploratory data analysis on network traffic.
* Handle the highly imbalanced attack classes.
* Compare multiple Machine Learning algorithms.
* Optimize model hyperparameters.
* Identify the most important network traffic features.
* Save the trained model for future API/dashboard integration.

---

## Dataset

The project uses the **CIC-IDS2017** dataset, a widely used network intrusion detection dataset containing benign network traffic and several types of cyberattacks.

The dataset contains network-flow features such as:

* Flow duration
* Forward and backward packet counts
* Packet lengths
* Packet rates
* TCP flags
* Inter-arrival times
* Header lengths
* TCP window information

### Dataset used in this project

* Initial sampled dataset: **400,000 records**
* Original features: **78 columns**
* Target column: **Label**
* Numeric modeling features after preprocessing: **69**

The dataset contains both benign traffic and multiple attack categories such as DDoS, DoS, PortScan, Bot, FTP-Patator, SSH-Patator, and Web Attacks.

> The raw dataset is not included in this repository because of its large size.

---

##  Machine Learning Workflow

The project follows this workflow:

```text
CIC-IDS2017 Dataset
        ↓
Data Loading
        ↓
Data Cleaning
        ↓
Exploratory Data Analysis
        ↓
Feature Preparation
        ↓
Train/Test Split
        ↓
Missing Value Handling
        ↓
Class Imbalance Handling
        ↓
Multiple Model Training
        ↓
Model Comparison
        ↓
Hyperparameter Optimization
        ↓
Feature Importance Analysis
        ↓
Final Model Selection
        ↓
Model Serialization
        ↓
Reusable Prediction Pipeline
```

---

##  Data Preprocessing

The following preprocessing steps were performed:

1. Column names were standardized.
2. Infinite values were replaced with missing values.
3. Completely empty columns were removed.
4. Duplicate rows were removed.
5. The target label was identified.
6. Extremely rare attack classes were grouped into `Rare_Attack`.
7. Numeric features were selected for modeling.
8. Constant features were removed.
9. Missing numerical values were handled using median imputation.
10. The dataset was divided into training and testing sets using stratified sampling.

After preprocessing:

* **69 numeric features** were used for modeling.
* The dataset was split into **80% training** and **20% testing**.

---

## Handling Class Imbalance

The CIC-IDS2017 dataset contains a large number of benign records compared with several minority attack classes.

To address this issue, **RandomOverSampler** was applied to the training data only.

This increased the representation of minority classes while keeping the test set unchanged.

The test set was deliberately kept separate from oversampling to prevent data leakage.

---

## Machine Learning Models

Four baseline models were trained and compared:

1. Logistic Regression
2. Random Forest
3. Extra Trees
4. XGBoost

The models were evaluated using:

* Accuracy
* Weighted Precision
* Weighted Recall
* Weighted F1-score
* Macro F1-score

Macro F1 was particularly important because it gives greater consideration to minority attack classes.

---

##  Model Comparison

The baseline models produced the following results:

| Model               |   Accuracy | Weighted F1 |   Macro F1 |
| ------------------- | ---------: | ----------: | ---------: |
| **XGBoost**         | **99.61%** |  **99.64%** | **85.75%** |
| Random Forest       |     99.43% |      99.47% |     80.75% |
| Extra Trees         |     99.13% |      99.23% |     73.16% |
| Logistic Regression |     42.01% |      53.65% |     24.98% |

###  Best Baseline Model

**XGBoost** achieved the highest Macro F1-score of **85.75%** and was selected as the final deployment model.

---

##  Hyperparameter Optimization

Random Forest hyperparameter optimization was performed using several configurations.

The best configuration was:

```text
n_estimators = 100
max_depth = None
min_samples_split = 5
```

Performance:

* Accuracy: **99.43%**
* Macro F1: **80.32%**

Although the optimized Random Forest performed well, it did not outperform XGBoost. Therefore, XGBoost was retained as the final deployment model.

---

##  Feature Importance

Feature importance analysis was performed using the optimized Random Forest model.

The top features included:

| Rank | Feature            | Importance |
| ---: | ------------------ | ---------: |
|    1 | Init_Bwd_Win_Bytes |     0.0338 |
|    2 | Bwd_Header_Length  |     0.0334 |
|    3 | Flow_IAT_Mean      |     0.0310 |
|    4 | Fwd_Packets_s      |     0.0307 |
|    5 | Bwd_Packets_s      |     0.0290 |
|    6 | Init_Fwd_Win_Bytes |     0.0281 |
|    7 | Flow_IAT_Max       |     0.0277 |
|    8 | Flow_Packets_s     |     0.0271 |
|    9 | Flow_Duration      |     0.0261 |
|   10 | Fwd_IAT_Std        |     0.0259 |

These results show that traffic timing, packet rates, TCP window characteristics, and header information were important for intrusion classification.

---

##  Final Model Evaluation

The final deployment model is **XGBoost**, selected because it achieved the strongest baseline Macro F1 score.

The detailed evaluation showed particularly strong performance on common classes such as:

* Benign
* DDoS
* DoS Hulk
* FTP-Patator

Some minority classes achieved lower F1 scores because they contained very few examples.

This demonstrates an important limitation of intrusion detection datasets: overall accuracy can be very high while rare attack categories remain more difficult to classify.

---

##  Saved Model

The trained XGBoost model and preprocessing components were saved as:

```text
models/nids_xgboost_model.joblib
```

The saved artifact contains:

* Trained XGBoost model
* Median imputer
* Label encoder
* Feature names
* Random state

This allows the trained system to be reused without retraining the model.

---

##  Reusable Prediction Pipeline

The repository includes:

```text
src/prediction.py
```

This module loads the saved model and provides a:

```python
predict_intrusion()
```

function for making predictions on new network traffic records.

The prediction pipeline returns:

* Predicted traffic/attack class
* Prediction confidence
* Risk level

Risk levels are assigned as:

```text
Benign → LOW

Malicious + confidence ≥ 70% → HIGH

Malicious + confidence < 70% → MEDIUM
```

---

##  Project Structure

```text
ML-Network-Intrusion-Detection/
│
├── README.md
├── requirements.txt
│
├── notebook/
│   └── NIDS_CICIDS2017.ipynb
│
├── models/
│   └── nids_xgboost_model.joblib
│
├── src/
│   └── prediction.py
│
└── results/
    ├── model_comparison.png
    ├── confusion_matrix.png
    └── feature_importance.png
```

---

##  Technologies Used

* Python
* Pandas
* NumPy
* Scikit-learn
* XGBoost
* Imbalanced-learn
* PyArrow
* Matplotlib
* Seaborn
* Joblib
* Google Colab
* GitHub

---

## Installation

Clone the repository:

```bash
git clone <YOUR-GITHUB-REPOSITORY-URL>
```

Move into the project directory:

```bash
cd ML-Network-Intrusion-Detection
```

Install the required packages:

```bash
pip install -r requirements.txt
```

---

## ▶ Running the Notebook

Open:

```text
notebook/NIDS_CICIDS2017.ipynb
```

The notebook can be opened in Google Colab or Jupyter Notebook.

The raw CIC-IDS2017 dataset must be downloaded separately because it is not included in this repository.

---

##  Limitations

The project has several limitations:

* The CIC-IDS2017 dataset is highly imbalanced.
* Some attack classes contain very few samples.
* Minority classes therefore have less reliable performance estimates.
* The project uses a sampled dataset to keep computation manageable in Google Colab.
* The current prediction pipeline is prepared for future dashboard/API integration rather than being a complete real-time network monitoring system.

---

##  Future Improvements

Future versions could include:

* Real-time network traffic streaming.
* FastAPI-based prediction API.
* Streamlit monitoring dashboard.
* SHAP-based explainable AI.
* More advanced imbalance techniques.
* Larger training datasets.
* Deep learning models.
* Graph-based network analysis.
* Real-time alert generation.

---

##  Conclusion

This project demonstrates a complete Machine Learning workflow for multiclass network intrusion detection using CIC-IDS2017.

The project successfully covers data preparation, exploratory analysis, feature engineering, class imbalance handling, model comparison, hyperparameter optimization, explainability, evaluation, and model serialization.

Among the tested baseline models, **XGBoost achieved the best performance with 99.61% accuracy and an 85.75% Macro F1-score**.

The resulting trained model has been saved as a reusable artifact and can serve as the foundation for a future network security dashboard or API-based intrusion detection system.

---

##  Project

**Machine Learning-Based Network Intrusion Detection & Threat Classification System**

**Dataset:** CIC-IDS2017
**Final Model:** XGBoost
**Environment:** Google Colab / Python
