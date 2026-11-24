# SVM vs. Ranking SVM for Software Defect Prediction

Software defect prediction experiments (2022). Code metrics from the PROMISE repository
are used to predict defect-prone classes. The scripts compare a standard RBF SVM classifier
with a pairwise **Ranking SVM**. The classifier is scored with 10-fold cross-validation
(accuracy, precision, recall, F1). The Ranking SVM is trained on one release of a project
and scored on the next (ROC AUC).

## Repository structure

```
.
├── SVM.py                    # RBF SVM, 10-fold CV on camel-1.2.csv (runs out of the box)
├── camel-1.2.csv             # PROMISE defect data: Apache Camel 1.2, 608 classes, 20 CK/OO metrics + bug count
├── compareSVMandSVMrank/
│   ├── SVM.py                # same SVM baseline; dataset path given as an argument (default: log4j-1.1)
│   ├── SVMrank.py            # Ranking SVM trained on one release, AUC on the next (default: xalan-2.5 -> 2.6)
│   ├── pair.py               # builds pairwise difference vectors x_i - x_k with label sign(y_i - y_k)
│   ├── r_train.py            # fits an RBF SVC on the pairs
│   ├── r_predict.py          # scores each item by summing decision values against all other items
│   ├── tool.py, baseline.py, pca.py   # thresholding, cosine-similarity baseline, optional PCA
│   ├── main.py, x.csv, y.csv # demo data and loader for the ranking-SVM helpers
│   └── README.md             # notes on the ranking-SVM helpers
└── requirements.txt
```

The pairwise Ranking SVM helpers in `compareSVMandSVMrank/` (`pair.py`, `r_train.py`,
`r_predict.py`, `tool.py`, `baseline.py`, `pca.py`, `main.py`, the demo `x.csv`/`y.csv` and
its README) appear to be adapted from a separate "Ranking_SVM" project. The original commit
pointed to it as a git submodule.

## Approach

- **Labels:** a class counts as defective if its `bug` count is greater than 0.
- **Features:** the 20 object-oriented metrics in each PROMISE CSV (`wmc`, `dit`, `noc`,
  `cbo`, `rfc`, `lcom`, `ca`, `ce`, `npm`, `lcom3`, `loc`, `dam`, `moa`, `mfa`, `cam`, `ic`,
  `cbm`, `amc`, `max_cc`, `avg_cc`).
- **SVM baseline** (`SVM.py`): `sklearn.svm.SVC(kernel='rbf')` with 10-fold `KFold`. It
  prints the mean accuracy, precision, recall and F1.
- **Ranking SVM** (`compareSVMandSVMrank/SVMrank.py`): every pair of training classes with
  different labels becomes one training example, the difference of their feature vectors,
  labelled with the sign of their label difference. An RBF SVC is trained on these pairs.
  Each test class is scored by summing the decision function over its differences with
  every other test class. The ROC AUC of these scores is reported on the next release
  (cross-version prediction).

## Getting started

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

python SVM.py                                   # 10-fold CV on camel-1.2.csv

cd compareSVMandSVMrank
python SVM.py ../camel-1.2.csv                  # same baseline on any PROMISE-format CSV
python SVMrank.py train.csv test.csv            # e.g. xalan-2.5.csv xalan-2.6.csv
```

Without arguments, the scripts in `compareSVMandSVMrank/` look for
`../data software defect prediction/dataApache(PROMISE)/{log4j-1.1,xalan-2.5,xalan-2.6}.csv`,
a local data folder that is not part of this repository. The Ranking SVM builds O(n²)
training pairs, so it is slow on full releases with several hundred classes.

## Data

`camel-1.2.csv` is the Apache Camel 1.2 release from the PROMISE software defect
prediction datasets: class-level CK/object-oriented metrics with a post-release bug count.
The other releases the scripts mention (log4j-1.1, xalan-2.5, xalan-2.6) come from the same
collection and need to be downloaded separately.

## Tech stack

Python 3, scikit-learn, NumPy, pandas, matplotlib.

## License

The code is released under the [MIT License](LICENSE). The PROMISE dataset keeps its
original terms.
