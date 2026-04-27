---
layout: page
title: Lab 4
description: >-
    LIDAR Sensor Models and Parameter Tuning.
mathjax: true
nav_exclude: true
---

# Lab 4
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Welcome to Lab 4! In this lab, we’ll implement the LIDAR sensor model for our MuSHR car in simulation. The sensor model allows us to calculate the likelihood of a LIDAR scan given a hypothesized robot pose and a map. This is the second major component of our HMM State Estimator (we already built the Motion Model in Lab 3).

## Sensor Model Implementation

The MuSHR car’s LIDAR unit emits infrared laser beams into the environment at fixed angular intervals and returns the measured distance along those beams. A sensor measurement $\mathbf{z}_t$ is a vector of distances, one for each beam: $\mathbf{z}_t = (z_t^1, \dots, z_t^K)$. 

Assuming that each distance $z_t^k$ is conditionally independent (given the state $\mathbf{x}_t$ and map $m$) allows us to consider each beam separately. The total likelihood is the product of the likelihoods of each beam:

$$P(\mathbf{z}_t | \mathbf{x}_t, m) = \prod_{k=1}^K P(z_t^k | \mathbf{x}_t, m)$$

### Sensor Model Modes

We model the probability $P(z_t^k | \mathbf{x}_t, m)$ using a mixture of four different modes:

1.  **Hit (Gaussian noise):** $p_{\text{hit}}$ models the case where the laser correctly measures the distance to an obstacle, but with some Gaussian measurement noise.
2.  **Short (Unexpected obstacles):** $p_{\text{short}}$ models the case where an unexpected obstacle (like a person) is between the car and the wall.
3.  **Max (Sensor failure):** $p_{\text{max}}$ models cases where the sensor fails to return a reading (e.g., due to a highly reflective surface).
4.  **Random (Phantom readings):** $p_{\text{rand}}$ models completely random "phantom" readings.

The mathematical definitions for these modes are:

$$
\begin{align}
p_{\text{hit}}(z_t^k | \mathbf{x}_t, m) &= \begin{cases} \frac{1}{\sqrt{2\pi\sigma_{\text{hit}}^2}} \exp \left\{ -\frac{1}{2} \left( \frac{z_t^k - z_t^{k*}}{\sigma_{\text{hit}}} \right)^2 \right\} & 0 \leq z_t^k \leq z_{\text{max}} \\ 0 & \text{otherwise} \end{cases} \\
p_{\text{short}}(z_t^k | \mathbf{x}_t, m) &= \begin{cases} 2 \frac{z_t^{k*} - z_t^k}{(z_t^{k*})^2} & z_t^k < z_t^{k*} \\ 0 & \text{otherwise} \end{cases} \\
p_{\text{max}}(z_t^k | \mathbf{x}_t, m) &= \mathbb{I}(z_t^k = z_{\text{max}}) \\
p_{\text{rand}}(z_t^k | \mathbf{x}_t, m) &= \begin{cases} \frac{1}{z_{\text{max}}} & 0 \leq z_t^k < z_{\text{max}} \\ 0 & \text{otherwise} \end{cases}
\end{align}
$$

The final probability is a weighted sum:

$$P(z_t^k | \mathbf{x}_t, m) = z_{\text{hit}} \cdot p_{\text{hit}} + z_{\text{short}} \cdot p_{\text{short}} + z_{\text{max}} \cdot p_{\text{max}} + z_{\text{rand}} \cdot p_{\text{rand}}$$

### Q1: Implement the Sensor Model

In this question, you will implement the logic to precompute the sensor model likelihood table. To speed up calculations at runtime, we pre-calculate a 2D table where:
*   **Rows** represent the actual measured LIDAR value ($z_t^k$).
*   **Columns** represent the expected LIDAR value based on raycasting ($z_t^{k*}$).
*   **Values** represent the probability $P(z_t^k | z_t^{k*})$.

**Requirement:** Implement the sensor model in the `SingleBeamSensorModel.precompute_sensor_model` method (`src/localization/sensor_model.py`). 

**Notes:**
- Your implementation should be **vectorized** (no Python loops!).
- Accept zero as a possible weight for any factor, as long as at least one weight is nonzero.
- Make sure each column of the table is normalized to sum to 1.
- The parameters $z_{\text{hit}}, z_{\text{short}}, z_{\text{max}}, z_{\text{rand}}$ and $\sigma_{\text{hit}}$ are provided as instance variables (e.g., `self.z_hit`).

#### Testing the Sensor Model
To verify your implementation, run:
```bash
python3 test/sensor_model.py
```

---

## Parameter Tuning

Once your sensor model is implemented, you need to find the right weights and noise parameters to match the physical car's performance.

### Q2: Exploring and Tuning Parameters

We’ve provided two tools for tuning. The first visualizes the likelihood for a single observation:

```bash
rosrun localization make_sensor_model_single_plot
```

The second visualizes the likelihood of different poses given a full LIDAR scan. Launch the simulator first:

```bash
roslaunch cse478 teleop.launch map:='$(find cse478)/maps/shapes_world_small.yaml'
```

Then run the likelihood visualization:

```bash
rosrun localization make_sensor_model_likelihood_plot
```

**Deliverable:** Tune the parameters of your sensor model ($z_{\text{hit}}, z_{\text{short}}, z_{\text{max}}, z_{\text{rand}}, \sigma_{\text{hit}}$) to match the reference behavior. Save three versions of the conditional probability plot as `sm1.png`, `sm2.png`, and `sm3.png` (where `sm3.png` is your final tuned version). 

In your writeup, provide a final likelihood plot for the robot positioned at $(-9.6, 0.0, -2.5)$ in the `maze_0` map. 

```bash
roslaunch cse478 teleop.launch map:='$(find cse478)/maps/maze_0.yaml' initial_x:=-9.6 initial_y:=0 initial_theta:=-2.5
```

---

## 📝 Write-up

Create a **new file** `localization/writeup/lab4.md`. **List the names and Northeastern emails** of students in your lab group at the top, and answer the following questions:

1.  What are the drawbacks of the conditional independence assumption for LIDAR beams? How does the `LaserScanSensorModelROS` class (which you can find in the codebase) mitigate these drawbacks?
2.  Document your tuning process for the sensor model parameters. What did you observe as you changed the weights for the four different modes?

Please also include the following images in the `localization/writeup/` directory:

3.  The `sm1.png`, `sm2.png`, and `sm3.png` figures showing your tuning process.
4.  The final sensor model likelihood plot for the `maze_0` pose specified in Q2.

---

## 🔥 Grading Breakdown

**Total Lab Points:** 60 (Estimated)

*   **Q1 Implementation:** 30 points if `python3 test/sensor_model.py` passes.
*   **Write-Up (Question Answers):** 10 points.
*   **Write-Up (Images):** 20 points for the tuning plots and final likelihood plot.

## 🚀 Submission

When you are finished with the lab, **make sure** to commit all your changes and push them to your private GitHub repository.

Use the Git tag `submit-lab4` to signal completion:

```bash
$ cd ~/mushr_ws/src/mushr478
$ git tag submit-lab4
$ git push origin main submit-lab4
```

### Re-submitting

If you need to make changes after you have already submitted, follow the same process as Lab 2 and 3:

```bash
$ git tag -d submit-lab4
$ git push origin :refs/tags/submit-lab4
$ git tag submit-lab4
$ git push origin submit-lab4
```

**Congratulations!** You've completed Lab 4. 🏎️💨
