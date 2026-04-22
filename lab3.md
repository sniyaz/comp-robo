---
layout: page
title: Lab 3
description: >-
    Advanced Indexing with NumPy and Kinematic Car Motion Modeling.
mathjax: true
nav_exclude: true
---

# Lab 3
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Welcome to Lab 3! In this lab, we will dive deeper into NumPy's powerful indexing capabilities and implement a kinematic motion model for our MuSHR car. These concepts are fundamental for state estimation and (in later labs) particle filtering with HMMs.

## NumPy: Advanced Indexing

Vectorization isn’t just limited to mathematical operations. NumPy supports a variety of different forms of [indexing](https://numpy.org/doc/stable/user/basics.indexing.html), but [integer array indexing](https://numpy.org/doc/stable/user/basics.indexing.html#integer-array-indexing) and [Boolean array indexing](https://numpy.org/doc/stable/user/basics.indexing.html#boolean-array-indexing) are especially important for this course.

### Q1: Integer Array Indexing

Complete the `extract_fibonacci_rows` function using **integer array indexing** in `src/introduction/indexing.py`.

Consider the example from the spec:

```python
>>> data = np.array([[ 4,  7,  8],
...                  [14, 17, 18],
...                  [24, 27, 28],
...                  [34, 37, 38]])
>>> extract_fibonacci_rows(data)
array([[ 4,  7,  8],   # Row 0
       [14, 17, 18],   # Row 1
       [14, 17, 18],   # Row 1
       [24, 27, 28],   # Row 2
       [34, 37, 38]])  # Row 3
```

In this example, we are extracting rows based on the Fibonacci sequence ($F_0=0, F_1=1, F_2=1, F_3=2, F_4=3, \dots$). Specifically, for a given input array, you should return a new array containing the rows at indices $[0, 1, 1, 2, 3, \dots, F_n]$ where $F_n$ is the largest Fibonacci number less than the number of rows in the input.

**Hint 1:** You'll need to compute the Fibonacci sequence up to the number of rows in your input. You can use the `fibonacci` function you implemented in Lab 1 (you'll need to import it!). 

**Hint 2:** You also will probably want to import `numpy` as `np`.

**Hint 3:** Integer array indexing allows you to pass a list or array of integers as an index to a NumPy array. For example, `data[[0, 1, 1, 2]]` will return a new array with the 0th, 1st, 1st, and 2nd rows of `data`.

**Hint 4:** If you need to, you can use `np.array()` to convert a Python list into a numpy array!

### Q2: Boolean Array Indexing

Complete the `increment_rows_with_odd_first_element` function using **Boolean array indexing** in `src/introduction/indexing.py`.

Consider the following example:

```python
>>> data = np.array([[ 1,  2,  3],
...                  [ 4,  5,  6],
...                  [ 7,  8,  9]])
>>> increment_rows_with_odd_first_element(data)
array([[ 2,  3,  4],   # Row 0 (1 is odd, so incremented)
       [ 4,  5,  6],   # Row 1 (4 is even, so unchanged)
       [ 8,  9, 10]])  # Row 2 (7 is odd, so incremented)
```

**Hint 1:** Boolean array indexing involves creating a "mask" (an array of True/False values) and using that mask to select elements. For example, `data[mask] += 1` will only increment the elements where `mask` is True.

**Hint 2:** You can create a mask by applying a condition to an entire column. For example, `data[:, 0] % 2 == 1` will return a Boolean array that is True for every row where the first element is odd.

**Hint 3:** You can use this mask to index into the original array and perform an in-place addition: `data[mask] += 1`. This will modify the rows that satisfy your condition while leaving the others untouched.

**NOTE:** This function does not have a return value: your implementation should modify the data argument in-place.

#### Testing Indexing
After completing both Q1 and Q2, expect your code to pass all the test cases when running:
```bash
python3 $(rospack find introduction)/test/indexing.py
```

---

## Motion Model

In this section, you will implement the kinematic car motion model, which is a key component of our HMM State Estimator.

### State Estimation Definitions
Recall these definitions from lecture:

*   **States** are defined as $x_t = (x_t, y_t, \theta_t)$ where $(x_t, y_t)$ is the car’s 2D position and $\theta_t$ is its heading direction, with respect to the map’s reference frame.
*   **Controls** are defined as $u_t = (v_t, \delta_t)$ where $v_t$ is the speed and $\delta_t$ is the steering angle, with respect to the car’s rear-axle reference frame.

With this in mind, the **motion model** specifies a probability distribution:

$$P(x_t | x_{t-1}, u_t)$$

In other words,  the probability of reaching a state $x_{t}$ given that control $u_{t}$ is applied from state $x_{t-1}$.

### Q3: Equations of Motion (Deterministic)

In this question, you will implement the deterministic car motion model. Let’s first review the deterministic equations of motion that we covered today in lecture. Suppose that action $u_{t}$ is applied from state $x_{t-1}$. We can compute the new state as follows:

$$ \theta_{t+1} = \theta_{t} + \frac{v}{L} \tan \delta \Delta t $$

$$ x_{t+1} = x_{t} + \frac{L}{\tan \delta} [ \sin \theta_{t+1} - \sin \theta_{t} ] $$

$$ y_{t+1} = y_{t} + \frac{L}{\tan \delta} [ -\cos \theta_{t+1} + \cos \theta_{t} ] $$

**Requirement:** Implement the kinematic car equations in the `KinematicCarMotionModel.compute_changes` method (`src/localization/motion_model.py`). This method is deterministic.

**Note on Steering Angle 0:**
If the steering angle is 0, the update rule simplifies to:

$$ \theta_{t+1} = \theta_{t} $$

$$ x_{t+1} = x_{t} + v \cos \theta \Delta t $$

$$ y_{t+1} = y_{t} + v \sin \theta \Delta t $$

**Actually**, in your implementation you should treat any steering angle $\delta$ with an **absolute value** of less than `delta_threshold` as a zero steering angle!

### Q4: Noisy Motion Model

In practice, our controls and models are **not** perfect. We account for this by adding noise:

1.  **Sample noisy controls** $u'_t = (v'_t, \delta'_t)$ where $v'_t \sim \mathcal{N}(v_t, \sigma_v^2)$ and $\delta'_t \sim \mathcal{N}(\delta_t, \sigma_\delta^2)$.

2.  **Integrate kinematic car equations** with noisy controls using your `compute_changes` method.

3.  **Add model noise** to the output $\Delta x_t \sim \mathcal{N}(\Delta x'_t, \mathrm{diag}(\sigma_x^2, \sigma_y^2, \sigma_\theta^2))$.

**Requirement:** Implement this noisy motion model in the `KinematicCarMotionModel.apply_motion_model` method. Wrap the resulting $\theta$ component to the interval $(-\pi, \pi]$.

### Q5: Exploring and Tuning Parameters

The noise in your motion model is controlled by $\sigma_v, \sigma_\delta$ (action noise) and $\sigma_x, \sigma_y, \sigma_\theta$ (model noise). In this question, you will tune these parameters.

**Requirement:** Visualize samples using the following command:
```bash
rosrun localization make_motion_model_plot
```
Tune your parameters to match the staff solution plots (as shown in the course materials). Save three versions of the figure representing your progress and final results as `mm1.png`, `mm2.png`, and `mm3.png`.

---

## 📝 Write-up

Create a **new file** `localization/writeup/lab3.md`, and answer the following questions:

1.  Explain in your own words the difference between the particles in Figure 2 vs Figure 3 of the localization project (referring to the effects of different noise parameters).
2.  Document your tuning process for the motion model parameters. What did you observe when you increased or decreased the different noise values?

Please also include the following images from **Q5** in the `localization/writeup/` directory:

3.  The `mm1.png`, `mm2.png`, and `mm3.png` figures showing your tuned motion model.

---

## 🔥 Grading Breakdown

**Total Lab Points:** 50

*   **Q1 & Q2:** 10 points for passing `python3 test/indexing.py`.
*   **Q3 & Q4:** 25 points for correct implementation of the motion model.
*   **Write-Up & Tuning:** 15 points.

## 🚀 Submission

When you are finished with the lab, **make sure** to commit all your changes and push them to your private GitHub repository.

Use the Git tag `submit-lab3` to signal completion:

```bash
$ cd ~/mushr_ws/src/mushr478
$ git tag submit-lab3
$ git push origin main submit-lab3
```

**Congratulations!** You've completed Lab 3. 🏎️💨
