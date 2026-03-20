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

The original dataset is structured such that each `gameid` corresponds to up to 12 rows: one for each of the 10 players (5 per team) and 2 additional rows containing team-level summary statistics. However, these team-level rows contain only a limited subset of aggregated features, meaning many detailed statistics (such as gold, vision, and combat metrics) are not fully represented.

To retain the most complete information, I construct the analysis dataset starting from player-level rows, and construct a team-level dataset. Throughout the analysis, I work with two datasets:

- `players`: player-level data for role-based analysis  
- `team_df`: aggregated team-level data for modeling gamelength 

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

I only include columns that represents in-game combat and farming statistics such as columns involving gold, xp, creep score, kills, deaths, and assists. These features reflects the state of the game instead of the strategics such as champion selection.

---

### Summary of Cleaning Decisions

These steps ensure that:

* The dataset retains **complete and detailed features** by starting from player-level data
* Each row in `team_df` represents **one team per match**, avoiding duplication
* Redundant and incomplete data are removed, improving **model reliability and interpretability**

As a result:

* `players` is used for **role-based analysis** (e.g., comparing vision activity across positions)
* `team_df` is used for **predictive modeling**, where each row corresponds to a team’s game state

> In addition, rows with missing values that are **missing at random (MAR)** conditional on league were removed. A substantial portion of missing data is concentrated within specific leagues (e.g.LPL, LDL), and a large portion of data within these leagues is missing, making standard imputation methods (such as group-wise mean imputation) inappropriate. Such methods would introduce bias and artificially reduce variance. A more detailed justification is provided in the *Assessment of Missingness* section.

Below is a representative subset of the cleaned team-level dataframe. While the full dataset contains additional features, the columns shown here capture key aspects of early-game performance and overall team combat dynamics:

|   gamelength | result   |   goldat10 |   xpat10 |   csat10 |   teamkills |   teamdeaths |   team kpm |   ckpm |
|-------------:|:---------|-----------:|---------:|---------:|------------:|-------------:|-----------:|-------:|
|         1782 | True     |      16291 |    20459 |      382 |          23 |            4 |     0.7744 | 0.9091 |
|         1782 | False    |      14498 |    18094 |      317 |           4 |           24 |     0.1347 | 0.9091 |
|         1753 | False    |      15623 |    19210 |      340 |           7 |           18 |     0.2396 | 0.8557 |
|         1753 | True     |      14864 |    19039 |      348 |          18 |            7 |     0.6161 | 0.8557 |
|         1777 | False    |      15043 |    16987 |      275 |          10 |           20 |     0.3376 | 1.0129 |

## Univariate Analysis
### Team Kills Per Minute (KPM)
We begin by examining the distributions of key variables related to combat performance and map control, which are central to understanding team dynamics.
<div style="text-align: center;">
<iframe
  src="assets/team-kpm.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>
</div>
The distribution of team kills per minute (`team_kpm`) is approximately unimodal and slightly right-skewed. Most games cluster around a moderate kill rate, with a smaller number of high-tempo games producing unusually high values.

This suggests that while most matches follow a relatively stable pace, there exists a subset of more aggressive games with significantly higher combat intensity.

### Vision Score Per Minute (VSPM)
We also examine the distribution of Vision Score Per Minute (vspm). VSPM measures how much vision a team provides or denies through warding, normalized per minute of gameplay. Higher values indicate stronger map control and information advantage.
<div style="text-align: center;">
<iframe
  src="assets/vspm.html"
  width="600"
  height="450"
  frameborder="0"
></iframe>
</div>
The distribution of vspm appears approximately symmetric and close to normal, with most teams falling within a relatively narrow range. This suggests that vision control is more consistent across games compared to combat metrics like KPM.

## Bivariate Analysis

To better understand the relationships between key variables in the dataset, we examine several bivariate plots focusing on game tempo, early-game advantage, and match outcomes.

### Game Length vs Combat Intensity (CKPM)

<div style="text-align: center;">
<iframe
  src="assets/ckpm-gamelength.html"
  width="600"
  height="450"
  frameborder="0"
  style="display: block; margin: 0 auto;">
</iframe>
</div>

This plot shows a negative relationship between combined kills per minute (CKPM) and game length. Games with higher combat intensity tend to end faster, suggesting that aggressive gameplay leads to quicker match resolution.

### Vision Score per Minute by Position
<div style="text-align: center;">
<iframe
  src="assets/vspm_by_position.html"
  width="800"
  height="600"
  frameborder="0"
  style="display: block; margin: 0 auto;">
</iframe>
</div>

Support players exhibit significantly higher vision score per minute compared to other roles, reflecting their primary responsibility for map control. Other positions show lower and more similar distributions.

Overall, these relationships suggest that both early-game advantage and combat intensity play important roles in determining match duration and outcomes. Teams that gain early leads or engage in high-tempo gameplay tend to close out games more quickly.

## Interesting Aggregates
To further explore patterns in the data, we examine several aggregate statistics that summarize how key features relate to game outcomes and duration.

### Vision Control Across Roles
Using the player-level dataset, we compute average vision score per minute (VSPM) for each position. Support players consistently have the highest VSPM, followed by junglers, while carries (mid, ADC, top) have lower values.

| position   |    False |    True |
|:-----------|---------:|--------:|
| bot        | 1.07065  | 1.28285 |
| jng        | 1.26212  | 1.4344  |
| mid        | 0.952555 | 1.10592 |
| sup        | 2.3897   | 2.58774 |
| top        | 0.910576 | 1.02063 |

This reflects the strategic division of responsibilities, where supports and junglers prioritize vision and map control, while carries focus more on farming and damage output.

---

# Assessment of Missingness

## MNAR Analysis

A column in this dataset that may exhibit **Missing Not At Random (MNAR)** behavior is `playerid`.

The column `playerid` contains missing values, and unlike gameplay statistics, its missingness is likely tied to the identity and prominence of the player itself. In particular, missing `playerid` values may occur more frequently for less-documented players, substitute players, or players from lower-tier leagues where record-keeping is less standardized. This suggests that the probability of `playerid` being missing depends on the underlying (unobserved) identity of the player.

## Missingness Dependency

To better understand the mechanism behind missing values in the dataset, I focus on the column `visionscore`, which has the highest proportion of missingness (~9.3%). In this section I will test the missingness dependensy of `visionscore` against `league` and  `side`.

### Dependency on `League` (Dependent Case)

I  test whether missingness depends on the categorical variable `league`. Notably, all rows with missing `visionscore` values come from only two leagues: **LPL and LDL**.

Below is a graph of distribution of missingness across leagues:

<div style="text-align: center;">
<iframe
  src="assets/missing_by_league.html"
  width="700"
  height="450"
  frameborder="0"
  style="display: block; margin: 0 auto;">
</iframe>
</div>

#### Hypotheses

- **Null Hypothesis (H₀):** The missingness of `visionscore` does not depend on league.
- **Alternative Hypothesis (H₁):** The missingness of `visionscore` does depend on league.
- **Test Statistic:** I use **Total Variation Distance (TVD)** to measure the difference in the distribution of missing vs. non-missing values between:
  - LPL/LDL leagues
  - All other leagues

#### Results

Here's the distribution of simulated TVDs:

<div style="text-align: center;">
<iframe
  src="assets/tvds_dist.html"
  width="700"
  height="450"
  frameborder="0"
  style="display: block; margin: 0 auto;">
</iframe>
</div>

- Observed TVD: **0.96**
- p-value: **≈ 0.0**

Since the p-value is extremely small, I reject the null hypothesis. This provides strong evidence that the missingness of `visionscore` **depends on league**.

This suggests that missingness is driven by **league-specific data collection practices**, rather than random variation.

### Dependency on Side (Non-Dependent Case)

Next, I test whether missingness depends on the variable `side` (Blue vs. Red team).

#### Hypotheses

- **Null Hypothesis (H₀):** The missingness of `visionscore` does not depend on side.
- **Alternative Hypothesis (H₁):** The missingness of `visionscore` does depend on side.

#### Test Statistic

I use the **absolute difference in proportions** of missing values between Blue and Red sides.

#### Results

- Observed statistic: **0.0**
- p-value: **1.0**

Since the observed statistic is exactly zero and the p-value is large, I fail to reject the null hypothesis. This indicates that missingness of `visionscore` **does not depend on side**.

---

## Hypothesis Testing

To better understand role-based differences in gameplay, I investigate whether support players contribute more to vision control than other roles.

### Question

Do support players have higher Vision Score Per Minute (`vspm`) than non-support players?

### Hypotheses

- **Null Hypothesis:**  
  The average `vspm` for support players is equal to the average `vspm` for non-support players.

- **Alternative Hypothesis:**  
  The average `vspm` for support players is **greater than** the average `vspm` for non-support players.

- **Significance Level:** 5%  

This is a **one-sided test**, since we specifically expect supports to provide more vision due to their role responsibilities.


### Test Statistic

I use the **difference in means**:

$\text{Mean VSPM (Support)} - \text{Mean VSPM (Non-Support)}$

This statistic directly measures how much more vision support players contribute on average.

### Methodology

I perform a **permutation test**:

- Shuffle the `is_support` labels
- Recompute the difference in means
- Repeat this process to build a null distribution

A permutation test is appropriate because:
- The distribution of `vspm` is not guaranteed to be normal
- It makes minimal assumptions about the data
- It directly tests whether group labels matter

### Results

- Observed difference: **≈ 1.36**
- p-value: **≈ 0.0**

Since the p-value is far below the significance level of **α = 0.05**, I reject the null hypothesis.

Below is a graph of distribution of simulated test statistics under null assumption:

<iframe src="assets/vspm_permutation" width="800" height="600" frameborder="0"></iframe>

The observed statistic lies far in the right tail of the permutation distribution, indicating that such a large difference is extremely unlikely under the null hypothesis.

### Hypothesis Test Conclusion

There is strong statistical evidence that support players have higher `vspm` than non-support players.

This aligns with domain knowledge: support players are responsible for warding and vision control, so their higher vision scores are expected. The result confirms that the dataset reflects meaningful role-based differences in gameplay behavior.

---

## Prediction Problem

The goal of this project is to **predict the final game length (`gamelength`) of a professional League of Legends match using early-game information**. Specifically, I use features available at 15 minutes into the game, such as gold, experience, creep score, and combat statistics, to estimate how long the match will ultimately last.

This is a **regression problem**, since the response variable, `gamelength`, is a continuous numerical value measured in seconds.

### Response Variable

The response variable is:

- **`gamelength`**: the total duration of a match

I chose this variable because game length is a fundamental measure of match dynamics and tempo. Understanding how early-game advantages influence match duration can provide insights into how quickly teams convert leads into victories, which is central to my project’s theme of early-game advantage.

### Features and Time-of-Prediction Justification

At the **time of prediction**, I assume that only **early-game statistics (up to 15 minutes)** are available. Therefore, I restrict my model to use features such as:

- `goldat15`, `xpat15`, `csat15`
- `golddiffat15`, `xpdiffat15`, `csdiffat15`
- early combat statistics (e.g., kills and deaths at 15 minutes)
- tempo-related features (e.g., `ckpm`, `vspm`)

I explicitly avoid using any features that depend on later stages of the game (such as gold at 20 or 25 minutes), since those would not be available at the time the prediction is made. This ensures that the model reflects a realistic prediction scenario and avoids data leakage.

### Evaluation Metric

I use **Root Mean Squared Error (RMSE)** as the evaluation metric.

RMSE is appropriate because:

- It is designed for regression problems with continuous targets
- It measures prediction error in the same units as the response variable (seconds)
- It penalizes larger errors more heavily, which is important since large mistakes in predicting game duration are more impactful than small ones

I use RMSE instead of $R^2$ because RMSE measures prediction error in the same units as the response variable (seconds), making it directly interpretable. Since the goal of this model is to accurately predict game duration, understanding the magnitude of prediction errors is more important than measuring relative variance explained.

---

## Baseline Model

### Model Description

For the baseline model, I use a **linear regression model** to predict game length (`gamelength`) from a small set of early-game features. The goal of this baseline is to establish a simple and interpretable reference point before building a more complex model.

### Features Used

The baseline model uses the following features:

- **`goldat15` (quantitative):** total team gold at 15 minutes  
- **`golddiffat15` (quantitative):** gold difference between teams at 15 minutes  
- **`firstPick` (nominal):** whether the team had first pick in champion selection  

These features were chosen because they capture early-game economy and draft advantage, which are likely to influence how quickly a game ends.

### Feature Transformations and Encoding

To prepare the data for modeling, I apply the following preprocessing steps:

- **Numerical features (`goldat15`, `golddiffat15`):**
  - Missing values are imputed using the mean
  - Features are standardized using `StandardScaler` to ensure they are on comparable scales

- **Categorical feature (`firstPick`):**
  - Encoded using **One-Hot Encoding** since it is a nominal variable with no inherent ordering and drop one column to avoid *multicollinearity*.

Additionally, I use grouped train-test splitting by game ID to prevent data leakage between teams from the same match.

### Model Performance

I evaluate the model using **Root Mean Squared Error (RMSE)**.

- **Training RMSE:** 325.943  
- **Test RMSE:** 311.449  

The relatively similar training and test RMSE values suggest that the model is not significantly overfitting and is able to generalize reasonably well to unseen data.

### Evaluation and Limitations

While the baseline model provides a reasonable starting point, its performance is limited. An RMSE of approximately 300 seconds (around 5 minutes) indicates that predictions can deviate substantially from actual game lengths.

This is expected, since the model:
- uses only a small number of features
- assumes a strictly linear relationship between features and the response
- does not capture more complex interactions or nonlinear effects present in the data

As a result, this baseline model is **not “good” in terms of predictive accuracy**, but it serves as an important reference point for evaluating improvements in the final model.

---

## Final Model

### Overview

To improve upon the baseline model, I developed a more expressive regression model that incorporates additional features capturing early-game dynamics, nonlinear relationships, and feature scaling.

### Feature Engineering

In addition to the baseline features, I engineered several new features to better reflect the structure of the game:

- **`abs_golddiffat15`, `abs_xpdiffat15`, `abs_csdiffat15` (quantitative):**
  
  These features measure the **magnitude of imbalance** between the two teams at 15 minutes. Since each match appears twice in the dataset (once per team), one row has a positive difference and the other a negative difference. Taking the absolute value ensures both rows reflect the same underlying game state (e.g. a 1700 gold lead), preventing the model from treating identical situations differently due to team perspective.

- **`combat_aggression15` (quantitative):**
  
  This feature captures **early-game combat intensity**, computed from kills and deaths at 15 minutes. Games with higher combat activity may progress faster due to increased skirmishing and objective pressure, making this a relevant predictor of game duration.

- **Polynomial features on `goldat15`, `xpat15`, `csat15`:**
  
  I apply polynomial transformations to these early-game economy variables to capture **nonlinear relationships**. For example, small differences in resources may not significantly affect game length, while large leads may accelerate game-ending conditions.

- **Log transformation of `ckpm` , `gold_advantage_ratio`, and `combat_aggression15`(quantitative):**
  
  These features are right-skewed, so I apply a log transformation to stabilize variance and reduce the influence of extreme values, improving model robustness.

- **Scaled linear features (`vspm` and engineered imbalance features):**  
  These features are standardized to ensure they contribute comparably to the model.

These feature choices are motivated by the data generating process: early-game resource differences, combat intensity, and tempo are key drivers of how quickly a team can close out a game.

### Model and Hyperparameter Tuning

To increase model flexibility, I tune the **degree of polynomial features** applied to `goldat15`, `xpat15`, and `csat15`. I consider degrees from 1 to 5 and use **k fold cross-validation**.

The best-performing hyperparameter was:

- **Polynomial degree = 2**

### Model Performance

I evaluate the final model using RMSE on both training and test data:

- **Final Model Test RMSE:** 261.55  
- **Final Model Training RMSE:** 264.73  

The final model achieves a substantial reduction in RMSE (approximately 50 seconds), indicating improved predictive accuracy.

Additionally, the training and test RMSE values are similar, suggesting that the model generalizes well and does not significantly overfit the training data.

---
## Fairness Analysis

### Group Definition

To evaluate fairness, I compare model performance between:

- **Group X (Ahead teams):** teams with `golddiffat15 > 0`  
- **Group Y (Behind teams):** teams with `golddiffat15 ≤ 0`  

This grouping is meaningful because it reflects early-game advantage. Importantly, the final model uses **absolute-value features** (e.g., `abs_golddiffat15`), which remove information about which team is ahead. This raises a natural fairness question:

> Does ignoring direction cause the model to perform worse for teams that are behind?

### Evaluation Metric

Since this is a regression task, I use **Root Mean Squared Error (RMSE)** to measure model performance. RMSE captures the magnitude of prediction errors in seconds and allows for direct comparison between groups.

### Hypotheses

- **Null Hypothesis:**  
  The model is fair. RMSE is the same for ahead and behind teams, and any observed difference is due to random chance.

- **Alternative Hypothesis:**  
  The model is unfair. RMSE is higher for **behind teams** than for ahead teams.

- **Significance Level:** 5%

### Test Statistic

The test statistic is the **difference in RMSE** between the two groups:

$\text{RMSE}_{\text{behind}} - \text{RMSE}_{\text{ahead}}$

A positive value indicates worse performance on behind teams.

### Results

- Observed test statistic: -2.728
- p-value: **0.656**

Below is a figure of distribution of simulated test statistics under the null hypothesis with observed statistic.

<div style="text-align: center;">
<iframe
  src="assets/fairness_permutation.html"
  width="800"
  height="600"
  frameborder="0"
  style="display: block; margin: 0 auto;">
</iframe>
</div>

### Conclusion

Since the p-value is greater than 0.05, I fail to reject the null hypothesis.

This suggests that there is **no statistically significant evidence** that the model performs worse for teams that are behind at 15 minutes compared to teams that are ahead.


### Interpretation

This result is particularly important given the model design. Although the model removes directional information by using absolute-value features, it does **not introduce detectable bias** between ahead and behind teams.

This indicates that the **magnitude of early-game advantage** is sufficient for predicting game duration, and that ignoring direction does not harm model fairness.




