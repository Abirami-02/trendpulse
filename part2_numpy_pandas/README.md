# TrendPulse — Part 2: NumPy & Pandas Data Wrangling

Computes a transparent **engagement score** with NumPy, loads and cleans the trending data with Pandas, and answers questions about *what trends and for how long*.

**Self-contained and offline.** Everything runs from this folder's own `data/` directory. No network access, and no file from Part 1 is read. Part 1's cleaning *decisions* (duplicate key, date format, blanks kept as missing) are carried over and re-applied with Pandas.

## Folder contents

```
part2_numpy_pandas/
├── part2_numpy_pandas.ipynb   # the full notebook (Sections 0-10)
├── README.md
├── requirements.txt
└── data/
    ├── USvideos.csv  GBvideos.csv  INvideos.csv
    └── categories.csv
```

## Install and run

```bash
cd part2_numpy_pandas
python -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab part2_numpy_pandas.ipynb
```

Then choose **Run → Run All Cells** (or *Restart Kernel and Run All*). Run it from inside `part2_numpy_pandas/` so that `data/` is found.

## Where each task lives

| Brief task | Notebook section |
|---|---|
| 1. NumPy arrays | 1 |
| 2. Engagement score without loops | 2 |
| 3. Reshape and broadcast | 3 |
| 4. Loading and inspection | 0 (loading) and 4 |
| 5. Filtering | 5 |
| 6. Missing values and duplicates | 6 |
| 7. Joining category names | 7 |
| 8. Aggregation | 8 |
| 9. Time to trend and lifespan | 9 |
| 10. Combining data (`concat`, `merge`) | 0 (`concat`), confirmed in 10.1, and 10.2–10.4 (`merge`) |

## Engagement-score weights

`engagement_score = 0.2 × views_norm + 0.3 × likes_norm + 0.5 × comments_norm`, where each `_norm` is min-max scaled to 0–1 and the weights sum to 1.

| Signal | Weight | Reasoning |
|---|---:|---|
| Views | 0.2 | Weakest evidence. A view can be one tap or an autoplay and costs the viewer nothing; it still shows reach. |
| Likes | 0.3 | A deliberate click, so more telling than a view, but still low effort. |
| Comments | 0.5 | Writing a comment takes the most effort, so it is the strongest sign people reacted. |

Limitations: min-max scaling of raw counts is dominated by the few largest videos, so most scores are tiny and the score is best used to **rank**; videos with comments disabled cannot earn comment points; the weights are a judgement call.

## Data-quality issues and handling

The notebook computes these counts live in Section 4.4. The figures below are for the full dataset.

| Issue | Figure (full dataset) | Handling |
|---|---|---|
| Duplicate rows (same `country` + `video_id` + `trending_date`) | 230 (US 80, GB 75, IN 75): 46,230 → 46,000 rows | Dropped, keeping the first copy |
| Missing `description` | 1,615 (3.51% of 46,000) | Kept rows; `fillna("")` |
| Missing `likes` | 12 (0.03%) | Kept as missing, not replaced by 0 |
| Missing `comment_count` | 36 (0.08%) | Kept as missing, not replaced by 0 |
| Removed / errored videos | 92 | Excluded from the analysis table (45,908 rows). Set `EXCLUDE_REMOVED = False` in the notebook to keep them |
| Like ratio undefined (zero votes or missing likes) | 551 | Left as `NaN` by safe division |
| Negative counts | 0 | None needed |
| Dates stored as text (`yy.dd.mm` and ISO) | all rows | Converted to datetimes |
| `likes` / `comment_count` stored as float | all rows | Converted to nullable `Int64` |

## Missing-value and duplicate decisions

- **Missing values** are never filled with 0: a blank is unknown, not zero. `likes` and `comment_count` stay `NaN`, so those rows get no engagement score and cannot enter the top 10% by accident. Missing `description` becomes an empty string because it only means the uploader wrote none. `dropna()` is shown on a copy to report its cost but is not applied.
- **Duplicate definition:** the same `video_id` on the same `trending_date` in the same `country`. A trending list shows a video once per country per day, and one video legitimately trends on many days and in many countries, so `video_id` alone would be wrong. The first copy is kept.

## Notes on the analysis

- Each row is one video on one trending day, so popular videos appear on many rows and carry extra weight in averages.
- Publishing day and hour use UTC. The notebook shows medians next to means because a few very popular videos can lift a group's average.
- Per-channel totals use each video's **peak** views, because `views` is cumulative and summing daily rows would count the same views many times.

## Reproducing the results

The notebook is saved with its outputs. To reproduce them, follow the install and run steps above: it runs fully offline from the files in `data/` and recomputes every number, table and score from scratch. Run it from inside `part2_numpy_pandas/` so the relative `data/` path resolves.
