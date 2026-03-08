# Narrative_Pattern_Evaluation
Evaluating Narrative Genre Patterns in Award-Winning Novellas using LLMs

This repository contains the data, code, and results for the paper:

> *"Evaluating Narrative Genre Patterns in Award-Winning Novellas"*  
> [Milka Kaplan, Armin Shmilovici, Mark Last], Text2Story 2026 Workshop  
> [Link to paper when available]

---

## Overview

We evaluate whether five narrative genre patterns from PatternTeller (Lima et al., 2025) — Comedy, Mystery, Romance, Satire, and Tragedy — are prevalent in award-winning novellas. A human expert and three LLMs (Claude, Gemini, DeepSeek) annotated 35 novellas using both direct genre knowledge and PatternTeller's structural framework.

**Key findings:**
- Direct Human–LLM agreement is moderate (F1 = 0.74)
- Agreement drops substantially when mediated by PatternTeller (F1 = 0.47–0.54)
- Inter-LLM agreement under PatternTeller is similarly low (F1 = 0.57–0.61)
- PatternTeller produces near-uniform genre distributions (N_eff = 4.72), failing to discriminate between genres
- Most structurally central elements across all genres: *Closure*, *Final Confrontation*, *Return*

---

## Repository Structure

```
narrative-pattern-evaluation/
├── README.md
├── requirements.txt
├── data/
│   ├── corpus_35_metadata.csv          # Title, author, year, word count, avg. raiting (GoodReads), score
│   ├── texts/                       # Cleaned texts (public domain only)
│   │   ├── 001_animal-farm.txt
│   │   ├── 054_doyle-hound-383.txt
│   │   └── ...
│   ├── annotations/
│   │   ├── human_general.csv        # Human scores (general genre knowledge)
│   │   ├── human_patternteller.csv  # Human scores (PatternTeller framework)
│   │   ├── llm_claude.csv           # Claude PatternTeller scores
│   │   ├── llm_gemini.csv           # Gemini PatternTeller scores
│   │   └── llm_deepseek.csv         # DeepSeek PatternTeller scores
│   └── raw_llm_outputs/
│       └── animal_farm_tragedy.txt  # Example raw LLM output
├── prompts/
│   ├── prompt_template.txt          # Base prompt template
│   └── run_annotation.py            # API call script
├── analysis/
│   ├── binomial_test.xlsx           # Binomial test (H0/H1)
│   ├── hhi_analysis.xlsx            # HHI and Effective N
│   ├── classification_metrics.xlsx  # Precision, Recall, F1 (Tables 1–3)
│   └── multitask_lasso.py           # MultiTaskLasso + stability selection
└── results/
    ├── table1_human_llm.csv
    ├── table2_human_pt_llm.csv
    ├── table3_inter_llm.csv
    ├── table4_mtlasso_concepts.csv
    └── table5_stability.csv
```

---

## Data

### Corpus
35 award-winning English novellas (7,500–102,810 words) from 5 genres (7 per genre): Comedy, Mystery, Romance, Satire, Tragedy. Selected from the [Goodreads Best Novellas list](https://www.goodreads.com/list/show/22695).

Full metadata available in `data/corpus_metadata.csv`.

> **Copyright note:** Only public domain texts (published before 1928) are included in full. For all other texts, only metadata is provided. The full list of texts used is in `corpus_metadata.csv`.

### Annotation Format
Each annotation file is a CSV with columns:

```
title, comedy, mystery, romance, satire, tragedy
```

Scores are on a 0–4 scale:
- 0 — no correspondence
- 1 — weak correspondence  
- 2 — moderate correspondence
- 3 — strong correspondence
- 4 — very strong correspondence

---

## Prompts

The LLM annotation used a bottom-up aggregation approach: each structural element was scored first (0/1/2), then aggregated into a genre coverage score.

See `prompts/prompt_template.txt` for the exact prompt used with all three LLMs.

---

## Reproducing Results

### Requirements

```bash
pip install -r requirements.txt
```

### Step 1 — Run annotation (requires API keys)

```bash
python prompts/run_annotation.py \
    --model claude-haiku \
    --input data/texts/ \
    --output data/annotations/llm_claude.csv
```

### Steps 2–4 - Classification metrics, Binomial test, HHI
These analyses were performed in Microsoft Excel.  
Annotated spreadsheets with all formulas are available in `analysis/`.

### Step 5 — MultiTaskLasso + stability selection (Table 4–5)

```bash
python analysis/multitask_lasso.py \
    --input data/annotations/llm_claude.csv \
    --n_runs 200 \
    --sample_frac 0.75
```

---

## Key Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| Threshold | ≥ 3 | Minimum score for positive genre label |
| Top-k genres | 2 | Number of genres per novella for binomial test |
| MTL n_runs | 200 | Stability selection subsampling runs |
| MTL sample_frac | 0.75 | Fraction of data per subsample |
| MTL alpha | CV-selected | Chosen via MultiTaskLassoCV (5-fold) |

---

## Citation

```bibtex
@inproceedings{kaplan2026narrative,
  title     = {Evaluating Narrative Genre Patterns in Award-Winning Novellas},
  author    = {Kaplan, Milka and Shmilovici, Armin and Last, Mark},
  booktitle = {Proceedings of the Text2Story 2026 Workshop},
  year      = {2026}
}
```

---

## License

Code: MIT License  
Data annotations: CC BY 4.0  
Texts: Public domain texts only (see `data/texts/`)

---

## Acknowledgments

This research was partially supported by the Data Science Research Center, Ben-Gurion University of the Negev, Ministry of Aliyah and Integration, Israel.
