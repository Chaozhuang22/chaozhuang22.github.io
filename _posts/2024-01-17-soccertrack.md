---
title: "Soccer Player Dynamics: An In-depth Data Analysis of Player Movements and Team Synchronization"
date: 2024-01-17T08:30:00+09:00
category: Data-Analysis
tag: Projects
header:
  teaser: /assets/images/soccertrack/teaser.jpg
---
> Discover how data analysis can reveal key insights into player dynamics and team synchronization in soccer. Learn about player trajectories, characteristic areas, order parameters, and anomaly detection.

# Analysis Goal
In this analysis, we will examine 30 minutes of annotated footage from a soccer game. Drawing inspiration from a study on player dynamics ([source](https://journals.aps.org/pre/pdf/10.1103/PhysRevE.104.024110?casa_token=1hUKBpJGWaAAAAAA%3A3EZJ3eSODJzs8Vzx6K8czlQ92fkfPzssWf1dwvo5S1Ed04QBq5yTvBA-Ma25lP_NBM-zl_fPnuFpRrU)), our focus will be on:
- Analyze player trajectories, spatial dispersion, and formation changes over time.
- Investigate the dynamic synchronization of player movements in response to game events.
- Apply anomaly detection to identify significant events, such as timeouts.
The original Kaggle notebook can be found [here](https://www.kaggle.com/code/chaozhuang/soccertrack-collective-dynamics-analysis#Team-Dynamics).

# Loading Data
The game's annotated data is divided into 30-second segments. To analyze the entire 30-minute game, we will concatenate these segments from the annotation folder to construct a comprehensive time series of player coordinates.

```python
# Loading data
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns
import os
import glob 
from matplotlib.patches import Ellipse
from matplotlib.animation import FuncAnimation
from IPython.display import HTML
import statsmodels.api
import statsmodels as sm

# Plot style
colors = sns.color_palette()

# Kaggle
path = '/kaggle/input/soccertrack/top_view/annotations/'

# Local
# path = 'top_view/annotations/'

# Loading data
csv_files = glob.glob(os.path.join(path, "*.csv"))
csv_files.sort()
data = pd.read_csv(csv_files[0], header=None)

# Dropping the original first few rows, and the first column
data = data.drop([0, 1, 2, 3])

# Concatenating all the annotation files
for file in csv_files[1:]: 
	
    # Preprocessing temp files
    temp = pd.read_csv(file, header=None)
    temp = temp.drop([0, 1, 2, 3])
    temp = temp.iloc[:,1:]

    # Concatenating with the previous file
    data = pd.concat([data, temp])

# Reset the messed-up indices and remove the wrong one
data = data.reset_index()
data = data.iloc[:,2:]

# Converting data type into floats
data = data.astype(float)
```

# Preprocessing Data
The annotation file includes four-column bounding box coordinates. For our analysis, these will be converted into two-column x-y coordinates, streamlining the data for efficient plotting.

```python
# Converting bounding box coordinates into xy coordinates
num_columns = len(data.columns)
group_size = 4
num_players = num_columns // group_size

for i in range(num_players):
    group_columns = data.iloc[:, i*group_size:(i+1)*group_size]
    result_column_name = f'player_{i}_x'
    data[result_column_name] = group_columns.iloc[:,1] + group_columns.iloc[:,3] / 2
    result_column_name = f'player_{i}_y'
    data[result_column_name] = group_columns.iloc[:,2] + group_columns.iloc[:,0] / 2

# Correcting the final coordinates as the ball coordinates
data = data.rename({'player_22_x':'ball_x', 'player_22_y':'ball_y'}, axis = 1)

# Remove the redundant bbox coordinates
data = data.iloc[:,-46:]

# Remove nan values
data = data.dropna()
```

# Individual-level Analysis
## Player Trajectories
Upon compiling the complete time series data, we can plot the trajectories of individual players throughout the 30-minute match.

```python
# Plotting the full-game trajectory of the Player 1
def draw_soccer_field(ax):
    # Standard soccer field dimensions in meters
    field_length_m = 105
    field_width_m = 68

    # Scale factors
    field_length_px = 3840 * 0.75
    field_width_px = field_length_px / (field_length_m / field_width_m)  # Maintain aspect ratio

    # Field position
    field_left = (3840 - field_length_px) / 2
    field_top = (2160 - field_width_px) / 2

    # Drawing the field
    rect = plt.Rectangle((field_left, field_top), field_length_px, field_width_px, edgecolor='black', facecolor='none', lw=2)
    ax.add_patch(rect)

    # Scale for meters to pixels
    scale_x = field_length_px / field_length_m
    scale_y = field_width_px / field_width_m

    # Center Circle
    center_circle_radius_m = 9.15  # Standard center circle radius
    center_circle = plt.Circle((3840 / 2, 2160 / 2), center_circle_radius_m * scale_x, color='black', fill=False, lw=2)
    ax.add_patch(center_circle)

    # Goal Areas (6-yard box)
    goal_area_length_m = 5.5
    goal_area_width_m = 18.32
    goal_area_left = plt.Rectangle((field_left, 2160 / 2 - (goal_area_width_m * scale_y / 2)), 
                                   goal_area_length_m * scale_x, goal_area_width_m * scale_y, 
                                   edgecolor='black', facecolor='none', lw=2)
    goal_area_right = plt.Rectangle((3840 - field_left - goal_area_length_m * scale_x, 2160 / 2 - (goal_area_width_m * scale_y / 2)), 
                                    goal_area_length_m * scale_x, goal_area_width_m * scale_y, 
                                    edgecolor='black', facecolor='none', lw=2)
    ax.add_patch(goal_area_left)
    ax.add_patch(goal_area_right)

    # Vertical Center Line
    plt.plot([3840 / 2, 3840 / 2], [field_top, field_top + field_width_px], color='black', lw=2)

df = data

fig, ax = plt.subplots(figsize=(19.2, 10.8))

# Draw the soccer field
draw_soccer_field(ax)

# Plotting player positions
ax.scatter(df.iloc[:,2],df.iloc[:,3], color = colors[0])
plt.xlim(0, 3840)
plt.ylim(0, 2160)
plt.gca().invert_yaxis()  # Invert y-axis to match image coordinates
plt.title('Team 1 player 1 Trajectory')
plt.show()
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img1.png" alt="A scatter plot showing the trajectory of a soccer player in Team 1 during a 30-minute match, plotted on a top-view diagram of a soccer field.">
  <figcaption>A player's trajectory in Team 1.</figcaption>
</figure>

## Player Activity Area
We can visualize a player's activity area as an ellipsoid, centered at their mean position, with axes derived from the largest eigenvectors of the position data. The visualization reveals:
- Goalkeepers have a significantly smaller activity area compared to outfield players.
- Team 2 displays more aggression, evidenced by several players having their activity areas predominantly in Team 1's half, a pattern not observed in Team 1.
- In line with Team 2's aggressive play, the ball is predominantly located in Team 1's half.

```python
# Plotting activity patterns for Team 1
def plot_std_dev_ellipses(player_data, ax, color):
    """
    Plot standard deviation ellipses and mean markers for players in a DataFrame.

    Parameters:
    player_data (DataFrame): DataFrame containing player coordinates.
    ax (Axes): Matplotlib Axes object to plot on.
    color (str): Color for the ellipse and marker.
    player_number (int): Player number to annotate.
    """

    # Calculate mean
    mean_x, mean_y = np.mean(player_data, axis=0)

    # Calculate the covariance matrix
    cov_matrix = np.cov(player_data.T)

    # Compute eigenvalues and eigenvectors
    eigenvalues, eigenvectors = np.linalg.eig(cov_matrix)

    # Sort the eigenvalues and eigenvectors in descending order
    sorted_indices = np.argsort(eigenvalues)[::-1]
    eigenvalues = eigenvalues[sorted_indices]
    eigenvectors = eigenvectors[:, sorted_indices]

    # Use the first eigenvector to calculate the angle in degrees
    angle = np.degrees(np.arctan2(eigenvectors[1, 0], eigenvectors[0, 0]))

    # Drawing the standard deviation ellipse with orientation for each player
    ellipse = Ellipse((mean_x, mean_y), width=2*np.sqrt(eigenvalues[0]), height=2*np.sqrt(eigenvalues[1]),
                      angle=angle, facecolor=color, alpha=0.5)
    ax.add_patch(ellipse)

    # Marking the mean
    ax.scatter([mean_x], [mean_y], color=color, marker='X', s = 250)

# Plotting activity patterns for Team 1
fig, ax = plt.subplots(figsize=(19.2, 10.8))
draw_soccer_field(ax)

for i in range(0, 22, 2):
    color = colors[i // 2 % len(colors)]
    plot_std_dev_ellipses(df.iloc[:, i:i+2], ax, color)

plt.xlim(0, 3840)
plt.ylim(0, 2160)
plt.gca().invert_yaxis()  # Invert y-axis to match image coordinates
plt.title('Activity Areas for Players in Team 1')
plt.show()
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img2.png" alt="A top-view diagram of a soccer field with ellipses representing the activity areas of different players in Team 1 during a 30-minute match.">
  <figcaption>The activity areas of players in Team 1.</figcaption>
</figure>

```python
# Plotting activity patterns for Team 2
fig, ax = plt.subplots(figsize=(19.2, 10.8))
draw_soccer_field(ax)
plt.xlim(0, 3840)
plt.ylim(0, 2160)
plt.gca().invert_yaxis()

for i in range(22, 44, 2):
    color = colors[i // 2 % len(colors)]
    plot_std_dev_ellipses(df.iloc[:, i:i+2], ax, color)

plt.title('Activity Areas for Players in Team 2')
plt.show()
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img3.png" alt="A top-view diagram of a soccer field with ellipses representing the activity areas of different players in Team 2 during a 30-minute match.">
  <figcaption>The activity areas of players in Team 2.</figcaption>
</figure>

```python
# Plotting activity patterns for the ball
fig, ax = plt.subplots(figsize=(19.2, 10.8))
draw_soccer_field(ax)
plt.xlim(0, 3840)
plt.ylim(0, 2160)
plt.gca().invert_yaxis()

# Using the same function plotting the ball
plot_std_dev_ellipses(df.iloc[:, -2:], ax, color = colors[0])

plt.gca().invert_yaxis()
plt.title('Activity Areas for the ball')
plt.show()
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img4.png" alt="A top-view diagram of a soccer field with an ellipse representing the activity area of the ball during a 30-minute match.">
  <figcaption>The activity areas of the ball.</figcaption>
</figure>

# Team-level Dynamics
We now shift focus to team dynamics by plotting the characteristic areas of both teams. These areas, defined by the standard deviation of player positions at given times, reflect the spatial distribution of players at any given point in time. Visualizing the entire match in this way reveals two dynamic blobs—representing each team—that move, rotate, expand, and contract in response to the game's flow.

```python
# A demo of characteristic area
# Specify the frame number to visualize
frame_number = 2000

# Plotting all player locations at a frame
fig, ax = plt.subplots(figsize=(19.2, 10.8))
draw_soccer_field(ax)
plt.xlim(0, 3840)
plt.ylim(0, 2160)
plt.gca().invert_yaxis()
df_frame = df.iloc[frame_number]
reshaped_df = pd.DataFrame(df_frame.values.reshape(-1, 2), columns=['x', 'y'])

# Team 1
ax.scatter(reshaped_df.iloc[:11, 0], reshaped_df.iloc[:11, 1], color = colors[0], marker='o', s=500)
plot_std_dev_ellipses(reshaped_df.iloc[:11], ax, color = colors[0])

# Team 2
ax.scatter(reshaped_df.iloc[11:22, 0], reshaped_df.iloc[11:22, 1], color = colors[1], marker='o', s=500)
plot_std_dev_ellipses(reshaped_df.iloc[11:22], ax, color = colors[1])

# Ball
ax.scatter(reshaped_df.iloc[22, 0], reshaped_df.iloc[22, 1], color = 'black', marker='o', s=500)

plt.gca().invert_yaxis()
plt.title('Characteristic Area of both teams at the frame number of 2000')
plt.show()
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img5.png" alt="A top-view diagram of a soccer field showing the characteristic areas of both teams at the 2000th frame, represented by two dynamic blobs that depict the spatial distribution of players.">
  <figcaption>The spatial distribution of both team at the frame number of 2000.</figcaption>
</figure>

## Animation
The code below plots the dynamic blobs representing each team and compiles them into a movie.

```python
# Animate the changing characteristic area in a fast-forward video
# Define the function to update each frame in the animation
def update(frame_number):
    ax.clear()
    draw_soccer_field(ax)
    df_frame = df.iloc[frame_number,:46]
    reshaped_df = pd.DataFrame(df_frame.values.reshape(-1, 2), columns=['x', 'y'])

    # Team 1
    color = colors[0]
    ax.scatter(reshaped_df.iloc[:11, 0], reshaped_df.iloc[:11, 1], color=color, marker='o', s=100)
    plot_std_dev_ellipses(reshaped_df.iloc[:11], ax, color)

    # Team 2
    color = colors[1]
    ax.scatter(reshaped_df.iloc[11:22, 0], reshaped_df.iloc[11:22, 1], color=color, marker='o', s=100)
    plot_std_dev_ellipses(reshaped_df.iloc[11:22], ax, color)

    # Ball
    color = 'black'
    ax.scatter(reshaped_df.iloc[22, 0], reshaped_df.iloc[22, 1], color=color, marker='o', s=100)

    plt.gca().invert_yaxis()
    plt.title('Frame: {}'.format(frame_number))

# Set up the figure
fig, ax = plt.subplots(figsize=(9.6, 5.4))
draw_soccer_field(ax)
plt.xlim(0, 3840)
plt.ylim(0, 2160)

# Frame rate
fps = 150

# Start frame
start_frame = 5000

# Calculate the number of frames for a 20-second video at the given fps
num_frames_for_video = 20 * fps

# Select a subset of rows starting from frame 3000 for the video
subset_df = df.iloc[start_frame:start_frame + num_frames_for_video]

# Create animation
ani = FuncAnimation(fig, update, frames=range(start_frame, start_frame + len(subset_df)), interval=1000/fps)

# Display the animation in the notebook
HTML(ani.to_html5_video())
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/gif1.gif" alt="An animated sequence showing the evolution of characteristic areas of both teams over a span of 100 seconds, represented by two dynamic blobs that move, rotate, expand, and contract in response to the game's flow.">
  <figcaption>The speedup 7.5x evolution of characteristic areas of both teams for 100 s.</figcaption>
</figure>

## Analysis on Characteristic Area
To investigate the dynamics, that is, the expansion and contraction of the characteristic areas, both the centers of mass and the total areas from both teams are calculated as a function of time. These values will be appended as additional columns to the original DataFrame. Moreover, the current frame-based index is too fine-grain for our analysis. Therefore, we will transform the DataFrame's index into a datetime format for more intuitive and convenient analysis.

```python
# Calculate the center coordinates and area for both teams and append them to the dataframe
def ellipse_area(cov_matrix):
    eigenvalues, _ = np.linalg.eig(cov_matrix)
    # Semi-major and semi-minor axes (sqrt of eigenvalues)
    semi_major = np.sqrt(eigenvalues[0])
    semi_minor = np.sqrt(eigenvalues[1])
    # Area of ellipse
    area = np.pi * semi_major * semi_minor
    return area

areas_team1 = []
areas_team2 = []

for frame_number in range(len(df)):
    df_frame = df.iloc[frame_number]
    reshaped_df = pd.DataFrame(df_frame.values.reshape(-1, 2), columns=['x', 'y'])

    # Calculate area for Team 1
    area_team1 = ellipse_area(np.cov(reshaped_df.iloc[:11].T))
    areas_team1.append(area_team1)

    # Calculate area for Team 2
    area_team2 = ellipse_area(np.cov(reshaped_df.iloc[11:22].T))
    areas_team2.append(area_team2)

# Convert areas to DataFrame and rename columns
areas_team1_df = pd.DataFrame(areas_team1, columns=['area_team1'])
areas_team2_df = pd.DataFrame(areas_team2, columns=['area_team2'])

df['center_team1_x'] = df.iloc[:,0:22:2].mean(axis=1)
df['center_team1_y'] = df.iloc[:,1:22:2].mean(axis=1)
df['center_team2_x'] = df.iloc[:,22:-2:2].mean(axis=1)
df['center_team2_y'] = df.iloc[:,23:-2:2].mean(axis=1)

# Reset index ensures smooth concatenation
df = df.reset_index(drop=True)
areas_team1_df = areas_team1_df.reset_index(drop=True)
areas_team2_df = areas_team2_df.reset_index(drop=True)

# Concatenate area data into the original dataframe
df = pd.concat([df, areas_team1_df, areas_team2_df], axis=1)

# As a bonus, calculate distances between team1, team2, and the ball, adding them as new columns
df['team1_team2'] = np.sqrt((df['center_team1_x'] - df['center_team2_x'])**2 + (df['center_team1_y'] - df['center_team2_y'])**2)
df['team1_ball'] = np.sqrt((df['center_team1_x'] - df['ball_x'])**2 + (df['center_team1_y'] - df['ball_y'])**2)
df['team2_ball'] = np.sqrt((df['center_team2_x'] - df['ball_x'])**2 + (df['center_team2_y'] - df['ball_y'])**2)

# Converting frame to second for a fewer data points and less clutter graph
def frame_to_second(start_time, df):
    # Set the starting point
    start_time = pd.Timestamp(start_time)
    # Calculate the number of rows in the DataFrame
    num_rows = len(df)
    # Generate a datetime index - frequency is set to '33L' for 1/30th of a second (approximately 33 milliseconds)
    datetime_index = pd.date_range(start=start_time, periods=num_rows, freq='33L')
    # Set this as the index of the DataFrame
    df.index = datetime_index
    # Resample and aggregate by seconds
    df_second = df.resample('S').mean()
    return df_second

df_second = frame_to_second('2022-02-02 16:01:00',df)
```

### Time Series Analysis
The characteristic area's evolution shows non-stationarity, evident from significant correlations at small lags in the autocorrelation plot. Visual analysis reveals an average persistence time (a cycle of expansion or contraction) of approximately 23 s for Team 1 and 15 s for Team 2, suggesting a more rapid playing style for Team 2.

Notably, a significant dip in Team 1's characteristic area around the 15-minute mark corresponds to a timeout before a corner kick, where Team 1 players formed a small circle for strategy discussion. A similar pattern is observed at the 28th minute, marking another short timeout. The timeouts are also apparent in the histogram as the small peak left of the main peak.

```python
# Plotting the time series and associated diagnostics for Team 1
def tsplot2(y, title, lags=None, figsize=(12, 8)):
    '''Examine the patterns of ACF and PACF, along with the time series plot and histogram.'''
    fig = plt.figure(figsize=figsize)
    layout = (2, 2)
    ts_ax = plt.subplot2grid(layout, (0, 0))
    hist_ax = plt.subplot2grid(layout, (0, 1))
    acf_ax = plt.subplot2grid(layout, (1, 0))
    pacf_ax = plt.subplot2grid(layout, (1, 1))

    y.plot (ax=ts_ax)
    ts_ax.set_title(title, fontsize=14, fontweight='bold' )
    y.plot (ax=hist_ax, kind='hist', bins=25)
    hist_ax.set_title('Histogram')
    sm.graphics.tsaplots.plot_acf (y, lags=lags, ax=acf_ax)
    sm.graphics.tsaplots.plot_pacf(y, lags=lags, ax=pacf_ax)
    [ax.set_xlim(0) for ax in [acf_ax, pacf_ax]]
    sns.despine()
    plt.tight_layout()
    return ts_ax, acf_ax, pacf_ax

tsplot2(df_second.area_team1, "Team 1's characteristic area");
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img6.png" alt="A time series plot with associated diagnostics showing the evolution of Team 1's characteristic area over the 30-minute match, revealing non-stationarity and significant correlations at small lags.">
  <figcaption>The time series of the characteristic areas of Team 1.</figcaption>
</figure>

```python
# Plotting the time series and associated diagnostics for Team 2
tsplot2(df_second.area_team2, "Team 2's characteristic area");
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img7.png" alt="A time series plot with associated diagnostics showing the evolution of Team 2's characteristic area over the 30-minute match, revealing non-stationarity and significant correlations at small lags.">
  <figcaption>The time series of the characteristic areas of Team 2.</figcaption>
</figure>

## Synchronization
The divergence and convergence of players within a team, relative to the team's frame of reference, reflect the intricate interplay of cooperation and competition. The temporal dynamics observed can serve as indicators of the team's collective attentiveness and responsiveness to changing situations, such as the ball's movement.

Another key aspect of team dynamics is synchronization, which refers to the coordinated movement of players during gameplay. To illustrate this, we will calculate player velocities and directions using the existing coordinate data and animate instances of synchronization among the players.

```python
# Calculate the velocity and direction of all players
# Calculate the differences in coordinates
diff = df.diff().iloc[1:,:46]

# Initialize DataFrames to store velocities and directions
df_velocity = pd.DataFrame(index=diff.index)
df_direction = pd.DataFrame(index=diff.index)

# Calculate the velocity and direction for each player
for i in range(22):
    dx = diff[f'player_{i}_x']
    dy = diff[f'player_{i}_y']
    
    # Velocity magnitude
    df_velocity[f'player_{i}_velocity'] = np.sqrt(dx**2 + dy**2)
    
    # Velocity direction in radians
    df_direction[f'player_{i}_direction'] = np.arctan2(dy, dx)
```

### Animation
To visualize synchronization, we use the following criteria: a link is drawn between two players if they move in the same direction for over 2 seconds. This link disappears when their movements diverge for more than 2 seconds, using a 20% tolerance in motion angle to define synchronization.

The animation below shows that synchronization commonly occurs during ball-chasing, often manifesting in waves and involving both teams simultaneously. Conversely, desynchronization typically happens when players from opposing teams converge towards the ball from different directions.

```python
# Animate the synchronization by adding links between players that move together
def are_synchronized(angle1, angle2, tolerance=0.2 * np.pi):
    """Check if two angles are within a certain tolerance."""
    # Normalize angles to [0, 2π] range
    angle1 = angle1 % (2 * np.pi)
    angle2 = angle2 % (2 * np.pi)

    # Calculate the absolute difference between angles
    diff = np.abs(angle1 - angle2)

    # Check if angles are within tolerance
    return diff <= tolerance or diff >= 2 * np.pi - tolerance

# Global variables to track synchronization over time
sync_tracker = {}  # Dictionary to track synchronization state

def update_sync_tracker(team, player_i, player_j, is_sync):
    """Update the synchronization tracker with the current state."""
    key = (team, min(player_i, player_j), max(player_i, player_j))
    sync_state = sync_tracker.get(key, {'count': 0, 'sync': False})

    if is_sync:
        sync_state['count'] += 1  # Increment sync count if players are synchronized
    else:
        sync_state['count'] -= 1  # Decrement sync count if players are not synchronized

    sync_state['count'] = max(0, min(sync_state['count'], 60))  # Keep count within [0, fps]
    sync_state['sync'] = sync_state['count'] >= 60  # Update sync status

    sync_tracker[key] = sync_state
    return sync_state['sync']

def update(frame_number):
    ax.clear()
    draw_soccer_field(ax)

    # Get data for the current frame
    df_frame = df.iloc[frame_number, :46]
    reshaped_df = pd.DataFrame(df_frame.values.reshape(-1, 2), columns=['x', 'y'])
    df_direction_frame = df_direction.iloc[frame_number]

    # Plot players and draw lines for synchronized players
    for team in range(2):
        start_idx = team * 11
        end_idx = (team + 1) * 11
        color = colors[0] if team == 0 else colors[1]

        # Draw players
        ax.scatter(reshaped_df.iloc[start_idx:end_idx, 0], reshaped_df.iloc[start_idx:end_idx, 1], color=color, marker='o', s=100)

        # Check and draw lines for synchronized players
        for i in range(start_idx, end_idx):
            for j in range(i+1, end_idx):
                is_sync = are_synchronized(df_direction_frame[f'player_{i}_direction'], df_direction_frame[f'player_{j}_direction'])
                if update_sync_tracker(team, i, j, is_sync):
                    ax.plot([reshaped_df.iloc[i, 0], reshaped_df.iloc[j, 0]], 
                            [reshaped_df.iloc[i, 1], reshaped_df.iloc[j, 1]], 
                            color=color, linewidth=1)

    # Plot the ball
    ax.scatter(reshaped_df.iloc[22, 0], reshaped_df.iloc[22, 1], color='black', marker='o', s=100)

    plt.gca().invert_yaxis()
    plt.title(f'Frame: {frame_number}')

# Set up the figure
fig, ax = plt.subplots(figsize=(9.6, 5.4))
draw_soccer_field(ax)
plt.xlim(0, 3840)
plt.ylim(0, 2160)

# Frame rate
fps = 150

# Start frame
start_frame = 5000

# Calculate the number of frames for a 20-second video at the given fps
num_frames_for_video = 20 * fps

# Select a subset of rows starting from frame 3000 for the video
subset_df = df.iloc[start_frame:start_frame + num_frames_for_video]

# Create animation
ani = FuncAnimation(fig, update, frames=range(start_frame, start_frame + len(subset_df)), interval=1000/fps)

# Display the animation in the notebook
HTML(ani.to_html5_video())
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/gif2.gif" alt="An animated sequence showing the synchronization of both teams over a span of 100 seconds, with links drawn between players moving in the same direction for over 2 seconds.">
  <figcaption>The speedup 7.5x evolution of synchronization of both teams for 100 s.</figcaption>
</figure>

### Order Parameter
Order parameters, akin to those used to describe liquid crystals or bird flocks, quantify the alignment and synchronization of constituent units. A high value indicates uniform movement direction across the group, while a low value signifies divergent movement directions.

We use the directions for all players in `df_direction` to compute the order parameter using the formula: $\phi(t)=\frac{1}{N}\sum_{n=1}^{11}\frac{v_n(t)}{|v_n(t)|}$. The time series of the order parameter $\phi$ for both teams is visualized below, which reveals that:
- There are unusual periods of desynchronization, where players move in random directions. These periods coincides with shrinking characteristic area observed previously.
- The autocorrelation plot suggests that Team 1 maintains synchronization slightly longer than Team 2, by about one second, though the difference is not statistically significant, which may indicate that the order parameter is a global feature of the game and does not reflect the strategies or play styles of teams

```python
# Plotting the time series and associated diagnostics for Team 1
# Converting the two dataframe into sec granularity
df_velocity_second = frame_to_second('2022-02-02 16:01:00',df_velocity)
df_direction_second = frame_to_second('2022-02-02 16:01:00',df_direction)

# Number of players per team
team_size = 11

# Initialize DataFrame to store order parameter for each team
df_order_param = pd.DataFrame(index=df_direction_second.index, columns=['team1_phi', 'team2_phi'])

# Calculate order parameter for each team
for t in df_direction_second.index:
    sum_vector_team1 = np.array([0.0, 0.0])
    sum_vector_team2 = np.array([0.0, 0.0])

    for i in range(22):
        direction = df_direction_second.at[t, f'player_{i}_direction']
        unit_vector = np.array([np.cos(direction), np.sin(direction)])
        
        # Summing unit vectors for each team
        if i < team_size:  # First 11 players for Team 1
            sum_vector_team1 += unit_vector
        else:  # Next 11 players for Team 2
            sum_vector_team2 += unit_vector

    # Calculating order parameter for each team
    df_order_param.at[t, 'team1_phi'] = np.linalg.norm(sum_vector_team1) / team_size
    df_order_param.at[t, 'team2_phi'] = np.linalg.norm(sum_vector_team2) / team_size

tsplot2(df_order_param.team1_phi, "Team 1's order parameter");
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img8.png" alt="A time series plot with associated diagnostics showing the evolution of the order parameter of Team 1 over the 30-minute match, which quantifies the alignment and synchronization of player movements.">
  <figcaption>The time series of order parameter of Team 1.</figcaption>
</figure>

```python
# Plotting the time series and associated diagnostics for Team 2
tsplot2(df_order_param.team2_phi, "Team 2's order parameter");
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img9.png" alt="A time series plot with associated diagnostics showing the evolution of the order parameter of Team 2 over the 30-minute match, which quantifies the alignment and synchronization of player movements.">
  <figcaption>The time series of order parameter of Team 2.</figcaption>
</figure>

### Anomaly Detection
Dips in order parameters, signaling desynchronization, often correspond to short timeouts during injuries, rule infringements, or set pieces like corner or free kicks.

By employing the `ruptures` package in Python with an l2 model, we can systematically detect these anomalies as shifts in the mean of the order parameter.

After adjusting the number of breakpoints, the model effectively identifies timeout periods for both Team 1 and Team 2. Cross-validation with Team 2's data corroborates the identified anomalies, confirming timeouts around 16:15 and 16:28. This aligns with the characteristic area reduction observed in previous analyses.

```python
!pip install ruptures
```

```python
# Detecting the periods of low synchronization using the ruptures package for Team 1
import ruptures as rpt

# Convert the smoothed signal to numpy array for rupture analysis
signal = df_order_param['team1_phi'].values

# Create a change point detection model
model = "l2"
algo = rpt.Binseg(model=model).fit(signal)

# Number of breakpoints to find
n_bkps = 9

# Find the breakpoints
bkps = algo.predict(n_bkps=n_bkps)

# Plotting
fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(df_order_param.index, df_order_param['team1_phi'], label='Original Signal')

# Highlight the detected breakpoints
for bkp in bkps[:-1]:  # Skip the last one as it is just the end of the signal
    ax.axvline(x=df_order_param.index[bkp], color='r', linestyle='--')

plt.title('Change Point Detection for Team 1 Order Parameter')
plt.xlabel('Time')
plt.ylabel('team2_phi')
plt.legend()
plt.show()
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img10.png" alt="A time series plot of Team 1's order parameter with detected change points, indicating potential anomalies or shifts in the mean of the order parameter.">
  <figcaption>Applying change point detection on the time series of order parameter of Team 1.</figcaption>
</figure>

```python
# Validating the anomalies in Team 1 by performing the same analysis on Team 2
# Convert the smoothed signal to numpy array for rupture analysis
signal = df_order_param['team2_phi'].values

# Create a change point detection model
model = "l2"
algo = rpt.Binseg(model=model).fit(signal)

# Number of breakpoints to find
n_bkps = 9

# Find the breakpoints
bkps = algo.predict(n_bkps=n_bkps)

# Plotting
fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(df_order_param.index, df_order_param['team2_phi'], label='Original Signal')

# Highlight the detected breakpoints
for bkp in bkps[:-1]:  # Skip the last one as it is just the end of the signal
    ax.axvline(x=df_order_param.index[bkp], color='r', linestyle='--')

plt.title('Change Point Detection for Team 2 Order Parameter')
plt.xlabel('Time')
plt.ylabel('team2_phi')
plt.legend()
plt.show()
```

<figure style="width: 500px" class="align-center">
  <img src="/assets/images/soccertrack/img11.png" alt="A time series plot of Team 2's order parameter with detected change points, indicating potential anomalies or shifts in the mean of the order parameter.">
  <figcaption>Applying change point detection on the time series of order parameter of Team 2.</figcaption>
</figure>

# Conclusion
In this analysis, we transformed the annotation file into a time series for detailed study. Our animation and time-series analysis revealed two key dynamics in high-level team play: the **characteristic area** and **synchronization order parameters**.

The characteristic area is represented by the team's mass center and positional standard deviation. This parameter fluctuates and reflects teamplay styles. The order parameter, on the other hand, is a global feature that signifies player synchronization, reflecting the game flow and ball-pursuit dynamics. Extended desynchronization often marks significant game events like timeouts.

These insights lay a foundation for broader analyses across multiple matches, offering a novel approach to team evaluation and comparison. Future research could explore how these dynamics vary when a team faces different opponents, investigating whether changes are driven by opponent competitiveness or are intrinsic to the team.

Thank you for reading and feel free to comment or leave a like in my [LinkedIn post](https://www.linkedin.com/feed/update/urn:li:activity:7149306123219591168/)!