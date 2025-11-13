# FootballPredict

A complete pipeline to build models that predict Premier League match outcomes. The project covers scraping historical leaderboard and match data, cleaning and preprocessing, exploratory data analysis (EDA), feature engineering, and building/evaluating machine learning models.

## Table of contents
- [Summary](#summary)
- [Data scraping](#data-scraping)
- [Data preprocessing](#data-preprocessing)
- [EDA](#eda)
- [Feature engineering](#feature-engineering)
- [Model training](#model-training)
- [Pipeline and notebooks](#pipeline-and-notebooks)
- [Results and artifacts](#results-and-artifacts)

## Summary
- Scraping: notebooks in `crawl_data/` collect historical club and season-level information.
- Data cleaning: `raw_data/` -> `post_processing/` and `src/clean_data_*.ipynb` prepare clean train/test CSVs.
- Feature engineering: notebooks under `feature_engineering/` create additional predictors used by models.
- Modeling: `train_model/` contains per-club model training notebooks, best-parameter files and final model outputs.

## Data scraping

- Data source: team and match statistics are scraped from `fbref.com`.
- Statistics: 
    + For training: 21 seasons (2000-2001 -> 2020-2021) ~ 15960 matches 
    + For testing: 3 seasons (2021-2022 -> 2023-2024) ~ 2280 matches

- Scraping framework & libraries used:
	- requests — for HTTP GET requests
	- BeautifulSoup (bs4) — HTML parsing and link extraction
	- pandas (`pd.read_html`) — convenient parsing of HTML tables ("Scores & Fixtures", "Shooting", etc.)
	- time / os — for rate-limiting and filesystem operations

- Scraping techniques implemented in the notebooks:
	- Use `pd.read_html` to extract the "Scores & Fixtures" tables directly into DataFrames.
	- Discover per-team "Shooting" (and other) pages by scanning anchor tags on the team page and selecting links containing the `all_comps/shooting/` substring.
	- Merge match-level tables (fixtures + shooting) on `Date` and `Opponent` to enrich rows with shot statistics.
	- Filter merged rows to `Comp == "Premier League"` to obtain league-only match entries.
	- Add `Season`, `Team` and source `url` columns, then save per-team CSVs under `raw_data/<team>/`.
	- Basic robustness: `try/except` around table parsing to skip missing pages, and `time.sleep()` between requests to avoid throttling.

## Data preprocessing
- Remove columns that are occupied by too many null values
- Fill null values in other columns
- Fix typos in string values

## EDA
- Purpose: Understand distributions, relationships and time patterns in match-level data to guide feature engineering and modeling.
- Techniques:
  - Summary statistics, null-rate checks, histograms and boxplots.
  - Rolling/form metrics (moving averages, recent-N aggregates).
  - Correlation heatmaps and pairwise plots for feature selection.
  - Head‑to‑head and home/away comparisons.
  - PCA/t‑SNE and K‑means for visualization / cluster features.
  - Model-driven checks: feature importances and SHAP / permutation explanations.


## Feature engineering
- Additional features: W/D/L rate, winning rate gap between 2 clubs, average scores in 5 recent matches, ELO, whether the opponent is in BIG6

## Model training
- Encoding: OneHotEncoder, LabelEncoder, BinaryEncoder
- Models: Logistic Regression, One-vs-rest, Random Forest, K-mean cluster

## Pipeline and notebooks
1. Data collection
	- `crawl_data/` contains notebooks to scrape club and season data from public sources. Use these to update or extend the dataset.

2. Raw -> clean data
	- `raw_data/` contains CSV exports of raw scrapes.
	- `src/clean_data_train.ipynb` and `src/clean_data_test.ipynb` show the cleaning steps and create `clean_data_train.csv` and `clean_data_test.csv`.

3. EDA (exploratory data analysis)
	- `src/data_anaylysis/` contains notebooks with visualizations and statistical analysis used to identify useful features.

4. Feature engineering
	- `feature_engineering/` notebooks compute derived features (rolling statistics, form, cluster-based features, etc.) used by the models. 

5. Model training and evaluation
	- `train_model/` contains club-specific training notebooks (Arsenal, MC, MU). Each folder includes `best_params/`, `final_model/` and evaluation outputs.
	- Notebooks demonstrate hyperparameter search, training, cross-validation and test evaluation.

## Results and artifacts
- Evaluation CSVs (test results) are available in `train_model/*/test_results.csv` and in some model folders as `test_results.csv`.
- Best hyperparameters are stored in `train_model/*/best_params/` and `train_model/*/final_best_params/`.
- Final model pickles or serialized objects may be in `train_model/*/final_model/` or `final_model_clustered/` depending on the experiment.