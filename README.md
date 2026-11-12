# Malicious URL Detection Using Machine Learning

**A machine learning-based cybersecurity application for classifying URLs as Benign, Defacement, Malware, or Phishing.**

The project compares three supervised learning algorithms—**Logistic Regression, XGBoost, and Random Forest**—using character-level TF-IDF features extracted from URLs. An interactive **Streamlit web application** lets users enter a URL and view predictions from all three models, their probability scores, and a session-based prediction history.

> **Best reported test accuracy: 95.43% (Logistic Regression)** on the original experiment's stratified 20% test split. These results are from the supplied notebook and have not been rerun after the improvements noted below.

## Features

- **Four-class URL classification:** Benign, Defacement, Malware, and Phishing.
- **Three-model comparison:** Logistic Regression, Random Forest, and XGBoost.
- **Character-level TF-IDF:** 3–5 character n-grams, capped at 5,000 features.
- **Model evaluation:** Accuracy, weighted precision, weighted recall, weighted F1-score, per-class classification reports, and confusion matrices.
- **Interactive Streamlit app:** Enter a URL and compare model predictions and class probabilities.
- **Prediction history:** Review URLs and results during the current Streamlit session.
- **Exploratory visualizations:** Word clouds by URL category.

## Tech Stack

| Area | Technologies |
| --- | --- |
| Language | Python |
| Data processing | Pandas, NumPy |
| Feature extraction | scikit-learn TF-IDF Vectorizer |
| Machine learning | scikit-learn, XGBoost |
| Evaluation | scikit-learn metrics |
| Visualization | Matplotlib, Seaborn, WordCloud |
| Web interface | Streamlit |
| Model persistence | Python `pickle` |
| Development environment | Kaggle Notebooks / Jupyter Notebook |
| Optional public demo tunnel | pyngrok |

## Dataset

The project uses the [Malicious URLs Dataset on Kaggle](https://www.kaggle.com/datasets/sid321axn/malicious-urls-dataset/data), containing **651,191 labeled URLs**.

| URL category | Samples | Share |
| --- | ---: | ---: |
| Benign | 428,103 | 65.74% |
| Defacement | 96,457 | 14.81% |
| Phishing | 94,111 | 14.45% |
| Malware | 32,520 | 4.99% |
| **Total** | **651,191** | **100%** |

The classes are imbalanced, with benign URLs forming the majority and malware URLs the smallest category.

## System Workflow

```text
Kaggle URL Dataset
        |
        v
Exploratory Data Analysis + Word Clouds
        |
        v
Character-level TF-IDF (3–5 grams, 5,000 features)
        |
        v
Stratified 80/20 Train–Test Split
        |
        +----------+------------------+
        |          |                  |
        v          v                  v
 Random Forest  Logistic Regression  XGBoost
        |          |                  |
        +----------+------------------+
                   |
                   v
       Metrics + Confusion Matrices
                   |
                   v
       Serialized Models + Vectorizer
                   |
                   v
             Streamlit App
                   |
                   v
       Predictions + Probabilities
                   |
                   v
        Session Prediction History
```

### Feature extraction

URLs are represented as TF-IDF vectors of character sequences. This helps capture patterns in URL strings without retrieving website content.

```python
vectorizer = TfidfVectorizer(
    analyzer="char",
    ngram_range=(3, 5),
    max_features=5000
)
```

The original notebook also defines a URL-cleaning helper that removes `http://`, `https://`, and `www.` prefixes and lowercases text. **In the supplied training code, TF-IDF is fitted on the original `url` column, whereas the Streamlit app vectorizes cleaned URLs.** This should be made consistent before publishing a corrected version of the application.

### Models and configuration

| Model | Key settings |
| --- | --- |
| Random Forest | `n_estimators=50`, `max_depth=10`, `random_state=42` |
| Logistic Regression | `max_iter=1000` |
| XGBoost | `n_estimators=100`, `max_depth=4`, `learning_rate=0.1`, `subsample=0.8`, `colsample_bytree=0.8`, `tree_method='hist'`, `random_state=42` |

The original code uses an **80/20 stratified train–test split** (`random_state=42`).

## Results

The following are the **reported results from the original notebook**, calculated on **130,239 test URLs**. Precision, recall, and F1-score are **weighted averages** across the four classes.

| Model | Accuracy | Precision | Recall | F1-score |
| --- | ---: | ---: | ---: | ---: |
| **Logistic Regression** | **95.43%** | **95.34%** | **95.43%** | **95.35%** |
| XGBoost | 92.74% | 92.55% | 92.74% | 92.42% |
| Random Forest | 85.01% | 86.38% | 85.01% | 80.93% |

**Logistic Regression performed best overall** in this comparison, outperforming the two tree-based models on accuracy and weighted F1-score.

### Class-level observations

The notebook uses `LabelEncoder`, which sorts the four labels alphabetically:

| Encoded class | Label | Logistic Regression F1-score | Random Forest F1-score | XGBoost F1-score |
| --- | --- | ---: | ---: | ---: |
| 0 | Benign | 0.97 | 0.91 | 0.96 |
| 1 | Defacement | 0.98 | 0.92 | 0.94 |
| 2 | Malware | 0.94 | 0.82 | 0.90 |
| 3 | Phishing | 0.85 | 0.25 | 0.76 |

Phishing was the most challenging class in the supplied reports, especially for Random Forest, which achieved only **0.14 recall** on that class. These results show why per-class metrics matter alongside overall accuracy.

## Streamlit Application

The application provides:

1. A text field for entering a URL.
2. A **Predict** button to run all three trained models.
3. Predicted URL categories and highest predicted class probability for each model.
4. Per-class probability breakdowns.
5. Green success messages for benign predictions and red alerts for malicious predictions.
6. A prediction history retained in the current Streamlit session.

The app uses `predict()` and `predict_proba()` from the trained models. The displayed probability is a **model output, not a calibrated guarantee of safety**.

## Running the Project Locally

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/Malicious-URL-Detection.git
cd Malicious-URL-Detection
```

Replace `<your-username>` with your GitHub username.

### 2. Create an environment and install dependencies

```bash
python -m venv .venv
```

Activate it using `.venv\Scripts\activate` on Windows or `source .venv/bin/activate` on macOS/Linux, then install the dependencies:

```bash
pip install pandas numpy matplotlib seaborn wordcloud scikit-learn xgboost streamlit tldextract
```

`pyngrok` is optional and is only needed if you want to expose a local Streamlit server through a tunnel.

### 3. Prepare the model artifacts

The app expects these four files in its working directory:

```text
vectorizer.pkl
rf_model.pkl
lr_model.pkl
xgb_model.pkl
```

Train the models using the notebook and serialize the fitted vectorizer and **all three** trained models. The supplied notebook writes the vectorizer and XGBoost model but **loads** the Random Forest and Logistic Regression files; make sure those files exist before starting the app.

> Only load pickle files that you created or obtained from a trusted source. Unpickling untrusted files can execute arbitrary code.

### 4. Start Streamlit

```bash
streamlit run app.py
```

Streamlit will display a local address in your terminal, typically `http://localhost:8501`.

**Note:** These instructions assume you have added the original notebook, `app.py`, and model artifacts to the repository or generated the artifacts locally. This README alone does not include the application code or trained models.

## Implementation Notes and Improvements

The supplied code demonstrates the end-to-end development workflow, but the following corrections are important for reproducibility and reliable predictions:

1. **Fit TF-IDF after the split.** The original notebook calls `fit_transform()` before `train_test_split()`, allowing the vectorizer to learn vocabulary from the test set. Fit on training URLs and use `transform()` on test URLs instead; then retrain and reevaluate the models.
2. **Use consistent URL preprocessing.** Train and predict using the same raw or cleaned URL representation.
3. **Correct the Streamlit label mapping.** The original app maps `2` to Phishing and `3` to Malware, but the fitted `LabelEncoder` maps `2` to Malware and `3` to Phishing. Save the encoder and use `inverse_transform()` or the correct mapping.
4. **Save every trained artifact.** Serialize `rf`, `lr`, `xgb`, and the fitted TF-IDF vectorizer together with their compatible versions.
5. **Validate the final app.** Retest predictions after fixing preprocessing and label mapping. The original accuracy figures above should not be presented as new, leakage-free results.

## Limitations and Future Enhancements

- URL-only features cannot inspect the actual content, behavior, or reputation of a website.
- The dataset's class imbalance can affect detection of less frequent threats.
- Prediction probabilities are not necessarily calibrated confidence estimates.
- A URL classified as benign is **not guaranteed to be safe**.
- Potential enhancements include stronger URL validation, consistent preprocessing pipelines, calibrated probabilities, class-imbalance strategies, additional security features, and deployment as an API or browser extension.

## References

- [Malicious URLs Dataset — Kaggle](https://www.kaggle.com/datasets/sid321axn/malicious-urls-dataset/data)
- Sahoo, D., Liu, C., & Hoi, S. C. H. *Malicious URL Detection using Machine Learning: A Survey*. [arXiv:1701.07179](https://arxiv.org/abs/1701.07179).
- Pedregosa, F., et al. (2011). *Scikit-learn: Machine Learning in Python*. Journal of Machine Learning Research, 12, 2825–2830.
- Chen, T., & Guestrin, C. (2016). *XGBoost: A Scalable Tree Boosting System*. Proceedings of KDD.

---

**Project category:** Machine Learning Application Development · Cybersecurity · Python
