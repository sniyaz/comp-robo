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

In Labs 7 through 9, you will implement a graph-based motion planner. We begin in this Lab by constructing roadmaps for various environments and motion planning problems.

## Getting Started

The motion planning code for CS 6983 lives in the `planning` subdirectory. The workflow to run these Labs is similar to the perception module (**remember** to always source)!

```bash
$ cd ~/mushr_ws/src/mushr478
$ cd ..
$ catkin build
$ source ~/mushr_ws/devel/setup.bash
```

If the build succeeds and you can run `roscd planning`, you’re ready to start! Please ask on the course discussion board or reach out to course staff if you run into any issues at this point.

## Code Overview

Labs 7 through 9 share a common codebase to handle the various components of the motion planning problem. We provide a walkthrough here, though please note that not all of these components will be implemented in **this** Lab. 

The first step of motion planning is to define the C-space (`src/planning/problems.py`). These labs only considers `PlanarProblem`s: we focus on the `R2Problem` child class **exclusively**. The `R2Problem` configuration space only considers `x` and `y` positions: as mentioned in Lecture, this space is used since it makes the visualizations easier to understand.

The `PlanarProblem` class implements shared functionality, such as collision-checking. The specific problems implement their own heuristic and steering function to connect two configurations. After defining these classes, the rest of your planning algorithm can abstract away these particulars of the configuration space. (To solve a new type of problem, just implement a corresponding problem class.)

The next step of sampling-based motion planning is to construct a roadmap by sampling configurations. Sampler classes include `HaltonSampler`, `LatticeSampler`, and `RandomSampler` (`src/planning/samplers.py`). You will fill in `HaltonSampler` to generate samples using the [Halton pseudorandom sampler](https://observablehq.com/@jrus/halton). Then, you will complete the `Roadmap` class (`src/planning/roadmap.py`) to finish constructing roadmaps. To search the roadmap, you will implement Lazy A* and path shortcutting (`src/planning/search.py`).

The `Roadmap` class contains many useful methods and fields. Three fields are of particular importance: `graph`, `vertices`, and `weighted_edges`. `Roadmap.graph` is a [NetworkX](https://networkx.org/) graph, either [undirected](https://networkx.org/documentation/stable/reference/classes/graph.html) or [directed](https://networkx.org/documentation/stable/reference/classes/digraph.html) depending on the problem type. The starter code already handles interacting with this object. Nodes in the NetworkX graph have integer labels. These are indices into `Roadmap.vertices`, a NumPy array of configurations corresponding to each node in the graph. `Roadmap.weighted_edges` is a NumPy array of edges and edge weights; each row `(u, v, w)` describes an edge where `u` and `v` are the same integer-labeled nodes and `w` is the length of the edge between them.

---

## Q1: Halton Sampling

Complete the `HaltonSampler` class in `samplers.py`. The `HaltonSampler` maintains a separate generator (with a separate `base`) for each dimension of the configuration space. The `compute_sample` method returns a deterministic sample between 0 and 1 for a given `index` and `base`, which is later scaled by the `sample` method to match the extents of the configuration space. This [blog post](https://observablehq.com/@jrus/halton) on Halton sequences might be a useful reference for implementing `compute_sample` (in addition to the slides from Lecture).

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

Implement `PlanarProblem.check_state_validity` in `problems.py`. *(Tip: While not strictly required, we strongly recommend vectorizing your implementation using NumPy to make collision checking run much faster!)* This method will be used to collision-check batches of many states (states that have been sampled as potential vertices or interpolated states along an edge). Your implementation should ensure that configurations are within the extents of the problem space (`PlanarProblem.extents`), and that the x and y components are collision-free (i.e. that they lie completely within `PlanarProblem.permissible_region`). For simplicity, we will assume that the robot is a point, so you only need to check one entry of the permissible region for each state.

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

Create a **new file** `planning/writeup/lab7.md`. **List the names and Northeastern emails** of students in your lab group at the top, and answer the following question:

1. How do quasi-random sequences like the Halton sequence differ from pseudo-random uniform sampling? What advantages does Halton sampling provide when constructing roadmaps for motion planning?

Please also include the following images in the `planning/writeup/` directory:

2. The plot of 100 Halton samples on empty space (`r2_empty.png`) generated in Q1.
3. The plot of 100 collision-free Halton samples on `map1.txt` (`r2_map1.png`) generated in Q2.

---

## 🔥 Grading Breakdown

**Total Lab Points:** 70

*   **Q1:** 20 points if `python3 $(rospack find planning)/test/samplers.py` passes.
*   **Q2:** 20 points if `python3 $(rospack find planning)/test/problems.py` passes.
*   **Write-Up (Question Answer):** 10 points (partial credit may be assigned).
*   **Write-Up (Plots):** 20 points (10 points per plot).

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
