# Football Match Outcome Predictor

Predict the probability of a home win, away win, or draw for any matchup between two football clubs, based on their historical home/away performance — weighted so recent seasons count more than older ones. Wrapped in a simple Gradio web app.

## How it works

1. **Data**: Loads `Country`, `League`, `Match`, `Player`, `Player_Attributes`, `Team`, and `Team_Attributes` tables from a local `database.sqlite` file (the [European Soccer Database](https://www.kaggle.com/datasets/hugomathien/soccer) — 11 countries/leagues, ~26,000 matches, 299 teams, 8 seasons from 2008/09 to 2015/16).
2. **Recency weighting**: Each season is assigned a weight via `decompose(n)`, which scales linearly by season order and normalizes across all seasons — so the most recent season contributes more to a team's rating than the earliest one.
3. **Home/away splits**: Matches are split into home and away perspectives. For each, win/loss/draw outcomes are flagged, multiplied by the season weight, and summed per team to produce weighted home and away performance profiles.
4. **Prediction**: `predict_result(home, away)` looks up both teams (by team ID or short name, e.g. `ARS`, `AVL`), cross-multiplies the home team's home-performance odds against the away team's away-performance odds for each outcome (home win / away win / draw), and normalizes the three products into probabilities.
5. **Interface**: A Gradio app takes the home and away team short codes as text input and returns the outcome probabilities.

## Requirements

- Python 3.12+
- `pandas`
- `gradio`
- `sqlite3` (standard library)
- The `database.sqlite` file from the European Soccer Database, placed in the project root

Install dependencies:

```bash
pip install pandas gradio
```

## Usage

1. Download `database.sqlite` from the [European Soccer Database on Kaggle](https://www.kaggle.com/datasets/hugomathien/soccer) and place it in the project root.
2. Create a `log/` directory (used to store a CSV of all possible home/away team permutations).
3. Run the notebook (`jupyter notebook`) cell by cell, or export it to a script.
4. The Gradio interface launches locally — enter two team short codes (e.g. `ARS` for Arsenal, `AVL` for Aston Villa) to get outcome probabilities.

Example output:

```
MATCH OUTCOME PROBABILITY
Arsenal : 55%
Draw: 23%
Aston Villa : 22%
```

## Notes & limitations

- Predictions are based purely on historical weighted win/loss/draw rates by team — they don't account for squad changes, injuries, form, or head-to-head history.
- Team lookups only cover clubs present in the underlying database (2008/09–2015/16 seasons), so more recent transfers or newly promoted clubs won't be recognized.
- `predict_result` returns an error string (rather than raising) for unrecognized teams or identical home/away input.

## License

Add a license of your choice (e.g. MIT) here.
