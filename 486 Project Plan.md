# **STAT 486 Project Plan — NFL Big Data Bowl**

**Date:** October 27, 2026  
**Team:** Rylan Van Komen, Jose De Leon, Nick Austin  
**Repository:** [JDeleon2342/Stat486-Kaggle-Project](https://github.com/JDeleon2342/Stat486-Kaggle-Project) 

## **1\. Project question and purpose**

**Research question:** Can data collected during the NFL Combine help explain or predict future NFL performance, and can tracking data reveal useful insights beyond traditional Combine measurements?

## **2\. Chosen route**

**Route B: Kaggle competition with two meaningfully different pipelines**, subject to confirming that the competition rules and course requirements support this approach.

This route fits our question because we can compare a conventional prediction approach with a sensor-enhanced approach using the same underlying dataset. This will allow us to investigate whether the additional tracking information provides measurable predictive value.

## **3\. Data source and access**

Our primary data source is the NFL Big Data Bowl dataset described in the competition materials. It contains nine CSV files totaling approximately 2.2 GiB and covers 510 rookie prospects drafted or signed between 2023 and 2025\.

The main files for our initial analysis will be:

* `players.csv`: player identifiers, draft year, position, college, and draft information.  
* `combine_results.csv`: traditional Combine measurements and Next Gen Stats scores.  
* `combine_tracking.csv`: 10 Hz tracking data from Combine drills, including position, speed, acceleration, distance traveled, and movement direction.  
* `player_career_successes.csv`: career participation measures, including games active, games started, snap counts, and accolades.  
* `player_play.csv`: play-level statistics and performance measures that may support additional analyses.

The `nfl_id` identifier will be used to connect player-level information across the relevant files.

We will obtain the data through the authorized competition download process and inspect the competition's license, rules, and redistribution restrictions before sharing any data. We will begin with a subset of the tracking data to manage memory and computation.

## **4\. Route-specific plan: Kaggle competition**

### **Pipeline 1: Traditional-metrics baseline**

Our first pipeline will use conventional Combine measurements and relevant player characteristics to predict a selected measure of early NFL participation.

Potential predictors include height, weight, 40-yard dash time, vertical jump, broad jump, agility drill results, and position. We will begin with a simple model, such as linear regression for a continuous outcome or logistic regression for a binary outcome.

### **Pipeline 2: Sensor-enhanced model**

Our second pipeline will add features derived from the Combine tracking data to the baseline predictors.

Potential features include peak speed, peak acceleration, changes in speed throughout a drill, acceleration patterns, and measures of movement consistency. We will summarize tracking frames and attempts at the player level rather than treating each frame as an independent player observation.

We will begin with a manageable supervised model and investigate whether the additional features improve predictions. The specific model and feature set will be selected after initial data exploration.

## **5\. Initial reference and proposed contribution**

Our initial reference will be a model using traditional Combine measurements and player characteristics. Our proposed contribution will be a reproducible comparison of that model with one incorporating sensor-derived features.

Candidate outcomes include games active, games started, and snap counts. We will choose an outcome after examining missingness, variability, and comparability across draft years and positions.

Because players entered the NFL in different years, raw career totals may not be directly comparable. We will consider restricting the analysis to comparable observation windows or using an outcome that accounts for the amount of time each player has had to accumulate NFL experience.

We will document any external code, packages, modeling approaches, and data transformations used in the project.

## **6\. Evaluation and evidence**

We will split the data at the player level so that a player's information cannot appear in both training and validation sets. We will use the same split and evaluation metric for both pipelines.

For a continuous outcome, we will consider root mean squared error (RMSE) or mean absolute error (MAE). For a binary outcome, we will consider an appropriate classification metric, such as ROC-AUC, alongside measures that reflect the balance of the outcome classes.

We will compare the two pipelines to determine whether sensor-derived features improve predictions on held-out players. We will also inspect feature importance or other interpretable summaries to identify which measurements appear most informative.

Evidence against our initial expectation would include little or no improvement from the sensor-enhanced model, unstable results across validation splits, or apparent improvements that disappear when differences in player exposure are addressed.

Potential outputs include a comparison table of validation metrics, plots of predicted versus observed outcomes, and summaries of the most informative traditional and sensor-derived features.

## **7\. Risks and backup scope**

The main risks include the size of the tracking dataset, missing measurements, the limited number of players, and differences in career exposure across draft years. There may also be limitations on the use of competition data or the submission of results.

If processing the full tracking data proves too demanding, we will focus on a subset of relevant drills or a smaller set of engineered features. If the sample is too small for reliable position-specific modeling, we will use a broader model and treat position-based comparisons as exploratory.

## **8\. Reproducibility and team contributions**

Code, configuration files, data instructions, and results will be organized in the team's GitHub repository. We will document data sources, processing steps, model settings, validation procedures, and results so another person can reproduce the analysis.

Our first runnable result will load the player and Combine results tables, check their identifiers and missing values, and produce a baseline model using a small set of traditional measurements.

Team members will divide responsibilities among data exploration, tracking feature engineering, model development, evaluation, and documentation. Each member will review teammates' work and help verify the reproducibility of the results.

## **9\. Initial sources**

1. **NFL Big Data Bowl competition materials and dataset description:** [NFL Big Data Bowl 2027 \| Kaggle](https://www.kaggle.com/competitions/nfl-big-data-bowl-2027/data) Primary source for the data, competition objectives, download instructions, and rules.  
2. **NFL Combine tracking data:** `combine_tracking.csv`. Source for high-frequency player movement features.  
3. **NFL Combine results:** `combine_results.csv`. Source for conventional Combine measurements and prospect scores.  
4. **NFL performance data:** `player_career_successes.csv` and `player_play.csv`. Sources for subsequent participation and on-field performance outcomes.



# Data and Reference Result

**Checkpoint:** November 5, 2026, by 11:59 PM Mountain Time  
**Team:** Jose De Leon, Rylan Van Komen, Nick Austin  
**Date:** November 6, 2026

## Data

Our project uses NFL Combine and player career data obtained from the NFL Big Data Bowl dataset on Kaggle. We downloaded four relevant CSV files and organized them in a local `data` directory for analysis in Jupyter Notebook.

The datasets currently available are:

| Dataset | Observations | Variables | Description |
|---|---:|---:|---|
| `combine_results.csv` | 510 | 18 | Traditional NFL Combine measurements and NGS scores |
| `combine_tracking.csv` | 463,189 | 14 | Player movement and sensor measurements during Combine drills |
| `player_career_successes.csv` | 510 | 9 | Career participation and achievement statistics |
| `player_play.csv` | 314,197 | 64 | Player participation and performance information during individual plays |

All 510 players in the Combine Results dataset are also represented in the Combine Tracking and Career Successes datasets. Player Play contains 497 of those 510 players.

Initial inspection identified missing values in several traditional Combine measurements, particularly the three-cone drill (330 missing), bench press (318 missing), and short shuttle (309 missing). The tracking and career datasets contained no missing values during our initial inspection. Player Play contains substantial missingness, likely because many variables apply only to specific positions or play situations.

The tracking dataset includes nine drill types, including the 40-yard dash, three-cone drill, and short shuttle. It records player position, speed, acceleration, distance, and direction at individual time intervals. Tracking coverage varies by drill, with the 40-yard dash providing the largest comparable sample between traditional and sensor-based measurements (418 players).

Remaining preparation includes engineering player-level movement features from the tracking observations, determining how to handle missing Combine measurements, and selecting an appropriate career outcome for prediction.

## Evaluation Plan

We plan to compare two modeling pipelines:

1. **Traditional Combine model:** Uses conventional Combine measurements such as height, weight, 40-yard dash time, vertical jump, and other available measurements.
2. **Combined model:** Uses traditional Combine measurements along with engineered features from sensor-based tracking data, such as maximum speed, average speed, acceleration, and movement characteristics.

Both pipelines will use the same players and the same training and testing split to ensure a fair comparison. We will split data by player so that observations from the same player cannot appear in both training and testing sets.

We will also account for missing measurements and differences in career exposure across draft years. All preprocessing and feature selection will be fitted using training data only to prevent information leakage.

For a continuous career outcome, we will consider regression metrics such as mean absolute error (MAE) and root mean squared error (RMSE). We will also examine whether the additional tracking features meaningfully improve predictive performance over the traditional Combine model.

## First Runnable Result

We have successfully loaded and inspected all four datasets in a local Jupyter Notebook. Our initial runnable results include dataset dimensions, missing-value summaries, player ID overlap, and tracking drill coverage.

No predictive baseline has been run yet. Our next step is to engineer player-level tracking features and establish a traditional Combine baseline before comparing it with the combined model.
