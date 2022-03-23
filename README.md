# What Makes Certain Basketball Teams Successful?

The use of data and analytics within professional sports has grown massively in the past several years. Data has become front and center for NBA games where almost every decision made in a game is based on analytics. In this spirit, we will analyze the NBA games' play-by-play data from `bigdataball.com`.

In this project, we would like to focus on the following question - **do great players make their teammates better?**

Some of the star players in today's NBA include LeBron James, Stephen Curry, Kevin Durant, Giannis Antetokounmpo, Nikola Jokic, and so on. A common saying about such star players is that they make their teammates better. To answer the first question, we would want to find evidence from the play-by-play data that supports the notion that star players indeed make their teammates better.

After some careful thinking, we have decided to focus on teammmates' Effective Field Goal Percentage (**eFG%**) while a star player is on or off the court. eFG%, in short, describes how successful one's team is from the field. We want to examine if the star player's presence on the court improves the eFG% of his teammates...

## Custom Functions
We will create some functions to help with our analysis.

The *filter_on* function below will filter the game's data frame down to just the records for which a given player is on the court.

```python
def filter_on(df, nm, h_o_a, tm):
    if(h_o_a == 'home'):
        row = df[((df['h1'] == nm) | (df['h2'] == nm) | (df['h3'] == nm) | (df['h4'] == nm) | (df['h5'] == nm)) & (df['Home'] == tm)]
        return row
    elif(h_o_a == 'away'):
        row = df[((df['a1'] == nm) | (df['a2'] == nm) | (df['a3'] == nm) | (df['a4'] == nm) | (df['a5'] == nm)) & (df['Away'] == tm)]
        return row
    else:
        raise TypeError('Error')
```

Let's test our *filter_on* function. We will only want to filter out the portion of our data frame that contains **Stephen Curry** in the **away** team.

```python
# testing the filter_on function
x = filter_on(df, 'Stephen Curry', 'away', 'GSW')
x[['h1', 'h2', 'h3', 'h4', 'h5', 'a1', 'a2', 'a3', 'a4', 'a5']].head()
```

![img1](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img1.png)

Looks like our *filter_on* function has worked!

The *filter_off* function below will do the same for when a given player is off the court. For this function, we will need to feed in the player's team name as one of the inputs because we do not want to filter out the part of the dataframe that is not the player's team.

```python
def filter_off(df, nm, h_o_a, tm):
    if(h_o_a == 'home'):
        row = df[(df['h1'] != nm) & (df['h2'] != nm) & (df['h3'] != nm) & (df['h4'] != nm) & (df['h5'] != nm) & (df['Home'] == tm)]
        return row
    elif(h_o_a == 'away'):
        row = df[(df['a1'] != nm) & (df['a2'] != nm) & (df['a3'] != nm) & (df['a4'] != nm) & (df['a5'] != nm) & (df['Away'] == tm)]
        return row
    else:
        raise TypeError('Error')
```

```python
# testing the filter_off function
x = filter_off(df, 'Stephen Curry', 'home', 'GSW')
x[['h1', 'h2', 'h3', 'h4', 'h5', 'a1', 'a2', 'a3', 'a4', 'a5', 'Home', 'Away']].head()
```

![img2](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img2.png)


In order to describe success that certain star players bring on the court, we have chosen to focus on the shooting percentage of *just the teammates* while a player is on vs. off the court. The *filter_eFG* function below will filter to just the shots attempted or made by the player's team that were **not** attempted/made by the player himself.

```python
def filter_eFG(df, nm = 'name', tm = 'teamAbbrev'):
        return df[(df['player'] != nm) & (df['team'] == tm) & ((df['event_type'] == 'shot') | (df['event_type'] == 'miss'))]
```

```python
# testing the filter_eFG function
x = filter_eFG(df, 'Stephen Curry', 'GSW')
x[['player', 'event_type', 'team', 'h1', 'h2', 'h3', 'h4', 'h5', 'a1', 'a2', 'a3', 'a4', 'a5']].head()
```

![img3](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img3.png)

We will also create a function called *eFG* to calculate the eFG: `eFG% = (FGM + 0.5 * 3PM) / FGA`
- Field Goals Made (**FGM**): The number of field goals that a player or team has made. This includes both 2 pointers and 3 pointers
- 3 Point Field Goals Made (**3PM**): The number of 3 point field goals that a player or team has made
- Field Goals Attempted (**FGA**): The number of field goals that a player or team has attempted. This includes both 2 pointers and 3 pointers

```python
def eFG(df):
    numerator = df.loc[df['event_type'] == 'shot', 'points'].count() + 0.5 * df.loc[df['points'] == 3, 'points'].count()
    denominator = df.loc[df['event_type'] == 'shot', 'points'].count() + df.loc[df['event_type'] == 'miss', 'points'].count()
    eFG = (numerator) / (denominator)
    return eFG
```

```python
# testing the eFG function
test_df = pd.DataFrame({'event_type': ['shot', 'shot', 'shot', 'shot', 'shot', 'miss', 'miss', 'miss', 'miss', 'miss'],
                   'points': [2, 2, 2, 2, 2, 0, 0, 0, 0, 0]})
test_df
```

<p align="center">
    <img src="https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img4.png" height=450, width=250>
</p>

Applying the *eFG* function to the above data frame should return 0.5 according to the formula.
FGM = 5, 3PM = 0, FGA = 10. Therefore eFG = (5 + 0.5 * 0) / 10 = 0.5.

```python
x = eFG(test_df)
print(x)
0.5
```

Looks like our *eFG* function has worked!

Finally, we will write a function to return the relative eFG% and a few more relevant pieces of information. We are interested in returning the number of shots attempted by teammates while a player is on/off the court.

<!-- If there’s a large discrepancy, which indicates that a player is either on the court or off the court for the vast majority of the game, the relative eFG% may be unreliable due to random fluctuations. We will want to account for that in our analysis below. -->

```python
def eFG_Ratio(df, nm, h_o_a, tm):

    df_on = filter_on(df, nm, h_o_a, tm)
    temp_on = filter_eFG(df_on, nm, tm)
    eFG_on = eFG(temp_on)
    
    df_off = filter_off(df, nm, h_o_a, tm)
    temp_off = filter_eFG(df_off, nm, tm)
    eFG_off = eFG(temp_off)
    
    eFG_Ratio = eFG_on / eFG_off
    
    return eFG_Ratio
```

```python
# testing the eFG_Ratio function
x = eFG_Ratio(df, 'LeBron James', 'home', 'CLE')
print(x)
1.116782109851913
```

Let's create a test for the *eFG_Ratio* function. The test data frame below contains only two types of events (i.e., 'shot' and 'miss'). We will be focusing on how the presence of SC affects the eFG% of the rest of the team. The player who is making the shot/missing the shot will be DG throughout the test.

When SC is on the court, the player makes one 2-pt shot and one 3-pt shot with a 2/4 shooting, giving a 0.625 eFG%. When SC is off the court, the player makes two 3-pt shots with a 2/4 shooting, giving a 0.75 eFG%. eFG_Ratio function will calculate the ratio between eFG_on to eFG_off. As such, the *eFG_Ratio* function is expected to spit out 0.625 / 0.75, which is **0.833**.

```python
# eFG_Ratio test 1
test_df = pd.DataFrame({'player': ['DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'DG'],
                    'h1': ['SC', 'SC', 'SC', 'SC', 'JP', 'JP', 'JP', 'JP'],
                    'h2': ['DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'DG'],
                    'h3': ['B', 'B', 'B', 'B', 'B', 'B', 'B', 'B'],
                    'h4': ['C', 'C', 'C', 'C', 'C', 'C', 'C', 'C'],
                    'h5': ['D', 'D', 'D', 'D', 'D', 'D', 'D', 'D'],
                    'a1': ['E', 'E', 'E', 'E', 'E', 'E', 'E', 'E'],
                    'a2': ['F', 'F', 'F', 'F', 'F', 'F', 'F', 'F'],
                    'a3': ['G', 'G', 'G', 'G', 'G', 'G', 'G', 'G'],
                    'a4': ['H', 'H', 'H', 'H', 'H', 'H', 'H', 'H'],
                    'a5': ['I', 'I', 'I', 'I', 'I', 'I', 'I', 'I'],
                    'event_type': ['shot', 'shot', 'miss', 'miss', 'shot', 'shot', 'miss', 'miss'],
                    'points': [2, 3, 0, 0, 3, 3, 0, 0],
                    'team': ['GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW'],
                    'Home': ['GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW'],
                    'Away': ['LAL', 'LAL', 'LAL', 'LAL', 'LAL', 'LAL', 'LAL', 'LAL']
                   })
test_df
```

![img9](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img9.png)

```python
# calculating the eFG_Ratio for the test_df above
x = eFG_Ratio(test_df, 'SC', 'home', 'GSW')
print(x)
0.8333333333333334
```

```python
# eFG_Ratio test 2
test_df = pd.DataFrame({'player': ['DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'LBJ'],
                    'h1': ['SC', 'SC', 'SC', 'SC', 'JP', 'JP', 'JP', 'JP', 'SC'],
                    'h2': ['DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'DG', 'DG'],
                    'h3': ['B', 'B', 'B', 'B', 'B', 'B', 'B', 'B', 'J'],
                    'h4': ['C', 'C', 'C', 'C', 'C', 'C', 'C', 'C', 'K'],
                    'h5': ['D', 'D', 'D', 'D', 'D', 'D', 'D', 'D', 'L'],
                    'a1': ['E', 'E', 'E', 'E', 'E', 'E', 'E', 'E', 'LBJ'],
                    'a2': ['F', 'F', 'F', 'F', 'F', 'F', 'F', 'F', 'N'],
                    'a3': ['G', 'G', 'G', 'G', 'G', 'G', 'G', 'G', 'O'],
                    'a4': ['H', 'H', 'H', 'H', 'H', 'H', 'H', 'H', 'P'],
                    'a5': ['I', 'I', 'I', 'I', 'I', 'I', 'I', 'I', 'Q'],
                    'event_type': ['shot', 'shot', 'miss', 'miss', 'shot', 'shot', 'miss', 'miss', 'shot'],
                    'points': [2, 3, 0, 0, 3, 3, 0, 0, 3],
                    'team': ['GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'CLE'],
                    'Home': ['GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW', 'GSW'],
                    'Away': ['LAL', 'LAL', 'LAL', 'LAL', 'LAL', 'LAL', 'LAL', 'LAL', 'CLE']
                   })
test_df
```

![img10](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img10.png)

```python
x = eFG_Ratio(test_df, 'SC', 'home', 'GSW')
print(x)
0.8333333333333334
```

## Time to plot!

We will take a look at the eFG% ratio of each player from the following teams: Golden State Warriors, Houston Rockets, Cleveland Cavaliers, and Boston Celtics. We will select these four teams for the analysis as they are all winners of the Conference Semifinals in the 2018 NBA playoffs.

By plotting eFG ratio of each player for these teams, we are attempting to understand whether a player's presence on the couurt makes his teammates more effective shooters. Star players like Stephen Curry, LeBron James, and James Harden tend to draw a lot of attention from the other team's defenders, and that attention may give more opportunities for their teammates for a better shot. As an example, Golden State Warriors designs a lot of plays based on making Stephen Curry the decoy on the court, freeing up other shooters like Klay Thompson.

For the purpose of the project, we will define star players for each of these four teams based on their popularity and past NBA All-Star Game appearance.

- Golden State Warriors: Stephen Curry, Kevin Durant
- Houston Rockets: James Harden, Chris Paul
- Cleveland Cavaliers: LeBron James
- Boston Celtics: Jayson Tatum, Kyrie Irving

Finally, we will only be selecting data for **home** games only.

![img5](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img5.png)

![img6](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img6.png)

![img7](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img7.png)

![img8](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img8.png)

## Summary and Future Outlook

From the plots, we can see that all of the **star players** for the four teams had an eFG ratio larger than 1, meaning that their presence on the court made their teammates shoot better than when they were off the court. For example, Kevin Durant from the Golden State Warriors had an eFG ratio of 1.11, suggesting that Durant's presence on the court made his teammates shoot 11% better compared to when he was off the court.

One factor that we did not account for in this analysis is the time each player has spent on the court. The standard for full credibility is when the player spent an equal amount of time on the court as off the court. The more disproportionate the on-the-court plays are vs. the off-the-court plays, the lower the credibility. This might explain why players that are not commonly considered as star players such as Kendrick Perkins from Cleveland Cavaliers or RJ Hunter from the Houston Rockets showed an unexpectedly high eFG ratio in the analysis. We could incorporate the credibility facotr in our future analysis.