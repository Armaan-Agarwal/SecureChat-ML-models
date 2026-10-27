# SecureChat ML Models: Spam and Phishing Detection

Machine learning models for **SecureChat**, a chat application that screens every message and blocks phishing and spam before it reaches the recipient. This repository contains only the ML code: spam detection on message text and phishing detection on URLs. The chat application itself (React frontend, Node.js/Express backend, MongoDB, encryption) is not included here.

Final-year Bachelor of Engineering project (Computer Engineering), PES Modern College of Engineering, Savitribai Phule Pune University, 2024-25.

## Repository contents

| File | Purpose |
|---|---|
| `Spam_detection_model.ipynb` | Trains, evaluates and saves the spam model, and runs predictions on new messages |
| `Phishing_detection_model.ipynb` | Extracts 30 features from a URL and classifies it with the trained phishing model |
| `spam_detection_svm_model.pkl`, `tfidf_vectorizer.pkl` | Saved spam model and its TF-IDF vectorizer |
| `model.pkl` | Saved phishing model (Gradient Boosting Classifier) |

## Models

| Task | Input | Algorithm | Features |
|---|---|---|---|
| Spam detection | Message text | SVM (linear kernel) | TF-IDF vectors |
| Phishing detection | URL | Gradient Boosting Classifier | 30 URL, domain and page-content features |

In the full application, a message is blocked if either model flags it.

### Spam model

- Dataset: `super_sms_dataset.csv` ([source]). Rows with missing values are dropped and labels are cast to integers (0 = not spam, 1 = spam).
- Split: 67% train, 33% test, stratified by label, `random_state=42`.
- Features: `TfidfVectorizer(stop_words='english', max_features=5000)`.
- Model: `SVC(kernel='linear', probability=True)`, so predictions come with a confidence score.
- Evaluation: accuracy, classification report and confusion matrix (plotted as a heatmap).
- Output: the model and vectorizer are saved with `joblib`.

### Phishing model

The `FeatureExtraction` class turns a URL into a vector of 30 values, each +1 (legitimate), 0 (suspicious) or -1 (phishing). The features fall into three groups:

- **URL-based:** IP address in URL, URL length, shortening service, `@` symbol, `//` redirection, hyphen in domain, number of subdomains, HTTPS, non-standard port, "https" in the domain name.
- **Domain-based:** registration length, domain age, DNS record, website traffic, page rank, Google index, match against known suspicious hosts and IPs.
- **Page-content-based:** favicon, request URLs, anchor URLs, links in script tags, form handler, email submission, abnormal URL, redirects, status-bar customization, right-click disabled, pop-up windows, iframe redirection, links pointing to the page.

The Gradient Boosting Classifier (an ensemble of decision trees built sequentially, each correcting the previous one's errors) classifies the feature vector and returns the phishing and non-phishing probabilities. The notebook loads the trained model from `model.pkl`. [Training code is not included in this repository.]

## Results

Results from the project report:

| Model | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| Spam (SVM + TF-IDF) | 98.22% | 98% | 97% (spam), 99% (not spam) | 98% |
| Phishing (Gradient Boosting) | 95.70% (test), 96.01% (train) | 95.31% | 97.08% | 96.18% |

Confusion matrices (test sets):

| | Spam model | Phishing model |
|---|---|---|
| True positives | 8,375 | 917 |
| True negatives | 13,345 | 1,199 |
| False positives | 129 | 36 |
| False negatives | 264 | 59 |

## Getting started

Requirements: Python 3.12 (tested), `pandas`, `numpy`, `scikit-learn`, `joblib`, `matplotlib`, `seaborn`, `requests`, `beautifulsoup4`, `lxml`, `python-whois`, `googlesearch-python`, `python-dateutil`.

```bash
git clone [repository URL]
cd [project folder]
pip install -r requirements.txt
jupyter notebook
```

### Spam detection

Open `Spam_detection_model.ipynb` and run the training and evaluation cell to produce `spam_detection_svm_model.pkl` and `tfidf_vectorizer.pkl`. Place `super_sms_dataset.csv` in the project folder first. To classify a message with the saved files:

```python
import joblib

model = joblib.load("spam_detection_svm_model.pkl")
vectorizer = joblib.load("tfidf_vectorizer.pkl")

message = "Dear user, confirm your banking details immediately to avoid account suspension"
features = vectorizer.transform([message])
label = model.predict(features)[0]                    # 1 = spam, 0 = not spam
confidence = model.predict_proba(features).max() * 100
print("SPAM" if label == 1 else "NOT SPAM", f"({confidence:.2f}% confidence)")
```

### Phishing detection

Open `Phishing_detection_model.ipynb`, run the `FeatureExtraction` cell, then:

```python
result = check_url_safety("https://example.com")
# {"url": ..., "is_safe": "safe" | "unsafe", "phishing_prob": ..., "non_phishing_prob": ...}
```

Feature extraction makes live HTTP and WHOIS requests to the URL being checked. Run it in an isolated environment when testing suspicious links.

## Limitations

- Both models are trained on specific datasets and may not generalize to new attack patterns. They can misclassify legitimate messages or miss malicious ones.
- Spam detection works on message text and phishing detection works on URLs. The phishing model does not read message content.
- Some phishing features depend on external services (WHOIS, web-ranking and search services) and fall back to a fixed value when a service is unavailable.
- Models need periodic retraining as spam and phishing patterns change.

## Team

Armaan Agarwal, Prathmesh Dhole, Yash Jaiswal, Harshad Parihar.
Guide: Mr. D. G. Modani, Department of Computer Engineering, PES Modern College of Engineering, Pune.

**My contribution:** Built the spam and phishing detection models, found and prepared the datasets, and led the four-member team, assigning responsibilities across the project.
