# FIT: measuring how good NBA players are, and how well they fit together

FIT is a basketball impact metric I'm building. Most public impact metrics give each player one number and assume he adds the same amount no matter who he plays with. Anyone who watches basketball knows that isn't true: two great players can be worse together than apart. FIT is built to measure that.

**Status:** work in progress. The data pipeline and the baseline model are built. The FIT layers are next.

## The problem

Today's best public metrics are built on **RAPM** (Regularized Adjusted Plus-Minus). RAPM is a ridge regression over every stretch of every game where the same 10 players are on the floor. It's a strong idea, but it has three weak spots:

1. **The scoreboard is noisy.** If opponents get hot from three, the defenders get blamed, even though open-three shooting is mostly luck.
2. **Some value never shows up in the box score.** Shots that never get taken because a rim protector is standing there, screens, gravity.
3. **It assumes a player's value is the same in every lineup.** Fit gets averaged away.

The usual way to look at fit is WOWY (with or without you): check the net rating when two specific players share the floor. But the samples are tiny, and it says nothing about players who have never played together, which is exactly the question in a trade.

## The idea

FIT turns every player into a small **skill profile** (spacing, rim pressure, creation, size, rim protection, perimeter defense), then learns from every lineup in the league how those skills combine:

- **Diminishing returns:** a second rim protector probably adds less than the first.
- **Synergy:** spacing is worth more next to a driver.

Because it learns from skill combinations instead of specific pairs, FIT can estimate how a lineup will do **before it has ever played together**. A pair's actual minutes still count, as evidence that nudges the estimate, weighted by sample size.

## The plan

| Step | What it does | Status |
|---|---|---|
| 1. Pull | Every game of a season from NBA.com: schedule, play-by-play, rotations | Built |
| 2. Clean | Build stints (same 10 players on the floor) and check every game against the official box score | Built |
| 3. Baseline | Plain RAPM: the number every FIT layer has to beat | Built |
| 4. Expected points | Replace made/missed with what each shot is usually worth, to take luck out of the scoreboard | Next |
| 5. Multi-factor RAPM | Separate ratings for shot quality, turnovers, rebounding, free throws, and rim attempts allowed, priced in points | Planned |
| 6. Skill profiles | About 6 explainable skill scores per player | Planned |
| 7. FIT model | Individual value + a fit bonus or penalty for the lineup's skill mix | Planned |
| 8. Validation | Predict games the model hasn't seen; the key tests are new lineups and traded players | Planned |

**How I'll know if it works:** train on past seasons, predict the next one, and compare against plain RAPM and public metrics on the same games. If a layer doesn't improve out-of-sample predictions, it doesn't make it into FIT.

## What's in the repo

- `01_pull_and_clean.ipynb`: downloads a season and turns it into clean stints. Every game is checked three ways: the points add up to the official final score, every stint has exactly 5 players per team, and the stints cover the whole game. Games that fail are saved to `data/bad_games.csv` and left out.
- `02_baseline_rapm.ipynb`: offense and defense RAPM with ridge regression, weighted by possessions, with the penalty picked by cross-validation split by game.

## How to run it

```
pip install -r requirements.txt
```

1. Open `01_pull_and_clean.ipynb`. It starts with `N_GAMES = 20` to test on a few games. Once that works, set it to `None` and run it again for the full season (about 1,230 games, roughly 1 to 1.5 hours because of the pause between requests). If it stops, just rerun it: games already downloaded are skipped.
2. Open `02_baseline_rapm.ipynb` and run it.

The data goes in `data/`, which isn't uploaded to GitHub.

## Data and credit

- Data: NBA.com stats through the [`nba_api`](https://github.com/swar/nba_api) Python package.
- RAPM builds on years of public work in basketball analytics.
- The idea of pricing box-score actions by what they actually do to a possession, instead of fitting their weights to RAPM, comes from Saint's ARC model ([court-share.com/luma](https://court-share.com/luma/arc)).
