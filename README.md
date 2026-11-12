# Malicious URL Detection Using Machine Learning

A four-class URL classification project that compares **Logistic Regression**, **XGBoost**, and **Random Forest** to identify **benign**, **defacement**, **malware**, and **phishing** URLs using character-level TF-IDF features. An interactive **Streamlit** interface displays predictions from all three models, class probabilities, confidence scores, and prediction history.

> **Project status:** Research prototype. The evaluation scores below are from the original notebook and have not been rerun with a leakage-free preprocessing pipeline. Do not use predictions as the sole basis for security decisions.

## Results

The original experiment used an 80/20 stratified split (`random_state=42`) and evaluated **130,239** test URLs. Precision, recall, and F1 are **weighted averages**.

| Model | Accuracy | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|
| **Logistic Regression** | **95.43%** | **95.34%** | **95.43%** | **95.35%** |
| XGBoost | 92.74% | 92.55% | 92.74% | 92.42% |
| Random Forest | 85.01% | 86.38% | 85.01% | 80.93% |

Logistic Regression performed best in this experiment. Random Forest had particularly low recall for phishing (0.14), while Logistic Regression achieved phishing recall of 0.81.

## Dataset

[Malicious URLs Dataset on Kaggle](https://www.kaggle.com/datasets/sid321axn/malicious-urls-dataset/data) — **651,191 URLs**.

| Class | Count |
|---|---:|
| Benign | 428,103 |
| Defacement | 96,457 |
| Phishing | 94,111 |
| Malware | 32,520 |

The dataset is imbalanced; benign URLs are the majority class.

## Approach

1. **Explore the data:** inspect URL categories and generate class-specific word clouds.
2. **Encode labels:** use `LabelEncoder` to convert string labels into model targets.
3. **Extract features:** character-level TF-IDF with 3–5-character n-grams and a maximum of 5,000 features.
4. **Train models:** Random Forest (50 trees, maximum depth 10), Logistic Regression (`max_iter=1000`), and XGBoost (100 estimators, maximum depth 4, learning rate 0.1).
5. **Evaluate:** accuracy, weighted precision/recall/F1, per-class classification reports, and confusion matrices.
6. **Predict interactively:** a Streamlit app compares the models' predictions, confidence scores, class probabilities, and session history.

### Workflow

```text
Kaggle URL dataset
       |
Exploration + label encoding
       |
Character TF-IDF (3–5 grams, max 5,000 features)
       |
80/20 stratified split
       |
Random Forest / Logistic Regression / XGBoost
       |
Classification reports + confusion matrices
       |
Streamlit: URL input -> three predictions -> probabilities + history
```

**Important:** This diagram reflects the original notebook, where TF-IDF was fitted before splitting. See *Known limitations*.

## Technologies

- **Python**, pandas, NumPy
- **scikit-learn**: TF-IDF, label encoding, model training, evaluation
- **XGBoost**
- **Matplotlib, Seaborn, WordCloud** for visualizations
- **Streamlit** for the interactive interface
- **pickle** for local model serialization
- **pyngrok** for optional temporary demo tunneling

## Running locally

The Streamlit application expects these files in the same directory:

```text
app.py
vectorizer.pkl
rf_model.pkl
lr_model.pkl
xgb_model.pkl
```

Once the trained artifacts have been generated and the issues below are addressed:

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install pandas numpy scikit-learn xgboost streamlit matplotlib seaborn wordcloud
streamlit run app.py
```

The notebook's Kaggle dataset path (`/kaggle/input/malicious-urls-dataset/malicious_phish.csv`) must be adjusted when running outside Kaggle. Only load pickle files that you trust.

## Known limitations and next improvements

- **Train/test leakage:** the notebook calls `vectorizer.fit_transform(df['url'])` *before* splitting. Fit TF-IDF on training URLs only and use `transform` on held-out URLs; then rerun evaluation and update the reported metrics.
- **Training/inference mismatch:** the notebook trains on original `url` strings, while the app calls `clean_url()` before vectorization. Apply identical preprocessing in both paths, ideally in a saved pipeline.
- **Incorrect Streamlit label dictionary:** `LabelEncoder` uses alphabetical ordering: `0=benign`, `1=defacement`, `2=malware`, `3=phishing`. The original app swaps malware and phishing. Save and load the fitted encoder, or correct the mapping.
- **Incomplete model-saving snippet:** the provided notebook writes `vectorizer.pkl` and `xgb_model.pkl`, but reads `rf_model.pkl` and `lr_model.pkl`. Save all three trained models before loading them in the app.
- **Model confidence:** `predict_proba` scores are not necessarily calibrated probabilities; confidence should not be treated as a security guarantee.
- **Generalization:** performance on a single random split may not reflect novel or changing threats. Consider time-based validation, duplicate URL checks, per-class metrics, and calibration.

## Future work

- Evaluate on a leakage-free holdout set and external, newer URLs.
- Add lexical features such as URL length, digits, symbols, and domain structure; these were discussed in the proposal but are **not** implemented in the provided training code.
- Improve minority-class detection and compare calibration methods.
- Package the preprocessing and best-performing model into a reproducible inference pipeline.
- Explore an API or browser extension after further security validation.

## References

- [Malicious URLs Dataset — Kaggle](https://www.kaggle.com/datasets/sid321axn/malicious-urls-dataset/data)
- Sahoo, D., Liu, C., & Hoi, S. C. H. (2019). *Malicious URL Detection using Machine Learning: A Survey*. [arXiv:1701.07179](https://arxiv.org/abs/1701.07179).
- Pedregosa, F., et al. (2011). *Scikit-learn: Machine Learning in Python*. JMLR.
- Chen, T., & Guestrin, C. (2016). *XGBoost: A Scalable Tree Boosting System*. KDD.
