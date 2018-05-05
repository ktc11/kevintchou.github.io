---
title: NCAA March Madness 2018 - Exploratory Data Analysis
date: 2018-03-24
tags: [Data Analytics]
header:
  image: "/image/marchmadness/marchmadness.jpg"
excerpt: A brief exploratory data analysis of the 2018 NCAA Men’s Basketball tournament dataset.
toc: true
toc_label: Table of Contents
---

This is a brief exploratory data analysis of the 2018 NCAA Men’s Basketball tournament dataset.
The datasets are pulled from the Kaggle and Google Cloud NCAA ML 2018 Competition.  

## File Descriptions
```r
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
