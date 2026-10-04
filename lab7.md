---
layout: page
title: Lab 7
description: >-
    Sampling-Based Motion Planning, Halton Sampling, and State Validity.
mathjax: true
nav_exclude: true
---

# Lab 7
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

![Planning]({{ site.baseurl }}/assets/lab7-assets/plan.png){: style="width: 60%;" }

## Overview

In this project, you will implement a sampling-based motion planning system. You will construct roadmaps for different environments and planning problems. You will implement the Lazy A* algorithm to search this graph efficiently, and implement a postprocessor to locally improve the path on the graph. By the end of this project, you will have an integrated system that combines all the components you’ve previously developed in this course!

Please complete this assignment with your group from the previous assignment.

The relevant lectures are:
* Introduction to Planning
* Heuristic Search
* Sampling-Based Motion Planning
* Lazy Search, Planning for Vehicles

## Getting Started

The MuSHR dependencies are the same as when Project 3 was released. To pull the planning starter code from our starter repository, follow these instructions (assuming you’ve already set the `upstream` remote previously):

```bash
$ cd ~/mushr_ws/src/mushr478
$ git pull upstream main
$ cd ..
$ catkin build
$ source ~/mushr_ws/devel/setup.bash
```

If the build succeeds and you can run `roscd planning`, you’re ready to start! Please ask on the course discussion board or reach out to course staff if you run into any issues at this point.

## Code Overview

The first step of motion planning is to define the problem space (`src/planning/problems.py`). This project only considers `PlanarProblem`s like `R2Problem` and `SE2Problem`. The `R2Problem` configuration space only considers x- and y- position, while the `SE2Problem` also includes orientation. The `PlanarProblem` class implements shared functionality, such as collision-checking. The specific problems implement their own heuristic and steering function to connect two configurations. After defining these classes, the rest of your planning algorithm can abstract away these particulars of the configuration space. (To solve a new type of problem, just implement a corresponding problem class.)

The next step of sampling-based motion planning is to construct a roadmap by sampling configurations. Sampler classes include `HaltonSampler`, `LatticeSampler`, and `RandomSampler` (`src/planning/samplers.py`). You will fill in `HaltonSampler` to generate samples using the [Halton pseudorandom sampler](https://observablehq.com/@jrus/halton). Then, you will complete the `Roadmap` class (`src/planning/roadmap.py`) to finish constructing roadmaps. To search the roadmap, you will implement (Lazy) A* and path shortcutting (`src/planning/search.py`).

The `Roadmap` class contains many useful methods and fields. Three fields are of particular importance: `graph`, `vertices`, and `weighted_edges`. `Roadmap.graph` is a [NetworkX](https://networkx.org/) graph, either [undirected](https://networkx.org/documentation/stable/reference/classes/graph.html) or [directed](https://networkx.org/documentation/stable/reference/classes/digraph.html) depending on the problem type.[^1] The starter code already handles interacting with this object. Nodes in the NetworkX graph have integer labels. These are indices into `Roadmap.vertices`, a NumPy array of configurations corresponding to each node in the graph. `Roadmap.weighted_edges` is a NumPy array of edges and edge weights; each row `(u, v, w)` describes an edge where `u` and `v` are the same integer-labeled nodes and `w` is the length of the edge between them.

The `PlannerROS` class in `src/control/planner_ros.py` provides a ROS interface to your planning algorithms. It adds the current robot state (estimated with your particle filter from Project 2) and the desired goal to the roadmap as the start and goal nodes. Then, it invokes Lazy A* and shortcutting to compute a path. Finally, this path is sent to a path tracking controller (your MPC algorithm from Project 3).

[^1]: `SE2Problem`s require directed edges.

---

## Q1: Halton Sampling

Complete the `HaltonSampler` class in `samplers.py`. The `HaltonSampler` maintains a separate generator (with a separate `base`) for each dimension of the configuration space. The `compute_sample` method returns a deterministic sample between 0 and 1 for a given `index` and `base`, which is later scaled by the `sample` method to match the extents of the configuration space. This [blog post](https://observablehq.com/@jrus/halton) on Halton sequences might be a useful reference for implementing `compute_sample`.

The extents describe the lower and upper bounds of the space being sampled. Your implementation of `sample` should scale them linearly: a 0 returned by `compute_sample` corresponds to the lower bound, while a 1 corresponds to the upper bound.

#### Testing Halton Sampling
After completing Q1, expect to pass all the test suites in:
```bash
python3 $(rospack find planning)/test/samplers.py
```
*(or `python3 test/samplers.py` from within the `planning` directory)*

You can visualize the sampled configurations with the following command. Your plot should match the reference plot below.

```bash
python3 scripts/roadmap --num-vertices 100 --lazy
```

![Figure 1: Halton samples]({{ site.baseurl }}/assets/lab7-assets/r2_empty.png)

---

## Q2: State Validity Checking

Implement `PlanarProblem.check_state_validity` in `problems.py`. Remember to vectorize! This method will be used to collision-check batches of many states (states that have been sampled as potential vertices or interpolated states along an edge), so it needs to be efficient. Your implementation should ensure that configurations are within the extents of the problem space (`PlanarProblem.extents`), and that the x and y components are collision-free (i.e. that they lie completely within `PlanarProblem.permissible_region`). For simplicity, we will assume that the robot is a point, so you only need to check one entry of the permissible region for each state.

#### Testing State Validity
You can verify your implementation on the provided test suites by running:
```bash
python3 $(rospack find planning)/test/problems.py
```
*(or `python3 test/problems.py` from within the `planning` directory)*

The following command will now sample until there are 100 collision-free vertices. Your plot should match the reference plot below.

```bash
python3 scripts/roadmap --text-map test/share/map1.txt --num-vertices 100 --lazy
```

![Figure 2: Halton samples on map1.txt]({{ site.baseurl }}/assets/lab7-assets/r2_map1.png)

---

## 📝 Write-up

Create a **new file** `planning/writeup/lab7.md`. **List the names and Northeastern emails** of students in your lab group at the top, and answer the following questions:

1. How do quasi-random sequences like the Halton sequence differ from pseudo-random uniform sampling? What advantages does Halton sampling provide when constructing roadmaps for motion planning?
2. Why is vectorization critical for `check_state_validity` when constructing roadmaps and evaluating edges?

Please also include the following images in the `planning/writeup/` directory:

3. The plot of 100 Halton samples on empty space (`r2_empty.png`) generated in Q1.
4. The plot of 100 collision-free Halton samples on `map1.txt` (`r2_map1.png`) generated in Q2.

---

## 🔥 Grading Breakdown

**Total Lab Points:** 60

*   **Q1:** 20 points if `python3 $(rospack find planning)/test/samplers.py` passes.
*   **Q2:** 20 points if `python3 $(rospack find planning)/test/problems.py` passes.
*   **Write-Up (Questions):** 10 points (5 points per question, partial credit may be assigned).
*   **Write-Up (Plots):** 10 points (5 points per plot).

---

## 🚀 Submission

When you are finished with the lab, **make sure** to commit all your changes and push them to your private GitHub repository.

Use the Git tag `submit-lab7` to signal completion:

```bash
$ cd ~/mushr_ws/src/mushr478
$ git tag submit-lab7
$ git push origin main submit-lab7
```

### Re-submitting

If you need to make changes after you have already submitted (and before the deadline), you can re-submit by deleting the old tag and creating a new one:

```bash
# Delete the tag locally
$ git tag -d submit-lab7
# Delete the tag on GitHub
$ git push origin :refs/tags/submit-lab7
# Create and push the new tag
$ git tag submit-lab7
$ git push origin submit-lab7
```

**Congratulations!** You've completed Lab 7! 🏎️💨
