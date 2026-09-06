# Comparative Analysis of Extractive Text Summarization Methods on Long Documents

MSc Computer Science final semester research project — a comparative evaluation of three extractive text summarization methods on long-form documents from the WikiHow dataset.

This is an **evaluation study**, not a novel model proposal. It compares statistical, graph-based, and embedding-based extractive summarization approaches specifically on documents 800+ words in length — a range underrepresented in most summarization literature, which tends to focus on short news datasets like CNN/DailyMail.

## Motivation

Most extractive summarization research is evaluated on short-to-medium documents. Real-world text — instructional guides, reports, structured articles — is often much longer, and the important content is distributed throughout rather than concentrated at the start (unlike news articles, which follow an "inverted pyramid" style that lets naive baselines like Lead-3 perform deceptively well).

WikiHow was chosen specifically because it resists this Lead-3 bias — each article's key information (its "Steps") is spread across the entire document, making it a more rigorous test of whether a summarization method can identify globally relevant content in long text.

## Methods Compared

| Method | Type | Approach |
|---|---|---|
| **TF-IDF** | Statistical baseline | Sentences scored by summed TF-IDF values; top 5 selected |
| **LexRank** | Graph-based | Sentence similarity graph (cosine similarity on TF-IDF vectors), ranked via eigenvector centrality |
| **SBERT** | Embedding-based | Sentences encoded via `all-MiniLM-L6-v2`, clustered with K-Means, one representative sentence per cluster |

## Dataset

- **Source:** [WikiHow dataset](https://github.com/mahnazkoupaee/WikiHow-Dataset) (Koupaee & Wang, 2018) — ~215,000 articles
- **Filtering:** Articles with fewer than 800 words excluded → ~30,700 long documents remain
- **Sample:** 500 randomly sampled articles (fixed random seed for reproducibility)
- **Reference summaries:** WikiHow's bolded step headlines, used as ground-truth summaries

## Evaluation Metrics

- **ROUGE-1 / ROUGE-2 / ROUGE-L** (F-measure) — standard summarization overlap metrics
- **Content Coverage** — ROUGE-1 recall, used as a proxy for how much reference content is captured
- **Redundancy** — mean pairwise cosine similarity between summary sentences (via SBERT embeddings)

## Results

Average scores across 500 sampled documents:

| Method | ROUGE-1 | ROUGE-2 | ROUGE-L | Coverage | Redundancy |
|---|---|---|---|---|---|
| TF-IDF | 0.250 | 0.070 | 0.136 | **0.532** | 0.399 |
| LexRank | 0.278 | **0.078** | 0.152 | 0.487 | 0.442 |
| SBERT | **0.309** | 0.072 | **0.166** | 0.370 | **0.325** |

**Key findings:**

- **SBERT achieved the best overall ROUGE performance** and produced the least redundant summaries, but had the lowest lexical coverage — likely because ROUGE is a purely lexical metric and doesn't credit semantically-equivalent but differently-worded sentences.
- **LexRank achieved the highest ROUGE-2**, suggesting its centrality-based sentence selection captures more exact phrase overlap with references than SBERT's semantically diverse selections.
- **TF-IDF had the highest coverage but also higher redundancy**, consistent with its purely frequency-driven sentence selection.
- **No evidence of performance degradation with increasing document length** within the 800+ word range studied (Pearson r ≈ 0.056–0.075 between word count and ROUGE-1 across all three methods). This finding is scoped to the long-document range only — it does not speak to the well-documented short-to-long degradation transition.

Full analysis, visualizations, and discussion are available in the [dissertation report](./report/dissertation.pdf).

## Repository Structure

```
├── notebooks/
│   ├── 01_tfidf.ipynb           # TF-IDF summarizer + evaluation
│   ├── 02_lexrank.ipynb         # LexRank summarizer + evaluation
│   ├── 03_sbert.ipynb           # SBERT summarizer + coverage/redundancy
│   └── 04_visualization.ipynb   # All charts and correlation analysis
├── data/
│   └── sample.pkl               # 500-article evaluation sample (pickled)
├── results/
│   ├── results_table.csv
│   └── charts/                  # Saved visualization outputs
├── report/
│   └── dissertation.pdf
├── requirements.txt
└── README.md
```

> **Note:** The full WikiHow dataset (~580MB) is not included in this repository. Download `wikihowAll.csv` from the [official dataset repository](https://github.com/mahnazkoupaee/WikiHow-Dataset) and place it in `data/raw/` to reproduce results from scratch.

## Setup & Reproduction

```bash
# Clone the repo
git clone https://github.com/YOUR-USERNAME/extractive-summarization-wikihow.git
cd extractive-summarization-wikihow

# Create virtual environment
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # macOS/Linux

# Install dependencies
pip install -r requirements.txt

# Run notebooks in order: 01 → 02 → 03 → 04
```

## Tools & Libraries

- `pandas` — data loading and preprocessing
- `scikit-learn` — TF-IDF vectorization, K-Means clustering
- `sumy` — LexRank implementation
- `sentence-transformers` — SBERT embeddings (`all-MiniLM-L6-v2`)
- `rouge-score` — ROUGE evaluation
- `matplotlib` / `seaborn` — visualization

## Limitations & Future Work

- ROUGE-based coverage is a lexical metric and may underestimate the semantic quality of embedding-based summaries — a limitation worth addressing with semantic similarity metrics in future work.
- This study evaluates only the long-document range (800+ words); it does not test whether performance degrades in the transition from short to long documents.
- Future work could extend evaluation to domain-specific long-document datasets (e.g., PubMed, legal documents) and additional baselines (e.g., Lead-3, TextRank).

## Author

Vivek — MSc Computer Science, Mumbai University

## License

This project is for academic purposes. Please cite appropriately if referencing this work.
