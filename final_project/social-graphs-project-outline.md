Got it. Here’s a tight, end-to-end roadmap you can follow for the project, aligned with the course goals (networks + NLP) and your Spotify + Genius plan.

## 🎯 Central idea (thesis)

Cluster songs by **audio signal** and test whether those clusters align with **language use** in lyrics. Concretely:

* Build an **audio similarity graph** of tracks, detect communities, and quantify how “genre-like” they are.
* Scrape lyrics per track, compute **TF-IDF**/topics, and test whether audio-based communities show **distinctive lexical signatures**.

---

## 🗺️ Project outline (what you’ll explore)

1. **Do audio features alone recover meaningful communities?**
   Compare audio-derived clusters to `track_genre` (purity/NMI/ARI) and to lyric-based topics (enrichment tests).
2. **Which audio dimensions drive separation?**
   PCA/UMAP on standardized features; loadings + partial dependence (feature importances if you train a simple classifier).
3. **Do communities have distinct language?**
   TF-IDF keywords per community; optional topic models (LDA/BerTopic) as sensitivity check.
4. **Network perspective:**
   Build a k-NN track similarity graph; run Louvain/Leiden; measure modularity, degree distribution, assortativity by genre, and community enrichment by lyric terms.

---

## 🧱 Data sources & artifacts

* **Spotify Kaggle CSV** (given).
* **Genius API** for lyrics: map track title + first artist; grab 1–3 top matches with fuzzy matching; rate-limit handling; store raw + cleaned text.
* **Derived data**:

  * `audio_features_clean.parquet`
  * `lyrics_clean.parquet`
  * `track_graph.edgelist` / `track_graph.gml`
  * `tfidf_per_community.csv`

---

## 🔧 Pipeline & tasks (with rough order)

### A. Ingestion & cleaning

* Load CSV; remove duplicate `track_id`; drop rows missing key audio features.
* Normalize categoricals (explicit→0/1); standardize numeric features (z-score).
* Keep a **core audio set**: `[danceability, energy, valence, tempo, loudness, acousticness, instrumentalness, liveness, speechiness]` (+ optional: `mode`, `key` via sin/cos encoding, `tempo` log-scaled if heavy-tailed).

### B. Lyrics scraping & preprocessing

* For each track: query Genius by `"track_name artist_name"`, keep best match; save URL, status, and fetched text.
  Fallback rules: strip parentheses/“feat.”, remove punctuation, try alternate artist ordering.
* Text clean: lower, strip HTML, remove `[Chorus]` brackets, drop non-language tokens, de-duplicate repeated lines.
* Tokenize → remove stopwords → lemmatize (keep nouns/verbs/adjectives). Keep doc length, tokens per track.

### C. Audio representation & clustering

* **Dimensionality reduction:**

  * PCA (explain variance; keep 90–95% variance, likely 4–6 PCs).
  * UMAP (optional) for 2D visualization (not for training metrics).
* **Clustering options (try 2):**

  * k-means on PCA space (choose k by elbow/silhouette; compare with #genres ≈ not required to match).
  * HDBSCAN on PCA space (density-based, handles noise).
* **Graph construction:**

  * Build k-NN (e.g., k=10–20) on PCA space (cosine or Euclidean).
  * Weight edges by similarity (e.g., `1 / (1 + distance)`), symmetrize.
  * Community detection with Louvain/Leiden; compute **modularity**.

### D. NLP features & analysis

* TF-IDF on cleaned lyrics (min_df to drop ultra-rare; cap max_features, e.g., 10–20k).
* Per **audio community**: aggregate TF-IDF to get top terms (mean or sum); run chi-square term enrichment vs. rest.
* Optional: topic modeling (LDA/BerTopic) to see if topics align with audio communities/genres.
* Optional sentiment/valence from lyrics; compare to Spotify `valence`.

### E. Tie networks + text (multilayer view)

* Treat TF-IDF topics or top-k keywords as **node attributes**.
* Compute **assortativity** of the k-NN graph by:

  * genre,
  * top topic,
  * sentiment bins,
  * lyric vocabulary clusters (k-means on TF-IDF embeddings).
* Run **community homogeneity tests**: for each audio community, test over-representation of specific genres/topics (hypergeometric/Fisher’s exact).

### F. Evaluation & baselines

* **Cluster quality:** silhouette (audio space);
* **Agreement with genre:** NMI/ARI/Purity between audio communities and `track_genre`.
* **Network quality:** modularity (higher = stronger community structure).
* **Text alignment:** per-community keyword enrichment p-values; topic coherence (if LDA).
* **Baselines:**

  * Randomly permute genre labels → NMI/ARI should drop.
  * Random k-partition with same sizes → compare modularity.
  * Lyrics-only clustering (k-means on TF-IDF) → compare to audio communities (are they orthogonal or consistent?).

### G. Visualization (what to show)

* PCA variance curve + feature loadings (bar plot).
* UMAP 2D scatter: points colored by **audio community**; shape or outline by **genre**.
* Degree distribution of the k-NN graph; community size histogram.
* Heatmap: community × top keywords (TF-IDF scores).
* Bar charts: NMI/ARI vs. methods; modularity vs. k.
* Sankey/alluvial: mapping between audio communities ↔ genres.

### H. Repro & structure

* `/notebooks`: 01_ingest_clean, 02_lyrics_scrape, 03_audio_pca_cluster, 04_graph_communities, 05_tfidf_topics, 06_alignment_eval, 07_figures.
* `/data_raw`, `/data_clean`, `/artifacts`, `/figures`.
* `config.yaml` with parameters (k, min_df, max_features, random_seed).
* Save seeds; store environment file; small sample CSV for grading.

---

## 📏 What to report (results you want)

* **Finding A:** Audio communities have **significantly higher** modularity than random partitions; their NMI with genre is X (vs. baseline Y).
* **Finding B:** Distinct **lyric vocabularies** enriched in specific audio communities (keywords + p-values).
* **Finding C (optional):** Lyrics sentiment/lexical valence partially tracks Spotify `valence` (correlation r).
* **Finding D (optional):** Network assortativity by genre/topic > 0, indicating homophily in the audio graph.

---

## 🧪 Sanity checks & pitfalls

* Lyrics coverage ≥70% of selected tracks; report coverage by genre.
* Handle title collisions/feats; keep a “match_quality” score.
* Balance genres when sampling for scraping (avoid a few huge genres dominating).
* Beware popularity bias: confirm results hold after controlling for `popularity` (e.g., partial correlation or stratified sampling).
* Check for **leakage** (don’t use genre labels to build the audio graph).

---

## ⏱️ Timeline & deliverables

**Week 1 (now):**

* Clean Spotify; define core feature set; PCA; initial k-means; pick a clustering setting.
* Prototype k-NN graph + Louvain; compute modularity; basic UMAP plot.

**Week 2:**

* Implement lyrics scraper; fetch for a **balanced subset** (e.g., 50–100 tracks × ~10 genres = 500–1000 tracks).
* Clean text; TF-IDF; per-community top terms; first enrichment tests.

**Week 3:**

* Full evaluation: NMI/ARI, modularity vs. baselines; assortativity; sensitivity (k-NN k, clustering method).
* Optional LDA/BerTopic; sentiment vs. Spotify valence.
* Polish figures and draft results section.

**Week 4:**

* Write paper: intro, data, methods, results, discussion, limitations, ethics/repro.
* Final plots; README; environment file.

---

## 🎬 60–90s video storyboard (first hand-in)

* **Hook (5s):** “Can sound alone group songs into meaningful communities—and do those groups sing about different things?”
* **Data (15s):** Spotify (N tracks, M features), Genius lyrics (coverage %), how you scraped.
* **Network (20s):** PCA → k-NN graph (k=15), Louvain communities, modularity number; quick UMAP with colored clusters.
* **Text (15s):** TF-IDF per community; show 3–4 keywords per cluster.
* **Tie-in (10s):** NMI with genre (early number), one neat enrichment example.
* **Plan (10s):** What’s next (lyrics expansion, evaluation, baselines).
* **Title card (5s):** Team names + GitHub repo.

---

## 📚 Minimal methods (what to actually run)

* **Standardize** features → **PCA(n_components≈6)** → **k-NN graph (k=15, cosine)** → **Louvain** → **modularity**.
* **Clustering alt:** k-means(k≈8–20), pick by silhouette.
* **TF-IDF:** `TfidfVectorizer(min_df=5, max_features=20000, ngram_range=(1,2))`.
* **Enrichment:** For each community, rank terms; run χ² vs. rest; report top terms + p-values (FDR-corrected).
* **Metrics:** silhouette (audio), NMI/ARI (cluster vs. genre), assortativity (genre/topic), modularity (graph).
* **Baselines:** label permutation for NMI; random partitions for modularity.

---

## 🧠 Compact glossary (for the report)

* **k-NN graph:** nodes = tracks; edges connect each node to its k most similar neighbors in feature space.
* **Modularity:** measures how well a partition captures dense within-group and sparse between-group edges. Higher is stronger community structure.
* **Assortativity:** tendency for connected nodes to share an attribute (e.g., same genre).
* **NMI/ARI:** agreement scores between two partitions (clusters vs. genre).
* **TF-IDF:** weights words that are frequent in a document but rare across the corpus, surfacing distinctive terms.

---

## ✅ Immediate next steps

1. Lock the **core audio feature list** and run PCA + variance explained.
2. Build a **k-NN graph** and compute **modularity** with Louvain; make the UMAP figure.
3. Define a **balanced scraping list** (per genre) and start the Genius pull with robust matching.

Once you have the first UMAP + modularity number and a tiny TF-IDF table for 2–3 communities, you’ll already have a compelling first video.
