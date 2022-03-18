# What Makes Certain Basketball Teams Successful?

The use of data and analytics within professional sports has grown massively in the past several years. Data has become front and center for NBA games where almost every decision made in a game is based on analytics. In this spirit, we will analyze the NBA games' play-by-play data from `bigdataball.com`.

In this project, we would like to focus on the two following questions:
1. Do great players make their teammates better?
2. How does a team best use a "power partnership"?

Some of the star players in today's NBA include LeBron James, Stephen Curry, Kevin Durant, Giannis Antetokounmpo, Nikola Jokic, and so on. A common saying about such star players is that they make their teammates better. To answer the first question, we would want to find evidence from the play-by-play data that supports the notion that star players indeed make their teammates better.

After some careful thinking, we have decided to focus on teammmates' Effective Field Goal Percentage (**eFG%**) while a star player is on or off the court. eFG%, in short, describes how successful one's team is from the field. We want to examine if the star player's presence on the court improves the eFG% of his teammates...

## Custom Functions
We will create some functions to help with our analysis.

The filter_on function below will filter the game's data frame down to just the records for which a given player is on the court.

```
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

Let's test our filter_on function. We will only want to filter out the portion of our data frame that contains **Stephen Curry** in the **away** team.

```
# testing the filter_on function
x = filter_on(df, 'Stephen Curry', 'away', 'GSW')
x[['h1', 'h2', 'h3', 'h4', 'h5', 'a1', 'a2', 'a3', 'a4', 'a5']]
```

![img1](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img1.png)

Looks like our filter_on function has worked!

The filter_off function below will do the same for when a given player is off the court. For this function, we will need to feed in the player's team name as one of the inputs because we do not want to filter out the part of the dataframe that is not the player's team.

```
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

```
# testing the filter_off function
x = filter_off(df, 'Stephen Curry', 'home', 'GSW')
x[['h1', 'h2', 'h3', 'h4', 'h5', 'a1', 'a2', 'a3', 'a4', 'a5', 'Home', 'Away']]
```

![img2](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img2.png)

In order to describe success that certain star players bring on the court, we have chosen to focus on the shooting percentage of *just the teammates* while a player is on vs. off the court. The filter_eFG function below will filter to just the shots attempted or made by the player's team that were **not** attempted/made by the player himself.

```
def filter_eFG(df, nm = 'name', tm = 'teamAbbrev'):
        return df[(df['player'] != nm) & (df['team'] == tm) & ((df['event_type'] == 'shot') | (df['event_type'] == 'miss'))]
```

```
# testing the filter_eFG function
x = filter_eFG(df, 'Stephen Curry', 'GSW')
x[['player', 'event_type', 'team', 'h1', 'h2', 'h3', 'h4', 'h5', 'a1', 'a2', 'a3', 'a4', 'a5']]
```

![img3](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img3.png)

We will also create a function called eFG to calculate the eFG: `eFG% = (FGM + 0.5 * 3PM) / FGA`
- Field Goals Made (**FGM**): The number of field goals that a player or team has made. This includes both 2 pointers and 3 pointers
- 3 Point Field Goals Made (**3PM**): The number of 3 point field goals that a player or team has made
- Field Goals Attempted (**FGA**): The number of field goals that a player or team has attempted. This includes both 2 pointers and 3 pointers

```
def eFG(df):
    numerator = df.loc[df['event_type'] == 'shot', 'points'].count() + 0.5 * df.loc[df['points'] == 3, 'points'].count()
    denominator = df.loc[df['event_type'] == 'shot', 'points'].count() + df.loc[df['event_type'] == 'miss', 'points'].count()
    eFG = (numerator) / (denominator)
    return eFG
```

```
# testing the eFG function
test_df = pd.DataFrame({'event_type': ['shot', 'shot', 'shot', 'shot', 'shot', 'miss', 'miss', 'miss', 'miss', 'miss'],
                   'points': [2, 2, 2, 2, 2, 0, 0, 0, 0, 0]})
test_df
```

![img4](https://github.com/mikiokaji/NBA-Play-by-Play-Analysis/blob/main/images/img4.png)
