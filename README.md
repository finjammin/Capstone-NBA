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
