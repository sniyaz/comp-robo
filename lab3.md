---
layout: page
title: Lab 3
description: >-
    Advanced Indexing with NumPy and Kinematic Car Motion Modeling.
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

Vectorization isn’t just limited to mathematical operations. NumPy supports a variety of different forms of indexing, but integer array indexing and Boolean array indexing are especially important for this course.

### Q1: Integer Array Indexing

Complete the `extract_fibonacci_rows` function using **integer array indexing** in `src/introduction/indexing.py`.

### Q2: Boolean Array Indexing

Complete the `increment_rows_with_odd_first_element` function using **Boolean array indexing** in `src/introduction/indexing.py`.

#### Testing Indexing
After completing both Q1 and Q2, expect your code to pass all the test cases when running:
```bash
python3 test/indexing.py
```

---

## Motion Model

In this section, you will implement the kinematic car motion model, which is a key component of the particle filter algorithm.

### State Estimation Definitions
Assume that the MuSHR car drives on a flat plane and localizes within a known map $m$.

*   **States** are defined as $\mathbf{x}_t = (x_t, y_t, \theta_t)$ where $(x_t, y_t)$ is the car’s 2D position and $\theta_t$ is its heading direction, with respect to the map’s reference frame.
*   **Controls** are defined as $\mathbf{u}_t = (v_t, \delta_t)$ where $v_t$ is the speed and $\delta_t$ is the steering angle, with respect to the car’s rear-axle reference frame.
*   The **motion model** specifies a probability distribution $P(\mathbf{x}_t | \mathbf{x}_{t-1}, \mathbf{u}_t)$, i.e. the probability of reaching a state $\mathbf{x}_t$ given that control $\mathbf{u}_t$ is applied from state $\mathbf{x}_{t-1}$.

### Q3: Kinematic Car Equations (Deterministic)

In this question, you will implement the kinematic car motion model. Let’s first review the kinematic model of the car and annotate the relevant lengths and angles. First, let’s assume that there is no control and that the velocity was stable and the steering angle is zero. We can then write out the change in states:
$$\dot{x} = v \cos \theta$$
$$\dot{y} = v \sin \theta$$
$$\dot{\theta} = \omega$$
where $\omega$ is the angular velocity from the center of rotation to the center of the rear axle.

By the definition of angular velocity:
$$\omega = \frac{v}{R} = \frac{v}{L} \tan \delta$$

Formally, the changes in states are:
$$\frac{\partial x}{\partial t} = v \cos \theta$$
$$\frac{\partial y}{\partial t} = v \sin \theta$$
$$\frac{\partial \theta}{\partial t} = \frac{v}{L} \tan \delta$$

After applying control $\mathbf{u}_t$, integrate over the time step:
$$\theta_{t+1} = \theta_{t} + \frac{v}{L} \tan \delta \Delta t$$
$$x_{t+1} = x_{t} + \frac{L}{\tan \delta} [ \sin \theta_{t+1} - \sin \theta_{t} ]$$
$$y_{t+1} = y_{t} + \frac{L}{\tan \delta} [ -\cos \theta_{t+1} + \cos \theta_{t} ]$$

**Requirement:** Implement the kinematic car equations in the `KinematicCarMotionModel.compute_changes` method (`src/localization/motion_model.py`). This method is deterministic.

**Note on Steering Angle 0:**
If the steering angle is 0, the update rule simplifies to:
$$\theta_{t+1} = \theta_{t}$$
$$x_{t+1} = x_{t} + v \cos \theta \Delta t$$
$$y_{t+1} = y_{t} + v \sin \theta \Delta t$$
In your implementation, you should treat any steering angle where $|\delta|$ is less than `delta_threshold` as a zero steering angle.

### Q4: Noisy Motion Model

In practice, our controls and models are not perfect. We account for this by adding noise:

1.  **Sample noisy controls** $\hat{\mathbf{u}}_t = (\hat{v}_t, \hat{\delta}_t)$ where $\hat{v}_t \sim \mathcal{N}(v_t, \sigma_v^2)$ and $\hat{\delta}_t \sim \mathcal{N}(\delta_t, \sigma_\delta^2)$.
2.  **Integrate kinematic car equations** with noisy controls using your `compute_changes` method.
3.  **Add model noise** to the output $\Delta \mathbf{x}_t \sim \mathcal{N}(\Delta \hat{\mathbf{x}}_t, \mathrm{diag}(\sigma_x^2, \sigma_y^2, \sigma_\theta^2))$.

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
