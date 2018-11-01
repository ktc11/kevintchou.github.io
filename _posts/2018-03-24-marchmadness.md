---
title: NCAA March Madness 2018 - Exploratory Data Analysis
date: 2018-03-24
tags: [Data Analytics]
header:
  image: "/images/marchmadness/marchmadness.jpg"
excerpt: A brief exploratory data analysis of the 2018 NCAA Men’s Basketball tournament dataset.
toc: true
toc_label: Table of Contents
---

-   [Importing the libraries and data](#importing-the-libraries-and-data)
    -   [File Descriptions and Preview](#file-descriptions-and-preview)
-   [Historical Tournament Performances](#historical-tournament-performances)
    -   [Number 1 Seeds](#number-1-seeds)
    -   [Regular Season Wins](#regular-season-wins)
    -   [National Titles](#national-titles)
    -   [Conference Titles](#conference-titles)
    -   [A side-by-side comparison](#a-side-by-side-comparison)
-   [Tournament Rounds Breakdown](#tournament-rounds-breakdown)
    -   [First Round Upsets](#first-round-upsets)
    -   [Round Appearances](#round-appearances)
-   [Regular Season Performances](#regular-season-performances)

This is a brief exploratory data analysis of the 2018 NCAA Men's Basketball tournament dataset. Before we create our prediction model, we will get a quick breakdown of the datasets and assess what we should look for. The datasets are pulled from the Kaggle and Google Cloud NCAA ML 2018 Competition.

Importing the libraries and data
--------------------------------

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

### File Descriptions and Preview

**Teams.csv**
This file identifies the different college teams present in the dataset. Each team has a 4-digit ID number.

``` r
glimpse(teams)
```

    ## Observations: 364
    ## Variables: 4
    ## $ TeamID        <int> 1101, 1102, 1103, 1104, 1105, 1106, 1107, 1108, ...
    ## $ TeamName      <chr> "Abilene Chr", "Air Force", "Akron", "Alabama", ...
    ## $ FirstD1Season <int> 2014, 1985, 1985, 1985, 2000, 1985, 2000, 1985, ...
    ## $ LastD1Season  <int> 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, ...

-   **TeamID** - a 4-digit ID number. A school's ID remains the same across all years
-   **TeamName** - a compact spelling of the school name
-   **FirstD1Season** - the first season in the dataset that the school was Division-I
-   **LastD1Season** - the last season in the dataset that the school was Dvision-I

**NCAATourneySeeds.csv**
This file identifies the seeds for all teams in each NCAA tournament, for all seasons in historical data.

``` r
glimpse(seeds)
```

    ## Observations: 2,150
    ## Variables: 3
    ## $ Season <int> 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1...
    ## $ Seed   <chr> "W01", "W02", "W03", "W04", "W05", "W06", "W07", "W08",...
    ## $ TeamID <int> 1207, 1210, 1228, 1260, 1374, 1208, 1393, 1396, 1439, 1...

-   **Season** - the year the tournament was played
-   **Seed** - the seeding information of the team. The first character indicates the region the team was in. (W, X, Y, Z)
-   **TeamID** - the 4-digit ID number for the team

**Conferences.csv**
This file indicates the Division-I conferences that have existed over the years since 1985. Each conference is listed with an abbreviation and a longer name.

``` r
glimpse(conferences)
```

    ## Observations: 51
    ## Variables: 2
    ## $ ConfAbbrev  <chr> "a_sun", "a_ten", "aac", "acc", "aec", "asc", "awc...
    ## $ Description <chr> "Atlantic Sun Conference", "Atlantic 10 Conference...

-   **ConfAbbrev** - the abbreviation for the conference
-   **Description** - the longer name for the conference

**TeamConferences.csv**
This file indicates the conference affiliations for each team during each season.

``` r
glimpse(team_conferences)
```

    ## Observations: 10,888
    ## Variables: 3
    ## $ Season     <int> 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, 198...
    ## $ TeamID     <int> 1114, 1147, 1204, 1209, 1215, 1223, 1273, 1359, 118...
    ## $ ConfAbbrev <chr> "a_sun", "a_sun", "a_sun", "a_sun", "a_sun", "a_sun...

-   **Season** - the year of the associated entry
-   **TeamID** - the 4-digit ID number for the team
-   **ConfAbbrev** - the abbreviation for the conference

**RegularSeasonCompactResults**
This file identifies the game-by-game results for seasons since 1985.

``` r
glimpse(season)
```

    ## Observations: 150,684
    ## Variables: 8
    ## $ Season  <int> 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, ...
    ## $ DayNum  <int> 20, 25, 25, 25, 25, 25, 25, 25, 25, 25, 25, 25, 25, 25...
    ## $ WTeamID <int> 1228, 1106, 1112, 1165, 1192, 1218, 1228, 1242, 1260, ...
    ## $ WScore  <int> 81, 77, 63, 70, 86, 79, 64, 58, 98, 97, 103, 75, 91, 7...
    ## $ LTeamID <int> 1328, 1354, 1223, 1432, 1447, 1337, 1226, 1268, 1133, ...
    ## $ LScore  <int> 64, 70, 56, 54, 74, 78, 44, 56, 80, 89, 71, 71, 72, 65...
    ## $ WLoc    <chr> "N", "H", "H", "H", "H", "H", "N", "N", "H", "H", "H",...
    ## $ NumOT   <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...

-   **Season** - the year of the associated entry
-   **DayNum** - the day the game was played on. The integer value ranges from 0-132. Day 132 is Selection Sunday
-   **WTeamID** - the ID number of the team that won the game, as listed in the "Teams.csv" file
-   **WScore** - the number of points scored by the winning team
-   **LTeamID** - the ID number of the team that lost the game, as list in the "Teams.csv" file
-   **LScore** - the number of points score by the losing team
-   **NumOT** - number of overtime periods in the game
-   **WLoc** - the location of the winning team. "H" for home game, "A" for away game, "N" for game played in neutral court

**RegularSeasonDetailedResults.csv**
This file is like the RegularSeasonCompactResults.csv but it contains additional information for each game. Since many of the variables are redundant and self-exlanatory, we will skip the descriptions.

``` r
glimpse(season_detail)
```

    ## Observations: 76,636
    ## Variables: 34
    ## $ Season  <int> 2003, 2003, 2003, 2003, 2003, 2003, 2003, 2003, 2003, ...
    ## $ DayNum  <int> 10, 10, 11, 11, 11, 11, 12, 12, 12, 12, 13, 13, 13, 13...
    ## $ WTeamID <int> 1104, 1272, 1266, 1296, 1400, 1458, 1161, 1186, 1194, ...
    ## $ WScore  <int> 68, 70, 73, 56, 77, 81, 80, 75, 71, 84, 106, 74, 66, 7...
    ## $ LTeamID <int> 1328, 1393, 1437, 1457, 1208, 1186, 1236, 1457, 1156, ...
    ## $ LScore  <int> 62, 63, 61, 50, 71, 55, 62, 61, 66, 56, 50, 73, 65, 48...
    ## $ WLoc    <chr> "N", "N", "N", "N", "N", "H", "H", "N", "N", "H", "H",...
    ## $ NumOT   <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, ...
    ## $ WFGM    <int> 27, 26, 24, 18, 30, 26, 23, 28, 28, 32, 41, 29, 26, 25...
    ## $ WFGA    <int> 58, 62, 58, 38, 61, 57, 55, 62, 58, 67, 69, 51, 66, 56...
    ## $ WFGM3   <int> 3, 8, 8, 3, 6, 6, 2, 4, 5, 5, 15, 7, 5, 10, 11, 10, 5,...
    ## $ WFGA3   <int> 14, 20, 18, 9, 14, 12, 8, 14, 11, 17, 25, 13, 19, 23, ...
    ## $ WFTM    <int> 11, 10, 17, 17, 11, 23, 32, 15, 10, 15, 9, 9, 9, 16, 1...
    ## $ WFTA    <int> 18, 19, 29, 31, 13, 27, 39, 21, 18, 19, 13, 11, 13, 23...
    ## $ WOR     <int> 14, 15, 17, 6, 17, 12, 13, 13, 9, 14, 15, 6, 21, 8, 15...
    ## $ WDR     <int> 24, 28, 26, 19, 22, 24, 18, 35, 22, 22, 29, 21, 23, 35...
    ## $ WAst    <int> 13, 16, 15, 11, 12, 12, 14, 19, 9, 11, 21, 18, 15, 18,...
    ## $ WTO     <int> 23, 13, 10, 12, 14, 9, 17, 19, 17, 6, 11, 15, 17, 13, ...
    ## $ WStl    <int> 7, 4, 5, 14, 4, 9, 11, 7, 9, 12, 10, 7, 12, 14, 18, 2,...
    ## $ WBlk    <int> 1, 4, 2, 2, 4, 3, 1, 2, 2, 0, 6, 1, 3, 19, 5, 6, 3, 5,...
    ## $ WPF     <int> 22, 18, 25, 18, 20, 18, 25, 21, 23, 13, 16, 5, 17, 13,...
    ## $ LFGM    <int> 22, 24, 22, 18, 24, 20, 19, 20, 24, 23, 17, 29, 24, 18...
    ## $ LFGA    <int> 53, 67, 73, 49, 62, 46, 41, 59, 52, 52, 52, 63, 56, 64...
    ## $ LFGM3   <int> 2, 6, 3, 6, 6, 3, 4, 4, 6, 3, 4, 10, 6, 8, 4, 7, 2, 2,...
    ## $ LFGA3   <int> 10, 24, 26, 22, 16, 11, 15, 17, 18, 14, 11, 22, 19, 24...
    ## $ LFTM    <int> 16, 9, 14, 8, 17, 12, 20, 17, 12, 7, 12, 5, 11, 4, 17,...
    ## $ LFTA    <int> 22, 20, 23, 15, 27, 17, 28, 23, 27, 12, 17, 5, 17, 8, ...
    ## $ LOR     <int> 10, 20, 31, 17, 21, 6, 9, 8, 13, 9, 8, 13, 14, 14, 20,...
    ## $ LDR     <int> 22, 25, 22, 20, 15, 22, 21, 25, 26, 23, 15, 16, 21, 26...
    ## $ LAst    <int> 8, 7, 9, 9, 12, 8, 11, 10, 13, 10, 8, 15, 17, 12, 17, ...
    ## $ LTO     <int> 18, 12, 12, 19, 10, 19, 30, 15, 25, 18, 17, 12, 18, 17...
    ## $ LStl    <int> 9, 8, 2, 4, 7, 4, 10, 14, 8, 1, 7, 6, 8, 10, 7, 5, 15,...
    ## $ LBlk    <int> 2, 6, 5, 3, 1, 3, 4, 8, 2, 3, 3, 2, 4, 0, 7, 1, 3, 4, ...
    ## $ LPF     <int> 20, 16, 23, 23, 14, 25, 28, 18, 18, 18, 15, 12, 13, 17...

**NCAATourneyCompactResults.csv**
This file identifies the game-by-game results for NCAA tournaments since 1985.

``` r
glimpse(tourney)
```

    ## Observations: 2,117
    ## Variables: 8
    ## $ Season  <int> 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, ...
    ## $ DayNum  <int> 136, 136, 136, 136, 136, 136, 136, 136, 136, 136, 136,...
    ## $ WTeamID <int> 1116, 1120, 1207, 1229, 1242, 1246, 1256, 1260, 1314, ...
    ## $ WScore  <int> 63, 59, 68, 58, 49, 66, 78, 59, 76, 79, 75, 96, 85, 83...
    ## $ LTeamID <int> 1234, 1345, 1250, 1425, 1325, 1449, 1338, 1233, 1292, ...
    ## $ LScore  <int> 54, 58, 43, 55, 38, 58, 54, 58, 57, 70, 64, 83, 68, 59...
    ## $ WLoc    <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "N",...
    ## $ NumOT   <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...

The DayNum is the day in which they play the tournament:

-   134 or 135: last-four in games
-   136 or 137: Round of 64
-   138 or 139: Round of 32
-   143 or 144: Sweet 16
-   145 or 146: Elite 8
-   152: Final 4
-   154: National Championship

**NCAATourneyDetailedResults.csv**
This file is like the NCAATourneyCompactResults.csv but it contains additional information for each game. Since many of the variables are redundant and self-exlanatory, we will skip the descriptions.

``` r
glimpse(tourney_detail)
```

    ## Observations: 981
    ## Variables: 34
    ## $ Season  <int> 2003, 2003, 2003, 2003, 2003, 2003, 2003, 2003, 2003, ...
    ## $ DayNum  <int> 134, 136, 136, 136, 136, 136, 136, 136, 136, 136, 136,...
    ## $ WTeamID <int> 1421, 1112, 1113, 1141, 1143, 1163, 1181, 1211, 1228, ...
    ## $ WScore  <int> 92, 80, 84, 79, 76, 58, 67, 74, 65, 64, 72, 72, 70, 71...
    ## $ LTeamID <int> 1411, 1436, 1272, 1166, 1301, 1140, 1161, 1153, 1443, ...
    ## $ LScore  <int> 84, 51, 71, 73, 74, 53, 57, 69, 60, 61, 68, 71, 69, 54...
    ## $ WLoc    <chr> "N", "N", "N", "N", "N", "N", "N", "N", "N", "N", "N",...
    ## $ NumOT   <int> 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, ...
    ## $ WFGM    <int> 32, 31, 31, 29, 27, 17, 19, 20, 24, 28, 22, 28, 23, 24...
    ## $ WFGA    <int> 69, 66, 59, 53, 64, 52, 54, 47, 56, 51, 51, 52, 54, 52...
    ## $ WFGM3   <int> 11, 7, 6, 3, 7, 4, 4, 6, 5, 2, 9, 5, 3, 10, 8, 8, 6, 7...
    ## $ WFGA3   <int> 29, 23, 14, 7, 20, 14, 13, 14, 14, 6, 16, 13, 13, 18, ...
    ## $ WFTM    <int> 17, 11, 16, 18, 15, 20, 25, 28, 12, 6, 19, 11, 21, 13,...
    ## $ WFTA    <int> 26, 14, 22, 25, 23, 27, 31, 37, 14, 11, 23, 18, 25, 24...
    ## $ WOR     <int> 14, 11, 10, 11, 18, 12, 13, 8, 15, 7, 11, 9, 11, 11, 1...
    ## $ WDR     <int> 30, 36, 27, 20, 20, 29, 27, 28, 23, 20, 20, 32, 33, 24...
    ## $ WAst    <int> 17, 22, 18, 15, 17, 8, 4, 12, 15, 13, 14, 7, 7, 16, 16...
    ## $ WTO     <int> 12, 16, 9, 18, 13, 14, 16, 12, 14, 11, 10, 23, 20, 14,...
    ## $ WStl    <int> 5, 10, 7, 13, 8, 3, 10, 2, 11, 8, 4, 4, 6, 8, 4, 6, 7,...
    ## $ WBlk    <int> 3, 7, 4, 1, 2, 8, 8, 2, 4, 4, 3, 6, 6, 3, 8, 1, 2, 5, ...
    ## $ WPF     <int> 22, 8, 19, 19, 14, 16, 23, 15, 14, 17, 23, 19, 19, 12,...
    ## $ LFGM    <int> 29, 20, 25, 27, 25, 20, 18, 26, 22, 23, 24, 26, 23, 21...
    ## $ LFGA    <int> 67, 64, 69, 60, 56, 64, 54, 66, 58, 56, 54, 65, 66, 52...
    ## $ LFGM3   <int> 12, 4, 7, 7, 9, 2, 3, 10, 8, 6, 5, 8, 8, 6, 4, 9, 7, 6...
    ## $ LFGA3   <int> 31, 16, 28, 17, 21, 17, 11, 27, 24, 17, 15, 19, 23, 20...
    ## $ LFTM    <int> 14, 7, 14, 12, 15, 11, 18, 7, 8, 9, 15, 11, 15, 6, 25,...
    ## $ LFTA    <int> 31, 7, 21, 17, 20, 13, 22, 10, 13, 10, 25, 21, 20, 8, ...
    ## $ LOR     <int> 17, 8, 20, 14, 10, 15, 11, 13, 17, 13, 14, 11, 14, 7, ...
    ## $ LDR     <int> 28, 26, 22, 17, 26, 26, 24, 22, 18, 19, 20, 18, 23, 22...
    ## $ LAst    <int> 16, 12, 11, 20, 16, 11, 8, 13, 10, 13, 14, 19, 15, 8, ...
    ## $ LTO     <int> 15, 17, 12, 21, 14, 11, 19, 10, 14, 13, 9, 13, 12, 18,...
    ## $ LStl    <int> 5, 10, 2, 6, 5, 8, 5, 7, 6, 6, 4, 6, 11, 7, 3, 3, 4, 8...
    ## $ LBlk    <int> 0, 3, 5, 6, 8, 4, 4, 6, 5, 1, 1, 0, 3, 2, 0, 3, 0, 4, ...
    ## $ LPF     <int> 22, 15, 18, 21, 19, 22, 19, 24, 16, 15, 20, 19, 25, 23...

**ConferenceTourneyGames.csv** This file indicates which games were part of each year's post-season conference tournaments (all of which finished on Selection Sunday or earlier), starting from the 2001 season.

``` r
glimpse(conference_tourney)
```

    ## Observations: 4,563
    ## Variables: 5
    ## $ Season     <int> 2001, 2001, 2001, 2001, 2001, 2001, 2001, 2001, 200...
    ## $ ConfAbbrev <chr> "a_sun", "a_sun", "a_sun", "a_sun", "a_sun", "a_sun...
    ## $ DayNum     <int> 121, 121, 122, 122, 122, 122, 123, 123, 124, 128, 1...
    ## $ WTeamID    <int> 1194, 1416, 1209, 1359, 1391, 1407, 1209, 1407, 120...
    ## $ LTeamID    <int> 1144, 1240, 1194, 1239, 1273, 1416, 1359, 1391, 140...

DayNum = 132 is the final day listed. The game played on day 132 is the conference championship game.

Historical Tournament Performances
----------------------------------

Now that we have a good idea of the data we are working with, let's start diving in some analysis. We will begin by assessing historical tournament performances.

### Number 1 Seeds

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

![](eda_files/figure-markdown_github/no.1%20seeds-1.png)

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

![](eda_files/figure-markdown_github/regular%20season%20wins-1.png)

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

![](eda_files/figure-markdown_github/national%20titles-1.png)

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

![](eda_files/figure-markdown_github/conference%20titles-1.png)

### A side-by-side comparison

![](eda_files/figure-markdown_github/plots-1.png)

Tournament Rounds Breakdown
---------------------------

Now let's take a look at what is going on at each level of the tournaments.

### First Round Upsets

Breaking down the how many times a team got upset in the first round and by which seed

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

![](eda_files/figure-markdown_github/first%20round%20upsets-1.png)

As expected, the number of upsets trends in the same downwards as the seed rankings goes lower. An interesting note to consider is the 5 v. 12. That matchup has traditionally been one where most of the surpising upsets seem to occur. Looking at the plot, we can see that is somewhat true. There is a jump from the number upsets made by a 13-seed to the number of upsets made by a 12-seed. From there, the number of upsets plateaus a bit, only increasing slightly, until we get to the 9-seed, but a 9-seed winning over a 8-seed may not be considered an upset. (or at least a surprising one)

### Round Appearances

A overview of how well each seed performs at each round.

<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:right;">
Seed
</th>
<th style="text-align:right;">
Round of 32
</th>
<th style="text-align:right;">
Sweet 16
</th>
<th style="text-align:right;">
Elite 8
</th>
<th style="text-align:right;">
Final 4
</th>
<th style="text-align:right;">
National Championship
</th>
<th style="text-align:right;">
Titles Won
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
132
</td>
<td style="text-align:right;">
114
</td>
<td style="text-align:right;">
92
</td>
<td style="text-align:right;">
54
</td>
<td style="text-align:right;">
32
</td>
<td style="text-align:right;">
20
</td>
</tr>
<tr>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
124
</td>
<td style="text-align:right;">
83
</td>
<td style="text-align:right;">
61
</td>
<td style="text-align:right;">
28
</td>
<td style="text-align:right;">
13
</td>
<td style="text-align:right;">
5
</td>
</tr>
<tr>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
111
</td>
<td style="text-align:right;">
68
</td>
<td style="text-align:right;">
32
</td>
<td style="text-align:right;">
15
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
4
</td>
</tr>
<tr>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
106
</td>
<td style="text-align:right;">
63
</td>
<td style="text-align:right;">
21
</td>
<td style="text-align:right;">
13
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
1
</td>
</tr>
<tr>
<td style="text-align:right;">
5
</td>
<td style="text-align:right;">
85
</td>
<td style="text-align:right;">
43
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
83
</td>
<td style="text-align:right;">
42
</td>
<td style="text-align:right;">
14
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
1
</td>
</tr>
<tr>
<td style="text-align:right;">
7
</td>
<td style="text-align:right;">
81
</td>
<td style="text-align:right;">
25
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
</tr>
<tr>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
67
</td>
<td style="text-align:right;">
13
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
1
</td>
</tr>
<tr>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
65
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
51
</td>
<td style="text-align:right;">
23
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:right;">
11
</td>
<td style="text-align:right;">
49
</td>
<td style="text-align:right;">
20
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:right;">
12
</td>
<td style="text-align:right;">
47
</td>
<td style="text-align:right;">
20
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:right;">
13
</td>
<td style="text-align:right;">
26
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:right;">
14
</td>
<td style="text-align:right;">
21
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:right;">
15
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
0
</td>
</tr>
</tbody>
</table>
Regular Season Performances
---------------------------

![](eda_files/figure-markdown_github/regular%20season-1.png)![](eda_files/figure-markdown_github/regular%20season-2.png)![](eda_files/figure-markdown_github/regular%20season-3.png)![](eda_files/figure-markdown_github/regular%20season-4.png)![](eda_files/figure-markdown_github/regular%20season-5.png)![](eda_files/figure-markdown_github/regular%20season-6.png)![](eda_files/figure-markdown_github/regular%20season-7.png)![](eda_files/figure-markdown_github/regular%20season-8.png)![](eda_files/figure-markdown_github/regular%20season-9.png)![](eda_files/figure-markdown_github/regular%20season-10.png)
