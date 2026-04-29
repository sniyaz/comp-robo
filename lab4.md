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

## Sensor Model Review

The MuSHR car’s LIDAR unit emits infrared laser beams into the environment at fixed angular intervals and returns the measured distance along those beams. A sensor measurement $z_t$ is a vector of distances, one for each beam: $z_t = (z_t^1, \dots, z_t^K)$.

Assuming that each beam/return distance $z_t^k$ is conditionally independent (given the state $x_t$) allows us to consider each beam separately. The total likelihood of the sensor measurement (i.e. the entire LIDAR scan) is thus the product of the likelihoods of each beam:

$$P(z_t | x_t) = \prod_{k=1}^K P(z_t^k | x_t)$$

**Note:** We actually handle **a lot** of the computation of the sensor model for you in Lab 4 (otherwise the Lab would take you several days). First, we’ve handled the code that generates simulated observations given that the robot state is $x_{t}$. If you're interested, we actually use a raycasting library called rangelibc (implemented in C++/CUDA), which casts rays into the map from the robot’s state/pose and measures the distance to any obstacle/wall the ray encounters.

We **also** handle the multiplying out of the individual beam/return probabilities (under our conditional probability assumption) to get the final probability of the entire LIDAR scan. The **only** thing we're asking you to compute in Lab 4 is the probability of an **individual** beam/return:

$$P(z_t^k | x_t)$$

Why? Because this is by far the hardest (and most important) part of the entire Sensor Model, of course! 😄 I'm nice, but I'm not _that_ nice 😊

### Sensor Model Modes For Single LIDAR Beam

As discussed in Lecture, we use a mixture of four different components (also called modes) to model the probability of a **single** beam, i.e.

$$P(z_t^k | x_t)$$

Just as a refresher, those four glorious components are:

1.  **Hit (Gaussian noise):** $p_{hit}$ models the case where the laser correctly measures the distance to an obstacle, but with some Gaussian measurement noise.
2.  **Short (Unexpected obstacles):** $p_{short}$ models the case where an unexpected obstacle (like a person) is between the car and the wall.
3.  **Max (Sensor failure):** $p_{max}$ models cases where the sensor fails to return a reading (e.g., due to an extremely dark surface).
4.  **Random (Phantom readings):** $p_{rand}$ models completely unexplainable "phantom" readings.

The mathematical definitions for these modes are:

$$
\begin{align}
p_{hit}(z_t^k | x_t) &= \begin{cases} \frac{1}{\sqrt{2\pi\sigma_{hit}^2}} \exp \left\{ -\frac{1}{2} \left( \frac{z_t^k - z_t^{k*}}{\sigma_{hit}} \right)^2 \right\} & 0 \leq z_t^k \leq z_{max} \\ 0 & \text{otherwise} \end{cases} \\
p_{short}(z_t^k | x_t) &= \begin{cases} 2 \frac{z_t^{k*} - z_t^k}{z_t^{k*}} & z_t^k < z_t^{k*} \\ 0 & \text{otherwise} \end{cases} \\
p_{max}(z_t^k | x_t) &= \mathbb{I}(z_t^k = z_{max}) \\
p_{rand}(z_t^k | x_t) &= \begin{cases} \frac{1}{z_{max}} & 0 \leq z_t^k < z_{max} \\ 0 & \text{otherwise} \end{cases}
\end{align}
$$

The final probability is a weighted dot product:

$$P(z_t^k | x_t) = w_{hit} \cdot p_{hit} + w_{short} \cdot p_{short} + w_{max} \cdot p_{max} + w_{rand} \cdot p_{rand}$$

### Q1: Implement the Sensor Model

As mentioned earlier, in this question you will implement the computation of single beam/return probabilities. All the other machinery of the Sensor Model is handled by the skeleton class we provide you.

Actually, we're going to do something **even cooler** (if that's even possible). We're going to speed up the sensor model by pre-computing and caching **every single possible** single-beam probability.

How is this even possible? That's actually one of the nice things about this being a **computational robotics** class: we're working completely in simulation. In the simulated MuSHR car world, we don't measure distance in meters or (as freedom-loving people do) feet. Instead, we measure distance in **integer numbers of pixels**. 

The fact that our beam/return distances are integer values means we can build a 2D table of all possible probabilities! Specifically, we will store the LIDAR range values in a table where the index of each row represents the actual measured value, the index of each column represents the expected value for a given LIDAR range, and the value at those coordinates is the probability.

Once you build this 2D table for us representing all possible single-beam probabilities, the rest of our code will take that table and use it to (very quickly) compute the rest of the Sensor Model.

Again: in order to speed up the Sensor Model at runtime, in this question you will implement logic to **precompute** the following sensor model likelihood table:
*   **Rows** represent the **real** measured beam/return LIDAR value ($z_t^k$).
*   **Columns** represent the **simulated** (expected) LIDAR value based on raycasting ($z_t^{k*}$).
*   **Values** represent the _cached_ probability of each real measured LIDAR reading $z_t^k$ (at that row) assuming the simulated LIDAR value $z_t^{k*}$ (at that column). That is:

$$P(z_t^k | x_{t})$$

**Requirement:** Implement the `SingleBeamSensorModel.precompute_sensor_model` method in `src/localization/sensor_model.py`. We have already set up the dimensions of the output table (`prob_table`) for you, but you need to fill it in.

> ### 🛑 HERE BE DRAGONS! (Read This!) 🛑
> CS 7680 is a graduate class, so we're going to hold your hand on this problem a bit less than we would in an undergraduate one. That said, this problem is **really hard**. Read these hints to save yourself a lot of existential dread and despair.
> 
> **Helper Functions:** We've intentionally left it open-ended how you should structure your code. The main thing we'll say though: if you try to do everything in one giant function, you're probably in for a bad time. Break things up.
>
> **Normalizing:** As an implementation detail, you'll need to make sure each column of `prob_table` is normalized to sum to 1 (this follows from the basic rule that the probabilities of all possible outcomes for an event must sum to 1). You can actually do this in a single line with NumPy: a quick Google search here will help a lot.
>
> **Parameters:** The parameters $w_{hit}$, $w_{short}$, $w_{max}$, $w_{rand}$ and $\sigma_{hit}$ are provided as instance variables. Note that for the weights the convention in code is a bit different than in lecture: $w_{hit}$, for example, is accessed by `self.z_hit`.
>
> **Max Sensor Range:** The max sensor range $z_{max}$ is actually passed as the `max_r` parameter to `precompute_sensor_model()`. Not `self.z_max`, which is actually $w_{max}$ (I'm so sorry guys).
>
> **Gaussian of Zero Standard Deviation** Because your TA and I are evil, we've included a test case where $\sigma_{hit}$ is set to zero. If you just plug this into the standard Gaussian equation, it actually causes a **divide by zero** error (you can stare at the formula and see why). If you see these errors in the autograder, it's probably this. Luckily, computing the probability of a Gaussian of zero standard deviation is actually pretty simple: the probability is 1 at the mean and zero everywhere else. In other words, setting $\sigma$ for a Gaussian to zero turns it into a Dirac Delta (where all the probability is stuffed into a single point).
{: .post-it }

#### Testing the Sensor Model
To verify your implementation, run:
```bash
python3 $(rospack find localization)/test/sensor_model.py
```

---

## Parameter Tuning

Congrats! Thanks to your hard work, our Sensor Model is fully implemented. Now we just need to find weights and noise parameters that approximate the real car's performance.

A little hint for you as well: which between $w_{hit}$, $w_{short}$, $w_{max}$, and $w_{rand}$ do you think should have the largest weight?

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
