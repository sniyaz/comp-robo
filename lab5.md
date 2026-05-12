---
layout: page
title: Lab 5
description: >-
    Particle Filters and Recorded Bag Files.
mathjax: true
nav_exclude: true
---

# Lab 5
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Welcome to Lab 5! In this final lab of the localization project, we’ll combine the Motion Model (from Lab 3) and the Sensor Model (from Lab 4) to implement a **Particle Filter**. The Particle Filter is a powerful algorithm for state estimation that uses a set of discrete "particles" to represent the robot's belief about its state.

We will also learn how to use **ROS bag files** to test our implementation on real-world data and record our own experiments on the physical car.

## Important Implementation Details

The Particle Filter is a complex system, but we’ve provided a lot of the scaffolding for you in `particle_filter.py`. Here are a few key things to keep in mind:

### Asynchronous Updates
The particle filter does not operate in a single synchronous loop. Instead, it uses an **asynchronous structure** to update particles and weights:
*   The **motion model** is applied whenever the filter receives a control message (containing motor speed and steering angle).
*   The **sensor model** is applied whenever the filter receives a laser scan.
*   This allows the filter to utilize all incoming messages as they arrive, rather than being limited by the rate of the slowest topic.

### Shared State and Synchronization
To ensure efficiency, the `particles` and `weights` NumPy arrays are **shared** across the main classes. 
*   **Concurrency Control:** Because updates are asynchronous (e.g., a laser scan might arrive while a control message is being processed), access to these arrays is synchronized using a Python `Lock` object (`self.state_lock`).
*   **In-place Modification:** Your implementations should modify the entries of the shared NumPy arrays **in-place** whenever possible to avoid the overhead of copying large arrays.

---

## Q1: Particle Filter Implementation

In this section, you will implement the two missing pieces of the Particle Filter algorithm: initialization and resampling.

### Q1.1: Particle Initialization

When the robot first starts, or when a user provides a new "pose estimate" in RViz, we need to initialize our particles. Instead of spreading them randomly across the entire map (which is called "Global Localization"), we will sample them from a Gaussian distribution centered around a known starting state.

**Requirement:** Implement the `ParticleInitializer.reset_click_pose` method in `src/localization/particle_filter.py`. 

**Task:** Initialize the robot's belief by sampling $M$ particles from a Gaussian distribution centered around the state provided by a `geometry_msgs/Pose` message. The spread of this distribution is controlled by parameters you can find in the class.

#### Testing Initialization
To verify your implementation, run:
```bash
python3 $(rospack find localization)/test/particle_initializer.py
```

### Q1.2: Low-Variance Resampling

Resampling is the process of dropping low-probability particles and duplicating high-probability ones. This focuses our computational resources on the parts of the state space where the robot is most likely to be.

We will implement the **Low-Variance Resampler**, which covers the sample space more systematically than basic random sampling and preserves particles more effectively when weights are equal.

**Algorithm:**
1. Choose a random number $r \in [0, 1/M]$.
2. Draw samples corresponding to the sequence of values $r, r + 1/M, r + 2/M, \dots, r + (M-1)/M$.
3. For each value in the sequence, find the particle whose cumulative weight corresponds to that value.

**Requirement:** Implement the `LowVarianceSampler.resample` method in `src/localization/resampler.py`.

**Tip:** You can use `np.searchsorted` to vectorize the lookup of particles based on the cumulative sum of weights. This is much faster than using a Python loop!

#### Testing Resampling
To verify your implementation, run:
```bash
python3 $(rospack find localization)/test/resample.py
```

---

## Q2: Testing and Bag Files

Now that your filter is implemented, it's time to see it in action!

### Simulation Tests

You can test your particle filter in simulation using teleoperation:

```bash
roslaunch localization particle_filter_teleop_sim.launch map:='$(find cse478)/maps/cse2_2.yaml'
```

*   Use the **2D Pose Estimate** tool in RViz to initialize the filter.
*   Use the **Publish Point** tool to move the robot to a new location (teleporting).
*   Drive the car around and watch the particles converge!

### Running on Recorded Bag Files

A "bag file" is a recording of ROS topics (laser scans, odometry, etc.) from a real or simulated run. We can "play back" these bags to test our localization algorithm offline.

1.  **Download the test bags:**
    ```bash
    rosrun localization download_bags.sh
    ```
2.  **Run the filter on a bag:**
    ```bash
    rostest localization particle_filter.test bag_name:="circle" --text
    ```
    You can add `rviz:=true` to see the visualization, or `plot:=true` to generate a path plot.

**Success Criteria:** For the `full.bag` test, your median errors for $x, y$, and $\theta$ should all be less than **0.1**.

### Recording Your Own Bag

Once you have your filter running on the physical car, you need to record a successful localization run.

**Requirement:** Record a bag file of the car driving for at least **25 feet**. The recording must include the initial pose, inferred pose, particles, laser scan, and joystick controls.

```bash
rosbag record -O localization.bag /initialpose /car/particle_filter/inferred_pose /car/particle_filter/particles /car/teleop/joy /map
```

---

## 📝 Write-up

Create a **new file** `localization/writeup/lab5.md`. **List the names and Northeastern emails** of students in your lab group at the top, and answer the following questions:

1.  Explain the "Kidnapped Robot Problem." How does your particle filter handle it if you manually move the robot in RViz using the "Publish Point" tool?
2.  What is the purpose of the "Low-Variance" part of the resampler? How does it differ from simply sampling $M$ particles independently based on their weights?

Please also include the following in your submission:

3.  A **path plot** generated from a 60-second drive through the CSE2 map in simulation.
4.  Your **localization.bag** file from the physical car.

---

## 🔥 Grading Breakdown

**Total Lab Points:** 100

*   **Q1.1 Initialization:** 20 points if `test/particle_initializer.py` passes.
*   **Q1.2 Resampling:** 30 points if `test/resample.py` passes.
*   **Q2.1 Bag Tests:** 20 points if `rostest localization particle_filter.test bag_name:="full"` passes with errors < 0.1.
*   **Write-Up (Question Answers):** 10 points.
*   **Write-Up (Images & Bag):** 20 points for the path plot and your recorded bag file.

## 🚀 Submission

When you are finished with the lab, **make sure** to commit all your changes and push them to your private GitHub repository.

Use the Git tag `submit-lab5` to signal completion:

```bash
$ cd ~/mushr_ws/src/mushr478
$ git tag submit-lab5
$ git push origin main submit-lab5
```

### Re-submitting

If you need to make changes after you have already submitted, follow the standard process:

```bash
$ git tag -d submit-lab5
$ git push origin :refs/tags/submit-lab5
$ git tag submit-lab5
$ git push origin submit-lab5
```

**Congratulations!** You've completed the Localization project! 🏎️💨
