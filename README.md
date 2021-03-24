# Capstone-NBA
General Assembly Capstone project

Using player form to predict NBA games between 2000 and 2020

This repository contains notebooks for my Capstone project completed during General Assembly's Data Science Immersive course.

# Introduction and aims:

The NBA is the most prestigous league in the world for men's basketball. This project aims to use player form data to try and predict home-wins. There are multiple steps to this project:
### Data collection using API's:
- Collect data from [BasketballReference](https://www.basketball-reference.com/)
- Collect data from the official NBA web-page
### Data merging
- Create an SQL database for easy merging of data
### Data cleaning
- Adjust player keys so that basketball-reference and NBA data have the same keys
- Make decisions about using outlier data, including player performances where the player played less than 5 minutes
- Make decisions about features to use from the multiple datasets
- Create observations for each player performance
### Feature Engineering
- Some advanced metrics are made of formulas of other counting stats. Adjust relevant features to account for dividing by 0.
- Explore clusters of players based on per 40 min player performance means to create new categorical variable
- Create rolling means for counting statistics to create modelling features
- Apply feature engineering such as John Hollinger's Game_score to rolling mean data
### Modelling
- Use a selection of models treating the problem both as a classification problem (predicting home wins) and a regression problem (predicting the point differential)

# Overview and findings

First I collected the data and created a database, then doing some minor data cleaning (i.e. if a player did not take a shot then a succesful shooting percentage of 0% should actually be a NaN value & converting imperial measurements to metric etc...).

### Heatmaps (time adjusted)
Using time adjusted player performances and adjusting co-dependent features accordingly (Adjusted image below-right) for the rest of the process
Unadjusted             |  Adjusted
:-------------------------:|:-------------------------:
![](/Pt4_Modelling/Images/heatmap_unadjusted.png)  |  ![](/Pt4_Modelling/Images/heatmap_adjusted.png)


### Clustering

Using the time adjusted data, I wanted to have a look at some clustering to try and identify differing quality players. The best silhouette score using K_means <br/>For n_clusters = 6 the average silhouette_score is: ~0.1714
![](/Pt4_Modelling/Images/kmeans_silhouette_6.png)
Starting positions            |  Clustered labels
:-------------------------:|:-------------------------:
![](/Pt4_Modelling/Images/clustering_played_pos.png)  |  ![](/Pt4_Modelling/Images/clustering_display.png)

##### Looking at the 'SuperStar' grouping (pink in Clustered labels image above).

|     |     |
| --- | --- |
| <img src="https://user-images.githubusercontent.com/67226231/111352806-81ff7000-867c-11eb-9ccb-572553b72a69.png" width="900"/> | Note that none of these players have played a game from the bench (in the past 20 years). <br><br> 3 players who have won an MVP in the last 20 years who are not in this list are semi-traditional bigmen: Tim Duncan, Dirk Nowitzki and Kevin Garnett <br><br> Alot of young player's currently playing in the league already showing up in this bracket: 'Ja Morant', 'Luka Doncic', 'Trae Young' ... <br><br> Nikola Jokic a 2021 MVP front runner is one of only two bigmen in this clustering. |

### Structuring and feature selection

Firstly it has to be decided what time span and method will be used to create the form used for the predictor variables. The image below shows Kawhi Leonard's points per game in orange, and in blue the exponentialy weighted means over a span of sixteen games. The right image shows the average time played for all players at the Toronto Raptor's during their Championship winning season, this is important to decide which players should be used for each games performance.
Kawhi Leonard's Points           |  Toronto minutes played
:-------------------------:|:-------------------------:
![](/Pt3_Data_processing/images/ewm16_pts_Kawhi.png)  |  ![](/Pt3_Data_processing/images/ewm16_min_TOR.png)

Finding the optimum timespan to create moving averages for our predictor variable is tricky, after a few tests, the best performing timeframes that I tested where around 16-20 games, using the exponentially weighted means. Other spans and methods could be tested in order to try and optimise.

##### Workflow :
1. Adjust for time, remove performances with less than 5 minutes of game time (this means we expect to know if a player will not be playing this night through injury or any other method
2. Group by player
3. Apply chosen timespan and method (using a minimum span of 2 in order to reduce the amount of games that we would loose if a player was starting their first game in the league)
4. Apply Polynomial features (only including strictly positive features)
5. Structure data by position: using starting position for the game (a limitation of this method), and for the 3 other players choosing the players that played the most amount of time and then ordering them by height to try and represent likely positions in the team)
6. Group by game: organise the teams into home then away order

##### Decide not to include post COVID data 'the bubble':
|     |     |
| --- | --- |
| <img src="https://raw.githubusercontent.com/finjammin/Capstone-NBA/main/Pt3_Data_processing/images/covid_anomaly.png" width="600"/>  |  The change in wins is pretty drastic compared to normal. We can see a small trend that home court advantage appears to be less important than it used to be, however 53% is quite alot lower than the almost 60% over the entire set <br><br> Becuase of the way the data has been structured and the anomalous nature of the pandemic <br><br> The resulting dataframe has a shape of (23002, 9003) <br><br> During the processing ~4000 games were lost including all games from the first season 1999-2000 |

### Results
##### Logistic Regression using no polynomial features (adjusted for time)
Train score :  0.676<br>Test score  :  0.661<br>Mean score. :  0.669
| Coefficients |     |
| --- | --- |
| <img src="https://github.com/finjammin/Capstone-NBA/blob/main/Pt4_Modelling/Images/coefficients_lrgrid_66.png" width="1200"/> | The features with the largest coefficients are as to be expected, plus_minus shows the average point differential while the player was on the court, although there is a lot of co-linearity with these features as players, particularly starting players are often on the court for large parts of the same time during a game.<br><br> Interesting to see age crop up, as well as type (regular season/playofff)<br><br> Also some hussle stats h_s2_ewm_fta, a_s2_ewm_blk, a_pf_ewm_stl et al. |

##### Logistic Regression using no polynomial features (not adjusted for time)
Train score :  0.681<br>Test score  :  0.663<br>Mean score. :  0.673
| Coefficients |     |
| --- | --- |
| <img src="https://github.com/finjammin/Capstone-NBA/blob/main/Pt4_Modelling/Images/coefficients_nonadj_lr.png" width="800"/> | Interesting to see that age becomes a more prevalent statistic when performances are not changed to a per 40 minute basis.<br><br> Nice to see that John Hollinger's GameScore (applied in feature engineering) also appears 'h_c__gs'. <br><br> Still alot of collinearity in the features, probably best to apply some sort of PCA. |
<br>
<img src="https://github.com/finjammin/Capstone-NBA/blob/main/Pt4_Modelling/Images/roc_precisionrecall.png" width="1200"/>
<br>
<img src="https://github.com/finjammin/Capstone-NBA/blob/main/Pt4_Modelling/Images/logreg_crossover_nonadj.png" width="400"/> 
<br>As we can see from the above precision-recall curves and roc curves, as well as the distributions of the prrobabilities show there is not much point in adjusting thresholds to try and improve the accuracy scores.<br>

##### Random Forest Classifier
Train score   :  0.713<br>Test score    :  0.660<br>Mean CVscore. :  0.666
<br>Best params : {'criterion': 'entropy', 'max_depth': 15, 'min_impurity_decrease': 0.001}

##### Gradient Boosting Classifier
Train score   :  0.713<br>Test score    :  0.660<br>Mean CVscore. :  0.668
<br>{'learning_rate': 0.025, 'max_depth': 4}

| RandomForestClassifier | GradientBoostingClassifier |
| --- | --- |
| <img src="https://github.com/finjammin/Capstone-NBA/blob/main/Pt4_Modelling/Images/feature_imp_rfc.png" width="400"/> | <img src="https://github.com/finjammin/Capstone-NBA/blob/main/Pt4_Modelling/Images/feature_imp_gfc.png" width="400"/> |
<br>
GradientBoostingClassifier train and test score performance.
<img src="https://github.com/finjammin/Capstone-NBA/blob/main/Pt4_Modelling/Images/gfc_iterations_performance.png" width="400"/>

### Limitations and future improvements

- The method's used mean that we assume that we know who will be playing in the game, and also the starting lineup, therefore should this be productionised it would rely on only being useful just before tipoff.
- Most teams rely on a rotation of 8 players however there are some notable exceptions, for instance the Toronto Raptor's in their 2019-2020 championship winning season regularly relied on 9 players or more, the way in which the data is structured would not take this into account.
- There is no match-up data involved, by creating a network between players and their one-one matchup it would be possible to include some inference about average player performance and apply that to their form statistics. A particularly good example of this is when Joel Embiid (one of the best players in the league) is matched up against Marc Gasol he performs at ~60% of his regular performances. [In November 2019 Marc Gasol held Joel Embiid to 0 points in 32 minutes](https://www.landofbasketball.com/games_between/joel_embiid_vs_marc_gasol.htm)
- Create features between players
- The only size metric for the player's used is height. In reality height is only one of many size metrics which are helpful for identifying features for winning basketball, in particular to defense. In the SQL database there is draft combine metrics, including statistics like hand size and wingspan. However top prospects often do not take part, therefore only about 70% of players had these physical attributes. If possible wingspan be would be a great addition to this dataset as it is widely considered one of the most important attributes for defense, an area of the game sorely missing from the dataset used.
- The NBA has for the past 3 years collected defensive statistics, however for the scope of this project they were not used.
- Aim to try and productionise a model with predictions made on game day.
