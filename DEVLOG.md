# Learning log

### The EDA pipeline I followed
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
- **19 tracks appear twice.** Probably the same songs in two different years (still to confirm). That's fine for a trend question, but it would double-count them in track-level questions, so I'll keep all 500 rows for the year trend and use a de-duplicated table for the rest.
- **`isna()` said 0 missing, but 16 artists have no genre.** An empty `[]` is a real string, so pandas doesn't treat it as missing. Lesson: a missing-value check only catches *empty cells*, not placeholders that look like data.
- **The genres are strings, not lists.** `"['pop', 'dance pop']"` has to be parsed into a real list before I can group by it.
- **The audio scores seem believable.** "Nonstop" by Drake has high danceability (0.912) but moderate energy (0.412), so the two columns clearly measure different things. "drivers license" and "golden hour" both have low valence, which fits how they sound. These are still Spotify's algorithmic scores, not direct measurements.
- **The year column is perfectly balanced** (100 per year), which suggests a top-100-per-year list. That means this is a dataset of *hits*, not music in general, so the limitations section should say so.

### Decisions for the cleaning stage
1. Work on a copy (`clean = df.copy()`) so the raw data stays untouched.
2. Keep all 500 rows for the year-trend question and use a de-duplicated table for the other questions.
3. Parse genres into real lists, labeling empty ones "Unknown".
4. Group the 200 genre labels into about 8 broader families, with a note that each artist gets one primary family.
5. Check value ranges and skew with `describe()`, which I haven't done yet.

### Things I learned
- Inspect before you clean: each surprise above turned directly into a cleaning step.
- `info()` shows both types and non-null counts in one view.
- "No missing values" doesn't mean "no missing information".
- Reading outputs matters as much as running the code, so I wrote a note after each result.

### Still to check
- `describe()` for impossible values and for how skewed `followers` is
- The distribution of `key`
- Which tracks are repeated and in which years
## Decisions and why
- Dropped duplicate tracks because ...
- Grouped genres into families because ...

## Things I learned
- How groupby works (split → calculate → combine)
- Why log-scaling followers matters
- Spotify's audio features are algorithmic scores, not measurements

## Mistakes and fixes
- Colab couldn't read my C: drive path → uploaded the file instead
- Backslashes in file paths cause unicode errors

## What I'd do next
- ...
