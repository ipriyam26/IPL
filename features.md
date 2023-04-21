
General calculations
```python

# Add columns for counts of boundaries, sixes, catches, stumpings, and run-outs
data['boundary_count'] = data['batsman_runs'].apply(lambda x: 1 if x == 4 else 0)
data['six_count'] = data['batsman_runs'].apply(lambda x: 1 if x == 6 else 0)
data['catch_count'] = data['dismissal_kind'].apply(lambda x: 1 if x == 'caught' else 0)
data['stump_count'] = data['dismissal_kind'].apply(lambda x: 1 if x == 'stumped' else 0)
data['run_out_throw_count'] = data['dismissal_kind'].apply(lambda x: 1 if x == 'run out' and x == 'throw' else 0)
data['run_out_catch_count'] = data['dismissal_kind'].apply(lambda x: 1 if x == 'run out' and x == 'catch' else 0)

# Calculate points for each player in each match
data['points'] = data.apply(calculate_points, axis=1)

# Calculate form (average points in the last few matches) for each player
num_matches_to_consider = 5
player_form = data.groupby(['match_id', 'batsman'])['points'].sum().rolling(window=num_matches_to_consider).mean().reset_index()
****
# Merge the features into a single DataFrame
features = pd.DataFrame({'batting_average': batting_average, 'bowling_average': bowling_average,
                         'strike_rate': strike_rate, 'economy_rate': economy_rate})

# Merge player_form with features
features = features.merge(player_form, left_index=True, right_on='batsman')
```


Batting Average

```python
import pandas as pd

# Assuming 'data' is a DataFrame containing the ball-by-ball analysis
data['player_dismissed'] = data['player_dismissed'].fillna(0)
data['player_dismissed'] = data['player_dismissed'].apply(lambda x: 1 if x != 0 else 0)
batsman_runs = data.groupby('batsman')['batsman_runs'].sum()
batsman_dismissals = data.groupby('batsman')['player_dismissed'].sum()
batting_average = batsman_runs / batsman_dismissals
```

Bowling Average

```python
bowler_wickets = data[data['is_wicket'] == 1].groupby('bowler')['is_wicket'].count()
bowler_runs = data.groupby('bowler')['total_runs'].sum()
bowling_average = bowler_runs / bowler_wickets
```

Strike Rate

```python
batsman_balls_faced = data.groupby('batsman')['ball'].count()
strike_rate = (batsman_runs / batsman_balls_faced) * 100
```

Economy Rate

```python
bowler_balls = data.groupby('bowler')['ball'].count()
bowler_overs = bowler_balls / 6
economy_rate = bowler_runs / bowler_overs
```

Form 

```python
# Calculate the points for each player in each match using the given criteria
def calculate_points(row):
    points = 0
    # Calculate batting points
    points += row['batsman_runs']
    # Add bonus points for boundaries
    points += row['boundary_count'] + 2 * row['six_count']
    # Add bonus points for reaching milestones
    if row['batsman_runs'] >= 100:
        points += 16
    elif row['batsman_runs'] >= 50:
        points += 8
    elif row['batsman_runs'] >= 30:
        points += 4
    # Subtract points for ducks
    if row['batsman_runs'] == 0 and row['player_dismissed'] == 1:
        points -= 2

    # Calculate bowling points
    points += 25 * row['is_wicket']
    if row['dismissal_kind'] in ['lbw', 'bowled']:
        points += 8 * row['is_wicket']
    # Add bonus points for multiple wickets
    if row['is_wicket'] >= 5:
        points += 16
    elif row['is_wicket'] >= 4:
        points += 8
    elif row['is_wicket'] >= 3:
        points += 4

    # Calculate fielding points
    points += 8 * row['catch_count']
    if row['catch_count'] >= 3:
        points += 4

    points += 12 * row['stump_count']
    points += 6 * row['run_out_throw_count']
    points += 6 * row['run_out_catch_count']
    
    return points
```
