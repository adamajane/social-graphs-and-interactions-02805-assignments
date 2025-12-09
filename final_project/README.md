# Beyond Labels: Clustering Musical Genres via Audio Similarity and Uncovering Lyrical Themes

## Social Graphs and Interactions (02805) - Final Project

**Group 37:** Vasiliki Tsanaktsidou, Arun Kumar Dhuraisamy, and Adam Ajane

## Overview

This project explores whether genres that _sound_ similar also _talk about_ similar things. We construct an audio similarity network from Spotify data, detect communities using Louvain algorithm, and analyze lyrical themes and sentiment within these communities.

**Key Question:** Do audio-based genre clusters share thematic and emotional patterns in their lyrics?

## Project Structure

### Analysis Notebooks

- **[final_explainer_notebook.ipynb](final_explainer_notebook.ipynb)** - Main explainer combining all three parts
- **[separate_analysis_notebooks/](separate_analysis_notebooks/)** - Individual analysis notebooks for each part

### Three-Part Analysis

1. **Data Cleaning** - Process ~114K Spotify tracks → 73,403 unique tracks with validated audio features
2. **Network Analysis** - Build k-NN audio similarity graph (k=10) and detect communities via Louvain
3. **Text Analysis** - Scrape lyrics, compute TF-IDF representations, and analyze sentiment (VADER)

## Data

All datasets are in [`data/`](data/):

- `spotify_tracks_dataset.csv` - Raw Spotify dataset (~114K tracks)
- `clean_spotify_tracks_dataset.csv` - Cleaned dataset (73,403 unique tracks)
- `spotify_knn_k10_removed_same_tracks.gexf` - Track-level audio similarity network (k=10)
- `spotify_genre_similarity_audio.gexf` - Genre-level aggregated network with communities
- `average_genre_signatures.csv` - Mean audio features per genre
- `text_analysis_output/` - Lyrics data, TF-IDF results, sentiment scores, and visualizations

## Requirements

See notebook imports for dependencies: `pandas`, `numpy`, `matplotlib`, `seaborn`, `networkx`, `scikit-learn`, `nltk`, `wordcloud`
