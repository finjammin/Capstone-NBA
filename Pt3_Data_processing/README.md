# Data processing
## Overview
- Clean data, player_id correction
- Time adjust to per 40 min stats
- Feature Selection
- Clustering
- Span selection & feature engineering
## Notebooks
#### PRE_collect_SQL
- Collect data from SQL
#### pt1_clean
- I discarded weight data as players weight fluctuates so much (example :[Shaquille O'Neal](https://www.stack.com/a/remember-when-shaq-was-ripped-and-super-athletic-heres-how-he-put-on-so-much-weight#:~:text=By%20the%202003%2D2004%20NBA,everything%20to%20do%20with%20basketball.)) and all data collected assumed the same weight throughout a player's career.
- Correct height data and some basketballreference data through the correction of player_id.
- Correct data for players by scraping the data from BasketballReference website using BeautifulSoup
- Statistical testing
- Removing observations under 300 seconds in time played
#### pt2_position_explore
- Looking at start_position
- Create position_dictionary based on which positions a player have started in
#### pt3_feature_selection
- Create age variable
- Collect data dictionary from BasketballReference
- Remove some colinear features
- Replace some 0 values with NaN values for some percent statistics (for rolling averages)
- Adjust data to a per 40 minute basis
