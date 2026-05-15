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

Welcome to Lab 5! In this final lab of the perception module, we’ll combine the Motion Model (from Lab 3) and the Sensor Model (from Lab 4) to implement an **Advanced Particle Filter** with Low-Variance Resampling. As mentioned in Lecture, this Particle Filter is an approximate/sampled version of the full state estimation HMM.

We will also learn how to use **ROS bag files** to test our implementation on real-world data.

## Important Implementation Details

As we mentioned in Lecture, you’ve actually already done most of the heavy lifting in Labs 3 and 4! The Particle Filter is really just the "glue" that brings those models together. We provide scaffolding in `particle_filter.py` that handles applying the motion model (when controls arrive) and weighing particles by the sensor model (when LIDAR scans arrive). You just need to implement the two remaining algorithmic steps: **initialization** and **resampling**.

That said, here are a few **important things** to keep in mind:

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

## Q1: Particle Initialization

When the robot first starts, or when a user provides a new "pose estimate" in RViz, we need to initialize our particles. Instead of spreading them randomly across the entire map (which is called "Global Initialization"), we will sample them from a Gaussian distribution centered around a known starting state.

**Requirement:** Implement the `ParticleInitializer.reset_click_pose` method in `src/localization/particle_filter.py`. 

**Task:** Initialize the robot's belief by sampling $M$ particles from a Gaussian distribution centered around the state provided by a `geometry_msgs/Pose` message. The spread of this distribution is controlled by parameters you can find in the class.

**Hint 1:** Before you start sampling the Gaussian centered at the specified initial pose, you'll need to convert it to the $[x, y, \theta]$ state representation we use in Lecture (this is also the state formulation we will use to store your particles). You will likely find the ROS [documentation](https://docs.ros.org/en/noetic/api/geometry_msgs/html/msg/Pose.html) for `geometry_msgs/Pose` useful.

**Hint 2:** When converting from the ROS message with the initial state (as mentioned in Hint 1), you'll notice the orientation component is stored as a **quaternion**. Don't worry too much about what this is: but there's a hint in the code on how you can convert this to $\theta$. You can [read more](https://en.wikipedia.org/wiki/Quaternion) about quaternions as well if you're interested (or just hate yourself and want to suffer).

**Hint 3:** Each particle should have the same starting weight: $1/M$.

**Hint 4:** As always, read the comments in the function for more on what exactly you need to do!

#### Testing Initialization
To verify your implementation, run:
```bash
python3 $(rospack find localization)/test/particle_initializer.py
```

---

## Q2: Low-Variance Resampling

Resampling is the process of dropping low-probability particles and duplicating high-probability ones. As mentioned in Lecture, this focuses our computational resources on the regions of the HMM belief where the robot is most likely to be.

To implement our resampling, we will also use the **low-variance** approach to random number generation mentioned towards the end of Lecture. This has the approach of covering the sample space more systematically than basic random sampling.

**Algorithm:**
1. Choose a random number $r \in [0, 1/M]$.
2. Draw samples corresponding to the sequence of values $r, r + 1/M, r + 2/M, \dots, r + (M-1)/M$.
3. For each value in the sequence, find the particle whose cumulative weight corresponds to that value (referring back to the resampling example from Lecture may be helpful here).

Here's a figure that illustrates this process:

![low-var-resample]({{ site.baseurl }}/assets/lab5-assets/low-var-resample.png)

**Requirement:** Implement the `LowVarianceSampler.resample` method in `src/localization/resampler.py`.

**Tip:** You can use `np.searchsorted` to vectorize the lookup of particles based on the cumulative sum of weights. This is much faster than using a Python loop!

**Hint 1:** One of the tests _specifically_ makes sure that you don't resample particles of zero weight. Make sure when you're putting things together that you don't introduce a bug where this happens.

**Hint 2:** This is quite a difficult problem, so it might be useful to print things to debug. Note that when you run the Python test below, print statements don't show up. They get saved to an `.xml` file (the test will give you the exact path).

**Hint 3:** 
> ### 🛑 STOP! READ THIS OR BE SAD! 🛑
>
> **WHAT DOES IN-PLACE MEAN?** In this lab, the `particles` and `weights` arrays are **shared** between several different classes. To make sure your changes actually "stick," you MUST modify the original objects in memory. 
> 
> **DO NOT** reassign the instance variables to a new object (e.g., `self.particles = new_particles`). This will break the shared memory link and your filter will stop working. Instead, you MUST edit the original array **index by index** to ensure the changes persist across the entire system!
{: .post-it }

#### Testing Resampling
To verify your implementation, run:
```bash
python3 $(rospack find localization)/test/resample.py
```

---

## Q3: Bag File Test

Now that your filter is implemented, it's time to see it in action! A "bag file" is a recording of ROS topics (laser scans, odometry, etc.) from a real or simulated run. We can "play back" these bags to test our localization algorithm offline.

### Simulation and Visualization

We'll replay some _real_ sensor readings from a _real_ MuSHR car using a bag file as mentioned above. To do this:

1.  **Download the test bags:**
    ```bash
    rosrun localization download_bags.sh
    ```
2.  **Run your particle filter on one of the test bags:**
    ```bash
    rostest localization particle_filter.test bag_name:="full" rviz:="true" plot:="true" --text
    ```

The second script above will let you watch the car drive around in RViz while the particles converge! It will also (more importantly) show you a figure of the **most likely** path according to your particle filter versus the path we know the car took when recording that particular bag file. Here's an example with my solution:

![low-var-resample]({{ site.baseurl }}/assets/lab5-assets/sherdil-PF.png)

### Path Comparison Deliverable

Once your filter is working, you'll need to generate a figure that compares the ground truth path (from the bag file) with your filter's estimated path. Just like the one from me above!

**Grading Policy:** I want to stress that your path just has to look **somewhat reasonable** compared to the ground truth! As I've said before, the point of this class is for you to learn these fundamental algorithms, not to spend forty hours tuning parameters. If your path follows the general shape of the ground truth, I will be very nice in grading.

**Tuning for Performance:** That said, if you *do* want to see better performance, you can always go back and tune your sensor model weights (from Lab 4) or your motion model noise parameters (from Lab 3). Finding the right balance between the two is a big part of "real" robotics engineering!

**Note on Variability:** Each time you run the plotting script, the car might follow a slightly different path (since the particle filter is a stochastic algorithm) or the script might pick a different bag file. Feel free to try a few runs until you get a plot you're happy with!

---

## 📝 Write-up

Create a **new file** `localization/writeup/lab5.md`. **List the names and Northeastern emails** of students in your lab group at the top, and answer the following questions:

1.  Consider the "Kidnapped Robot Problem": this is a classic challenge in robotics where an autonomous robot, while operating, is abruptly moved to an unknown location. How does your particle filter react to it if you manually move the robot in RViz using the "Publish Point" tool?
2.  What is the purpose of the "Low-Variance" part of the resampler? How does it differ from simply sampling $M$ particles independently based on their weights?

Please also include the following in your submission:

3.  A **path plot** generated from a 60-second drive through the CSE2 map in simulation.

---

## 🔥 Grading Breakdown

**Total Lab Points:** 100

*   **Q1 Initialization:** 20 points if `test/particle_initializer.py` passes.
*   **Q2 Resampling:** 30 points if `test/resample.py` passes.
*   **Q3 Bag Tests:** 30 points if `rostest localization particle_filter.test bag_name:="full"` passes with errors < 0.1.
*   **Write-Up (Question Answers):** 10 points.
*   **Write-Up (Path Plot):** 10 points for the path plot.

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
