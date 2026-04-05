```markdown
# US 2020 Election Twitter Analysis

This project analyzes tweets related to the 2020 US presidential election in order to study:

- Cleaning of Trump and Biden datasets  
- Sentiment analysis  
- User analysis (activity / influence)  
- Trump vs Biden comparison (Which share of tweets is negative? Visual difference in emotional intensity?)  
- Geographical analysis (Which candidate is most favored in which area?)  
- Exported figures and tables
in order to identify opinion trends and influence on Twitter.
---

##  Project Overview

The goal is to start from raw CSV files of tweets and build a complete pipeline:

- Loading and cleaning tweets  
- Feature extraction (sentiment, user information, geography, etc.)  
- Exploratory analysis and visualizations  
- Notebooks to document each step of the study

###  Database

The Kaggle dataset provided the data required for our project to function properly (https://www.kaggle.com/datasets/manchunhui/us-election-2020-tweets).

###  Global Project Pipeline

The general pipeline is:

1. **Raw data**: CSV files containing tweets (text, date, user, metadata)  
2. **Pre‑processing**: text cleaning, normalization, retweet removal  
3. **Enrichment**: sentiment computation, derived variables (length, hashtags, etc.)  
4. **Analyses**: Trump/Biden comparison, sentiment over time, user analysis  
5. **Visualizations & reporting**: charts and tables to interpret the results

> In the final version, insert here a diagram illustrating this pipeline, for example `docs/img/pipeline.png`.  
> We suggest creating a `docs/img/` folder to store images for each feature.

```text
Raw CSV → Cleaning & enrichment → Analysis modules (sentiment, comparison, users, geography) → Visualizations → Report
```

###  Main Features

- **Tweet loading & cleaning**  

  This part corresponds to the pre‑processing of all textual data.  
  The notebook `01_load_clean_separate.ipynb` reads the raw CSV files (Trump and Biden tweets) and, for each tweet, applies a series of cleaning operations defined in the module `src/utils_text.py`.  
  This module groups all low-level functions: URL, mention and hashtag removal, lowercasing, removal of punctuation and emojis, whitespace normalization, etc.  
  The notebook acts as an **orchestrator** (it loads data, calls functions from `utils_text.py`, and saves the result in `data/interim/`), while `utils_text.py` acts as a **reusable text toolbox**.

  #### Example snippet for clarity

  ```python
  import pandas as pd
  from src.utils_text import clean_tweet

  # Load raw data
  df = pd.read_csv("data/raw/trump_tweets.csv")

  # Apply cleaning function defined in utils_text.py
  df["clean_text"] = df["text"].apply(clean_tweet)

  # Save cleaned data
  df.to_csv("data/processed/trump_tweets_clean.csv", index=False)
  ```

- **Trump vs Biden comparison**  

  Basic statistics (number of tweets per candidate, frequent hashtags, distributions, etc.) and visual comparison of sentiment and emotional intensity between the two candidates.

- **Sentiment analysis**  

  Computation of tweet sentiment and aggregation by candidate and time period.

- **User analysis**  

  Statistics on the most active accounts, retweets, location, etc.

- **Geographical analysis**  

  Study of where each candidate is more favored based on tweet content and sentiment.

---

##  UML Diagram

> UML diagram of the main classes / modules (`SentimentAnalyzer`, text utilities, user analysis, etc.).

```text
                         +------------------------------+
                         |          DataLoader          |
                         +------------------------------+
                         | + read_tweets_csv(path)      |
                         | + clean_single_candidate(...)|
                         +-----------+------------------+
                                     |
                                     | returns DataFrames
                                     v
+----------------------+     +-------------------------+     +----------------------+
|      utils_text      |     |    SentimentAnalyzer    |     |     UserAnalyzer     |
+----------------------+     +-------------------------+     +----------------------+
| + fix_mojibake(s)    |     | - model                |     |                      |
| + clean_text(t)      |     +-------------------------+     +----------------------+
| + is_retweet_from_   |     | + load_model()         |     | + top_active_users()  |
|   text(t)            |     | + predict(text)        |     | + users_by_candidate()|
+----------+-----------+     | + predict_many(df)     |     | + user_stats(df)      |
           ^                 +-----------+-------------+     +-----------+----------+
           |                   ^         ^                            |
           | used by           |         | uses                       | uses
           |-------------------+         |                            v
           |                 +-------------------------+   +-------------------------+
           |                 |      GeoAnalyzer        |   |     Notebooks 01–05     |
           |                 +-------------------------+   +-------------------------+
           |                 | + favorite_by_region()  |   | 01_load_clean_...       |
           |                 | + build_geo_map()       |   | 02_compare.ipynb        |
           |                 +-------------------------+   | 03_sentiment.ipynb      |
           |                                               | 04_users.ipynb          |
           |                                               | 05_Geographic.ipynb     |
           |                                               |                         |
           +---------------------------------------------->+-------------------------+
```

This UML diagram summarizes the logical architecture of the project.  
`DataLoader` loads raw and cleaned data as pandas DataFrames.  
The `utils_text` module provides low-level tweet cleaning functions (`fix_mojibake`, `clean_text`, retweet detection).  
`SentimentAnalyzer` encapsulates the sentiment model and exposes methods to predict sentiment for a single tweet or a whole dataset.  
`UserAnalyzer` groups functions for user‑level analysis (top accounts, per‑candidate stats, etc.).  
`GeoAnalyzer` handles the geographic part: favorite candidate by region and map/barplot creation.  
Finally, notebooks 01–05 orchestrate the whole pipeline: they call functions from these modules, produce visualizations, and serve as support for exploratory analysis and presentation.

The link between sentiment and geography is as follows:

- `SentimentAnalyzer` computes a sentiment score per tweet.  
- The resulting DataFrame (with columns like `sentiment`, `target`, `user_location`, `lat`, `long`, etc.) is passed to `GeoAnalyzer`.  
- `GeoAnalyzer` aggregates by region / country and builds maps or barplots.

The Trump vs Biden comparison feature uses the sentiment scores and user information to compare both candidates.

---

## 🛠 Installation Tutorial

This section explains how to install and quickly test the project.

### 1. Clone the repository

```bash
git clone https://github.com/<marie-account>/Analyse-de-tweet-US-election-2020-.git
cd Analyse-de-tweet-US-election-2020-
```

### 2. Create a virtual environment

**Using venv (standard)**:

```bash
python -m venv .venv
```

After creating the venv, install dependencies (if needed):

```bash
pip install pandas pyarrow numpy matplotlib seaborn vaderSentiment wordcloud spacy gensim
python -m spacy download en_core_web_sm  # download English language model for spaCy
```

Activate the environment:

```bash
# Windows
.\.venv\Scripts\activate

# Linux / macOS
source .venv/bin/activate
```

### 3. Install dependencies from `requirements.txt`

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Minimal installation test

Create a `quick_test.py` file:

```python
from src.utils_text import clean_tweet
from src.sentiment import SentimentAnalyzer

raw = "I love this candidate!!! https://example.com #election2020"
print("Raw tweet:", raw)

cleaned = clean_tweet(raw)
print("Cleaned tweet:", cleaned)

analyzer = SentimentAnalyzer()
score = analyzer.predict("I love this candidate!")
print("Sentiment score:", score)
```

Then run:

```bash
python quick_test.py
```

If everything is correctly installed, you should see:

- the raw tweet  
- the cleaned tweet  
- a numeric sentiment score

###  Tested platforms

- Windows 11 / Python 3.11 (venv)

> Add other systems here if you test them (Ubuntu, macOS, etc.).

---

#  Features in Detail and Examples

## 1. Tweet Loading & Cleaning

**Summary**

- Reads raw CSV files from `data/raw/`.  
- Cleans text: lowercasing, removal of URLs, mentions, hashtags, emojis, punctuation, stopwords, extra spaces, etc.  
- Saves cleaned data into `data/interim/`.  

The notebook `01_load_clean_separate.ipynb` implements a candidate‑wise loading/cleaning pipeline via the function `clean_single_candidate`.  
This function relies on:

- `read_tweets_csv` (DataLoader) for robust CSV reading  
- `utils_text` for text cleaning operations (`fix_mojibake`, `clean_text`, `is_retweet_from_text`)

The result is a set of clean intermediate files (`trump_all.csv`, `trump_nort.csv`, `biden_all.csv`, `biden_nort.csv`) used later for EDA, sentiment analysis, user analysis, and geographic analysis.

### Data format

Expected raw files in `data/raw/`:

- `trump.csv`  
- `biden.csv`

Minimal columns:  
`created_at,tweet_id,tweet,likes,retweet_count,source,user_id,user_name,user_screen_name,user_description,user_join_date,user_followers_count,user_location,lat,long,city,country,continent,state,state_code,collected_at`

### `utils_text` description

This is the central text‑cleaning module.

Functions:

- `fix_mojibake(s)`  
  Fixes encoding issues (e.g. `Ã©` → `é`) when they appear in tweets.

- `clean_text(t)`  
  Full cleaning pipeline for a tweet:
  - remove URLs (`URL_RE`)  
  - remove mentions (`@user`)  
  - remove `#` but keep the word (`#Trump` → `Trump`)  
  - lowercase  
  - remove emojis and strong punctuation (`EMOJI_PUNCT_RE`)  
  - collapse multiple spaces and trim  

  This function is called in notebooks to create the cleaned text column.

- `is_retweet_from_text(t)`  
  Detects whether a tweet is a retweet by checking if it starts with `"RT @"`.

### `DataLoader` description

The logical `DataLoader` module groups functions that manage tweet CSV files:

- `read_tweets_csv(path)`  
  Reads a raw CSV handling encoding issues, numeric conversions (`likes`, `retweet_count`, etc.), and date parsing (`created_at`, `user_join_date`, `collected_at`).

- `clean_single_candidate(path_in, path_all_out, path_nort_out, target_label)`  
  - reads one candidate’s raw tweet file (Trump or Biden)  
  - ensures presence and order of main columns  
  - fixes encodings and cleans text using `fix_mojibake` and `clean_text`  
  - creates `text_clean` and `is_retweet` columns  
  - filters invalid rows (missing user / location)  
  - produces two cleaned files in `data/interim/`:
    - `*_all.csv` (with retweets)  
    - `*_nort.csv` (without retweets, using `is_retweet`)

**Real code illustration**

```python
# Trump
df_trump_all, df_trump_nort = clean_single_candidate(
    path_in       = RAW_DIR / "hashtag_donaldtrump.csv",
    path_all_out  = INT_DIR / "trump_all.csv",
    path_nort_out = INT_DIR / "trump_nort.csv",
    target_label  = "Trump",
)
```

### Transformations applied during cleaning

- remove URLs `https://t.co/...`  
- remove multiple line breaks / underscores  
- remove emojis  
- remove `#` but keep the word (`#JoeBiden` → `joebiden`)  
- lowercase  
- remove strong punctuation (`: . ! ? , '`) and double spaces  
- drop tweets with missing `user_name` or missing `user_location`

**Command‑line example**

```bash
python -m src.users \
    --input data/raw/trump_tweets.csv \
    --output data/processed/trump_tweets_clean.csv
```

**Python example**

```python
import pandas as pd
from src.utils_text import clean_tweet

df = pd.read_csv("data/raw/trump_tweets.csv")
df["clean_text"] = df["text"].apply(clean_tweet)
print(df[["text", "clean_text"]].head())
```

> (Bonus) Add a screenshot of the DataFrame before/after cleaning, with a short caption.

---

## 2. Geographic Analysis

**Summary**
The geographic analysis aims to understand where tweets mentioning each candidate originate from and to compare the relative presence of Joe Biden and Donald Trump across different geographic areas.

This analysis focuses on Twitter activity, not on voting behavior.
It reflects online political engagement and visibility, highlighting regions where discussions about the 2020 US election were particularly active.

This part of the project was fully redesigned using object-oriented programming, in order to clearly separate:

- data aggregation logic,

- geographic normalization,

- and visualization (charts and maps).

 **Methodology**
The geographic analysis follows a structured pipeline composed of several steps.

# 1. Location extraction*

Tweets include a user_location field provided freely by users.
This field is highly heterogeneous and may contain:

- city names,

- country names,

- abbreviations,

- emojis or informal text.
Because of this variability, raw locations cannot be used directly.

# 2. Location cleaning and normalization*

Locations are cleaned using a rule-based approach:

lowercasing text,

removing special characters and emojis,

trimming extra spaces,

mapping frequent cities to countries
(e.g. “paris” → France, “london” → United Kingdom).

# 3. Aggregation by geographic unit*

After normalization, tweets are aggregated by geographic area:

- country level for global analysis,

- city level for more detailed comparisons.

For each geographic unit, the following metrics are computed:

- number of Biden tweets,

- number of Trump tweets,

- total tweet volume,

- relative dominance score.
  
 # 4. Candidate comparison*
To compare candidates fairly across regions, we compute a relative dominance score:

- dominance = (Biden − Trump) / (Biden + Trump)

This score allows us to visualize: which candidate dominates in each region, and how strong that dominance is.

**Object-Oriented Design**

The geographic logic is encapsulated in dedicated Python classes.

GeographyAnalyzer (src/geography_analysis.py)

This class is responsible for:
- loading cleaned tweet datasets,

- detecting and cleaning location columns,

- normalizing geographic information,

- aggregating tweets by country or city,

- computing dominance metrics between candidates.

Using a dedicated class allows:

- clear separation of concerns,

- reusable geographic logic,

- lighter notebooks focused on exploration rather than computation.
  
# WorldMapBuilder (src/world_map.py)
Visualization is handled separately by the WorldMapBuilder class, which:

- receives aggregated geographic data,

- applies color scales based on dominance scores,

- builds static and interactive world maps,

- exports maps as PNG or HTML files.

This separation avoids mixing analysis logic and rendering logic.

**Associated Notebook**

All geographic experiments and figures are produced in: notebooks/geography_exploration.ipynb
This notebook:
- instantiates geographic analysis classes,

- triggers aggregations and comparisons,

- generates and saves all visual outputs into outputs/figures/.

 **Visualizations**
 
Top geographic locations by tweet volume
The following bar chart shows the most active geographic locations worldwide
in terms of tweet volume for each candidate.
![Top geographic locations by tweet volume](outputs/figures/geo_barplot.png)

# Interpretation 
- The United States dominates tweet activity for both candidates.

- Several international locations (Europe, India, Australia) also show strong engagement.

- Trump tends to generate higher tweet volume in multiple major regions.

**Balanced geographic comparison**

To avoid dominance by very large regions, we also visualize relative differences between candidates.

![Balanced tweet volume difference by location](outputs/figures/geo_diff_barplot.png)
*Interpretation*

- Blue bars represent Biden-dominant locations.

- Red bars represent Trump-dominant locations.

- This view highlights meaningful contrasts even for smaller regions.

  **Global dominance map**

The central visualization of this feature is a world dominance map.

![World dominance map](outputs/figures/world_map.png)

Color encoding:

- Blue shades → Biden-dominant regions

- Red shades → Trump-dominant regions

- Color intensity reflects the strength of dominance

Hovering over a country (in the interactive versions) displays:

- tweet counts for each candidate,

- dominance score.

  **Interactive Maps**

In addition to static figures, several interactive maps are generated:

- outputs/figures/world_dominance_22colors_LEGEND.html

- outputs/figures/world_dominance_22colors_OOP.html

- outputs/figures/usa_cities_dominance.html

These maps allow zooming, hovering, and deeper exploration of geographic patterns.

**Interpretation and Limitations**

This geographic analysis highlights where online political discussion was most active during the 2020 US election.

However, several limitations must be acknowledged:

- Twitter users are not representative of the general population.

- Some countries show strong activity despite not being directly involved in the election.

- Location data is self-reported and may be incomplete or ambiguous.

Despite these limitations, the geographic analysis provides valuable insight into global political engagement and visibility on social media.

---

## 3. Sentiment Analysis

**Summary**

- Uses the sentiment analysis RoBERTa 
- Computes a sentiment score for each tweet  
- Aggregates sentiment per day and per candidate

**Command‑line example**

```bash
python -m src.sentiment \
    --input data/processed/all_tweets_clean.csv \
    --output outputs/sentiment_by_day.csv
```

**Python example**

```python
import pandas as pd
from src.sentiment import SentimentAnalyzer

df = pd.read_csv("data/processed/all_tweets_clean.csv")
analyzer = SentimentAnalyzer()

df["sentiment"] = df["clean_text"].apply(analyzer.predict)
daily = df.groupby(["date", "candidate"])["sentiment"].mean()
print(daily.head())
```

##  Key Visualizations

The project includes several visualizations, with at least one central, relevant plot.

### Example: Average sentiment per day and per candidate

- X‑axis: date  
- Y‑axis: average sentiment  
- One line per candidate

```markdown

```

This figure shows how Twitter sentiment evolves over the campaign for each candidate.

Other possible plots:

- Barplot of top hashtags  
- Distribution of tweet lengths  
- Barplot of the most active users

> (Bonus) Add a screenshot of the sentiment‑over‑time chart.

---

## 4. User Analysis

**Summary**

- Identifies most active accounts (number of tweets / retweets)  
- Computes simple metrics (followers, friends, etc., when available)  
- Can filter by candidate or hashtag

**Associated notebook**

- `notebooks/04_users.ipynb`

**Example**

```python
import pandas as pd
from src.users import top_active_users

df = pd.read_csv("data/processed/all_tweets_clean.csv")
top10 = top_active_users(df, n=10)
print(top10)
```

> (Bonus) Add a screenshot of the table or a barplot of the top 10 users.

---

## 5. Trump vs Biden Comparison

**Summary**

- Uses sentiment scores and user data to compare both candidates  
- Proportion of negative / neutral / positive tweets per candidate  
- Visual difference in emotional intensity (e.g. mean sentiment over time, distributions)

**Associated notebook**

- `notebooks/02_compare.ipynb` (or equivalent)

**Example**

```python
import pandas as pd

df = pd.read_csv("data/processed/all_tweets_clean.csv")
by_candidate = df.groupby("candidate")["sentiment"].agg(["mean", "median", "count"])
print(by_candidate)
```

---

##  Quick Steps

1. Run Part 2 of `main.ipynb` → produces `data/clean/trump_clean.csv`, `data/clean/biden_clean.csv` and saves figures.   
2. Run Part 3 of `main.ipynb` → computes sentiment with R.O.B.E.R.T.A  → produces `data/analysed/sentiment_summary.csv`, `data/analysed/tweets_with_sentiment.csv` and saves figures.  
3. Run Part 4 of `main.ipynb` → produces voters representation map.  
4. Run Part 5 of `main.ipynb` for Trump/Biden comparison with plots.
5. 4. Run Part 6 of `main.ipynb` user's influences on the campaign.

---

##  Project Structure

```text
.
├── data/
│   ├── raw/                   # Raw CSV data
│   ├── clean/                 # Cleaned CSV data
│   └── analysed/              # Enriched data
├── outputs/                   # Generated images
├── src/
│   ├── main.ipynb                # The main projet code
│   ├── visualization.py       # Comparison plots functions
│   ├── utils_text.py          # Text preprocessing functions
│   ├── sentiment_analysis.py  # Sentiment analysis utilities
│   ├── sentiment_model.py     # Model
│   ├── polytical_labeler.py     # Model 
│   ├── polytical_visualizer.py     # Model 
│   ├── data_loader.py         # Data loading functions
│   └── geography_analysis.py  # geography analysis utilities
├── quick_test.py           # Minimal test script
├── requirements.txt
└── README.md
```

---

##  Example End‑to‑End Pipeline

```bash
# 1. Create and activate environment
python -m venv .venv
.\.venv\Scripts\activate  # Windows

# 2. Install dependencies
pip install -r requirements.txt

# 3. Clean tweets
python -m src.users \
    --input data/raw/all_tweets.csv \
    --output data/processed/all_tweets_clean.csv

# 4. Compute sentiment
python -m src.sentiment \
    --input data/processed/all_tweets_clean.csv \
    --output outputs/sentiment_by_day.csv
```

Then open notebooks (for example `notebooks/03_sentiment.ipynb`) to generate figures and explore results.

---

## Contributors
- Marie-Pauline WAGUE – Data completion and Sentiment analysis 
- Danielle Keune – Data Cleaning 
- Sandrine Bodjio – Geographic analysis 
- Tonye Kaptue – Users Analysis
- Silvio – Candidates Comparison
---

##  License

Project developed as part of a course.  
For educational use only.
```
