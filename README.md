## CONDOR Project: Analysis and Visualization of the Spatio-Temporal Particle Distribution

# Overview
This repository contains the code and scripts developed for the CONDOR Project, focusing on the analysis and visualization of the spatio-temporal distribution of incident particles on a detector array. The project is divided into several stages: data preprocessing, binning of particle positions and times, and visualization through 2D and 3D plots to interpret the results.

The main goal is to group the particles that reach different areas of the detector array within short time intervals (in nanoseconds) and visualize how they are distributed across space and time. This allows studying the dynamics of particles in the detector and their relationship with parameters such as incidence energy, angle, and position.

# Project Structure
The project is organized into three main parts:

- Data Preprocessing
- Spatio-Temporal Binning
- Results Visualization

# 1. Data Preprocessing
The preprocessing step involves cleaning and organizing the raw data of incident particles. In the original particle dataframe, the columns include x, y (spatial positions on the detector array), t (time in nanoseconds), and energy (particle energy).

Key steps in this section:

- Spatial filtering: Particles outside the bounds of the detector array are removed. The total detector area covers 122 x 113 meters.
- Temporal adjustment: Time values are normalized by subtracting the minimum time so that the analysis starts at t = 0 (nanoseconds).

```python
particles_df['t'] = particles_df['t'] - particles_df['t'].min()
particles_df = particles_df.sort_values(by='t', ascending=True).reset_index(drop=True)
```

# 2. Spatio-Temporal Binning
The core idea is to bin particles into fixed spatial and temporal intervals to count how many particles arrive at certain areas within the same nanosecond. The particles are grouped into "bins" of space (e.g., 1x1 meter) and time (e.g., 1 ns).

Key steps:

Create fixed-size bins for both the spatial coordinates (x_bin, y_bin) and the time (t_bin).
Group the particles by these bins and count how many fall into each.

```python
particles_df['x_bin'] = np.floor(particles_df['x']).astype(int)
particles_df['y_bin'] = np.floor(particles_df['y']).astype(int)
particles_df['t_bin'] = np.floor(particles_df['t'] / time_bin_size).astype(int)
binned_particles = particles_df.groupby(['x_bin', 'y_bin', 't_bin']).size().reset_index(name='particle_count')
```

# 3. Results Visualization
To interpret the results, both 2D and 3D animated plots are created to show the spatio-temporal distribution of particles. The plots are designed to highlight areas with higher particle concentrations and show how these change over time.

- 2D Animation: Represents particles in the x-y plane for different times (t_bin), showing how the spatial distribution of particles evolves over time.
- 3D Animation: Displays the evolution of the spatial distribution (x, y) with a third dimension representing time (z = t_bin).

Both types of animation are saved as .gif files and are configured to display the exact time of each frame in the title of the plots.

# Example of 3D Plot

```python
x = filtered_particles['x_bin']
y = filtered_particles['y_bin']
z = filtered_particles['t_bin']
particle_count = filtered_particles['particle_count']

fig = go.Figure(data=[go.Scatter3d(
    x=x,
    y=y,
    z=z,
    mode='markers',
    marker=dict(
        size=5,
        color=particle_count,  # Color based on particle count
        colorscale='tealrose',
        colorbar=dict(title='Particle Count'), 
        opacity=0.5
    )
)])

fig.update_layout(
    title='3D Particle Distribution in Time and Space',
    scene=dict(
        xaxis_title='X Bin',
        yaxis_title='Y Bin',
        zaxis_title='Time Bin'
    ),
    margin=dict(l=0, r=0, b=0, t=0)
)

# Display the plot
fig.show()
```

# Files
- Preprocessing Scripts: Scripts that preprocess the raw particle data (datapreprocessing.ipynb).
- Binned Dataframe: The dataframe (at .csv format) that bins the particles spatially and temporally.
- Visualization gifs: Gifs that generate 2D and 3D animations of the spatio-temporal distribution.
