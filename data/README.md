# Data

The PDFs and OCR output are **not** stored in this repository. Only the small label files (`data/labels/`) are meant to be shared.

## Where the documents come from

- **Project documents:** downloaded from the public [YESAB Registry](https://yesabregistry.ca/) (Yukon Environmental and Socio-economic Assessment Board), which hosts documents for mining and other projects assessed in Yukon.
- **Project selection:** the projects were not chosen at random. They were selected with help from mining department, as projects of interest for a mining use case. The criteria they used for choosing them are not known, so the sample may not be representative of the whole registry.
- **Correspondence documents:** YESAB documents contained too few correspondence pages, so additional letters and emails were provided by the same person who helped with project selection. They are understood to be publicly available documents, but their original source was not recorded, so it cannot be stated here.

## Labels

Each page was labelled with one of the classes `chart`, `map`, `correspondence` or `other`. Charts and maps are merged into a single `chart_map` class when training (see notebook 02). Pages labelled `table` were excluded.

Each label CSV has one row per page, with columns such as `path` (the PDF file), `page_num` and `label`.

## Expected layout

```
data/
├── labels/                  # label CSVs (included in this repo)
│   ├── YESAB_data_train.csv
│   ├── YESAB_data_test.csv
│   └── correspondences_data.csv
├── yesab_ocr.json           # created by notebooks/01_ocr_extraction.ipynb
├── correspondences_ocr.json # created by notebooks/01_ocr_extraction.ipynb
└── pages_ocr.json           # final OCR dataset used for training
```

The `path` column contains the file paths from the machine where the labels were created. Set `OLD_PATH_PREFIX` and `NEW_PATH_PREFIX` at the top of notebook 01 to point to wherever you saved the PDFs.
