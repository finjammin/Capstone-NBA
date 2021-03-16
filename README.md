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

Using the time adjusted data, I wanted to have a look at some clustering to try and identify differing quality players. The best silhouette score using K_means <br/>For n_clusters = 6 the average silhouette_score is: 0.17137413370989096
![](/Pt4_Modelling/Images/kmeans_silhouette_6.png)
Starting positions            |  Clustered labels
:-------------------------:|:-------------------------:
![](/Pt4_Modelling/Images/clustering_played_pos.png)  |  ![](/Pt4_Modelling/Images/clustering_display.png)

##### Looking at the 'SuperStar' grouping (pink in Clustered labels image above).

:-------------------------:|:-------------------------:
<img src="https://user-images.githubusercontent.com/67226231/111352806-81ff7000-867c-11eb-9ccb-572553b72a69.png" width="600"/>

- Note that none of these players have played a game from the bench (in the past 20 years).
- 3 players who have won an MVP in the last 20 years who are not in this list are semi-traditional bigmen: Tim Duncan, Dirk Nowitzki and Kevin Garnett
- Alot of young player's currently playing in the league already showing up in this bracket: 'Ja Morant', 'Luka Doncic', 'Trae Young' ...
- Nikola Jokic a 2021 MVP front runner is one of only two bigmen in this clustering

### Structuring and feature selection

Firstly it has to be decided what time span and method will be used to create the form used for the predictor variables. The image below shows Kawhi Leonard's points per game in orange, and in blue the exponentialy weighted means over a span of sixteen games.
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
| <img src="https://raw.githubusercontent.com/finjammin/Capstone-NBA/main/Pt3_Data_processing/images/covid_anomaly.png" width="1000"/>  |  The change in wins is pretty drastic compared to normal. We can see a small trend that home court advantage appears to be less important than it used to be, however 53% is quite alot lower than the almost 60% over the entire set <br><br> Becuase of the way the data has been structured and the anomalous nature of the pandemic <br><br> The resulting dataframe has a shape of (23002, 9003) <br><br> During the processing ~4000 games were lost including all games from the first season 1999-2000 |

### Results
##### Logistic Regression using no polynomial features
