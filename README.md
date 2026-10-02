# Page-level classification of environmental assessment documents (OCR + LayoutLM)

Classifies individual pages of scanned and digital PDF documents into three classes using OCR text and the position of each word on the page:

| Class | Meaning |
|---|---|
| `chart_map` | pages dominated by a chart or a map |
| `correspondence` | letters and emails |
| `other` | everything else (report text, tables of contents, forms, ...) |

## Pipeline

```
PDF pages ──► Tesseract OCR ──► words + normalised bounding boxes ──► LayoutLM (fine-tuned) ──► page label
              (notebook 01)                                             (notebook 02)
```

1. **`notebooks/01_ocr_extraction.ipynb`** renders every labelled page, runs Tesseract and saves the words with their bounding boxes (scaled to 0–1000, as LayoutLM expects).
2. **`notebooks/02_layoutlm_classification.ipynb`** fine-tunes `microsoft/layoutlm-base-uncased` and evaluates it.

## Method

- **Document-wise split.** All pages of a document go either to train/validation or to test, so the model is never tested on a document it saw during training. Splits are stratified by each document's most frequent label (80% / 20% of documents).
- **Data:** public documents from the [YESAB Registry](https://yesabregistry.ca/): 1,340 labelled pages (1,087 train/validation pages from 380 documents, 253 test pages from 95 documents). Pages with no OCR text were removed.
- **Model:** LayoutLM base, 3 epochs, learning rate 5e-5, batch size 1, AdamW, sequence length 512.
- **5-fold cross-validation** on the train/validation documents (also grouped by document). The fold with the best validation accuracy is evaluated on the held-out test set.

## Results (held-out test set, 253 pages)

| Class | Precision | Recall | F1 | Pages |
|---|---|---|---|---|
| chart_map | 0.60 | 0.74 | 0.66 | 34 |
| correspondence | 0.99 | 0.96 | 0.98 | 114 |
| other | 0.87 | 0.83 | 0.85 | 105 |
| **accuracy** | | | **0.88** | 253 |
| macro avg | 0.82 | 0.84 | 0.83 | 253 |
| weighted avg | 0.89 | 0.88 | 0.88 | 253 |

## Limitations

- `chart_map` is the weakest class (F1 0.66): the model only sees OCR words and their positions, not the image itself, and charts and maps contain little text.
- Validation accuracy varied a lot between folds (from about 18% to 94%), so training with batch size 1 is not very stable. Results come from a single run, and the test set is small.
- OCR quality on scanned pages is imperfect, and errors propagate to the classifier.

## How to run

1. Install the Tesseract binary (e.g. `sudo apt install tesseract-ocr`; on macOS `brew install tesseract`).
2. Install the Python dependencies: `pip install -r requirements.txt`
3. Put the label CSVs in `data/labels/` and the PDFs on disk (see `data/README.md`), then set the paths at the top of notebook 01.
4. Run notebook 01, then notebook 02. A GPU is strongly recommended for notebook 02.

## Repository structure

```
├── notebooks/
│   ├── 01_ocr_extraction.ipynb
│   └── 02_layoutlm_classification.ipynb
├── data/README.md        # expected data layout and sources
├── requirements.txt
└── README.md
```
