---
title: NCAA March Madness 2018 - Exploratory Data Analysis
date: 2018-03-24
tags: [Data Analytics]
header:
  image: "/images/marchmadness/marchmadness.jpg"
excerpt: A brief exploratory data analysis of the 2018 NCAA Men’s Basketball tournament dataset.
layout: single
---

This is a brief exploratory data analysis of the 2018 NCAA Men’s Basketball tournament dataset.
The datasets are pulled from the Kaggle and Google Cloud NCAA ML 2018 Competition. More information and the data used can be found [here](https://www.kaggle.com/c/mens-machine-learning-competition-2018).

## File Descriptions
``` r
library(data.table)
library(dplyr)
library(plyr)
library(MASS)
library(magrittr)
library(ggplot2)
library(gridExtra)
library(ggExtra)
library(knitr)
library(kableExtra)

teams <- fread('~/Documents/NCAA tourney/Teams.csv')
seeds <- fread('~/Documents/NCAA tourney/NCAATourneySeeds.csv')
conferences <- fread('~/Documents/NCAA tourney/Conferences.csv')
team_conferences <- fread('~/Documents/NCAA tourney/TeamConferences.csv')
season <- fread('~/Documents/NCAA tourney/RegularSeasonCompactResults.csv')
season_detail <- fread('~/Documents/NCAA tourney/RegularSeasonDetailedResults.csv')
tourney <- fread('~/Documents/NCAA tourney/NCAATourneyCompactResults.csv')
tourney_detail <- fread('~/Documents/NCAA tourney/NCAATourneyDetailedResults.csv')
conference_tourney <- fread('ConferenceTourneyGames.csv')
```

**Teams.csv:** This file identifies the different college teams present in the dataset. Each team has a 4-digit ID number.  

**NCAATourneySeeds.csv:** This file identifies the seeds for all teams in each NCAA tournament, for all seasons in historical data.
* **Season -** The year the tournament was played
* **Seed -** The seeding information of the team. The first character indicates the region the team was in. (W, X, Y, Z)
* **TeamID -** The 4-digit ID number for the team  

**Conferences.csv:** This file indicates the Division-I conferences that have existed over the years since 1985. Each conference is listed with an abbreviation and a longer name.  

**TeamConferences.csv:** This file indicates the conference affiliations for each team during each season.

**RegularSeasonCompactResults.csv:** This file identifies the game-by-game results for seasons since 1985.
* **Season -** The year of the associated entry
* **DayNum -** The day the game was played on. The integer value ranges from 0-132. Day 132 is Selection Sunday
* **WTeamID -** The ID number of the team that won the game, as listed in the “Teams.csv” file
* **WScore -** The number of points scored by the winning team
* **LTeamID -** The ID number of the team that lost the game, as list in the “Teams.csv” file
* **LScore -** The number of points score by the losing team
* **NumOT -** Number of overtime periods in the game
* **WLoc -** The location of the winning team. “H” for home game, “A” for away game, “N” for game played in neutral court  

**RegularSeasonDetailedResults.csv:** This file is like the RegularSeasonCompactResults.csv but it contains additional information for each game.  

**NCAATourneyCompactResults.csv:** This file identifies the game-by-game results for NCAA tournaments since 1985.
The DayNum is the day in which they play the tournament:
* **134 or 135:** last-four in games
* **136 or 137:** Round of 64
* **138 or 139:** Round of 32
* **143 or 144:** Sweet 16
* **145 or 146:** Elite 8
* **152:** Final 4
* **154:** National Championship  

**NCAATourneyDetailedResults.csv:** This file is like the NCAATourneyCompactResults.csv but it contains additional information for each game.  

**ConferenceTourneyGames.csv:** This file indicates which games were part of each year’s post-season conference tournaments (all of which finished on Selection Sunday or earlier), starting from the 2001 season.
* DayNum = 132 is the final day listed. The game played on day 132 is the conference championship game.  

## Historical Tournament Performances
Now that we have a good idea of the data we are working with, let’s start diving in some analysis. We will begin by assessing historical tournament performances.  

### Number 1 seeds
We look at the top ten teams with most number 1 seeds entering the tournament since 1985  

``` r
setkey(teams, TeamID)
setkey(seeds, TeamID)

no1_seeds <- teams[seeds][, one_seed := as.numeric(substr(Seed, 2, 3)) == 1][, sum(one_seed), by = TeamName]
no1_seeds <- no1_seeds[with(no1_seeds, order(-V1, TeamName)), ]

no1_seeds_plot <- ggplot(no1_seeds[1:10,], aes(reorder(TeamName, V1), V1)) +
  geom_bar(stat = 'identity', fill = 'steelblue') +
  labs(x = '', y = 'No. 1 Seeds', title = 'No. 1 Seeds since 1985') +
  coord_flip()
no1_seeds_plot
```  

<img src="{{ site.url }}{{ site.baseurl }}/images/marchmadness/num1seeds.jpeg" alt="number 1 seeds">  

### Regular Season Wins
The top ten teams with the most regular season wins since 1985  

``` r
setkey(season, WTeamID)

reg_wins <- season[teams][, .(Wins = .N), by = TeamName][order(-Wins)]

reg_wins_plot <- ggplot(reg_wins[1:10,], aes(reorder(TeamName, Wins), Wins)) +
  geom_bar(stat = 'identity', fill = 'steelblue') +
  labs(x = '', y = 'Wins', title = 'Regular Season Wins since 1985') +
  coord_flip()
reg_wins_plot
```  
<img src="{{ site.url }}{{ site.baseurl }}/images/marchmadness/regwins.jpeg" alt="regular season wins">  

### National Titles
The top ten teams that have won the most national championships since 1985  

``` r
setkey(tourney, WTeamID)

titles <- tourney[teams][DayNum == 154, .(Titles = .N), by = TeamName][order(-Titles)]

titles_plot <- ggplot(titles[1:10,], aes(reorder(TeamName, Titles), Titles)) +
  geom_bar(stat = 'identity', fill = 'steelblue') +
  labs(x = '', y = 'Titles', title = 'National Titles since 1985') +
  coord_flip()
titles_plot
```  
<img src="{{ site.url }}{{ site.baseurl }}/images/marchmadness/titles.jpeg" alt="national titles">  

### Conference Titles
The top ten teams that have won their respective conference championships since 1985  

``` r
setkey(conference_tourney, WTeamID)

conf_titles <- conference_tourney[teams][DayNum == 131, .(Titles = .N), by = TeamName][order(-Titles)]

conf_titles_plot <- ggplot(conf_titles[1:10,], aes(reorder(TeamName, Titles), Titles)) +
  geom_bar(stat = 'identity', fill = 'steelblue') +
  labs(x = '', y = 'Conference titles', title = 'Conference Titles since 1985') +
  coord_flip()
conf_titles_plot
```  
<img src="{{ site.url }}{{ site.baseurl }}/images/marchmadness/conftitles.jpeg" alt="conference titles">  

## Tournament Rounds Breakdown
Now let’s take a look at what is going on at each level of the tournaments.  

### First Round Upsets

``` r
setkey(seeds, TeamID)

tourney_results <- seeds[, .(Season, WTeamID = TeamID, Winner_seed = as.numeric(substr(Seed, 2, 3)))][tourney, on = c('Season', 'WTeamID')][seeds[, .(Season, LTeamID = TeamID, Loser_seed = as.numeric(substr(Seed, 2, 3)))], on = c('Season', 'LTeamID')]

round1_wins <- tourney_results[DayNum == 136 | DayNum == 137, .(Wins = .N), by = Winner_seed][order(-Winner_seed)]

round1_plot <- ggplot(round1_wins[1:7, ], aes(Winner_seed, Wins)) +
  geom_bar(stat = 'identity', fill = 'steelblue') +
  labs(x = 'Seed', y = 'Wins', title = 'Round of 64 Upsets since 1985') +
  scale_x_continuous(breaks = seq(min(round1_wins$Winner_seed), max(round1_wins$Wins), by = 1)) +
  coord_flip()
round1_plot
```  

<img src="{{ site.url }}{{ site.baseurl }}/images/marchmadness/upsets.jpeg" alt="upsets">  

As expected, the number of upsets trends in the same downwards as the seed rankings goes lower. An interesting note to consider is the 5 v. 12. That matchup has traditionally been one where most of the surpising upsets seem to occur. Looking at the plot, we can see that is somewhat true. There is a jump from the number upsets made by a 13-seed to the number of upsets made by a 12-seed. From there, the number of upsets plateaus a bit, only increasing slightly, until we get to the 9-seed, but a 9-seed winning over a 8-seed may not be considered an upset. (or at least a surprising one)  

### Round Appearances
A overview of how well each seed performs at each round.  

<img src="{{ site.url }}{{ site.baseurl }}/images/marchmadness/appearances.jpeg" alt="appearances">  

## Regular Season performances
A comparison of in-game statistics in the regular season for the winning teams against losing teams.  

<img src="{{ site.url }}{{ site.baseurl }}/images/marchmadness/regseason.jpeg" alt="regular season performance">
