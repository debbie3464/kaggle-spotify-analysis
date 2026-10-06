# Learning log

A running record of how I approached this analysis: what I did, what I noticed, and what I learned at each stage.

## The EDA pipeline I'm following

| Stage | Status |
|---|---|
| 1. Ask questions | ✅ Done |
| 2. Inspect the data | ✅ Done |
| 3. Clean | ✅ Done |
| 4. Explore (EDA) | ⏳ Next |
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
- **19 tracks appear twice.** Probably the same songs in two different years (confirmed in Stage 3). That's fine for a trend question, but it would double-count them in track-level questions.
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

### Remaining checks (finished during cleaning)
- `describe()`: all audio features fall in 0-1, tempo is 65-202 BPM, loudness is always negative (-15.2 to -1.2 dB). Nothing impossible.
- `key`: only values 0-11, no `-1` (undetected) values.
- Zeros: 3 tracks have popularity 0. No zeros in followers, artist popularity, or tempo.
- `followers` is heavily right-skewed (mean about 24M, median about 12.5M).
- `instrumentalness` is almost always 0, so I won't use it.

### What I learned
- Inspect before you clean: each surprise above turned directly into a cleaning step.
- `info()` shows both types and non-null counts in one view.
- "No missing values" doesn't mean "no missing information".
- Reading outputs matters as much as running the code, so I wrote a note after each result.

---

## Stage 3: Cleaning (Oct 6, 2026)

**Goal:** make sure the numbers I chart can be trusted, and record every decision. I worked on copies so the raw `df` stayed untouched.

### Problem 1: songs listed more than once
- **Noticed:** matching on `track_uri` found 19 repeated songs, all identical except `year`. Matching on song name + artist found **27 songs (54 rows)**, so some songs have different IDs in different years.
- **Example:** *Blinding Lights* has popularity 18 under one ID (2020) and 90 under another (2021). My best guess is that Spotify has several IDs for one song and plays pile up on one of them. I can't confirm this from the data.
- **Did:** built two tables.
  - `clean`: all 500 rows, for the year-trend question (Q2), where a song that stays on the list should count in each year.
  - `tracks`: one row per song (473), keeping the copy with the highest popularity, for all other questions.
- **Why:** counting a song twice would bias averages and correlations.
- **Learned:** `year` is the year of the list, not the release year. `track_popularity` is a snapshot, so it can't compare years.

### Problem 2: genres stored as text
- **Noticed:** `artist_genre` looked like a list but was a string, with 200 unique combinations and 16 empty `[]` values.
- **Did:** parsed the text into real lists with `ast.literal_eval`, then grouped artists into broad families with keyword rules (first matching rule wins, so "pop" is checked last). Empty lists became "Unknown".
- **Checked by reading the artist lists, which caught a bug:** George Ezra and Vance Joy landed in K-pop. Their tag "folk-pop" contains "k-pop" inside the word. Fixed with a word-boundary pattern (`\bk-pop`). I also added "drill" so Pop Smoke and Central Cee moved from "Other" to Hip-hop/Rap.
- **Simplifications I accepted:** each artist gets only one family, so some are debatable (Halsey in Rock/Indie/Alt, Adele in R&B/Soul, ROSALÍA in R&B/Soul). Country (4 songs) was merged into "Other".

### Problem 3: odd rows that turned out to be real
- **Noticed:** 25 well-known songs have popularity under 40 (*Dynamite*, *positions*, *drivers license*). 28 songs by tiny artists appear almost entirely in 2022 (27 in 2022, 1 in 2021). The median followers in 2022 is about 2.5M, versus 12-23M in other years.
- **My first guess was wrong:** I expected popularity to rise with year (a recency effect). It didn't (yearly means: 74.7, 75.8, 65.1, 66.6, 75.5). A split-ID explanation fits better for some of these songs, but not all of them.
- **Did:** flagged instead of deleting (`low_pop` for popularity under 40, `niche` for under 100k followers). I'll test whether Q1 and Q4 results change with and without them.
- **Why:** "unusual" isn't "wrong". Deleting everything under 40 would have removed real hits.

### Other steps
- Stripped extra spaces from track and artist names.
- Added `log_followers` (log10) because followers are heavily skewed.
- Ran assertions (row counts, no duplicate songs, popularity within 0-100, tempo above 0, genre families filled), and all passed.

### What I learned
- Cleaning is about fixing only what would make my answers wrong, and writing down why.
- `isna()` doesn't catch placeholders like `[]`, and checking duplicates by ID alone missed repeats.
- Keyword matching can match inside longer words, so I always check results by eye.
- Group sizes matter: with under about 20 songs, one track can swing an average, so small groups should show their n.

### Limitations noted so far
- Only hits, from five yearly lists, so this isn't "music in general".
- 2022's list looks different (many small artists), so 2021 → 2022 changes may reflect list composition.
- Popularity is a snapshot and can't compare years.
- Audio features are Spotify's algorithmic scores.
- One genre family per artist is a simplification.

---

## Mistakes and fixes
- Colab couldn't read my `C:\` drive path, because it runs on a cloud server and can't see my laptop → uploaded the CSV to Colab instead.
- Backslashes in a Windows path (`"C:\Users\..."`) cause a `unicodeescape` error in Python → use forward slashes or a raw string `r"..."`.
- My upload landed in the server's root folder rather than `/content` → used the root path, then moved the file.
- Colab session reset → `df` not defined (`NameError`), and the uploaded CSV was gone → re-uploaded and re-ran setup. Notebooks should run cleanly from top to bottom ("Restart session and run all").
- Assumed low-popularity rows were data errors, but they were real hits → investigated before deleting anything.
- Keyword "k-pop" matched inside "folk-pop" → used a word-boundary pattern.

## What I want to learn next
- How `groupby` works (split → calculate → combine)
- Why log-scaling a skewed column like `followers` helps
- How to read residuals from a simple regression (for the breakout-hits question)
- How to read a correlation heatmap
- How to choose a chart type from the question

## What I'd do next
- Start Stage 4: explore the data (distributions, correlations, genre group sizes)
- Then answer each question in order: Q5, Q2, Q3, Q1, Q4
