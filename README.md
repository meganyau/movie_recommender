# Movie Recommender (Letterboxd-Style Prototype)

This repository contains a small content-based movie recommender inspired by **Letterboxd**.  
It currently has two main notebooks:

- **`v1_kaggle_widget/kaggle_widget.ipynb`**  
  A first version that uses only the public **TMDB 5000 Kaggle dataset**.  
  You choose four favorite movies, and it recommends a few similar titles using TF–IDF over genres, keywords, cast, director, and overview text.

- **`v2_letterboxd_demo/letterboxd_demo.ipynb`**  
  A richer prototype that incorporates **real Letterboxd exports** (ratings + favorites), a tiny **friends graph**, and a more Letterboxd-like interactive UI with explanations.

The rest of this README focuses on the Version 2 prototype, which is the more “Letterboxd-ish” system.
> **Privacy note**  
> Letterboxd export folders for real users are **not included in this repo** and are ignored via `.gitignore`.  
> The notebook expects these CSVs to exist locally if you want to fully reproduce the demo.

---

## Data Sources

**Public data**

- `tmdb_5000_movies.csv`  
- `tmdb_5000_credits.csv`  

From TMDB 5000 Kaggle dataset, used to build the content-based model (genres, keywords, cast, crew, overview text).

**Private / local data (not in repo)**

Per-user Letterboxd exports:

- `letterboxd-*/ratings.csv`
- `letterboxd-*/profile.csv`

For the demo, I collected data for **5 real users** (friends) and exported their
Letterboxd logs. These live in local folders such as:

- `letterboxd-megwhat/`
- `letterboxd-gabybrown/`
- `letterboxd-ariannagk/`
- `letterboxd-bobblet11/`
- `letterboxd-tharaaaaa/`

These folders are **ignored by Git** but can be used locally to run the notebook.

---

## Model Overview

### 1. Content features (TMDB)

In `letterboxd_demo.ipynb` we first prepare a content representation for each
movie:

- Merge `tmdb_5000_movies.csv` and `tmdb_5000_credits.csv`.
- Parse JSON-like strings into Python lists:
  - `genres_clean`
  - `keywords_clean`
  - `cast_clean` (top ~5 actors)
  - `director_clean` (from the `crew` field)
- Clean the text overview and keep the first ~50 words.
- Build a **feature "soup"** per movie:
  - genres + keywords + cast + director + overview tokens
- Run a **TF–IDF vectorizer** over the soup to get a matrix `X`  
  (one row per movie, one column per token).

This reproduces the Version 1 content model, but we now use it as the base for
Letterboxd-driven recommendations.

---

### 2. Mapping Letterboxd ratings to TMDB movies

For each Letterboxd export folder:

1. Load `ratings.csv` and tag each rating with `user_id`.
2. Normalize titles to a lowercase `title_norm` field.
3. Build a small mapping from TMDB movies:

   ```text
   movie_idx (row index in TF–IDF matrix)
   title_norm
   year
4. Join ratings to this mappig on (title_norm, year) to attach a movie_idx 

For favorites:
- Parse profile.csv to extract up to 4 favorite films per user.
- Expand the favorite URLs into individual rows.
- Join them back to ratings

As a result, lb_ratings_all becomes a single table of user–movie interactions:

user_id, Name (movie title), Year, Rating, favorite (0/1), movie_idx (link back into TMDB + TF–IDF matrix)

---

### 3. Building a user profile
For each user we build a taste vector as a weighted sum of movie vectors:
- Start from the TF–IDF row for each movie they rated (X[movie_idx]).
- Weight it by a function of the rating and favorite flag, (ex: extra boost if favorite == 1)
- Normalize by the total weight to get a single user vector.

This gives us a point in the same vector space as the movies, representing that
user’s overall taste.

---

### 4. Scoring candidate movies
To recommend movies for a given user:
1. Compute cosine similarity between user vector and all movie vectors
2. Drop any movies that user has already rated
3. Friend boost: (if friend rated movie >4, add small boost; if friend has it in top 4, add large boost) 
                           
This is implimented in function recommend_for_user_with_explanations    
                           
---

### 5. Explanation generation 
For a user:
- Collect set of directors, actors, and genres they strongly like
- For each candidate movie, build reasons such as:
       - "Same director as The Truman Show (Peter Weir)"
       -  "Overlapping Cast: Keira Knightley (also in the ____)
       - "Matches genres you often rate highly: Drama, Romance"
- Use friends graph to add reasons 
       - "Your friend ______ has this in their Top 4"
       - "Your friend ______ rated this 4.5"
- Pad with at most one generic "trust us" reason if vector is high, but no explanation

The final result is a list of 2-3 bullet point explanations per recommendation

---

## Limitations & Caveats

This is a small, exploratory prototype, not a production system. Some important caveats:

- **No official API integration (yet)**  
  All data access is via static CSVs:
  - TMDB 5000 Kaggle dataset for movie metadata.
  - Letterboxd CSV exports for ratings/favorites.
  There is no live TMDB or Letterboxd API integration in this version.

- **Catalog limited to TMDB 5000 (~pre-2015)**  
  The TMDB 5000 dataset mostly covers movies released up to around 2015.  
  When I map Letterboxd ratings + favorites into this catalog using `(title_norm, year)`, only about **half of the rated films** end up with a valid `movie_idx`. Anything that doesn’t match the Kaggle catalog is effectively invisible to the recommender.

- **Tiny, hand-crafted friends graph**  
  Letterboxd friend relationships are not available in the exports I used, so the “friends” network in this demo is manually defined:
  - it only covers a handful of users,
  - relationships are hard-coded (e.g. everyone is friends with everyone except one user),
  - it’s meant to show how friend-based explanations could work, not to reflect real social data.

- **Only 5 real user datasets**  
  The prototype is built around exports from **five real Letterboxd accounts**, plus a bit of manual cleanup (e.g. mapping favorites that weren’t logged as ratings). This is enough to exercise the logic for:
  - mapping ratings → TMDB,
  - building user profiles,
  - generating friend-aware explanations,  
  but it’s not representative of a large-scale deployment.

- **Heuristic scoring and explanations**  
  The scoring blend (content similarity + friend boosts) and explanation rules (how many reasons, which ones to prioritize) are manually chosen heuristics. A production system would likely:
  - learn weights from data (learning-to-rank),
  - A/B test how users respond to different explanation styles.

## Future Work

Planned extensions / improvements:

- Integrate a proper **TMDB or Letterboxd API** for a complete, up-to-date catalog and more reliable ID matching.
- Add **watchlists** and curated **lists** as signals (e.g. “good cry” lists).
- Introduce **online learning**: adjust the user profile and scoring based on “Yes / Next / I’ve watched this” feedback over time.
- Impliment recommendations from favourite actors/ directors with their top 4

                           

