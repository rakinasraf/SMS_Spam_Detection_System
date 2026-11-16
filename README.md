# 📩 SMS Spam Detection System

An NLP project that classifies SMS messages as **spam** or **ham** (legitimate) using classical machine learning. We compare Bag of Words, TF-IDF (unigram and unigram+bigram), and average Word2Vec representations with a majority-class baseline, Multinomial Naive Bayes, and Logistic Regression, and we run an ablation study on stop-word removal and stemming.

> **Best result:** TF-IDF (unigram + bigram) + Logistic Regression → **98.69% accuracy** and **97.06% macro-F1** on the held-out test set.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Dataset](#-dataset)
- [Methodology](#-methodology)
- [Results](#-results)
- [Error Analysis](#-error-analysis)
- [Limitations](#-limitations)
- [Getting Started](#-getting-started)
- [Project Structure](#-project-structure)
- [Team](#-team)
- [References](#-references)

---

## 🔎 Overview

Unwanted and fraudulent SMS messages (fake prizes, credit scams, unsolicited ads) are a real nuisance and a financial risk. Rule-based filters break when spammers change their wording, so this project learns patterns directly from labelled messages.

**Main contributions**

1. A reusable SMS preprocessing pipeline, with an ablation study on stop-word removal and stemming.
2. A comparison of BoW, TF-IDF (unigram and unigram+bigram), and average Word2Vec features.
3. An evaluation of a majority-class baseline, Multinomial Naive Bayes, and Logistic Regression on a stratified train/validation/test split.
4. Identification of TF-IDF (uni+bi) + Logistic Regression as the best configuration.

---

## 📊 Dataset

- **Source:** [UCI SMS Spam Collection](https://doi.org/10.24432/C5CC84) (Almeida & Hidalgo, 2011)
- **Size:** 5,574 labelled English SMS messages (4,827 ham / 747 spam)
- **CSV copy used:** [`sms_spam.csv`](https://raw.githubusercontent.com/stedy/Machine-Learning-with-R-datasets/master/sms_spam.csv)

| Split      | Messages |
|------------|---------:|
| Training   | 3,901    |
| Validation | 836      |
| Test       | 837      |
| **Total**  | **5,574** |

Stratified 70:15:15 split with `random_state = 42`. Ham is about 86.6% of the training data and spam about 13.4%, so **macro-F1** is the primary metric.

---

## ⚙️ Methodology

**Workflow:** Dataset → Stratified split → Preprocessing → Feature extraction → Model training and tuning → Evaluation → Ham / Spam

### 1. Preprocessing
- Lowercasing
- Removal of HTML tags/entities, URLs, and numbers
- Removal of punctuation, special symbols, and emojis
- Whitespace tokenization
- Porter stemming (stop words **kept**, chosen via the ablation study)

### 2. Text Representations
- **Bag of Words (BoW)**
- **TF-IDF**: unigram and unigram+bigram
- **Average Word2Vec**: 100-dimensional, trained on the training messages only

### 3. Classifiers
- Majority-class baseline
- Multinomial Naive Bayes (α ∈ {0.01, 0.1, 0.5, 1, 2})
- Logistic Regression (C ∈ {0.1, 1, 10, 100})

Hyperparameters were selected with validation macro-F1, and the test set was used only for the final evaluation.

### 4. Preprocessing Ablation (validation macro-F1)

| Stop-word removal | Stemming | NB + BoW | LR + TF-IDF | Mean |
|:--:|:--:|:--:|:--:|:--:|
| No  | No  | 0.9715 | 0.9738 | 0.9727 |
| No  | Yes | 0.9642 | 0.9818 | **0.9730** |
| Yes | No  | 0.9591 | 0.9653 | 0.9622 |
| Yes | Yes | 0.9570 | 0.9628 | 0.9599 |

Keeping stop words and applying Porter stemming performed best.

---

## 🏆 Results

Test-set performance:

| Model | Acc. | Macro-P | Macro-R | Macro-F1 |
|---|:--:|:--:|:--:|:--:|
| **LR + TF-IDF (Uni+Bi)** | **0.9869** | **0.9883** | **0.9547** | **0.9706** |
| LR + TF-IDF (Uni) | 0.9857 | 0.9876 | 0.9502 | 0.9679 |
| LR + BoW | 0.9845 | 0.9869 | 0.9457 | 0.9650 |
| MNB + TF-IDF (Uni) | 0.9821 | 0.9855 | 0.9368 | 0.9593 |
| MNB + TF-IDF (Uni+Bi) | 0.9797 | 0.9756 | 0.9354 | 0.9543 |
| MNB + BoW | 0.9749 | 0.9607 | 0.9289 | 0.9440 |
| LR + Word2Vec | 0.9534 | 0.9304 | 0.8599 | 0.8907 |
| Majority Baseline | 0.8662 | 0.4331 | 0.5000 | 0.4641 |

**Best model (LR + TF-IDF uni+bi):** Accuracy 98.69% · Macro-F1 97.06% · Macro-Recall 95.47% · Spam Recall 91.07%

**Confusion matrix (837 test messages):** 724 ham and 102 spam were classified correctly, with 1 false alarm (ham → spam) and 10 missed spam messages (spam → ham).

The majority baseline reaches 86.62% accuracy but only 0.4641 macro-F1, which shows why accuracy alone is misleading on imbalanced data.

---

## 🧐 Error Analysis

Across all seven trained models, 49 distinct test messages were misclassified by at least one model. A rule-based grouping gave:

| Error Category | Count |
|---|---:|
| Missed spam: promotional or commercial cues | 26 |
| Missed spam: SMS slang and informal language | 8 |
| False alarm: genuine message with spam-like terms | 7 |
| False alarm: other genuine message | 4 |
| False alarm: short or ambiguous message | 4 |
| **Total** | **49** |

The categories are approximate: they come from keyword and message-length rules, not manual inspection.

**Most informative words (Naive Bayes log-ratio)**
- Spam: `claim`, `prize`, `tone`, `cs`, `pobox`, `guarante`, `ppm`, `award`, `servic`, `rington`
- Ham: `he`, `lor`, `later`, `she`, `my`, `but`, `home`, `wat`, `da`, `way`

---

## ⚠️ Limitations

- English-only dataset, so results may not transfer to other languages or newer messaging platforms.
- 403 duplicate messages were found. Because the split is at the message level, similar messages may appear in different subsets and could affect test results.
- Slang, misspellings, negation, and context remain hard. Removing URLs, numbers, and symbols may also discard useful signals.

**Future work:** duplicate-aware splitting, larger and more diverse data, URL/phone-number features, and Transformer models (BERT, DistilBERT).

---

## 🚀 Getting Started

### Prerequisites
- Python 3.8+
- Jupyter Notebook / JupyterLab

### Installation

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
pip install pandas numpy scikit-learn nltk gensim matplotlib seaborn jupyter
```

### Run

```bash
jupyter notebook
```

Open the project notebook, then run all cells from top to bottom. The notebook downloads the dataset CSV from GitHub, so an internet connection is required on the first run.

---

## 🗂️ Project Structure

```
.
├── <notebook-name>.ipynb      # Full pipeline: EDA → preprocessing → modelling → evaluation
├── <paper-name>.pdf           # IEEE-style project report
└── README.md
```

> Update the file names above to match your repository.

---

## 👥 Team

| Name | ID | Contribution |
|---|---|---|
| Saruar Hossain Shovo | 23-50314-1 | Feature representations (BoW, TF-IDF, Word2Vec) and classifier training |
| Farhan Chowdhury | 23-52314-2 | Dataset loading setup, text preprocessing, IEEE report writing |
| Md. Asraf Uddin | 23-52310-2 | Text preprocessing pipeline (regex cleaning, tokenization, stop-words, stemming) |
| Shahriyar Lipu | 24-56330-1 | Evaluation, confusion matrix, error analysis, project report writing |

---

## 📚 References

1. N. N. A. Sjarif et al., "SMS spam message detection using term frequency-inverse document frequency and random forest algorithm," *Procedia Computer Science*, vol. 161, pp. 509–515, 2019.
2. T. Şenol, H. Takçı, and A. Şeker, "SMS spam detection via a systematic machine learning pipeline," *Journal of Engineering Research*, vol. 14, no. 3, pp. 2941–2952, 2026.
3. H. Jain and Mahadev, "An analysis of SMS spam detection using machine learning model," *CCICT 2022*, pp. 151–156.
4. T. Almeida and J. Hidalgo, "SMS Spam Collection," 2011. https://doi.org/10.24432/C5CC84

---

## 🤖 AI-Use Statement

Claude and ChatGPT were used to assist with understanding concepts, debugging code, and improving the report's language and structure. All outputs were reviewed and verified by the group members, who take full responsibility for the submitted work.
