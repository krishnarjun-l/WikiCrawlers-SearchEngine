# The WikiCrawlers — A Wikipedia Computer-Science Search Engine

**Course:** CSC 575 — Intelligent Information Retrieval
**Team:** Krishnarjun Lakshminarayanan, Sanjaiya Mohandas

---

## 1. Overview

This project is an information-retrieval system over the Computer Science portion of
Wikipedia. It crawls full-text articles, builds an inverted index, and ranks search
results with two classical models — **BM25** (probabilistic) and **TF-IDF cosine**
(vector space) — and improves results with **Rocchio relevance feedback** (query
expansion). A **Flask** web app provides the search interface, and a separate notebook
evaluates both rankers on the standard **CISI** benchmark.

All retrieval logic (inverted index, BM25, TF-IDF cosine, and Rocchio) is implemented
from scratch — no external search/IR library is used for ranking.

---

## 2. Files in this submission

| File | Purpose |
|------|---------|
| `crawler.ipynb` | Crawls Wikipedia CS articles and builds the inverted index (writes `index.json`). |
| `main.ipynb` | The search engine: loads `index.json` and starts the Flask web app (BM25 / TF-IDF / Rocchio). |
| `eval.ipynb` | Evaluation on the CISI collection (MAP, MRR, P@10) with a comparison chart. |
| `index.json` | Pre-built inverted index for 2,050 articles, so you can run the search engine **without** re-crawling. |
| `README.md` | This file. |

> The CISI evaluation data is **not** bundled (it is a third-party collection). See Section 6 to obtain it.

---

## 3. Requirements

- **Python 3.9+**
- **Jupyter** (Notebook or JupyterLab), or **VS Code** with the Jupyter extension
- Python packages: `Wikipedia-API`, `nltk`, `flask`, `matplotlib`

Install the packages:

```bash
pip install Wikipedia-API nltk flask matplotlib
```

Download the one NLTK resource the project needs (run once, in Python or a notebook cell):

```python
import nltk
nltk.download('stopwords')
```

> The Porter stemmer ships with NLTK and needs no download. Tokenization is
> regular-expression based, so the `punkt` tokenizer is **not** required.

---

## 4. How to run

All three files are **Jupyter notebooks**. Open each one and use **Run All**, or run the
cells top to bottom. There are **no command-line arguments**; instead, each notebook has a
few configuration variables near the top, listed in Section 5.

### 4.1 `main.ipynb` — run the search engine (start here)

This is the main deliverable. It loads `index.json` and starts the Flask server.

1. Keep `index.json` in the **same folder** as `main.ipynb`.
2. **Run all cells.** The final cell calls `app.run(...)` and prints a line like:
   `* Running on http://127.0.0.1:5000`
3. Open **http://127.0.0.1:5000** in your browser.
4. In the page: type a query (e.g., `machine learning`), choose the ranking model
   (**BM25** or **TF-IDF**), and click **Search**. Each result shows the article title and
   a relevance score, best match first.
5. **Relevance feedback:** tick the checkbox next to one or more results that are relevant,
   then click the **Rocchio feedback** button. The query is expanded with terms taken from
   those documents and the results are re-ranked.
6. **To stop the server,** interrupt the kernel (the ■ / Stop button). `app.run()` runs
   until interrupted, so the cell will appear to "keep running" — that is expected; it is a
   server, not a script that finishes.

### 4.2 `crawler.ipynb` — rebuild the index (optional)

You only need this if you want to regenerate `index.json` from scratch — a pre-built
`index.json` is already included. The crawler performs a breadth-first traversal of the
Wikipedia Computer Science category tree (the top category mostly contains sub-categories,
so it recurses into them), preprocesses each article
(regex tokenize → stop-word removal → Porter stemming), and writes the inverted index,
document lengths, and article titles to `index.json`.

- **Run all cells.** Output: `index.json`.
- **Note:** the crawl is the only slow step — roughly **30–60 minutes**, single-threaded,
  using polite sequential requests to the MediaWiki API.

### 4.3 `eval.ipynb` — evaluation on CISI

Reproduces the quantitative results (MAP, MRR, P@10) for BM25 vs TF-IDF. It builds a fresh
index over the CISI collection using the **same** preprocessing and scoring as the live
system, then computes the metrics and a bar chart.

1. Obtain the CISI data (Section 6) and place it in a `cisi/` folder.
2. Confirm `CISI_DIR` at the top of the notebook points to that folder (default: `cisi`).
3. **Run all cells.** Runtime is under ~10 seconds. Output: a results table and a comparison chart.

---

## 5. Configuration variables (the "arguments")

These are set near the top of the relevant cell in each notebook. (If your identifiers
differ slightly, adjust the names accordingly.)

| Notebook | Variable | Default | Meaning |
|----------|----------|---------|---------|
| `main.ipynb` | `k1` | 1.5 | BM25 term-frequency saturation |
| `main.ipynb` | `b` | 0.75 | BM25 document-length normalization |
| `main.ipynb` | `alpha` | 1.0 | Rocchio weight on the original query |
| `main.ipynb` | `beta` | 0.75 | Rocchio weight on relevant-document terms (top 12 expansion terms) |
| `crawler.ipynb` | `start_category` | `Category:Computer_science` | Root category for the crawl |
| `crawler.ipynb` | `target` | 2000 | Approximate number of articles to collect |
| `eval.ipynb` | `CISI_DIR` | `cisi` | Folder containing `CISI.ALL`, `CISI.QRY`, `CISI.REL` |

---

## 6. Getting the CISI dataset (for `eval.ipynb`)

CISI is a standard, publicly available IR test collection. It is not bundled with the code.

1. Download it from the University of Glasgow IR test collections page:
   **https://ir.dcs.gla.ac.uk/resources/test_collections/cisi/**
2. Extract the archive (`cisi.tar.gz`).
3. Place the three files **`CISI.ALL`**, **`CISI.QRY`**, and **`CISI.REL`** inside a folder
   named **`cisi/`** next to `eval.ipynb` (or set `CISI_DIR` to wherever they are).

---

## 7. Libraries and tools used

| Library / tool | Used for |
|----------------|----------|
| **Wikipedia-API** (`import wikipediaapi`) | Fetching article text and category members through the official MediaWiki API. |
| **NLTK** | English stop-word list and the Porter stemmer. |
| **Flask** | The web search interface (HTML is served with `render_template_string`, so no `templates/` folder is needed). |
| **matplotlib** | The BM25-vs-TF-IDF results chart in `eval.ipynb`. |
| **Python standard library** | `json` (read/write the index), `math` (scoring), `re` (tokenization), `collections.deque` (the breadth-first crawl queue). |

The inverted index, BM25, TF-IDF cosine similarity, and the Rocchio update are all written
by the team. No external IR/search engine (e.g., Lucene, Elasticsearch) and no
machine-learning ranking library (e.g., scikit-learn) is used for retrieval.

---

## 8. Data and results (for reference)

- **Corpus:** 2,050 articles, 42,445 unique stemmed terms, ~644 tokens per document on average, 0 empty documents.
- **CISI evaluation** (76 judged queries):

| Model | MAP | MRR | P@10 |
|-------|-----|-----|------|
| **BM25** | **0.232** | **0.653** | **0.370** |
| TF-IDF (cosine) | 0.192 | 0.560 | 0.292 |

BM25 outperforms TF-IDF on all three metrics.

---

## 9. Troubleshooting

- **`No module named wikipediaapi`** — the *import* name is `wikipediaapi`; install the
  package with `pip install Wikipedia-API`.
- **NLTK `LookupError` for stopwords** — run `nltk.download('stopwords')` once.
- **`main.ipynb` last cell never finishes** — that is expected. `app.run()` is a web
  server; use the browser at http://127.0.0.1:5000, and interrupt the kernel to stop it.
- **Port 5000 already in use** — change the port in the `app.run(...)` call, e.g.
  `app.run(port=5001)`, then open http://127.0.0.1:5001.
- **`eval.ipynb` can't find the data** — make sure `CISI.ALL`, `CISI.QRY`, `CISI.REL` are
  in the folder named by `CISI_DIR` (default `cisi/`).
