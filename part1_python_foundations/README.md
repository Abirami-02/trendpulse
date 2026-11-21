# TrendPulse — Part 1: Python Foundations & Data Structures

Part 1 of the **TrendPulse** mini project ("What's actually trending right now?"). It loads daily YouTube *Trending* lists for three countries, cleans them with **core Python**, organises them with the right data structures, and labels every video with **transparent rule-based logic** (no machine learning).

The only Pandas calls are `pd.read_csv()` and `df.to_dict("records")`. Everything after loading is plain Python: loops, functions, conditions, lists, tuples, sets and dictionaries.

## Folder contents

```
part1_python_foundations/
├── TrendPulse_Part1.ipynb   # the notebook (run top to bottom)
├── README.md                # this file
├── requirements.txt         # pandas + jupyter
├── .gitignore
└── data/
    ├── USvideos.csv         # 16,080 rows
    ├── GBvideos.csv         # 15,075 rows
    ├── INvideos.csv         # 15,075 rows
    └── categories.csv       # category_id -> category_name (17 categories)
```

## Data source

Public **Trending YouTube Video Statistics** dataset (daily snapshots of YouTube's Trending list, one CSV per country plus a category lookup), available on Kaggle: <https://www.kaggle.com/datasets/datasnaek/youtube-new>. The CSV files used here were supplied by the program team on the LMS and are included unmodified in `data/`.

Countries used: **US, GB, IN** (46,230 rows loaded in total, above the 40,000 minimum).

> Note: this copy of the data has anonymised-looking channel names (e.g. `QuietChallenge75`), so channel-level results describe patterns in the data, not real creators.

## Install and run

**Locally (Jupyter)**
```bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPO.git
cd YOUR-REPO/part1_python_foundations
pip install -r requirements.txt
jupyter notebook TrendPulse_Part1.ipynb
```
Then choose **Kernel → Restart & Run All**. The notebook reads from the relative `data/` folder, so start Jupyter from inside `part1_python_foundations/`.

**Google Colab**
1. Open `TrendPulse_Part1.ipynb` in Colab (File → Open notebook → GitHub tab).
2. In the first code cell, uncomment the two lines that `git clone` the repo and `%cd` into `part1_python_foundations`.
3. Choose **Runtime → Restart and run all**.

## Cleaning rules

| Field(s) | Rule |
|---|---|
| `views`, `likes`, `dislikes`, `comment_count` | Converted to `int`. Blank, negative or non-whole values become `None` and are counted (blank vs bad format reported per field). Never replaced with `0`, because that would invent data. |
| `comments_disabled`, `ratings_disabled`, `video_error_or_removed` | Converted to `bool`; anything unrecognised becomes `None`. |
| `trending_date` | Parsed with `%y.%d.%m` (this copy stores `17.14.11` = 14 Nov 2017) into a `date`. |
| `publish_time` | ISO timestamp (`2017-11-14T07:04:28.000Z`) parsed into a timezone-aware `datetime`. |
| Failed conversions | Wrapped in `try/except`; the record is **kept** with `None` so good fields are not lost. |
| Exact duplicates | Rows with the same `(country, video_id, trending_date)` are **dropped** (230 rows: US 80, GB 75, IN 75). |

Result: 46,230 rows loaded → **46,000 records kept** (US 16,000 · GB 15,000 · IN 15,000). Only `likes` (12) and `comment_count` (36) had blanks; there were no bad-format values.

## Classification thresholds (`classify_video`)

Like ratio = `likes / (likes + dislikes)`. Thresholds were chosen from the real distribution (median like ratio ≈ 0.95).

| Setting | Value |
|---|---|
| `VIRAL_VIEWS` | 1,000,000 |
| `STEADY_VIEWS` | 100,000 |
| `HEALTHY_RATIO` | 0.90 |
| `DIVISIVE_RATIO` | 0.85 |

Rules, checked in order:
1. **Unrated**: ratings or comments disabled, video removed/errored, any needed value missing, or zero likes **and** zero dislikes (avoids division by zero).
2. **Low Engagement**: zero views, or under 100,000 views with an acceptable ratio.
3. **Divisive**: like ratio below 0.85.
4. Otherwise by views: 1M+ views and ratio ≥ 0.90 → **Viral Hit**; 1M+ views with ratio below 0.90, or 100k–999,999 views → **Steady Performer**; under 100k → **Low Engagement**.

| Label | Records | Share |
|---|---:|---:|
| Viral Hit | 8,586 | 18.7% |
| Steady Performer | 25,316 | 55.0% |
| Divisive | 2,781 | 6.0% |
| Low Engagement | 7,649 | 16.6% |
| Unrated | 1,668 | 3.6% |

These are analytical categories defined by this project, not YouTube classifications. The notebook includes a hand-traced example and edge-case tests.

## What the notebook covers

Setup and config · reusable `load_country()` and `category_names` · cleaning with failure counts · list of dicts, tuples, sets (intersection/difference), dictionary counter, slicing, nested dictionary · refactor (`unique_ids`) · variable scope demo · `title_features` with `for`, `range()`, `while`, `break`, `continue` · rule-based classification with label counts.
