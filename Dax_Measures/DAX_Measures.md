# DAX Measures --- European Football Dashboard 25/26

This document contains the DAX measures used in the **European Football
Dashboard 25/26** Power BI project.

The measures are organized in a dedicated `_measures` table to keep the
semantic model clean and make calculations easier to locate and
maintain.

> **Note:** The explanations below document the measures as implemented
> in the project. They describe the current DAX logic and do not change
> or "correct" the formulas.

------------------------------------------------------------------------

## Table of Contents

1.  [Overview](#overview)
2.  [KPI & Counting Measures](#kpi--counting-measures)
3.  [Goal & Assist Measures](#goal--assist-measures)
4.  [Match & Playing-Time Measures](#match--playing-time-measures)
5.  [Discipline Measures](#discipline-measures)
6.  [Shooting Measures](#shooting-measures)
7.  [Average & Rate Measures](#average--rate-measures)
8.  [Peak / Highlight Measure](#peak--highlight-measure)
9.  [DAX Functions Used](#dax-functions-used)

------------------------------------------------------------------------

# Overview

The dashboard uses explicit DAX measures for its KPIs, charts, tables,
and player/team/competition analysis.

Power BI measures are evaluated dynamically according to the current
filter context, meaning that selections such as **Competition, Club,
Player, Nationality, Position, and Age Group** can affect the displayed
results.

The measures in this project mainly use:

-   `SUM`
-   `DISTINCTCOUNT`
-   `MAX`
-   `AVERAGE`
-   `DIVIDE`
-   `MAXX`
-   `ALL`
-   `CALCULATE`
-   `IF`
-   `VAR`

------------------------------------------------------------------------

# KPI & Counting Measures

## Players count

### DAX

``` dax
Players count =
DISTINCTCOUNT(dim_players[PlayerID])
```

### Purpose

Counts the number of distinct players using `PlayerID`.

### Main use

Used in KPI cards and player/competition/team-level analysis.

------------------------------------------------------------------------

## Clubs count

### DAX

``` dax
Clubs count =
DISTINCTCOUNT(dim_clubs[ClubID])
```

### Purpose

Counts the number of distinct clubs using `ClubID`.

### Main use

Used to display the number of clubs in the selected filter context.

------------------------------------------------------------------------

## Competition count

### DAX

``` dax
Competition count =
DISTINCTCOUNT(dim_competitions[CompID])
```

### Purpose

Counts the number of distinct competitions.

### Main use

Used as a high-level KPI in the Overview page.

------------------------------------------------------------------------

## Countries count

### DAX

``` dax
Countries count =
DISTINCTCOUNT(dim_players[Nationality])
```

### Purpose

Counts the number of distinct player nationalities represented in the
current filter context.

### Main use

Used in the Overview and Team pages.

------------------------------------------------------------------------

# Goal & Assist Measures

## Total goals

### DAX

``` dax
Total goals =
SUM(fact_stats[Goals])
```

### Purpose

Calculates the total number of goals by summing the `Goals` column in
`fact_stats`.

### Main use

Used throughout the dashboard for:

-   Total goals KPI
-   Competition analysis
-   Team analysis
-   Player analysis
-   Goal distribution
-   Goal contribution analysis

------------------------------------------------------------------------

## Total Assists

### DAX

``` dax
Total Assists =
SUM(fact_stats[Assists])
```

### Purpose

Calculates the total number of assists.

### Main use

Used in KPI cards and team/player/competition analysis.

------------------------------------------------------------------------

## Total goalscontribution

### DAX

``` dax
Total goalscontribution =
SUM(fact_stats[GoalsContributions])
```

### Purpose

Calculates the total number of goal contributions by summing goals and
assists represented by the `GoalsContributions` column.

### Main use

Used for player performance analysis and age-based goal contribution
visuals.

------------------------------------------------------------------------

## Penalty goals count

### DAX

``` dax
Penalty goals count =
SUM(fact_stats[PenaltyGoals])
```

### Purpose

Calculates the total number of penalty goals.

### Main use

Used in player-level performance KPIs.

------------------------------------------------------------------------

# Match & Playing-Time Measures

## Total rounds

### DAX

``` dax
Total rounds =
MAX(fact_stats[MatchesPlayed])
```

### Purpose

Returns the maximum value of `MatchesPlayed` within the current filter
context.

### Main use

Used as the rounds component of the competition match calculation.

------------------------------------------------------------------------

## Total matches for competition

### DAX

``` dax
Total matches for competition =
([Clubs count]/2) * [Total rounds]
```

### Purpose

Estimates the total number of competition matches using the number of
clubs and the number of rounds.

The calculation uses:

-   Number of clubs
-   Total rounds
-   Two clubs per match

### Main use

Used for competition-level match analysis and calculating average goals
per match.

------------------------------------------------------------------------

## Total_matches

### DAX

``` dax
Total_matches =
SUM(fact_stats[MatchesPlayed])
```

### Purpose

Sums the `MatchesPlayed` values from the fact table.

### Main use

Used for match-related player and competition analysis.

------------------------------------------------------------------------

## Total minutes

### DAX

``` dax
Total minutes =
SUM(fact_stats[MinutesPlayed])
```

### Purpose

Calculates the total playing minutes represented in the current filter
context.

### Main use

Used as a player performance KPI.

------------------------------------------------------------------------

# Discipline Measures

## Yellow cards count

### DAX

``` dax
Yellow cards count =
SUM(fact_stats[YellowCards])
```

### Purpose

Calculates the total number of yellow cards.

### Main use

Used in player discipline analysis.

------------------------------------------------------------------------

## Red cards count

### DAX

``` dax
Red cards count =
SUM(fact_stats[RedCards])
```

### Purpose

Calculates the total number of red cards.

### Main use

Used in player discipline analysis.

------------------------------------------------------------------------

# Shooting Measures

## TotalShots

### DAX

``` dax
TotalShots =
SUM(fact_stats[Shots])
```

### Purpose

Calculates the total number of shots.

### Main use

Used in player finishing and shooting analysis.

------------------------------------------------------------------------

## ShotsOnTarget

### DAX

``` dax
ShotsOnTarget =
SUM(fact_stats[ShotsOnTarget])
```

### Purpose

Calculates the total number of shots that were on target.

### Main use

Used in player finishing effectiveness analysis.

------------------------------------------------------------------------

# Average & Rate Measures

## Average age

### DAX

``` dax
Average age =
AVERAGE(dim_players[Age])
```

### Purpose

Calculates the average age of players in the current filter context.

### Main use

Used in competition and team comparison.

------------------------------------------------------------------------

## Average goals

### DAX

``` dax
Average goals =
AVERAGE(fact_stats[Goals])
```

### Purpose

Calculates the average goals value across the rows in `fact_stats`
within the current filter context.

### Main use

Used for player/competition performance analysis.

------------------------------------------------------------------------

## Average goals per match

### DAX

``` dax
Average goals per match =
DIVIDE([Total goals], [Total matches for competition])
```

### Purpose

Calculates the average number of goals scored per competition match.

The measure divides total goals by the calculated number of competition
matches.

### Main use

Used in the competition summary analysis.

### Why `DIVIDE`?

`DIVIDE` is used instead of the `/` operator to provide safer division
behavior when the denominator is zero or blank.

------------------------------------------------------------------------

# Peak / Highlight Measure

## Peak Goals Contribution

### DAX

``` dax
Peak Goals Contribution =

VAR MaxValue =
    MAXX(
        ALL('dim_players'[Age]),
        CALCULATE([Total goalscontribution])
    )

VAR CurrentValue =
    [Total goalscontribution]

RETURN
    IF(CurrentValue = MaxValue, CurrentValue, BLANK())
```

### Purpose

Identifies the peak value of total goal contribution across player ages
and returns that value only when the current context matches the
maximum.

If the current value is not the maximum, the measure returns `BLANK()`.

### DAX logic

#### 1. `VAR MaxValue`

``` dax
VAR MaxValue =
    MAXX(
        ALL('dim_players'[Age]),
        CALCULATE([Total goalscontribution])
    )
```

`ALL('dim_players'[Age])` removes the current filter from the `Age`
column so that the measure can evaluate goal contribution across all
player ages.

`MAXX` then finds the highest goal-contribution value across those age
values.

#### 2. `VAR CurrentValue`

``` dax
VAR CurrentValue =
    [Total goalscontribution]
```

Stores the goal contribution for the current filter context.

#### 3. `IF`

``` dax
IF(CurrentValue = MaxValue, CurrentValue, BLANK())
```

Returns the current value only when it equals the maximum value.

Otherwise, the measure returns blank.

### Main use

This measure can be used to highlight the **highest point** on the Goals
Contribution by Player Age line chart.

------------------------------------------------------------------------

# DAX Functions Used

  -----------------------------------------------------------------------
  Function                Used in                 Purpose
  ----------------------- ----------------------- -----------------------
  `SUM`                   Goal, assist, match,    Adds values from a
                          minute, card, shooting  column
                          measures                

  `DISTINCTCOUNT`         Players, clubs,         Counts unique values
                          competitions, countries 

  `MAX`                   Total rounds            Returns the maximum
                                                  value

  `AVERAGE`               Average age, average    Calculates an
                          goals                   arithmetic mean

  `DIVIDE`                Average goals per match Performs safe division

  `MAXX`                  Peak Goals Contribution Evaluates an expression
                                                  over a table and
                                                  returns the maximum

  `ALL`                   Peak Goals Contribution Removes filters from
                                                  the specified column

  `CALCULATE`             Peak Goals Contribution Evaluates an expression
                                                  in a modified filter
                                                  context

  `IF`                    Peak Goals Contribution Returns different
                                                  results based on a
                                                  condition

  `VAR`                   Peak Goals Contribution Stores intermediate
                                                  results for reuse

  `BLANK`                 Peak Goals Contribution Returns an empty result
                                                  when the condition is
                                                  not met
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Measure Organization

All measures are stored in the dedicated:

``` text
_measures
```

table.

This approach keeps calculations separate from the underlying fact and
dimension tables and makes the Power BI model easier to navigate.

The measures are then reused across the dashboard's:

-   KPI cards
-   Charts
-   Tables
-   Player analysis
-   Team analysis
-   Competition analysis
-   Age-group analysis

------------------------------------------------------------------------

# Measure Categories Summary

  -----------------------------------------------------------------------
  Category                            Measures
  ----------------------------------- -----------------------------------
  Counts                              Players count, Clubs count,
                                      Competition count, Countries count

  Goals & Assists                     Total goals, Total Assists, Total
                                      goalscontribution, Penalty goals
                                      count

  Matches & Time                      Total rounds, Total matches for
                                      competition, Total_matches, Total
                                      minutes

  Discipline                          Yellow cards count, Red cards count

  Shooting                            TotalShots, ShotsOnTarget

  Averages                            Average age, Average goals, Average
                                      goals per match

  Highlighting                        Peak Goals Contribution
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Notes

-   These measures are designed to respond to the current Power BI
    filter context.
-   The dashboard's slicers and relationships allow the same measures to
    be reused across different analytical perspectives.
-   `Peak Goals Contribution` is specifically designed to highlight the
    maximum goal-contribution point by age.
-   `Age Group` and `Age Group Sort` are model columns rather than
    measures and therefore are not included in this document.
-   The project uses a dedicated `_measures` table to organize the DAX
    layer of the semantic model.

------------------------------------------------------------------------

## References

-   [Microsoft Learn --- Measures in Power BI
    Desktop](https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-measures)
-   [Microsoft Learn --- DAX
    Overview](https://learn.microsoft.com/en-us/dax/dax-overview)
-   [Microsoft Learn --- DAX Function
    Reference](https://learn.microsoft.com/en-us/dax/dax-function-reference)
