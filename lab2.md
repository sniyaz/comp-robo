---
layout: page
title: Lab 2
description: >-
    Introduction to NumPy and ROS Subscribers.
nav_exclude: true
---

# Lab 2
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Welcome to Lab 2! In this lab, we'll combine ROS with the NumPy and Matplotlib scientific computing libraries. NumPy makes computations faster by using vectorization, and Matplotlib creates plots and other data visualizations.

⚠️ **NOTE:** This lab provides **important foundational knowledge** that we will use when we start implementing Hidden Markov Models (HMMs) in later labs. Please pay close attention to these concepts in NumPy, as they will be critical later on!

## NumPy Tutorial: Euclidean Norms

In this section, we will compute the Euclidean norm in two ways: with regular Python for loops and with functions from NumPy. 

The input will be a 2D NumPy array of shape (N, D), which contains N vectors each of dimension D. The expected output will be a 1D NumPy array of shape (N,), where each entry is the Euclidean norm of the corresponding vector. For example:

![lab2_mat]({{ site.baseurl }}/assets/lab2-assets/lab2_mat.png)

### Q1: norm_python

Complete the `norm_python` function in `introduction/src/introduction/listener.py`. 

**Requirement:** Use Python for loops to compute the Euclidean norm. To index into a 2D array, NumPy extends the Python indexing syntax to take in multiple indices, with one for each dimension (e.g., `data[i, j]`).

### Q2: norm_numpy

In the same file, complete the `norm_numpy` function. 

**Requirement:** Use NumPy functions like `np.sqrt`, `np.square`, and `np.sum` to compute the norm without explicit for loops.

#### Testing Norms
To verify your implementation of the norms, run:
```bash
python3 $(rospack find introduction)/test/norms.py
```

You can also compare the performance of your two implementations by running:
```bash
python3 $(rospack find introduction)/scripts/compare_norm
```
This script will use matplotlib to plot the mean and standard deviation of your run time. Save the figure as `runtime_comparison.png`.

## ROS Subscriber: PoseListener

Next, we'll implement a `PoseListener` class that subscribes to the car's pose and stores its location over time.

### Q3: Subscriber Initialization

In the `PoseListener` class in `introduction/src/introduction/listener.py`, initialize a subscriber to the car’s pose topic. 

To find the right topic name to subscribe to, launch the car simulation again and run `rostopic list` to see all the topics being published. There’s one topic containing `car_pose` in its name. To figure out what type of messages are being sent on that topic, use `rostopic info XXX/car_pose`. When you construct the subscriber, pass `self.callback` as the callback function.

**⚠️ HINT:** You need to set `self.subscriber` to be not `None` by initializing it with a `rospy.Subscriber` object:
```python
self.subscriber = rospy.Subscriber(topic_name, message_type, callback)
```
Where:
- `topic_name` (string): The name of the topic you found using `rostopic list`.
- `message_type` (class): The ROS message class for the topic (you must import this at the top of your file).
- `callback` (method): The method that will handle incoming messages. In this case, you should pass `self.callback`.

### Q4: Callback Implementation

Fill in the `PoseListener.callback` method.

**Requirement:** Extract the x and y position from the pose messages and save them in `self.storage`. You can see what a car pose message looks like by running `rostopic echo` on the topic.

#### Testing PoseListener
To verify your subscriber implementation, run:
```bash
rostest introduction pose_listener.test
```

## Data Collection and Visualization

Now, let’s use the `PoseListener` to actually collect some data! In `scripts/pose_listener`, we’ve already used `matplotlib` to plot the xy-locations of your car and save the resulting plot to `locations.png`. Use your `norm_numpy` function to compute the car’s distance to the origin for all the xy-locations captured by the PoseListener. Then, use `matplotlib` to plot this distance as a function of `time`, and save the resulting plot to `distances.png`.

1.  First, you need to **launch the simulator** using the sandbox map:
    ```bash
    roslaunch cse478 teleop.launch map:='$(find mushr_sim)/maps/sandbox.yaml'
    ```
2.  **In another terminal**, launch the path publisher to have the car follow a "figure 8" plan (**feel free** to follow along in RViz if you would like!)
    ```bash
    roslaunch introduction path_publisher.launch plan_file:='$(find introduction)/plans/figure_8.txt'
    ```

### Q5: Distance Calculation and Plotting

In `scripts/pose_listener`, use your `norm_numpy` function (as desbribed above) to compute the car’s distance to the origin for all the xy-locations captured by the `PoseListener`. 

**Requirement:** Use matplotlib to plot this distance as a function of time as desbribed above, and save the resulting plot to `distances.png`. The script already handles plotting the xy-locations to `locations.png`.

## 📝 Write-up

Create a **new file** `introduction/writeup/lab2.md`, and answer the following questions:

1.  What is "vectorization" in the context of NumPy, and why is it generally faster than using Python for loops?
2.  Explain the role of a "callback" function in a ROS subscriber.

Please also include the following images from **Q5** in the `introduction/writeup/` directory:

3.  The `runtime_comparison.png` plot generated by `compare_norm`.
4.  The `locations.png` plot generated by `pose_listener`.
5.  The `distances.png` plot generated by `pose_listener`.

## 🔥 Grading Breakdown

**Total Lab Points:** 30

*   **Q1 & Q2:** 10 points if `norms.py` passes.
*   **Q3 & Q4:** 5 points if `pose_listener.test` passes.
*   **Write-Up (Question Answers):** 10 points.
*   **Write-Up (Images):** 5 points for the correct plots.

## 🚀 Submission

When you are finished with the lab, **make sure** to commit all your changes and push them to your private GitHub repository.

Use the Git tag `submit-lab2` to signal completion:

```bash
$ cd ~/mushr_ws/src/mushr478
$ git tag submit-lab2
$ git push origin main submit-lab2
```

### Re-submitting

If you need to make changes after you have already submitted (and before the deadline), don't panic! You can re-submit by deleting the old tag and creating a new one:

```bash
# Delete the tag locally
$ git tag -d submit-lab2
# Delete the tag on GitHub
$ git push origin :refs/tags/submit-lab2
# Create and push the new tag
$ git tag submit-lab2
$ git push origin submit-lab2
```

**Congratulations!** You've completed Lab 2. 🏎️💨
