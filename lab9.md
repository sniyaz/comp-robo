---
layout: page
title: Lab 9
description: >-
    Building Lazy PRM and LazySP.
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

Welcome to Lab 9! In Lab 7, we sampled configurations and implemented collision checking for states. In Lab 8, we completed roadmap graph construction by validating and adding weighted edges, and implemented graph search (Dijkstra / A*) to compute shortest paths across our roadmap graph.

In this lab (the last motion planning lab!) we will optimize our PRM planner. First, we will implement **Lazy PRM** (Lazy Dijkstra or Lazy A*), which avoids collision-checking all edges during roadmap construction by deferring edge evaluation until search time. Then, we will implement **path shortcutting** as a post-processing step to smooth out and shorten paths found on the discrete roadmap graph. Finally, we will implement **LazySP**, an even lazier planner that only collision-checks edges that lie on a candidate shortest path.

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

😈 **Make sure to re-pull from skeleton:** I've added a fun new problem for you! :)

If the build succeeds and you can run `roscd planning`, you’re ready to start!

---

## Q1: Lazy PRM

One of the main computational bottlenecks in motion planning algorithms is checking whether an edge is collision-free. This requires checking collisions between the robot and the environment for every intermediate state along an edge. Rather than checking all edges when constructing the roadmap, Lazy PRM only collision-checks an edge when attempting to choose the edge during search.

**Requirement:** Implement Lazy PRM in `src/planning/search.py`.

The `Roadmap` class decides whether to collision-check edges on construction based on `Roadmap.lazy`. You will use this same field to decide when edges need to be lazily evaluated during search. When a new node is expanded, the edge from its parent to that node must be collision-checked (with `Roadmap.check_edge_validity`) to determine whether search can continue. If it is in collision, the entry cannot be used and is discarded. If it is collision-free, the algorithm can proceed as before.

#### Testing Lazy PRM Search

You can verify your implementation on the provided test suite by running:
```bash
python3 $(rospack find planning)/test/dijkstra.py
```
*(or `python3 test/dijkstra.py` from within the `planning` directory)*

Your implementation should pass the final `test_lazy_prm` test suite in `test/dijkstra.py`.

You should now be able to run Lazy PRM! Note that you must be in the `planning` package directory (run `roscd planning`) to run the script. Your plot should match the reference plot below. Note that some edges are in collision. Save your generated plot as `halton_map1_lazy_prm_path.png`.

```bash
python3 scripts/run_search -m test/share/map1.txt -n 25 -r 3.0 --lazy --show-edges r2 -s 1 1 -g 7 8
```

![Figure 1: Lazy PRM path in blue on map1.txt, from (1, 1) in green to (7, 8) in red.]({{ site.baseurl }}/assets/lab9-assets/r2_map1_lazy_prm_path.png)

---

## Q2: Path Shortcutting

**Requirement:** Implement path shortcutting in `src/planning/search.py`.

PRM returns the shortest path on the graph, which is not necessarily the shortest path in the environment because not all vertices are directly connected by edges. Randomly select two indices to try and connect. Be sure to verify that an edge connecting the two vertices would be collision-free, and that connecting the two indices would actually be shorter.

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

![Figure 2: Lazy PRM path with shortcut postprocessing on map1.txt.]({{ site.baseurl }}/assets/lab9-assets/r2_map1_shortcut_path.png)

---

## Q3: LazySP

Lazy PRM defers collision checking until search time, but it still checks the edge to *every* node it expands, including many nodes that never end up on the final path. **LazySP** (Lazy Shortest Path) is lazier still: it only collision-checks edges that lie on a candidate shortest path. You can read more in [A Unifying Formalism for Shortest Path Problems with Expensive Edge Evaluations via Lazy Best-First Search over Paths with Edge Selectors](https://arxiv.org/abs/1603.03490) (Dellin & Srinivasa, ICRA 2016), which formalizes the LazySP class of algorithms and investigates different edge selectors.

LazySP repeats the following loop:

1. **Search:** find the shortest path from the start to the goal, *optimistically assuming every edge that hasn't been checked yet is collision-free*.
2. **Evaluate (forward selector):** walk along that path from the start towards the goal, collision-checking each edge in order.
3. **Hit a collision?** Delete that edge from the graph and **immediately** go back to step 1 (don't bother checking the rest of the path: it is no longer the shortest path).
4. **All clear?** If every edge on the path is collision-free, return the path.

Because the same edge can appear on many candidate paths, LazySP must **remember** the result of every edge it has checked (both collision-free and in-collision), and never collision-check the same edge twice.

**Requirement:** Implement `check_edge` and `solve` in the `LazySP` class (`src/planning/lazysp.py`).

*   `check_edge(u, v)` should return whether an edge is collision-free, memoizing results in `self.edge_status`. Only call `self.rm.check_edge_validity` (which increments the `Roadmap.edges_evaluated` counter) for edges that haven't been checked yet. Use the provided `self.edge_key(u, v)` to build dictionary keys (R2 roadmaps are undirected, so `(u, v)` and `(v, u)` are the same edge).
*   `solve(start, goal)` should implement the loop above. Remove edges in collision from `self.graph` (a copy of the roadmap's graph) with [`self.graph.remove_edge(u, v)`](https://networkx.org/documentation/stable/reference/classes/generated/networkx.Graph.remove_edge.html).

> **HINT:** You do **not** need to write a new graph search! A `LazySP` object can stand in for a `Roadmap`: it has its own `graph` (with collision edges removed) and `lazy = False` (so your search never collision-checks edges itself), and forwards everything else to the underlying roadmap. That means you can reuse your search (and `extract_path`) from Lab 8 directly with `search.dijkstra(self, start, goal)`.
{: .post-it }

#### Testing LazySP

You can verify your implementation on the provided test suite by running:
```bash
python3 $(rospack find planning)/test/lazysp.py
```
*(or `python3 test/lazysp.py` from within the `planning` directory)*

Your implementation should pass the `test_lazysp_r2` test suite. Note that on the R2 test problem, LazySP finds the same path as Lazy PRM while evaluating only 10 edges (compared to 14 for Lazy PRM with Dijkstra).

You should now be able to run LazySP! Note that you must be in the `planning` package directory (run `roscd planning`) to run the script. Your plot should match the reference plot below, and the script should report that **14** edges were evaluated (compared to 24 for Lazy PRM and 60 for PRM). Save your generated plot as `halton_map1_lazysp_path.png`.

```bash
python3 scripts/run_search -m test/share/map1.txt -n 25 -r 3.0 --algorithm lazysp --show-edges r2 -s 1 1 -g 7 8
```

![Figure 3: LazySP path in blue on map1.txt, from (1, 1) in green to (7, 8) in red.]({{ site.baseurl }}/assets/lab9-assets/r2_map1_lazysp_path.png)

---

## 📝 Write-up

Create a **new file** `planning/writeup/lab9.md`. **List the names and Northeastern emails** of students in your lab group at the top, and answer the following questions:

1. **PRM vs. Lazy PRM vs. LazySP Comparison:** For your choice of number of vertices and connection radius (from your Lab 8 experiments on `map2.txt`), compute the shortest path on `map2.txt` with standard PRM, with Lazy PRM, and with LazySP:
   ```bash
   # PRM
   python3 scripts/run_search -m test/share/map2.txt -n <N> -r <R> r2 -s 252 115 -g 350 350
   # Lazy PRM
   python3 scripts/run_search -m test/share/map2.txt -n <N> -r <R> --lazy r2 -s 252 115 -g 350 350
   # LazySP
   python3 scripts/run_search -m test/share/map2.txt -n <N> -r <R> --algorithm lazysp r2 -s 252 115 -g 350 350
   ```
   Report the path length, planning time, and number of edges evaluated for all three planners in a table. Explain any variation and trade-offs you observe. In particular: which planner evaluates the fewest edges, and which is fastest? Why might they differ, and when would LazySP be the best choice? (Also compare Dijkstra and A* if you implemented both.)

   > **NOTE:** LazySP re-runs a full graph search every time it finds an edge in collision, so it can be slow on large roadmaps (with 600 vertices it may take a minute or more). If it takes too long, use a smaller number of vertices for all three planners.
2. **Shortcutting Evaluation:** Compare the time spent on planning and shortcutting on `map1.txt`. Describe qualitatively how the paths differ before and after shortcutting.

Please also include the following images in the `planning/writeup/` directory:

3. `halton_map1_lazy_prm_path.png`: The plot of the Lazy PRM path on `map1.txt` generated in Q1.
4. `halton_map1_shortcut_path.png`: The plot of the Lazy PRM path with shortcut postprocessing on `map1.txt` generated in Q2.
5. `halton_map1_lazysp_path.png`: The plot of the LazySP path on `map1.txt` generated in Q3.

---

## 🔥 Grading Breakdown

**Total Lab Points:** 100

*   **Q1:** 25 points if `python3 $(rospack find planning)/test/dijkstra.py` passes the `test_lazy_prm` test suite.
*   **Q2:** 20 points if `python3 $(rospack find planning)/test/shortcut.py` passes.
*   **Q3:** 25 points if `python3 $(rospack find planning)/test/lazysp.py` passes.
*   **Write-Up (Questions & Analysis):** 15 points (10 points for the planner comparison, 5 points for the shortcutting evaluation).
*   **Write-Up (Plots):** 15 points (5 points per plot: `halton_map1_lazy_prm_path.png`, `halton_map1_shortcut_path.png`, and `halton_map1_lazysp_path.png`).

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
