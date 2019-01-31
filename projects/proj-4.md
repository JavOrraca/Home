---
layout: post
title: 'Super Bowl 53: Predictive Modeling'
---
Two weeks ago, I entered a Super Bowl LIII predictive analytics competition with my classmate [Kaitlyn Drake](https://www.linkedin.com/in/kaitdrake/) to enhance our data analysis and processing skills via R. We ended up having a great time and walked away with some tools and machine learning models that can be applied to topics beyond sports analytics. I've been using Shiny, Plotly, and gganimate recently to make flat charts come to life, so this was a great opportunity to further explore interactive, web-based visualization tools. Kaitlyn was great because prior to this exercise, I knew very little about the NFL and football!

We spent a significant amount of time exploring NFL data sets (by team, player, season, _you-name-it_), but ultimately, our regression model only relied on four variables, best suited for our simulation, with more variables considered for the J48 decision tree model. We built the decision tree in WEKA, so this post will only cover the regression modeling and game simulation aspects of our project executed with R. While we did not use all of the data wrangled in the below example codes, I figured it could be beneficial to just keep it here. If you can use it in the future, awesome.

**Prerequisites**
* R notebook: I use [JupyterLab](https://blog.jupyter.org/jupyterlab-is-ready-for-users-5a6f039b8906) almost exclusively nowadays for projects like these, but if you're just starting out, [RStudio (desktop)](https://www.rstudio.com/) is a much better beginner platform in my opinion
* R packages:
  * dplyr, sqldf, fastDummies (this one is new to me, but worked great for fast conversions of categorical variables to binary variables), ggplot2, 


# Super Bowl 53: Final Score Predictions
_**BANA 290: Applied Forecast Modeling**_
<br/>_Kaitlyn Drake_
<br/>_Javier Orraca_

# Overview & Process
1. Data Collection
2. Data Manipulation
3. Exploration through Visualizations
4. Poisson Regression
5. Simulation Modeling
6. Forecast Conclusions

# 1. Data Collection


```R
setwd("C:/Users/orrac/OneDrive/Documents/UCI Courses/2019.01_BANA 290_Predictive Modeling/NFL Data")

library(dplyr)

Teams_2018_Stats <- read.csv("NFL 2018 Team Season Data.csv")
Teams_2018_Valuation <- read.csv("NFL Team Valuations.csv")
Teams_2018_Valuation_by_Year <- read.csv("NFL Team Valuations by Year.csv")
Teams_Coach_Tenure <- read.csv("NFL Coach Tenure.csv")

# Rename / remove certain columns for cleanliness
colnames(Teams_2018_Stats)[1] <- "Team"
colnames(Teams_2018_Valuation)[1] <- "Team"
colnames(Teams_2018_Valuation_by_Year)[1] <- "Team"
colnames(Teams_Coach_Tenure)[1] <- "CoachName"
Teams_2018_Valuation[c(2:14)] <- NULL
colnames(Teams_2018_Valuation)[c(2,3)] <- c("Valuation2018inMil", "ValuationChange5Yr")
colnames(Teams_2018_Stats)[4] <- "GameLocation"
colnames(Teams_2018_Stats)[colnames(Teams_2018_Stats)=="Game.Number"] <- "GameNumber"

# Change character types as needed
Teams_2018_Stats$Team <- as.character(Teams_2018_Stats$Team)
Teams_2018_Stats$Opponent <- as.character(Teams_2018_Stats$Opponent)
Teams_2018_Stats$GameLocation <- as.character(Teams_2018_Stats$GameLocation)
Teams_2018_Valuation$Team <- as.character(Teams_2018_Valuation$Team)
Teams_2018_Valuation_by_Year$Team <- as.character(Teams_2018_Valuation_by_Year$Team)
Teams_2018_Valuation$Valuation2018inMil <- as.integer(Teams_2018_Valuation$Valuation2018inMil)
Teams_2018_Valuation_by_Year$Valuation <- as.integer(Teams_2018_Valuation_by_Year$Valuation)
Teams_Coach_Tenure$CoachName <- as.character(Teams_Coach_Tenure$CoachName)
Teams_Coach_Tenure$Team <- as.character(Teams_Coach_Tenure$Team)

# Check variable types of each data.frame running string function on each
str(Teams_2018_Stats)
str(Teams_2018_Valuation)
str(Teams_2018_Valuation_by_Year)
str(Teams_Coach_Tenure)
```

    Warning message:
    "package 'dplyr' was built under R version 3.5.2"
    Attaching package: 'dplyr'
    
    The following objects are masked from 'package:stats':
    
        filter, lag
    
    The following objects are masked from 'package:base':
    
        intersect, setdiff, setequal, union
    
    

    'data.frame':	532 obs. of  17 variables:
     $ Team                : chr  "Arizona Cardinals" "Arizona Cardinals" "Arizona Cardinals" "Arizona Cardinals" ...
     $ Opponent            : chr  "Washington Redskins" "Los Angeles Rams" "Chicago Bears" "Seattle Seahawks" ...
     $ GameNumber          : int  1 2 3 4 5 6 7 8 9 10 ...
     $ GameLocation        : chr  "HOME" "AWAY" "HOME" "HOME" ...
     $ Team_Score          : int  6 0 14 17 28 17 10 18 14 21 ...
     $ Opponent_Score      : int  24 34 16 20 18 27 45 15 26 23 ...
     $ Team_Passing        : int  145 83 168 171 164 208 154 233 166 128 ...
     $ Team_Rushing        : int  68 54 53 92 56 60 69 88 94 154 ...
     $ Total_Yards         : int  213 137 221 263 220 268 223 321 260 282 ...
     $ Team_Turnovers      : int  2 1 4 1 0 2 5 2 2 2 ...
     $ Opponent_Passing    : int  247 342 194 160 300 216 178 160 212 173 ...
     $ Opponent_Rushing    : int  182 90 122 171 147 195 131 107 118 152 ...
     $ Opponent_Total_Yards: int  429 432 316 331 447 411 309 267 330 325 ...
     $ Opponent_Turnovers  : int  1 1 2 0 5 2 1 0 0 0 ...
     $ Net_Yards           : int  -216 -295 -95 -68 -227 -143 -86 54 -70 -43 ...
     $ Net_Turnovers       : int  1 0 2 1 -5 0 4 2 2 2 ...
     $ Net_Score           : int  -18 -34 -2 -3 10 -10 -35 3 -12 -2 ...
    'data.frame':	32 obs. of  3 variables:
     $ Team              : chr  "Los Angeles Rams" "Oakland Raiders" "Atlanta Falcons" "Los Angeles Chargers" ...
     $ Valuation2018inMil: int  29 15 20 11 8 10 14 16 17 18 ...
     $ ValuationChange5Yr: num  3.44 2.49 2.31 2.29 2.16 ...
    'data.frame':	448 obs. of  3 variables:
     $ Team     : chr  "Arizona Cardinals" "Arizona Cardinals" "Arizona Cardinals" "Arizona Cardinals" ...
     $ Year     : int  2005 2006 2007 2008 2009 2010 2011 2012 2013 2014 ...
     $ Valuation: int  673 789 888 914 935 919 901 922 961 1000 ...
    'data.frame':	32 obs. of  3 variables:
     $ CoachName  : chr  " Bruce AriansÂ " " Dan QuinnÂ " " John HarbaughÂ " " Rex RyanÂ " ...
     $ Team       : chr  "Arizona Cardinals" "Atlanta Falcons" "Baltimore Ravens" "Buffalo Bills" ...
     $ CoachTenure: int  6 4 11 4 8 4 16 3 9 4 ...
    

# 2. Data Manipulation


```R
# Left join Teams_2018_Stats and valuation data frames by Team
NFL_Trim <- left_join(Teams_2018_Stats, Teams_2018_Valuation, by=c("Team" = "Team"))

# Use fastDummies package to create dummy variables for Teams. Note: The first
# dummy variable for Team and Opponent can be removed to avoid multicollinearity by
# including final criteria in dummy_cols function as "remove_first_dummy = TRUE".

library(fastDummies)

NFL_Trim <- dummy_cols(NFL_Trim, select_columns = "Team")
NFL_Trim[c(20:36,38:39,41:51)] <- NULL
colnames(NFL_Trim)[c(20,21)] <- c("Team_Rams", "Team_Patriots")

NFL_Trim$Valuation2018inMil <- ifelse(is.na(NFL_Trim$Valuation2018inMil), 0, NFL_Trim$Valuation2018inMil)
NFL_Trim$ValuationChange5Yr <- ifelse(is.na(NFL_Trim$ValuationChange5Yr), 0, NFL_Trim$ValuationChange5Yr)

NFL_Trim$GameLost <- as.integer(ifelse(NFL_Trim$Net_Score < 0, 1, 0))
NFL_Trim$GameTie <- as.integer(ifelse(NFL_Trim$Net_Score == 0, 1, 0))
NFL_Trim$GameWon <- as.integer(ifelse(NFL_Trim$Net_Score > 0, 1, 0))

str(NFL_Trim)
```

    Warning message:
    "package 'fastDummies' was built under R version 3.5.2"

    'data.frame':	532 obs. of  24 variables:
     $ Team                : chr  "Arizona Cardinals" "Arizona Cardinals" "Arizona Cardinals" "Arizona Cardinals" ...
     $ Opponent            : chr  "Washington Redskins" "Los Angeles Rams" "Chicago Bears" "Seattle Seahawks" ...
     $ GameNumber          : int  1 2 3 4 5 6 7 8 9 10 ...
     $ GameLocation        : chr  "HOME" "AWAY" "HOME" "HOME" ...
     $ Team_Score          : int  6 0 14 17 28 17 10 18 14 21 ...
     $ Opponent_Score      : int  24 34 16 20 18 27 45 15 26 23 ...
     $ Team_Passing        : int  145 83 168 171 164 208 154 233 166 128 ...
     $ Team_Rushing        : int  68 54 53 92 56 60 69 88 94 154 ...
     $ Total_Yards         : int  213 137 221 263 220 268 223 321 260 282 ...
     $ Team_Turnovers      : int  2 1 4 1 0 2 5 2 2 2 ...
     $ Opponent_Passing    : int  247 342 194 160 300 216 178 160 212 173 ...
     $ Opponent_Rushing    : int  182 90 122 171 147 195 131 107 118 152 ...
     $ Opponent_Total_Yards: int  429 432 316 331 447 411 309 267 330 325 ...
     $ Opponent_Turnovers  : int  1 1 2 0 5 2 1 0 0 0 ...
     $ Net_Yards           : int  -216 -295 -95 -68 -227 -143 -86 54 -70 -43 ...
     $ Net_Turnovers       : int  1 0 2 1 -5 0 4 2 2 2 ...
     $ Net_Score           : int  -18 -34 -2 -3 10 -10 -35 3 -12 -2 ...
     $ Valuation2018inMil  : int  10 10 10 10 10 10 10 10 10 10 ...
     $ ValuationChange5Yr  : num  2.15 2.15 2.15 2.15 2.15 2.15 2.15 2.15 2.15 2.15 ...
     $ Team_Rams           : int  0 0 0 0 0 0 0 0 0 0 ...
     $ Team_Patriots       : int  0 0 0 0 0 0 0 0 0 0 ...
     $ GameLost            : int  1 1 1 1 0 1 1 0 1 1 ...
     $ GameTie             : int  0 0 0 0 0 0 0 0 0 0 ...
     $ GameWon             : int  0 0 0 0 1 0 0 1 0 0 ...
    


```R
# Make new columns for later use in Poisson regression and simulations
NFL_Trim$Home <- ifelse(NFL_Trim$GameLocation=="HOME", 1, 0)
NFL_Trim$Away <- ifelse(NFL_Trim$GameLocation=="AWAY", 1, 0)
NFL_Trim$HomeGoals <- ifelse(NFL_Trim$Home==1, NFL_Trim$Team_Score, 0)
NFL_Trim$AwayGoals <- ifelse(NFL_Trim$Away==1, NFL_Trim$Team_Score, 0)
str(NFL_Trim)
```

    'data.frame':	532 obs. of  28 variables:
     $ Team                : chr  "Arizona Cardinals" "Arizona Cardinals" "Arizona Cardinals" "Arizona Cardinals" ...
     $ Opponent            : chr  "Washington Redskins" "Los Angeles Rams" "Chicago Bears" "Seattle Seahawks" ...
     $ GameNumber          : int  1 2 3 4 5 6 7 8 9 10 ...
     $ GameLocation        : chr  "HOME" "AWAY" "HOME" "HOME" ...
     $ Team_Score          : int  6 0 14 17 28 17 10 18 14 21 ...
     $ Opponent_Score      : int  24 34 16 20 18 27 45 15 26 23 ...
     $ Team_Passing        : int  145 83 168 171 164 208 154 233 166 128 ...
     $ Team_Rushing        : int  68 54 53 92 56 60 69 88 94 154 ...
     $ Total_Yards         : int  213 137 221 263 220 268 223 321 260 282 ...
     $ Team_Turnovers      : int  2 1 4 1 0 2 5 2 2 2 ...
     $ Opponent_Passing    : int  247 342 194 160 300 216 178 160 212 173 ...
     $ Opponent_Rushing    : int  182 90 122 171 147 195 131 107 118 152 ...
     $ Opponent_Total_Yards: int  429 432 316 331 447 411 309 267 330 325 ...
     $ Opponent_Turnovers  : int  1 1 2 0 5 2 1 0 0 0 ...
     $ Net_Yards           : int  -216 -295 -95 -68 -227 -143 -86 54 -70 -43 ...
     $ Net_Turnovers       : int  1 0 2 1 -5 0 4 2 2 2 ...
     $ Net_Score           : int  -18 -34 -2 -3 10 -10 -35 3 -12 -2 ...
     $ Valuation2018inMil  : int  10 10 10 10 10 10 10 10 10 10 ...
     $ ValuationChange5Yr  : num  2.15 2.15 2.15 2.15 2.15 2.15 2.15 2.15 2.15 2.15 ...
     $ Team_Rams           : int  0 0 0 0 0 0 0 0 0 0 ...
     $ Team_Patriots       : int  0 0 0 0 0 0 0 0 0 0 ...
     $ GameLost            : int  1 1 1 1 0 1 1 0 1 1 ...
     $ GameTie             : int  0 0 0 0 0 0 0 0 0 0 ...
     $ GameWon             : int  0 0 0 0 1 0 0 1 0 0 ...
     $ Home                : num  1 0 1 1 0 0 1 1 0 1 ...
     $ Away                : num  0 1 0 0 1 1 0 0 1 0 ...
     $ HomeGoals           : num  6 0 14 17 0 0 10 18 0 21 ...
     $ AwayGoals           : num  0 0 0 0 28 17 0 0 14 0 ...
    


```R
# Run SQL queries for summarizing conditional sum / mean metrics by team
library(sqldf)

WhenWonbyTeam <- sqldf("SELECT Team, AVG(Team_Score) AS PointsAvgWhenWon, AVG(Total_Yards) AS YardsAvgWhenWon, AVG(Net_Yards) AS YardsAvgNetWhenWon FROM NFL_Trim WHERE GameWon = 1 GROUP BY Team")
WhenAwaybyTeam <- sqldf("SELECT Team, AVG(Team_Score) AS PointsAvgWhenAway, AVG(Total_Yards) AS YardsAvgWhenAway, AVG(Net_Yards) AS YardsAvgNetWhenAway FROM NFL_Trim WHERE GameLocation = 'AWAY' GROUP BY Team")
WhenWonAwaybyTeam <- sqldf("SELECT Team, AVG(Team_Score) AS PointsAvgWhenWonAway, AVG(Total_Yards) AS YardsAvgWhenWonAway, AVG(Net_Yards) AS YardsAvgNetWhenWonAway FROM NFL_Trim WHERE GameLocation = 'AWAY' AND GameWon = 1 GROUP BY Team")

# Left join all using plyr package and re-order columns  
library(plyr)
NFL_ConditionalTbl <- join_all(list(WhenWonbyTeam, WhenAwaybyTeam, WhenWonAwaybyTeam), by="Team", type="left")
NFL_ConditionalTbl <- NFL_ConditionalTbl[,c(1,2,5,8,3,6,9,4,7,10)]

# Since SF 49ers won no games away, clean up NAs
NFL_ConditionalTbl[is.na(NFL_ConditionalTbl)] <- 0

# Re-load dplyr in case of potential plyr function overrides
library(dplyr)

str(NFL_ConditionalTbl)
```

    Loading required package: gsubfn
    Loading required package: proto
    Loading required package: RSQLite
    ------------------------------------------------------------------------------
    You have loaded plyr after dplyr - this is likely to cause problems.
    If you need functions from both plyr and dplyr, please load plyr first, then dplyr:
    library(plyr); library(dplyr)
    ------------------------------------------------------------------------------
    
    Attaching package: 'plyr'
    
    The following objects are masked from 'package:dplyr':
    
        arrange, count, desc, failwith, id, mutate, rename, summarise,
        summarize
    
    

    'data.frame':	32 obs. of  10 variables:
     $ Team                  : chr  "Arizona Cardinals" "Atlanta Falcons" "Baltimore Ravens" "Buffalo Bills" ...
     $ PointsAvgWhenWon      : num  22 32 27.3 26.8 30.3 ...
     $ PointsAvgWhenAway     : num  15.9 22.2 21.5 14.8 21.5 ...
     $ PointsAvgWhenWonAway  : num  24 32 23.8 34 27 ...
     $ YardsAvgWhenWon       : num  285 446 390 331 365 ...
     $ YardsAvgWhenAway      : num  225 390 378 285 375 ...
     $ YardsAvgWhenWonAway   : num  268 469 385 372 372 ...
     $ YardsAvgNetWhenWon    : num  -61 36.3 156.6 67.2 33.7 ...
     $ YardsAvgNetWhenAway   : num  -160.6 29.4 85.5 -20.4 25.6 ...
     $ YardsAvgNetWhenWonAway: num  -118.5 57.3 205 126 54.5 ...
    


```R
# Summarize average metrics by team
NFL_by_Team <- NFL_Trim %>%
    group_by(Team) %>%
    transmute(
        GamesTotal = max(GameNumber),
        GamesTotalLost = sum(GameLost),
        GamesTotalTied = sum(GameTie),
        GamesTotalWon = sum(GameWon),
        PointsAvgPerGame = mean(Team_Score),
        PointsTotal = sum(Team_Score),
        YardsTotalPassing = sum(Team_Passing),
        YardsTotalRushing = sum(Team_Rushing),
        YardsTotal = sum(Total_Yards),
        YardsAvg = mean(Total_Yards),
        YardsTotalNet = sum(Net_Yards),
        YardsAvgNet = mean(Net_Yards),
        TurnoversTotal = sum(Team_Turnovers),
        TurnoversAvg = mean(Team_Turnovers),
        Valuation2018 = mean(Valuation2018inMil),
        ValuationChange5Yr = mean(ValuationChange5Yr)
    )

# Run distinct function to remove duplicate instances carried from NFL_Trim
NFL_by_Team <- distinct(NFL_by_Team, Team, .keep_all = TRUE)

# Left join to NFL conditional sum / avg table and re-order columns
NFL_by_Team <- left_join(NFL_by_Team, NFL_ConditionalTbl, by=c("Team" = "Team"))
NFL_by_Team <- as.data.frame(NFL_by_Team)
NFL_by_Team <- NFL_by_Team[,c(1:5,7,6,18:20,8:11,21:23,12:13,24:26,14:17)]
str(NFL_by_Team)
```

    'data.frame':	32 obs. of  26 variables:
     $ Team                  : chr  "Arizona Cardinals" "Atlanta Falcons" "Baltimore Ravens" "Buffalo Bills" ...
     $ GamesTotal            : num  16 16 17 16 16 17 16 16 18 16 ...
     $ GamesTotalLost        : int  13 9 7 10 9 5 10 8 7 10 ...
     $ GamesTotalTied        : int  0 0 0 0 0 0 0 1 0 0 ...
     $ GamesTotalWon         : int  3 7 10 6 7 12 6 7 11 6 ...
     $ PointsTotal           : int  225 414 406 269 376 436 368 359 385 329 ...
     $ PointsAvgPerGame      : num  14.1 25.9 23.9 16.8 23.5 ...
     $ PointsAvgWhenWon      : num  22 32 27.3 26.8 30.3 ...
     $ PointsAvgWhenAway     : num  15.9 22.2 21.5 14.8 21.5 ...
     $ PointsAvgWhenWonAway  : num  24 32 23.8 34 27 ...
     $ YardsTotalPassing     : int  2523 4653 3697 2794 3836 3855 3290 4007 4012 3695 ...
     $ YardsTotalRushing     : int  1342 1573 2531 1984 2136 2003 1682 1893 2177 1907 ...
     $ YardsTotal            : int  3865 6226 6228 4778 5972 5858 4972 5900 6189 5602 ...
     $ YardsAvg              : num  242 389 366 299 373 ...
     $ YardsAvgWhenWon       : num  285 446 390 331 365 ...
     $ YardsAvgWhenAway      : num  225 390 378 285 375 ...
     $ YardsAvgWhenWonAway   : num  268 469 385 372 372 ...
     $ YardsTotalNet         : int  -1876 74 1298 72 321 763 -1646 -388 163 -240 ...
     $ YardsAvgNet           : num  -117.25 4.62 76.35 4.5 20.06 ...
     $ YardsAvgNetWhenWon    : num  -61 36.3 156.6 67.2 33.7 ...
     $ YardsAvgNetWhenAway   : num  -160.6 29.4 85.5 -20.4 25.6 ...
     $ YardsAvgNetWhenWonAway: num  -118.5 57.3 205 126 54.5 ...
     $ TurnoversTotal        : int  28 18 23 32 22 24 17 24 18 21 ...
     $ TurnoversAvg          : num  1.75 1.12 1.35 2 1.38 ...
     $ Valuation2018         : num  10 20 19 1 12 26 3 4 32 22 ...
     $ ValuationChange5Yr    : num  2.15 2.31 1.73 1.71 1.84 ...
    


```R
# Normalized totals for Saints and Chiefs to reflect 16 vs 17 games, and
# adjusted Patriots and Rams to reflect 16 vs 18 games. Created new fields.

NFL_by_Team <- NFL_by_Team %>%
    mutate(
        PointsTotalNorm = ifelse(GamesTotal==17, PointsTotal/17*16, PointsTotal),
        YardsTotalPassingNorm = ifelse(GamesTotal==17, YardsTotalPassing/17*16, YardsTotalPassing),
        YardsTotalRushingNorm = ifelse(GamesTotal==17, YardsTotalRushing/17*16, YardsTotalRushing),
        YardsTotalNorm = ifelse(GamesTotal==17, YardsTotal/17*16, YardsTotal),
        YardsTotalNetNorm = ifelse(GamesTotal==17, YardsTotalNet/17*16, YardsTotalNet),
        TurnoversTotalNorm = ifelse(GamesTotal==17, TurnoversTotal/17*16, TurnoversTotal),
        PointsTotalNorm = ifelse(GamesTotal==18, PointsTotal/18*16, PointsTotal),
        YardsTotalPassingNorm = ifelse(GamesTotal==18, YardsTotalPassing/18*16, YardsTotalPassing),
        YardsTotalRushingNorm = ifelse(GamesTotal==18, YardsTotalRushing/18*16, YardsTotalRushing),
        YardsTotalNorm = ifelse(GamesTotal==18, YardsTotal/18*16, YardsTotal),
        YardsTotalNetNorm = ifelse(GamesTotal==18, YardsTotalNet/18*16, YardsTotalNet),
        TurnoversTotalNorm = ifelse(GamesTotal==18, TurnoversTotal/18*16, TurnoversTotal)
    )
NFL_by_Team <- NFL_by_Team[,c(1:6,27,7:13,28:30,14:18,31,19:23,32,24:26)]
str(NFL_by_Team)

# Save data frame as csv file, if needed
write.csv(NFL_by_Team, "NFL_by_Team.csv")
```

    'data.frame':	32 obs. of  32 variables:
     $ Team                  : chr  "Arizona Cardinals" "Atlanta Falcons" "Baltimore Ravens" "Buffalo Bills" ...
     $ GamesTotal            : num  16 16 17 16 16 17 16 16 18 16 ...
     $ GamesTotalLost        : int  13 9 7 10 9 5 10 8 7 10 ...
     $ GamesTotalTied        : int  0 0 0 0 0 0 0 1 0 0 ...
     $ GamesTotalWon         : int  3 7 10 6 7 12 6 7 11 6 ...
     $ PointsTotal           : int  225 414 406 269 376 436 368 359 385 329 ...
     $ PointsTotalNorm       : num  225 414 382 269 376 ...
     $ PointsAvgPerGame      : num  14.1 25.9 23.9 16.8 23.5 ...
     $ PointsAvgWhenWon      : num  22 32 27.3 26.8 30.3 ...
     $ PointsAvgWhenAway     : num  15.9 22.2 21.5 14.8 21.5 ...
     $ PointsAvgWhenWonAway  : num  24 32 23.8 34 27 ...
     $ YardsTotalPassing     : int  2523 4653 3697 2794 3836 3855 3290 4007 4012 3695 ...
     $ YardsTotalRushing     : int  1342 1573 2531 1984 2136 2003 1682 1893 2177 1907 ...
     $ YardsTotal            : int  3865 6226 6228 4778 5972 5858 4972 5900 6189 5602 ...
     $ YardsTotalPassingNorm : num  2523 4653 3480 2794 3836 ...
     $ YardsTotalRushingNorm : num  1342 1573 2382 1984 2136 ...
     $ YardsTotalNorm        : num  3865 6226 5862 4778 5972 ...
     $ YardsAvg              : num  242 389 366 299 373 ...
     $ YardsAvgWhenWon       : num  285 446 390 331 365 ...
     $ YardsAvgWhenAway      : num  225 390 378 285 375 ...
     $ YardsAvgWhenWonAway   : num  268 469 385 372 372 ...
     $ YardsTotalNet         : int  -1876 74 1298 72 321 763 -1646 -388 163 -240 ...
     $ YardsTotalNetNorm     : num  -1876 74 1222 72 321 ...
     $ YardsAvgNet           : num  -117.25 4.62 76.35 4.5 20.06 ...
     $ YardsAvgNetWhenWon    : num  -61 36.3 156.6 67.2 33.7 ...
     $ YardsAvgNetWhenAway   : num  -160.6 29.4 85.5 -20.4 25.6 ...
     $ YardsAvgNetWhenWonAway: num  -118.5 57.3 205 126 54.5 ...
     $ TurnoversTotal        : int  28 18 23 32 22 24 17 24 18 21 ...
     $ TurnoversTotalNorm    : num  28 18 21.6 32 22 ...
     $ TurnoversAvg          : num  1.75 1.12 1.35 2 1.38 ...
     $ Valuation2018         : num  10 20 19 1 12 26 3 4 32 22 ...
     $ ValuationChange5Yr    : num  2.15 2.31 1.73 1.71 1.84 ...
    

**3. Exploration through Visualizations**
Unfortunately, I'm having issues bringing in my Plotly and gganimate interactive plots and animations to this GitHub post, but feel free to contact me via LinkedIn or email and I'll get those HTML files to you. For now, below are some static charts built with ggplot + plotly + gganimate.  

**4. Poisson Regression**
The first thing we do is to create two data frames and row-bind them for running Poisson regressions and later for calculating game simulation probabilities.

```R
NFL_Poisson <- rbind(
    data.frame(Points=NFL_Trim$HomeGoals,
               Team=NFL_Trim$Team,
               Opponent=NFL_Trim$Opponent,
               Home=1),
    data.frame(Points=NFL_Trim$AwayGoals,
               Team=NFL_Trim$Opponent,
               Opponent=NFL_Trim$Team,
               Home=0)) %>%
glm(Points ~ Home + Team + Opponent, family=poisson(link=log), data=.)
summary(NFL_Poisson)
```

    Call:
    glm(formula = Points ~ Home + Team + Opponent, family = poisson(link = log), 
        data = .)
    
    Deviance Residuals: 
       Min      1Q  Median      3Q     Max  
    -6.407  -4.769  -3.894   3.085   9.295  
    
    Coefficients:
                                  Estimate Std. Error z value Pr(>|z|)    
    (Intercept)                   2.051457   0.081417  25.197  < 2e-16 ***
    Home                          0.094649   0.017971   5.267 1.39e-07 ***
    TeamAtlanta Falcons           0.494885   0.078686   6.289 3.19e-10 ***
    TeamBaltimore Ravens          0.276548   0.080051   3.455 0.000551 ***
    TeamBuffalo Bills             0.226990   0.082911   2.738 0.006186 ** 
    TeamCarolina Panthers         0.321063   0.080845   3.971 7.15e-05 ***
    TeamChicago Bears             0.271657   0.078837   3.446 0.000569 ***
    TeamCincinnati Bengals        0.408172   0.079095   5.161 2.46e-07 ***
    TeamCleveland Browns          0.217660   0.082372   2.642 0.008232 ** 
    TeamDallas Cowboys            0.241876   0.079675   3.036 0.002399 ** 
    TeamDenver Broncos            0.100502   0.082234   1.222 0.221651    
    TeamDetroit Lions             0.264208   0.080591   3.278 0.001044 ** 
    TeamGreen Bay Packers         0.287966   0.080466   3.579 0.000345 ***
    TeamHouston Texans            0.283781   0.082098   3.457 0.000547 ***
    TeamIndianapolis Colts        0.291924   0.080449   3.629 0.000285 ***
    TeamJacksonville Jaguars     -0.099105   0.089485  -1.108 0.268073    
    TeamKansas City Chiefs        0.506154   0.075156   6.735 1.64e-11 ***
    TeamLos Angeles Chargers      0.190751   0.079741   2.392 0.016750 *  
    TeamLos Angeles Rams          0.601312   0.073030   8.234  < 2e-16 ***
    TeamMiami Dolphins            0.306510   0.082076   3.734 0.000188 ***
    TeamMinnesota Vikings         0.203569   0.081838   2.487 0.012865 *  
    TeamNew England Patriots      0.389060   0.077399   5.027 4.99e-07 ***
    TeamNew Orleans Saints        0.574709   0.075185   7.644 2.11e-14 ***
    TeamNew York Giants           0.371601   0.080633   4.609 4.05e-06 ***
    TeamNew York Jets             0.499575   0.078820   6.338 2.33e-10 ***
    TeamOakland Raiders           0.340988   0.077951   4.374 1.22e-05 ***
    TeamPhiladelphia Eagles       0.144412   0.082357   1.753 0.079518 .  
    TeamPittsburgh Steelers       0.415679   0.079157   5.251 1.51e-07 ***
    TeamSan Francisco 49ers       0.170797   0.080523   2.121 0.033915 *  
    TeamSeattle Seahawks          0.294163   0.078023   3.770 0.000163 ***
    TeamTampa Bay Buccaneers      0.301204   0.081737   3.685 0.000229 ***
    TeamTennessee Titans          0.205374   0.083943   2.447 0.014422 *  
    TeamWashington Redskins       0.186148   0.083679   2.225 0.026111 *  
    OpponentAtlanta Falcons       0.071204   0.076715   0.928 0.353327    
    OpponentBaltimore Ravens     -0.189036   0.079433  -2.380 0.017322 *  
    OpponentBuffalo Bills        -0.139305   0.080691  -1.726 0.084278 .  
    OpponentCarolina Panthers     0.015709   0.076398   0.206 0.837087    
    OpponentChicago Bears        -0.092515   0.077185  -1.199 0.230675    
    OpponentCincinnati Bengals    0.095233   0.075442   1.262 0.206829    
    OpponentCleveland Browns      0.070000   0.075265   0.930 0.352347    
    OpponentDallas Cowboys       -0.081397   0.076309  -1.067 0.286120    
    OpponentDenver Broncos       -0.038357   0.076429  -0.502 0.615762    
    OpponentDetroit Lions        -0.070459   0.077885  -0.905 0.365652    
    OpponentGreen Bay Packers     0.176179   0.073056   2.412 0.015884 *  
    OpponentHouston Texans        0.067971   0.077615   0.876 0.381170    
    OpponentIndianapolis Colts    0.146496   0.074353   1.970 0.048806 *  
    OpponentJacksonville Jaguars -0.131281   0.080687  -1.627 0.103728    
    OpponentKansas City Chiefs    0.414984   0.068892   6.024 1.70e-09 ***
    OpponentLos Angeles Chargers  0.200529   0.071166   2.818 0.004836 ** 
    OpponentLos Angeles Rams      0.128184   0.072208   1.775 0.075865 .  
    OpponentMiami Dolphins        0.116249   0.075775   1.534 0.124996    
    OpponentMinnesota Vikings     0.027711   0.075641   0.366 0.714104    
    OpponentNew England Patriots  0.077337   0.074022   1.045 0.296123    
    OpponentNew Orleans Saints   -0.091669   0.076013  -1.206 0.227832    
    OpponentNew York Giants       0.160109   0.075747   2.114 0.034537 *  
    OpponentNew York Jets         0.009482   0.078704   0.120 0.904102    
    OpponentOakland Raiders      -0.038871   0.077220  -0.503 0.614696    
    OpponentPhiladelphia Eagles   0.065689   0.073953   0.888 0.374402    
    OpponentPittsburgh Steelers   0.022132   0.076957   0.288 0.773660    
    OpponentSan Francisco 49ers   0.198540   0.072088   2.754 0.005885 ** 
    OpponentSeattle Seahawks      0.133579   0.072706   1.837 0.066174 .  
    OpponentTampa Bay Buccaneers  0.300799   0.072116   4.171 3.03e-05 ***
    OpponentTennessee Titans     -0.164258   0.082708  -1.986 0.047032 *  
    OpponentWashington Redskins  -0.106113   0.080839  -1.313 0.189302    
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    
    (Dispersion parameter for poisson family taken to be 1)
    
        Null deviance: 19855  on 1063  degrees of freedom
    Residual deviance: 19315  on 1000  degrees of freedom
    AIC: 22013
   
    Number of Fisher Scoring iterations: 6

Similar to logistic regression, we take the exponent of the parameter Estimate values. A positive value implies more points (e<sup>x</sup>>1∀x>0), while values closer to zero represent more neutral effects (e<sup>0</sup>=1). Home has an Estimate of 0.094649. Historically when predicing NFL scores, Home would have captured the fact that home teams generally score more points than away teams. However, as we take the exponent of the Estimate coefficient for Home, home teams were only 9.9% (e<sup>0.094649</sup>=1.099) more likely to score.

The Los Angeles Rams have a Team estimate coefficient of 0.601312, while the New England Patriots have a 0.389060. This indicates that the Los Angeles Rams are generally better scorers than the "average" NFL team (and well... the New England Patriots are _also_ better scorers than the average NFL team, but _not as good of scorers_ as the Los Angeles Rams). The Opponent values penalize and reward teams based on the quality of their opposition. This mimics the defensive strength of each team (Los Angeles Rams: 0.128184; New England Patriots: 0.077337). In this case, you are less likely to score against the New England Patriots.

Below, we'll create additional regression models for consideration in our ensemble model.


```R
# Additional 
```

**5. Simulation Modeling**

In order to predict the outcome of Super Bowl 53, one of the finalists would (or should) be assigned a home team advantage. Both Super Bowl 53 teams are playing away, however, the Los Angeles Rams do not have a wide-reaching fanbase and sports forecasters / ticket brokers are predicting that majority of the fanbase present at Super Bowl 53 to be Patriots fans. As such, we'll assign the Patriots with the home team advantage.

We will pass the teams into NFL_Poisson and the model will return the expected average number of points for the team (we need to run it twice to calculate the expected average number of points for each team separately). Below is the code and results for how many points we expect the Los Angeles Rams and the New England Patriots to score, giving the home team advantage to the Patriots.

```R
predict(NFL_Poisson,
        data.frame(Home=1, Team="New England Patriots",
                   Opponent="Los Angeles Rams"), type="response")
```
14.3442631436516

```R
predict(NFL_Poisson,
        data.frame(Home=0, Team="Los Angeles Rams",
                   Opponent="New England Patriots"), type="response")
```
15.3345232802124

The above results show a super tight range, predicting that the Los Angeles Rams will beat the New England Patriots 15 to 14, depsite the Patriots having the home team advantage. Given the tight above results, we'll create a Super Bowl simulation function as follows:

```R
# Create function (and prepare underlying data frames) for simulation
SuperBowl_Simulate <- function(NFL_Model, HomeTeam, AwayTeam, MaxPoints=40){
    HomePointsAvg <- predict(NFL_Model,
                            data.frame(Home=1, Team=HomeTeam,
                                       Opponent=AwayTeam), type="response")
    AwayPointsAvg <- predict(NFL_Model,
                            data.frame(Home=0, Team=AwayTeam,
                                       Opponent=HomeTeam), type="response")
    dpois(0:MaxPoints, HomePointsAvg) %o% dpois(0:MaxPoints, AwayPointsAvg) 
}

# Simulate for the teams at Super Bowl 53, listing the Home-advantage team first.
set.seed(123)
SuperBowl_Simulate(NFL_Poisson, "New England Patriots", "Los Angeles Rams", MaxPoints=16)
```

The code above produces a matrix of probabilities of the New England Patriots (rows of the matrix) and the Los Angeles Rams (matrix columns) scoring a specific number of points. For example, along the diagonal, both teams have the same number of points (e.g. P(0-0) = 1.290229e-13). We can calculate the odds of draw by summing all the diagonal entries. Everything below the diagonal represents a Patriots win (e.g P(8-0) = 3.198777e-09). Using matrix manipulation functions, we can perform these types of calculations to understand the chances of winning.

```R
PatriotsVsRams <- SuperBowl_Simulate(NFL_Poisson, "New England Patriots", "Los Angeles Rams", MaxPoints=40)
# Chances of a Patriots win
sum(PatriotsVsRams[lower.tri(PatriotsVsRams)])
```
0.391993996454314

```R
# Chances of a Tie, though this is an impossibility for the Super Bowl
sum(diag(PatriotsVsRams))
```
0.0723591319608067

```R
# Chances of a Rams win
sum(PatriotsVsRams[upper.tri(PatriotsVsRams)])
```
0.535646822597374


**6. Forecast Conclusions**
* Our predictions indicate that the Super Bowl will be an extremely close game!!!
* Even with the New England Patriots having the home team advantage (due to heavier fanbase predicted to be Super Bowl 53), they're projected to lose
* Our regression analysis and simulations favor the Los Angeles Rams narrowly leading with a final score of 15-14
* The Rams have a 53.6% chance of winning.

**Acknowledgements**
The main idea for the game simulation came from one of David Sheehan's GitHub repos, [Predicting Futbol Results With Statistical Modelling](https://dashee87.github.io/data%20science/football/r/predicting-football-results-with-statistical-modelling/).
