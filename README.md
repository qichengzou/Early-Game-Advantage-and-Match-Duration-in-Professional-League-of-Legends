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
The original dataset contains **148464** rows. Since each gameid includes player rows and two team-summary rows, I focus on player-level rows for exploratory data analysis on different game statistics and the team-level rows for modeling so that each observation represents one team’s game state within a match.

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
