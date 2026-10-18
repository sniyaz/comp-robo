---
layout: page
title: Lab 9
description: >-
    Building Lazy PRM.
mathjax: true
nav_exclude: true
---

# Lab 9
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

Welcome to Lab 9! In Lab 7, we sampled configurations and implemented collision checking for states. In Lab 8, we completed roadmap graph construction by validating and adding weighted edges, and implemented the A* search algorithm to compute shortest paths across our roadmap graph.

In this lab (the last motion planning lab!) we will optimize our PRM planner. First, we will implement **Lazy A\***, which avoids collision-checking all edges during roadmap construction by deferring edge evaluation until search time. Then, we will implement **path shortcutting** as a post-processing step to smooth out and shorten paths found on the discrete roadmap graph.

As a reminder, all motion planning labs (Labs 7 through 9) share the common architecture and problem representation described in the [Lab 7 Code Overview]({{ site.baseurl }}/lab7#code-overview).

## Getting Started

Make sure you are in your project workspace, have pulled any updates, and built the workspace:

```bash
$ cd ~/mushr_ws/src/mushr478
$ git pull skeleton main
$ cd ..
$ catkin build
$ source ~/mushr_ws/devel/setup.bash
```

If the build succeeds and you can run `roscd planning`, you’re ready to start!

---

## Q1: Lazy A*

One of the main computational bottlenecks in motion planning algorithms is checking whether an edge is collision-free. This requires checking collisions between the robot and the environment for every intermediate state along an edge. Rather than checking all edges when constructing the roadmap, Lazy A* only collision-checks an edge when attempting to choose the edge during search.

**Requirement:** Implement Lazy A* in `src/planning/search.py`.

The `Roadmap` class decides whether to collision-check edges on construction based on `Roadmap.lazy`. You will use this same field to decide when edges need to be lazily evaluated by A*. When a new node is expanded, the edge from its parent to that node must be collision-checked (with `Roadmap.check_edge_validity`) to determine whether search can continue. If it is in collision, the entry cannot be used and is discarded. If it is collision-free, the algorithm can proceed as before.

#### Testing Lazy A* Search

You can verify your implementation on the provided test suite by running:
```bash
python3 $(rospack find planning)/test/astar.py
```
*(or `python3 test/astar.py` from within the `planning` directory)*

Your implementation should pass the final `test_lazy_astar` test suite in `test/astar.py`.

You should now be able to run Lazy A*! Note that you must be in the `planning` package directory (run `roscd planning`) to run the script. Your plot should match the reference plot below. Note that some edges are in collision. Save your generated plot as `halton_map1_lazy_astar_path.png`.

```bash
python3 scripts/run_search -m test/share/map1.txt -n 25 -r 3.0 --lazy --show-edges r2 -s 1 1 -g 7 8
```

![Figure 1: Lazy A* path in blue on map1.txt, from (1, 1) in green to (7, 8) in red.]({{ site.baseurl }}/assets/lab9-assets/r2_map1_lazy_astar_path.png)

---

## Q2: Path Shortcutting

**Requirement:** Implement path shortcutting in `src/planning/search.py`.

A* returns the shortest path on the graph, which is not necessarily the shortest path in the environment because not all vertices are directly connected by edges. Randomly select two indices to try and connect. Be sure to verify that an edge connecting the two vertices would be collision-free, and that connecting the two indices would actually be shorter.

#### Testing Path Shortcutting

You can verify your implementation on the provided test suite by running:
```bash
python3 $(rospack find planning)/test/shortcut.py
```
*(or `python3 test/shortcut.py` from within the `planning` directory)*

Your plot for the following command may not match the reference plot exactly. Note that the shortcut uses edges that did not exist in the graph. Note that you must be in the `planning` package directory (run `roscd planning`) to run the script. Save your generated plot as `halton_map1_shortcut_path.png`.

```bash
python3 scripts/run_search -m test/share/map1.txt -n 25 -r 3.0 --lazy --shortcut --show-edges r2 -s 1 1 -g 7 8
```

![Figure 2: Lazy A* path with shortcut postprocessing on map1.txt.]({{ site.baseurl }}/assets/lab9-assets/r2_map1_shortcut_path.png)

---

## 📝 Write-up

Create a **new file** `planning/writeup/lab9.md`. **List the names and Northeastern emails** of students in your lab group at the top, and answer the following questions:

1. **A* vs. Lazy A* Comparison:** For your choice of number of vertices and connection radius (from your Lab 8 experiments on `map2.txt`), compute the shortest path on `map2.txt` with standard A* and with Lazy A*:
   ```bash
   python3 scripts/run_search -m test/share/map2.txt -n <N> -r <R> [--lazy] r2 -s 252 115 -g 350 350
   ```
   Report the path length, planning time, and number of edges evaluated in a table. Explain any variation and trade-offs observed between regular A* and Lazy A*.
2. **Shortcutting Evaluation:** Compare the time spent on planning and shortcutting on `map1.txt`. Describe qualitatively how the paths differ before and after shortcutting.

Please also include the following images in the `planning/writeup/` directory:

3. `halton_map1_lazy_astar_path.png`: The plot of the Lazy A* path on `map1.txt` generated in Q1.
4. `halton_map1_shortcut_path.png`: The plot of the Lazy A* path with shortcut postprocessing on `map1.txt` generated in Q2.

---

## 🔥 Grading Breakdown

**Total Lab Points:** 100

*   **Q1:** 30 points if `python3 $(rospack find planning)/test/astar.py` passes the `test_lazy_astar` test suite.
*   **Q2:** 30 points if `python3 $(rospack find planning)/test/shortcut.py` passes.
*   **Write-Up (Questions & Analysis):** 20 points (10 points per question/analysis).
*   **Write-Up (Plots):** 20 points (10 points per plot: `halton_map1_lazy_astar_path.png` and `halton_map1_shortcut_path.png`).

---

## 🚀 Submission

When you are finished with the lab, **make sure** to commit all your changes and push them to your private GitHub repository.

Use the Git tag `submit-lab9` to signal completion:

```bash
$ cd ~/mushr_ws/src/mushr478
$ git tag submit-lab9
$ git push origin main submit-lab9
```

### Re-submitting

If you need to make changes after you have already submitted (and before the deadline), you can re-submit by deleting the old tag and creating a new one:

```bash
# Delete the tag locally
$ git tag -d submit-lab9
# Delete the tag on GitHub
$ git push origin :refs/tags/submit-lab9
# Create and push the new tag
$ git tag submit-lab9
$ git push origin submit-lab9
```

**Congratulations!** You've completed Lab 9 and finished the motion planning labs! 🏎️💨
