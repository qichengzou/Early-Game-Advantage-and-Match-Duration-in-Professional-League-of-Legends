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
The original dataset is structured such that each gameid corresponds to up to 12 rows: one for each of the 10 players (5 per team) and 2 additional rows containing team-level summary statistics. However, these team-level rows only include a limited subset of aggregated features, meaning many detailed statistics (such as player-level gold, vision, and combat metrics) are not fully represented at the team level.

Because of this, I chose to construct my analysis dataset using player rows, which contain more complete information, and then aggregate them into a consistent team-level dataset. In my exploratory data analysis, I will use both the player dataframe and the team dataframe. This is achieved by 
1) Filtering to player rows
   
   Although team rows already exist, they do not contain all relevant features. Player rows provide a more complete view of the game state, making    them a better foundation for building a full dataset.
2) Removing incomplete matches
   
   Incomplete matches may have missing or unreliable early-game statistics, which would negatively affect both hypothesis testing and modeling.
3) Removing redundant opponent columns.

   The dataset includes many columns prefixed with opp_, which duplicate information about the opposing team. Since these columns do not add new independent information and may introduce redundancy, I removed them.
4) Converting the columns which should be of type Boolean

   Certain columns were converted to more appropriate types. For example, the result column was converted from numeric values to Boolean (`True` for win, `False` for loss) to improve interpretability and consistency.
5) Selecting relevant columns

I selected a subset of columns that capture:

- early-game economy (gold, XP, CS)
- combat statistics (kills, deaths, assists)
- tempo and vision (ckpm, vspm)
- player and team identifiers

This ensures the dataset focuses on features relevant to both hypothesis testing and prediction.
6) Constructing a team-level dataset from player rows

Although my final analysis is conducted at the team level, I do not use the original team rows because they are incomplete. Instead, I construct a team-level dataset from player rows.

Within each match, all players on the same team share identical team-level statistics (e.g. gamelength). Therefore, I remove duplicate rows within each (gameid, teamid) pair

This results in one row per team per match, while retaining the full set of features available in the player data.

These cleaning steps ensure that:

- The dataset retains complete and detailed features by starting from player-level data.

- Each row in the modeling dataset (team_df) represents a single team in a match, avoiding duplicated information.

- Redundant and incomplete data are removed, improving model reliability and interpretability.

As a result:

players is used for role-based analysis (e.g., testing whether supports have higher vision activity).

team_df is used for predictive modeling, where each row corresponds to one team’s game state.

> In addition, rows with missing values that are **missing at random (MAR)** conditional on league were removed. A substantial portion of missing data is concentrated within specific leagues (e.g., LDL), making standard imputation methods (such as group-wise mean imputation) inappropriate. Such methods would introduce bias and artificially reduce variance. A more detailed justification is provided in the *Assessment of Missingness* section.

Below is the cleaned teams dataframe, with additional player rows dropped:
| gameid                | league   | teamid                                  |   gamelength | result   |   kills |   deaths |   assists |   teamkills |   teamdeaths |   team kpm |   ckpm |   visionscore |   vspm |   totalgold |   earnedgold |   earned gpm |   earnedgoldshare |   goldspent |   total cs |   minionkills |   monsterkills |   cspm |   goldat10 |   goldat15 |   xpat10 |   xpat15 |   csat10 |   csat15 |   golddiffat10 |   golddiffat15 |   xpdiffat10 |   xpdiffat15 |   csdiffat10 |   csdiffat15 |
|:----------------------|:---------|:----------------------------------------|-------------:|:---------|--------:|---------:|----------:|------------:|-------------:|-----------:|-------:|--------------:|-------:|------------:|-------------:|-------------:|------------------:|------------:|-----------:|--------------:|---------------:|-------:|-----------:|-----------:|---------:|---------:|---------:|---------:|---------------:|---------------:|-------------:|-------------:|-------------:|-------------:|
| ESPORTSTMNT03/1632489 | KeSPA    | oe:team:ef69efa6acebe94107f0cf1ba716806 |         1782 | True     |       7 |        1 |         3 |          23 |            4 |     0.7744 | 0.9091 |            25 | 0.8418 |       12065 |         8154 |      274.546 |          0.196019 |       10875 |        209 |           193 |             16 | 7.037  |       3421 |       5407 |     5043 |     7536 |       73 |      114 |            436 |            748 |          550 |          -56 |            1 |           -4 |
| ESPORTSTMNT03/1632489 | KeSPA    | oe:team:5cd2cd09ec94296f605dd13a2924d6c |         1782 | False    |       0 |        5 |         2 |           4 |           24 |     0.1347 | 0.9091 |            20 | 0.6734 |        8863 |         4952 |      166.734 |          0.197362 |        8525 |        215 |           215 |              0 | 7.2391 |       2985 |       4659 |     4493 |     7592 |       72 |      118 |           -436 |           -748 |         -550 |           56 |           -1 |            4 |
| ESPORTSTMNT03/1632500 | KeSPA    | oe:team:5cd2cd09ec94296f605dd13a2924d6c |         1753 | False    |       0 |        3 |         6 |           7 |           18 |     0.2396 | 0.8557 |            26 | 0.8899 |        9201 |         5349 |      183.08  |          0.191556 |        8650 |        201 |           201 |              0 | 6.8796 |       3183 |       4905 |     4995 |     7319 |       82 |      119 |            335 |            169 |          495 |           80 |           13 |            0 |
| ESPORTSTMNT03/1632500 | KeSPA    | oe:team:ef69efa6acebe94107f0cf1ba716806 |         1753 | True     |       1 |        2 |         8 |          18 |            7 |     0.6161 | 0.8557 |            32 | 1.0953 |        9579 |         5727 |      196.018 |          0.158084 |        9125 |        197 |           193 |              4 | 6.7427 |       2848 |       4736 |     4500 |     7239 |       69 |      119 |           -335 |           -169 |         -495 |          -80 |          -13 |            0 |
| ESPORTSTMNT03/1632502 | KeSPA    | oe:team:5cd2cd09ec94296f605dd13a2924d6c |         1777 | False    |       3 |        3 |         3 |          10 |           20 |     0.3376 | 1.0129 |            20 | 0.6753 |        8821 |         4920 |      166.123 |          0.188789 |        8725 |        153 |           148 |              5 | 5.166  |       3110 |       4898 |     4157 |     6894 |       61 |      112 |            670 |            683 |         1069 |          940 |           13 |           11 |

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
