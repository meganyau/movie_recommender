# Movie Recommender (Content-Based)

[![Launch in Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/meganyau/movie_recommender/HEAD?labpath=movie_recommender.ipynb)


A small content-based movie recommender inspired by Letterboxd “Top 4” favorites.

Given four favorite movies, the notebook:
- represents each film as a TF–IDF + count-based feature vector (genres, keywords, cast, director, overview text),
- uses cosine similarity to find similar movies,
- detects dominant genres from your favorites, and
- recommends 3–5 movies with match scores and explanations (shared genres/cast/director + a one-sentence synopsis).

> Note: The dataset is the TMDB 5000 Movies & Credits dataset from Kaggle.  
> To run the notebook yourself, download the dataset from Kaggle and place the files in this folder as `tmdb_5000_movies.csv` and `tmdb_5000_credits.csv`.

---

## How to run locally

```bash
git clone https://github.com/meganyau/movie_recommender.git
cd movie_recommender

python -m venv .venv
source .venv/bin/activate   # on Windows: .venv\Scripts\activate

pip install -r requirements.txt
jupyter notebook


-- 
## Example

For example, if you choose these four favorites:

- (500) Days of Summer  
- 10 Things I Hate About You  
- 27 Dresses  
- You’ve Got Mail  

the recommender detects dominant genres **Comedy** and **Romance** and returns rom-com style recommendations such as:

- **Don Jon** — high match due to overlapping cast and similar genre mix  
- **The Wedding Date** — similar genres (Comedy, Romance) and wedding-themed plot  
