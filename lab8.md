---
layout: page
title: Lab 8
description: >-
    Steering and Graph Search.
mathjax: true
nav_exclude: true
---

# Lab 8
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

Welcome to Lab 8! In Lab 7, we sampled configurations and implemented collision checking for states. In this lab, we will complete our roadmap graph construction by validating and adding weighted edges, and then implement **Dijkstra's algorithm** (or **A\***) to compute shortest paths across our roadmap graph, completing our **Probabilistic Roadmap (PRM)** planner.

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

## Q1: Roadmap Edge Validation

In Lab 7, we sampled collision-free vertices for our roadmap. Now, we need to connect those vertices with valid edges to construct the graph.

**Requirement:** Implement `Roadmap.check_weighted_edges_validity` in `src/planning/roadmap.py`. This method is called by the `Roadmap` constructor to prevent edges in collision from being added to the graph.

#### Testing Edge Validity
You can verify your implementation on the provided test suite by running:
```bash
python3 $(rospack find planning)/test/roadmap.py
```
*(or `python3 test/roadmap.py` from within the `planning` directory)*

The following command samples 25 collision-free vertices and collision-checks edges that have length less than the connection radius of 3. Note that you must be in the `planning` package directory (run `roscd planning`) to run the script. Your plot should match the reference plot below. Save your generated plot as `halton_map1_edges.png`.

```bash
python3 scripts/roadmap -m test/share/map1.txt -n 25 -r 3.0 --show-edges
```

![Figure 1: Halton graph on map1.txt with 25 vertices and connection radius of 3.]({{ site.baseurl }}/assets/lab8-assets/r2_map1_edges.png)

---

## Q2: Searching the Roadmap with Dijkstra

Dijkstra's algorithm expands nodes in order of their cost-to-come $g(n)$, finding the shortest path from the start to the goal across the roadmap graph. (You are also welcome to implement **A\*** search by adding an admissible heuristic: $f(n) = g(n) + h(n)$! Either Dijkstra or A* is fully supported and accepted.)

**Requirement:** Implement graph search in `src/planning/search.py`. For each neighbor, compute the cost-to-come via the current node (cost-to-come for the current node, plus the length of the edge to the neighbor). Insert a `QueueEntry` with this information into the priority queue. (If implementing A*, add the heuristic to compute the f-value.) Then, implement `extract_path` to follow the parent pointers from the goal to the start; this function is called after the goal has been expanded to finally return the sequence of nodes.

#### Testing Graph Search
You can verify your implementation on the provided test suites by running:
```bash
python3 $(rospack find planning)/test/dijkstra.py
```
*(or `python3 test/dijkstra.py` from within the `planning` directory)*

Your implementation should pass the `test_dijkstra_r2` and `test_dijkstra_se2` suites.

You should now be able to run a PRM planner from the start to the goal! The following command specifies that it is an `R2Problem`, planning from the start of (1, 1) to the goal of (7, 8). Note that you must be in the `planning` package directory (run `roscd planning`) to run the script. Your plot should match the reference plot below. Save your generated plot as `halton_map1_prm_path.png`.

```bash
python3 scripts/run_search -m test/share/map1.txt -n 25 -r 3.0 --show-edges r2 -s 1 1 -g 7 8
```

![Figure 2: PRM path in blue on map1.txt, from (1, 1) in green to (7, 8) in red.]({{ site.baseurl }}/assets/lab8-assets/r2_map1_prm_path.png)

---

## 📝 Write-up

Create a **new file** `planning/writeup/lab8.md`. **List the names and Northeastern emails** of students in your lab group at the top, and answer the following questions:

1. **Shortest Path on `map2.txt`:** Compute the shortest path on `map2.txt` between (252, 115) and (350, 350) with the following command:
   ```bash
   python3 scripts/run_search -m test/share/map2.txt -n 600 -r 100 r2 -s 252 115 -g 350 350
   ```
   Save this plot as `map2_prm_path.png` in `planning/writeup/` and include it in your writeup.
2. **Varying Connection Radius:** Holding the number of vertices constant at 600, decrease the connection radius from 100 until the planner fails to find a solution on `map2.txt`. Now, double the connection radius to 200. For each radius, report the radius, path length, and planning time in a table. Explain any variation you observe.
3. **Varying Number of Vertices:** Holding the connection radius constant at 100, decrease the number of vertices from 600 until the planner fails to find a solution on `map2.txt`. Now, double the number of vertices to 1200. For each number of vertices, report the number of vertices, path length, and planning time in a table. Explain any variation you observe.

Please also include the following images in the `planning/writeup/` directory:

4. `halton_map1_edges.png`: The plot of the roadmap with 25 vertices and connection radius 3.0 generated in Q1.
5. `halton_map1_prm_path.png`: The plot of the PRM path from (1, 1) to (7, 8) generated in Q2.
6. `map2_prm_path.png`: The plot of the shortest path on `map2.txt` between (252, 115) and (350, 350).

---

## 🔥 Grading Breakdown

**Total Lab Points:** 100

*   **Q1:** 25 points if `python3 $(rospack find planning)/test/roadmap.py` passes.
    *   That's 5 points per test (5 tests, all or nothing per test).
*   **Q2:** 30 points if `python3 $(rospack find planning)/test/dijkstra.py` passes the search test suites.
    *   15 points for `test_dijkstra_r2` and 15 points for `test_dijkstra_se2`.
*   **Write-Up (Questions & Analysis):** 15 points (5 points per question/experiment analysis).
*   **Write-Up (Plots):** 30 points (10 points per plot: `halton_map1_edges.png`, `halton_map1_prm_path.png`, and `map2_prm_path.png`).

---

## 🚀 Submission

When you are finished with the lab, **make sure** to commit all your changes and push them to your private GitHub repository.

Use the Git tag `submit-lab8` to signal completion:

```bash
$ cd ~/mushr_ws/src/mushr478
$ git tag submit-lab8
$ git push origin main submit-lab8
```

### Re-submitting

If you need to make changes after you have already submitted (and before the deadline), you can re-submit by deleting the old tag and creating a new one:

```bash
# Delete the tag locally
$ git tag -d submit-lab8
# Delete the tag on GitHub
$ git push origin :refs/tags/submit-lab8
# Create and push the new tag
$ git tag submit-lab8
$ git push origin submit-lab8
```

**Congratulations!** You've completed Lab 8! 🏎️💨
