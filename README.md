# Early Game Advantage and Match Duration in Professional League of Legends
Final Project for DSC 80 at UCSD

by: Qicheng Zou

---
# Introduction
## Context of the Dataset
League of Legends (LOL) is a popular multiplayer online battle arena (MOBA) game developed by Riot Games, and a blueprint for many MOBA games today. This project is working on 2021 professional League of Legends games data recorded and maintained by[Oracle's Elixir](https://oracleselixir.com/tools/downloads) which is interesting because it captures not just who won, but how a game developed over time, making it well-suited for studying early-game advantage, match tempo, and predictive modeling.

This project is centered around the following question:
**Can we predict the length of a professional League of Legends match using statistics available at 15 minutes?**

This question matters because game length reflects the pace and competitiveness of a match. In professional play, some games end quickly after one team builds a decisive early lead, while others remain close and extend much longer. Understanding which early-game signals are associated with shorter or longer matches can help reveal how teams convert advantages into wins and which aspects of the game state are most informative for forecasting match flow.

## Introduction to the Dataset
The original dataset contains **148464** rows of match data collected from professional League of Legends tournaments. Each match contains multiple rows describing individual players and team-level summaries. Since each gameid includes player rows and two team-summary rows, I focus on player-level rows for exploratory data analysis on different game statistics and the team-level rows for modeling so that each observation represents one team’s game state within a match.

The columns most relevant to the analysis include:

| Column         | Description                                                                             |
| -------------- | --------------------------------------------------------------------------------------- |
| `gameid`       | Unique identifier for each match.                                                       |
| `position`     | The role played by a player (e.g., top, jungle, mid, bot, support).                     |
| `gamelength`   | The total duration of the match; this is the response variable in the prediction model. |
| `goldat15`     | Total team gold at 15 minutes.                                                          |
| `xpat15`       | Total team experience at 15 minutes.                                                    |
| `csat15`       | Total creep score at 15 minutes.                                                        |
| `golddiffat15` | The team’s gold advantage or deficit relative to the opposing team at 15 minutes.       |
| `xpdiffat15`   | Experience advantage or deficit relative to the opposing team at 15 minutes.            |
| `csdiffat15`   | Creep score advantage or deficit relative to the opposing team at 15 minutes.           |
| `ckpm`         | Combined kills per minute, measuring the overall combat tempo of the game.              |
| `vspm`         | Vision score per minute, measuring how actively a player contributes to map vision.     |
| `killsat15`    | Total number of kills achieved by the team at 15 minutes.                               |
| `assistsat15`  | Total number of assists recorded by the team at 15 minutes.                             |
| `deathsat15`   | Total number of deaths suffered by the team at 15 minutes.                              |

---
# Data Cleaning and Exploratory Data Analysis
## Data Cleaning

The original dataset is structured such that each `gameid` corresponds to up to 12 rows: one for each of the 10 players (5 per team) and 2 additional rows containing team-level summary statistics. However, these team-level rows only include a limited subset of aggregated features, meaning many detailed statistics (such as player-level gold, vision, and combat metrics) are not fully represented.

Because of this, I construct the analysis dataset using **player-level rows**, which contain more complete information, and then aggregate them into a consistent team-level dataset. Throughout the exploratory data analysis, both the player dataframe (`players`) and the aggregated team dataframe (`team_df`) are used.

The data cleaning process consists of the following steps:

### 1. Filtering to Player Rows

Although team rows already exist, they do not contain all relevant features. Player rows provide a more complete view of the game state and therefore serve as a better foundation for building a full dataset.

### 2. Removing Incomplete Matches

Incomplete matches may contain missing or unreliable early-game statistics. These were removed to avoid introducing bias into hypothesis testing and predictive modeling.

### 3. Removing Redundant Opponent Columns

The dataset includes many columns prefixed with `opp_`, which duplicate information about the opposing team. Since these columns do not provide new independent information and may introduce redundancy, they were removed.

### 4. Converting Columns to Appropriate Data Types

Certain columns were converted to more appropriate types. For example, the `result` column was converted from numeric values to Boolean (`True` for win, `False` for loss) to improve interpretability and consistency.

### 5. Selecting Relevant Columns

A subset of columns was selected to focus the analysis on key aspects of gameplay:

* **Early-game economy**: `goldat10`, `xpat10`, `csat10`
* **Combat statistics**: `kills`, `deaths`, `assists`
* **Tempo and vision**: `ckpm`, `vspm`
* **Identifiers**: player and team identifiers

This ensures the dataset is aligned with both hypothesis testing and predictive modeling goals.

### 6. Constructing a Team-Level Dataset

Although the final analysis is conducted at the team level, the original team rows are not used due to missing features. Instead, a team-level dataset is constructed from player rows.

Within each match, all players on the same team share identical team-level statistics (e.g., `gamelength`). Therefore, duplicate rows within each (`gameid`, `teamid`) pair are removed.

This results in **one row per team per match**, while retaining the full set of features derived from player-level data.

---

### Summary of Cleaning Decisions

These steps ensure that:

* The dataset retains **complete and detailed features** by starting from player-level data
* Each row in `team_df` represents **one team per match**, avoiding duplication
* Redundant and incomplete data are removed, improving **model reliability and interpretability**

As a result:

* `players` is used for **role-based analysis** (e.g., comparing vision activity across positions)
* `team_df` is used for **predictive modeling**, where each row corresponds to a team’s game state

---

> In addition, rows with missing values that are **missing at random (MAR)** conditional on league were removed. A substantial portion of missing data is concentrated within specific leagues (e.g., LDL), making standard imputation methods (such as group-wise mean imputation) inappropriate. Such methods would introduce bias and artificially reduce variance. A more detailed justification is provided in the *Assessment of Missingness* section.

Below is a representative subset of the cleaned team-level dataframe. While the full dataset contains additional features, the columns shown here capture key aspects of early-game performance and overall team combat dynamics:

|   gamelength | result   |   goldat10 |   xpat10 |   csat10 |   teamkills |   teamdeaths |   team kpm |   ckpm |
|-------------:|:---------|-----------:|---------:|---------:|------------:|-------------:|-----------:|-------:|
|         1782 | True     |       3421 |     5043 |       73 |          23 |            4 |     0.7744 | 0.9091 |
|         1782 | False    |       2985 |     4493 |       72 |           4 |           24 |     0.1347 | 0.9091 |
|         1753 | False    |       3183 |     4995 |       82 |           7 |           18 |     0.2396 | 0.8557 |
|         1753 | True     |       2848 |     4500 |       69 |          18 |            7 |     0.6161 | 0.8557 |
|         1777 | False    |       3110 |     4157 |       61 |          10 |           20 |     0.3376 | 1.0129 |

## Univariate Analysis
### Team Kills Per Minute (KPM)
We begin by examining the distributions of key variables related to combat performance and map control, which are central to understanding team dynamics.

<iframe
  src="assets/team-kpm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of team kills per minute (`team_kpm`) is approximately unimodal and slightly right-skewed. Most games cluster around a moderate kill rate, with a smaller number of high-tempo games producing unusually high values.

This suggests that while most matches follow a relatively stable pace, there exists a subset of more aggressive games with significantly higher combat intensity.

### Vision Score Per Minute (VSPM)
We also examine the distribution of Vision Score Per Minute (vspm). VSPM measures how much vision a team provides or denies through warding, normalized per minute of gameplay. Higher values indicate stronger map control and information advantage.

<iframe
  src="assets/vspm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of vspm appears approximately symmetric and close to normal, with most teams falling within a relatively narrow range. This suggests that vision control is more consistent across games compared to combat metrics like KPM.

## Bivariate Analysis
## Interesting Aggregates
