# Football Match Outcome Predictor

Predicts the probability of a home win, away win, or draw for any matchup between two football clubs, based on their historical home and away performance — weighted so recent seasons count more than older ones. Wrapped in a simple Gradio web app.

## Contents

| File | Description |
|---|---|
| `Sports Project.ipynb` | Notebook containing all data loading, weighting logic, prediction function, and the Gradio app. |
| `README.md` | This file. |

## How It Works

1. **Data**: Connects to a local `database.sqlite` file (the [European Soccer Database](https://www.kaggle.com/datasets/hugomathien/soccer) — 11 countries/leagues, ~26,000 matches, 299 teams, 8 seasons from 2008/09 to 2015/16) and loads the `Country`, `League`, `Match`, `Player`, `Player_Attributes`, `Team`, and `Team_Attributes` tables. Only `Country`, `League`, `Match`, and `Team` are actually used downstream — the player-level tables are loaded for exploration but don't feed into the prediction logic.
2. **Recency weighting**: `decompose(n)` builds a weight for each of the `n` seasons — season order (1, 2, 3, ...) divided by `n`, then normalized so all season weights sum to 1. This means the most recent season gets the largest weight and the earliest season the smallest.
3. **Home perspective**: `Match` is merged with `Team`, `Country`, and `League` on the home team, and each match is flagged `home_win` / `home_loss` / `home_draw` by comparing goals scored. Each flag is multiplied by its season's weight, and the results are summed per team (`home_odds`) to get a weighted home win/loss/draw profile for every club.
4. **Away perspective**: The same process is repeated from the away team's side (`away_win` / `away_loss` / `away_draw`) to build `away_odds`.
5. **Prediction**: `predict_result(home, away)` accepts either a team's numeric `team_api_id` or its short name (e.g. `ARS`, `AVL`), looks up both teams' weighted profiles, and cross-multiplies: home team's `home_win_odds` × away team's `away_loss_odds` (home win), home team's `home_loss_odds` × away team's `away_win_odds` (away win), and both teams' draw odds together (draw). The three products are normalized to sum to 100%.
6. **Interface**: A Gradio `Interface` takes two text inputs (home and away team short codes) and returns the formatted probability string.

A `log/log.csv` file is also generated, listing every possible ordered pairing of team short names (via `itertools.permutations`) — useful as a reference list of valid inputs, though it isn't read back by the app itself.

## Requirements

* Python 3.12+
* `pandas`
* `gradio`
* `sqlite3` (standard library)
* The `database.sqlite` file from the European Soccer Database, placed in the project root

Install dependencies:
```
pip install pandas gradio
```

## Usage

1. Download `database.sqlite` from the [European Soccer Database on Kaggle](https://www.kaggle.com/datasets/hugomathien/soccer) and place it in the project root.
2. Create a `log/` directory (used to store the CSV of all possible home/away team permutations).
3. Run the notebook (`jupyter notebook`) cell by cell, or export it to a script.
4. The Gradio interface launches locally — enter two team short codes (e.g. `ARS` for Arsenal, `AVL` for Aston Villa) to get outcome probabilities.

Example output:
```
MATCH OUTCOME PROBABILITY
Arsenal : 55%
Draw: 23%
Aston Villa : 22%
```

## Notes & Limitations

* Predictions are based purely on historical weighted win/loss/draw rates by team — they don't account for squad changes, injuries, form, or head-to-head history.
* Team lookups only cover clubs present in the underlying database (2008/09–2015/16 seasons), so more recent transfers or newly promoted clubs won't be recognized.
* `predict_result` returns an error string (rather than raising) for unrecognized teams or identical home/away input.
* Team short names are matched case-insensitively (input is upper-cased before lookup).
