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

Welcome to Lab 4! In this lab, we’ll implement the LIDAR sensor model for our MuSHR car in simulation. The sensor model allows us to calculate the likelihood of a LIDAR scan given a hypothesized robot state (aka pose) and a map. This is the second major component of our HMM State Estimator (we already built the Motion Model in Lab 3). 

## Sensor Model Implementation

The MuSHR car’s LIDAR unit emits infrared laser beams into the environment at fixed angular intervals and returns the measured distance along those beams. For an unsuccessful measurement, it returns NaN or 0. A sensor measurement $z_t$ is a vector of distances, one for each beam: $z_t = (z_t^1, \dots, z_t^K)$.

Assuming that each beam/return distance $z_t^k$ is conditionally independent (given the state $x_t$) allows us to consider each beam separately. The total likelihood is the product of the likelihoods of each beam:

$$P(z_t | x_t) = \prod_{k=1}^K P(z_t^k | x_t)$$

### Sensor Model Modes

As discussed in Lecture, we use a mixture of four different components (also called modes) to model the probability of a single beam, i.e.

$$P(z_t^k | x_t)$$

Just as a refresher, those four glorious components are:

1.  **Hit (Gaussian noise):** $p_{hit}$ models the case where the laser correctly measures the distance to an obstacle, but with some Gaussian measurement noise.
2.  **Short (Unexpected obstacles):** $p_{short}$ models the case where an unexpected obstacle (like a person) is between the car and the wall.
3.  **Max (Sensor failure):** $p_{max}$ models cases where the sensor fails to return a reading (e.g., due to a highly reflective surface).
4.  **Random (Phantom readings):** $p_{rand}$ models completely random "phantom" readings.

The mathematical definitions for these modes are:

$$
\begin{align}
p_{hit}(z_t^k | x_t) &= \begin{cases} \frac{1}{\sqrt{2\pi\sigma_{hit}^2}} \exp \left\{ -\frac{1}{2} \left( \frac{z_t^k - z_t^{k*}}{\sigma_{hit}} \right)^2 \right\} & 0 \leq z_t^k \leq z_{max} \\ 0 & \text{otherwise} \end{cases} \\
p_{short}(z_t^k | x_t) &= \begin{cases} 2 \frac{z_t^{k*} - z_t^k}{(z_t^{k*})^2} & z_t^k < z_t^{k*} \\ 0 & \text{otherwise} \end{cases} \\
p_{max}(z_t^k | x_t) &= \mathbb{I}(z_t^k = z_{max}) \\
p_{rand}(z_t^k | x_t) &= \begin{cases} \frac{1}{z_{max}} & 0 \leq z_t^k < z_{max} \\ 0 & \text{otherwise} \end{cases}
\end{align}
$$

The final probability is a weighted dot product:

$$P(z_t^k | x_t) = w_{hit} \cdot p_{hit} + w_{short} \cdot p_{short} + w_{max} \cdot p_{max} + w_{rand} \cdot p_{rand}$$

### Q1: Implement the Sensor Model

In this question, you will implement the logic to precompute the sensor model likelihood table. To speed up calculations at runtime, we pre-calculate a 2D table where:
*   **Rows** represent the **real** measured beam/return LIDAR value ($z_t^k$).
*   **Columns** represent the **simulated** (expected) LIDAR value based on raycasting ($z_t^{k*}$).
*   **Values** represent the _cached_ probability of each real measured LIDAR reading $z_t^k$ (at that row) assuming the simulated LIDAR value $z_t^{k*}$ (at that column). That is:

$$P(z_t^k | x_{t})$$

**Requirement:** Implement the sensor model in the `SingleBeamSensorModel.precompute_sensor_model` method (`src/localization/sensor_model.py`).

**Notes:**
- Accept zero as a possible weight for any factor, as long as at least one weight is nonzero.
- Make sure each column of the table is normalized to sum to 1 (this follows from the basic rule that the probabilities of all possible outcomes for an event must sum to 1).
- The parameters $w_{hit}$, $w_{short}$, $w_{max}$, $w_{rand}$ and $\sigma_{hit}$ are provided as instance variables. Note that for the weights the convention in code is a bit different than in lecture: $w_{hit}$, for example, is accessed by `self.z_hit`.

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

**Deliverable:** Tune the parameters of your sensor model ($z_{hit}, z_{short}, z_{max}, z_{rand}, \sigma_{hit}$) to match the reference behavior. Save three versions of the conditional probability plot as `sm1.png`, `sm2.png`, and `sm3.png` (where `sm3.png` is your final tuned version). 

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
