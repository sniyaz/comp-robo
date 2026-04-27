---
layout: page
title: Lab 4
description: >-
    Sensor Models!
mathjax: true
nav_exclude: true
---

# Lab 4

## Q2. Sensor Model

In this question, you’ll implement the LIDAR sensor model described in lecture. The MuSHR car’s LIDAR unit emits infrared laser beams into the environment at fixed angular intervals and returns the measured distance along those beams. For an unsuccessful measurement, it returns NaN or 0. A sensor measurement $\mathbf{z}_t$ is a vector of distances, one for each beam: $\mathbf{z}_t = (z_t^1, \dots, z_t^K)$. Assuming that each distance $z_t^k$ is conditionally independent (given the state $\mathbf{x}_t$ and map $m$) allows us to consider each beam separately.

First, we’ve provided some starter code to generate a simulated observation $\mathbf{z}_t^{k*}$ given that the robot state is $\mathbf{x}_t$. We use a raycasting library called `rangelibc` (implemented in C++/CUDA), which casts rays into the map $m$ from the robot’s state and measures the distance to any obstacle/wall the ray encounters. This is similar to how the physical LIDAR unit measures distances.

Then, you’ll implement the sensor model $P(z_t^k | \mathbf{x}_t, m)$ by considering the four different modes discussed in lecture. Specifically, you will implement a model that measures how likely you are to see a real observation $z_t^k$, given that the simulated observation was $z_t^{k*}$. You may find the lecture slides or the following summary useful. (Note that the $p_{\text{short}}$ factor should be modeled more simply as with a linear probability density function.)

### Sensor Model Modes

$$
\begin{align}
p_{\text{hit}}(z_t^k | \mathbf{x}_t, m) &= \begin{cases} \mathcal{N}(z_t^k; z_t^{k*}, \sigma_{\text{hit}}^2) & 0 \leq z_t^k \leq z_{\text{max}} \\ 0 & \text{otherwise} \end{cases} \\
p_{\text{short}}(z_t^k | \mathbf{x}_t, m) &= \begin{cases} 2 \frac{z_t^{k*} - z_t^k}{(z_t^{k*})^2} & z_t^k < z_t^{k*} \\ 0 & \text{otherwise} \end{cases} \\
p_{\text{max}}(z_t^k | \mathbf{x}_t, m) &= \mathbb{I}(z_t^k = z_{\text{max}}) \\
p_{\text{rand}}(z_t^k | \mathbf{x}_t, m) &= \begin{cases} \frac{1}{z_{\text{max}}} & 0 \leq z_t^k < z_{\text{max}} \\ 0 & \text{otherwise} \end{cases}
\end{align}
$$

The notation $\mathcal{N}(z_t^k; z_t^{k*}, \sigma_{\text{hit}}^2)$ means that we’re computing the value of the probability density function, not that we’re sampling from a Gaussian distribution. (We would use $x \sim \mathcal{N}(\mu, \sigma^2)$ to denote that $x$ is being sampled from that Gaussian distribution.) The probability density function is a Gaussian parameterized with mean $z_t^{k*}$ and variance $\sigma_{\text{hit}}^2$, and we want to find its value for the input $z_t^k$, i.e.

$$
\frac{1}{\sqrt{2\pi\sigma_{\text{hit}}^2}} \exp \left\{ -\frac{1}{2} \left( \frac{z_t^k - z_t^{k*}}{\sigma_{\text{hit}}} \right)^2 \right\}
$$

The sensor model mixes the factors together with weights $z_{\text{hit}}, z_{\text{short}}, z_{\text{max}}, z_{\text{rand}}$. In addition, noise in the $p_{\text{hit}}$ factor is parameterized by $\sigma_{\text{hit}}$. Later, you will choose values for these parameters.

You can speed up the sensor model by pre-calculating and caching the sensor probabilities. On our 2D map, continuous values are converted to discrete pixel distances. You can then store the LIDAR range values in a table where each row is the actual measured value, the column is the expected value for a given LIDAR range, and the value is the probability. At runtime, you’ll use `rangelibc` to generate simulated range measurements, then use the cached table to quickly look up sensor probabilities.

**Q2.1: Implement the sensor model** in the `SingleBeamSensorModel.precompute_sensor_model` method (`src/localization/sensor_model.py`). Your implementation should accept zero as a possible weight for the factors, as long as one of them is nonzero. Make sure the columns of the table are normalized to ensure probabilities sum to 1. Your implementation should be vectorized.

After completing Q2.1, expect your code to pass all the test cases when running `python3 test/sensor_model.py`.

### Exploring the Sensor Model Parameters

We’ve provided two tools for tuning the sensor model. The following script visualizes the combined sensor model likelihood for a single simulated observation. Based on the similar plots from lecture and the textbook, this conditional probability plot should help you characterize the relative importance of $z_{\text{hit}}, z_{\text{short}}, z_{\text{max}}, z_{\text{rand}}$. (Hint: which should be the largest weight?) Your tuned sensor model parameters will be specific to the MuSHR LIDAR unit, so it won’t match the exact plots from lecture.

```bash
$ rosrun localization make_sensor_model_single_plot
```

As you’re tuning, it will also be helpful to visualize the likelihood of different poses given a single LIDAR scan. We’ve included a script that will pull the latest scan from the robot and evaluate your sensor model over a large number of poses covering every map cell. To use it, first launch the simulation with a new map.

```bash
$ roslaunch cse478 teleop.launch map:='$(find cse478)/maps/shapes_world_small.yaml'
```

The more interesting the map, the more interesting the laser scan. Try some different maps and poses! To reposition the robot in simulation, use the “Publish Point” tool in the RViz toolbar: select the tool, then click where on the map you want the robot to be. You can, of course, teleoperate the robot too. Reposition the robot, then when you’re happy run:

```bash
$ rosrun localization make_sensor_model_likelihood_plot
```

The node may take a few minutes to calculate all of the probabilities for larger maps. When it’s done, a plot will open showing you what your model “thinks” about each position in the map. Each pixel represents the most likely particle for the given position, with its color telling about the weight of the particle. The darker purple pixels had low weights assigned, while the brighter, yellower pixels were assigned higher weights.

The staff solution produces the following plots with our sensor model that is tuned to match the physical MuSHR LIDAR. Try to match these plots by tuning your parameters.

*Figure 4: Sensor model likelihood of the staff solution with the vehicle positioned at (4.25, 5.50, 0) on the shapes_world_small map.*

*Figure 5: Sensor model likelihood of the staff solution with the vehicle positioned at (9, 9, 0) on the shapes_world_small map.*

In general, you should expect the model to have multiple modes when the scan is ambiguous, i.e. if the car could’ve been in any number of places and “seen” the same thing with its sensor. This pattern is especially obvious in environments with symmetry and happens all the time in realistic maps. For instance, expect hallways to produce a similar effect, with a mode becoming a streak along the axis of the passage. Tuning won’t change the fact that the sensor can’t tell some locations from others, but it will adjust how much confidence the model puts in different types of plausible locations.

> **Deliverable:** Tune the parameters of your sensor model to match the reference sensor model likelihood plots in Figure 4 and Figure 5. To explain your tuning process, please save three versions of the conditional probability plot for $P(z_t^k | z_t^{k*})$ that were generated by different sensor model parameters (`sm1.png`, `sm2.png`, `sm3.png`). In your writeup, document your intermediate parameter choices and explain the visual differences between your plots.

> **Deliverable:** With the final tuned parameters, provide the sensor model likelihood plot for the robot positioned at $(-9.6, 0.0, -2.5)$ in the `maze_0` map. You can position the robot there precisely by providing arguments to the simulation launch file:
> ```bash
> roslaunch cse478 teleop.launch map:='$(find cse478)/maps/maze_0.yaml' initial_x:=-9.6 initial_y:=0 initial_theta:=-2.5
> ```
