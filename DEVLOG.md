# Learning log

A running record of how I approached this analysis: what I did, what I noticed, and what I learned at each stage.

## The EDA pipeline I'm following

| Stage | Status |
|---|---|
| 1. Ask questions | ✅ Done |
| 2. Inspect the data | ✅ Done (a few checks left) |
| 3. Clean | ⏳ Next |
| 4. Explore (EDA) | ⬜ To do |
| 5. Analyze | ⬜ To do |
| 6. Visualize | ⬜ To do |
| 7. Conclude and publish | ⬜ To do |

---

## Stage 1: Asking questions

**Goal:** decide what I want to find out before touching the data.

My five questions:
1. Is it who you are or what you sound like? Does artist fame (followers) predict a hit's popularity better than audio features do?
2. How have hit songs changed from 2018 to 2022?
3. What does each genre sound like, and which perform best?
4. Which songs are breakout hits, far more popular than their artist's following would predict?
5. Do hits share a tempo? Is there a "sweet spot" BPM? (bonus)

**Decision:** I leaned toward questions built on concrete columns (followers, popularity, year, tempo, loudness, genre). Spotify's audio scores such as danceability and valence are algorithmic estimates, so I use them with care.

---

## Stage 2: Inspecting the data (Oct 4, 2026)

**Goal:** understand what's in the dataset before changing anything.

### What I did
- Loaded the CSV into a DataFrame and checked its shape and first rows (`shape`, `head`)
- Checked column types and non-null counts (`info`)
- Counted missing values (`isna().sum()`)
- Checked for duplicates, both fully identical rows and repeated `track_uri` values
- Counted tracks per year (`value_counts` on `year`)
- Looked at the `artist_genre` column
- Sanity-checked the audio features on a random sample of songs I know

### What I found
| Check | Result |
|---|---|
| Size | 500 rows × 19 columns, one row per track |
| Data types | Numbers are int/float, text columns are objects, nothing needs converting |
| Missing values | 0 |
| Duplicate rows | 0 fully identical rows, but only **481 unique tracks** out of 500 |
| Years | Exactly 100 tracks for each year from 2018 to 2022 |
| Genres | 200 unique values, stored as text that looks like a list, and 16 rows are an empty `[]` |

### What I noticed
- **19 tracks appear twice.** Probably the same songs in two different years (still to confirm). That's fine for a trend question, but it would double-count them in track-level questions.
- **`isna()` said 0 missing, but 16 artists have no genre.** An empty `[]` is a real string, so pandas doesn't treat it as missing.
- **The genres are strings, not lists.** `"['pop', 'dance pop']"` has to be parsed into a real list before I can group by it.
- **The audio scores seem believable.** "Nonstop" by Drake has high danceability (0.912) but moderate energy (0.412), so the two columns clearly measure different things. "drivers license" and "golden hour" both have low valence, which fits how they sound. They're still Spotify's algorithmic scores, not direct measurements.
- **The year column is perfectly balanced** (100 per year), which suggests a top-100-per-year list. So this is a dataset of *hits*, not music in general, which belongs in the limitations section.

### Decisions for the cleaning stage
1. Work on a copy (`clean = df.copy()`) so the raw data stays untouched.
2. Keep all 500 rows for the year-trend question and use a de-duplicated table for the other questions.
3. Parse genres into real lists, labeling empty ones "Unknown".
4. Group the 200 genre labels into about 8 broader families. Each artist gets one primary family, which is a simplification I'll note in the limitations.
5. Check value ranges and skew with `describe()`.

### What I learned
- Inspect before you clean: each surprise above turned directly into a cleaning step.
- `info()` shows both types and non-null counts in one view.
- "No missing values" doesn't mean "no missing information".
- Reading outputs matters as much as running the code, so I wrote a note after each result.

### Still to check
- [ ] `describe()` for impossible values and for how skewed `followers` is
- [ ] The distribution of `key`
- [ ] Which tracks are repeated and in which years

---

## Mistakes and fixes
- Colab couldn't read my `C:\` drive path, because it runs on a cloud server and can't see my laptop → uploaded the CSV to Colab instead.
- Backslashes in a Windows path (`"C:\Users\..."`) cause a `unicodeescape` error in Python → use forward slashes or a raw string `r"..."`.
- My upload landed in the server's root folder rather than `/content` → used the root path, then moved the file.

## What I want to learn next
- How `groupby` works (split → calculate → combine)
- Why log-scaling a skewed column like `followers` helps
- How to read residuals from a simple regression (for the breakout-hits question)

## What I'd do next
- Finish the remaining inspection checks
- Start Stage 3: clean the data
