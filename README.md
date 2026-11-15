# CSBP711 Assignment 2 Paper Critique

## Paper

Chen et al. (ACL 2025), *Can Indirect Prompt Injection Attacks Be Detected and Removed?*

## Purpose

This repository contains the reproducible code, data splits, and saved outputs
for the Assignment 2 reality check. The study evaluates a paper-inspired
`DeBERTa-v3-base` indirect prompt-injection detector on BIPIA
EmailQA, then compares it fairly with the Assignment 1 winner: a TF-IDF
unigram Linear SVM.

This is a scaled, paper-inspired reproduction, not an exact reproduction of
Chen et al. The original paper uses substantially larger QA datasets and a
different computing setting. This repository uses the BIPIA EmailQA data from
Assignment 1 and a category-disjoint grouped cross-validation protocol.

## Repository Structure

```text
.
├── README.md
├── requirements.txt
├── data/
│   ├── train.csv
│   └── test.csv
├── notebooks/
│   ├── 01\_data\_audit\_and\_setup.ipynb
│   └── 02\_category\_disjoint\_cv\_comparison.ipynb
└── results/
    ├── README.md
    ├── grouped\_cv\_deberta/
    ├── category\_disjoint\_cv\_deberta/
    └── pilot\_validation/
```

## Data

The data are paired BIPIA EmailQA samples. Each `context\_id` has one benign
email and one injected-email counterpart. The model uses only `text` as input
and `label` as the binary target. Metadata fields are excluded from model
input.

|Split|Rows|Contexts|Benign|Attack|
|-|-:|-:|-:|-:|
|Training|100|50|50|50|
|Test|74|37|37|37|

## Notebooks

1. **`01\_data\_audit\_and\_setup.ipynb`** validates the data, checks pair and
split integrity, examines duplicated texts and token lengths, and verifies
the DeBERTa tokenizer and forward pass.
2. **`02\_category\_disjoint\_cv\_comparison.ipynb`** is the primary reality-check
experiment. It runs ordinary grouped cross-validation as a robustness
reference, then runs the stricter category-disjoint grouped cross-validation
and compares DeBERTa with Linear SVM on the same folds.

## Evaluation Protocol

The primary protocol is five-fold category-disjoint grouped cross-validation
on the 100 training rows. Each validation fold contains 10 benign and 10
attack samples from 10 contexts. Three attack categories are held out in each
fold, so no validation attack category appears in that fold's training data.
The paired benign/attack emails remain in the same fold to prevent context
leakage.

The DeBERTa model uses head-tail truncation at 512 tokens, a head-only warm-up
phase, and full fine-tuning with a lower encoder learning rate. The Linear SVM
uses `TfidfVectorizer(ngram\_range=(1, 1), norm="l2")` and
`LinearSVC(class\_weight="balanced", random\_state=42)`.

## Primary Results

The strict category-disjoint results are the main evidence. Values below are
mean ± standard deviation across five folds.

|Metric|DeBERTa-v3-base|Linear SVM|
|-|-:|-:|
|Accuracy|0.900 ± 0.224|0.770 ± 0.091|
|Macro-F1|0.867 ± 0.298|0.758 ± 0.107|
|Attack recall|1.000 ± 0.000|0.620 ± 0.192|
|False-positive rate|0.200 ± 0.447|0.080 ± 0.110|
|ROC-AUC|0.930 ± 0.157|0.894 ± 0.081|

DeBERTa detects more attacks but has a higher false-positive rate. In one
held-out category group it predicted every benign email as an attack, which is
the over-defense behavior discussed in the critique. The Linear SVM is more
conservative but misses more attacks.

## Reproduction

1. Clone the repository or upload its contents to Google Drive.
2. In Google Colab, open each notebook from the `notebooks/` directory.
3. Select a GPU runtime before running the DeBERTa experiment.
4. Run `01\_data\_audit\_and\_setup.ipynb` first.
5. Run `02\_category\_disjoint\_cv\_comparison.ipynb` from top to bottom.

The notebooks detect Colab and mount Google Drive. Outside Colab, they use the
repository root as the project directory. Generated outputs are written to
`results/`.

## Important Interpretation Note

The original 74-example hold-out test was examined during early exploration.
The reported reality-check conclusion is based on category-disjoint
cross-validation only, because this protocol was fixed before interpreting its
results and provides a stricter test of transfer to unseen attack categories.

## Author

Najwa Abu Amssimir
CSBP711 Advanced Artificial Intelligence

