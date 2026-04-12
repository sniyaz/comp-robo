---
layout: page
title: Lab 1
description: >-
    Instructions for setting up the CS 7680 Virtual Machine.
nav_exclude: true
---

# Lab 1
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Welcome to Lab 1! Here, we'll set up the class VM for the rest of the semester and learn a bit about ROS. We'll also get familiar with the GitHub process you'll use to submit your assignments in CS 7680.

## GitHub Account and Class Repo

You are required to have a GitHub account for this course, as we will use it for all lab assignments. You should have created both a GitHub account and your copy of the class repo by following the [GitHub Linking]({{ site.baseurl }}/account_linking#github-account) directions before Lab today.

If you haven't already done this, **please do this immediately**. You will need a GitHub account and a copy of the class repo to set up the class VM next!

## Virtual Machine Setup

Note that the only **real** equipment requirement for this course is that you have access to a personal laptop capable of running the class Virtual Machine (VM). Most modern laptops should be capable of this. Furthermore, you are **required** to bring this personal laptop with you to every class: we will be working on labs in-person during almost every session.

Of course, that means the first thing we'll do in Lab 1 is **set up** this fabled VM!

### Requirements

Before you begin, ensure your disk has at least **14GB** of free space.

### Download VM Image

Download the appropriate virtual machine image for your hardware. Use the Google Drive folder [here](https://drive.google.com/drive/folders/1NS3p4AtmgWsnuty7PZSxREThAKnUy_sC?usp=drive_link).

*   **For Windows / Linux / Intel Macs:** You will need both the `.vmdk` and the `.ovf` files.
*   **For Apple Silicon (M1/M2/M3):** Download the `.utm` virtual machine image.

### Install Hypervisor

Depending on your operating system, you will need to install a "hypervisor" to run the VM:

*   **Windows / Linux:** Use **VMware Workstation Pro**. You will need to register for a free account on the Broadcom Support portal to download it [here](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion). Search in the "Free Downloads" area for VMware Workstation Pro. Once installed, import the `.ovf` file.
*   **Intel Macs (pre-Apple Silicon):** Use **VMware Fusion**. You will need to register for a free account on the Broadcom Support portal to download it [here](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion). Search in the "Free Downloads" area for VMware Fusion. Once installed, import the `.ovf` file.
*   **Apple Silicon Macs:** Use the [**UTM hypervisor**](https://mac.getutm.app/). **Unzip** the downloaded VM file and double-click the `.utm` image to open it in UTM.

### Login Credentials

Once the VM is booted, use the following credentials to log in. Note that if you are using Apple Silicon/UTM, you likely won't be asked for a password at all.

*   **Username:** `robotics`
*   **Password:** `robotics` 
    *   *Note: For Apple Silicon/UTM, if prompted for a different password, it may be `prl_robot`.*

### Git & GitHub Setup

Once inside the VM, you should configure Git to work with your GitHub account:

1.  Open a terminal in the VM.
2.  Configure your name and email:
    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "your_email@example.com"
    ```
3.  Generate an SSH key pair:
    ```bash
    ssh-keygen -t ed25519
    ```
    Press enter to accept defaults.
4.  Add the public key to your [GitHub account](https://github.com/settings/keys). You can view your public key by running:
    ```bash
    cat ~/.ssh/id_ed25519.pub
    ```

## VM Troubleshooting

#### Not enough physical memory
If you see an error stating "Not enough physical memory is available," navigate to **Virtual Machine > Settings > Processors & Memory** and reduce the allocated RAM from the default (typically 4096MB).

#### 3D Graphics Acceleration
It is highly recommended to enable **3D Graphics Acceleration** in the VM settings to improve the frame rate in RViz and other simulation tools. This usually requires a full VM shutdown before the setting can be changed.

## Getting Started with ROS

The Robot Operating System (ROS) is a robotics middleware framework (not actually an operating system) that is commonly used in robotics platforms. ROS is used as an interprocess communications provider. By implementing functionalities in many separate “nodes” (each their own process) and relying on ROS mechanisms to communicate, we can seamlessly integrate custom software with existing, off-the-shelf ROS components. **Gaining a grasp of fundamental ROS concepts will help you make the most of the many robots which use ROS.**

⚠️ **NOTE:** To be clear, the point of this class is to learn important algorithms in Computational Robotics, not to learn ROS: that said, hopefully learning it is a cool side benefit of this class!

ROS code is organized into packages, which can contain executables, libraries and other information like message formats, geometric meshes, and maps. Some of these executables may be nodes, processes that register themselves with a ROS Master and communicate with other nodes. A package can have multiple nodes. Each package has a `package.xml` file which specifies dependencies, as well as a `CMakeLists.txt` makefile. Most ROS code is written in Python or C++.

ROS development is done in a workspace, a specially structured file tree that enables you to work on many separate packages and build them all at once. For this course, we’ve crafted a repository `~/dependencies_ws` that combines all of the key packages you’ll need in order to run the simulator alongside starter code and packages for each assignment. **This means you won’t have to create any packages yourself.**

ROS workspaces are designed to be layered, which allows you to build sets of packages separately and to override packages selectively. In fact, ROS itself is just a bunch of packages living in a workspace in `/opt/ros/noetic`, and for the most part every ROS workspace you’ll encounter will overlay this root workspace. We won’t be using the full power of workspace overlays, but we will be using two overlaid workspaces (in addition to the root installation workspace) to better manage the packages you’ll need to work with.

### Workspaces

First, let’s take a look at our workspaces.

```bash
$ ls ~/dependencies_ws/
build devel logs src
$ ls ~/mushr_ws/
build devel logs src  # You may be missing directories here, but we will go through how to build this workspace!
```

**Terminal Tip: ls**

The `ls` command lists the contents of a directory. In a path like `~/dependencies_ws`, the `~` is shorthand for your home directory (`/home/robotics` for this virtual machine).

You can use tab completion rather than typing out the entire command. Try typing `ls ~/dep<TAB>`. If there are multiple completions, press the tab key twice to see all the options.

As the names indicate, one workspace is dedicated to the dependencies you’ll need for your projects. They’re set up for you, and you shouldn’t need to modify them. The `mushr_ws` is for your project code.

The key part of the workspace is the `src` folder which contains packages. The others are automatically generated at build time. 

### Building the Dependencies Workspace

Let’s build the dependencies workspace first.

⚠️ **NOTE:** The `dependencies_ws` must be built first (before `mushr_ws`) since it contains important packages that let our labs assignments actually run. **Never** modify anything inside the `dependencies_ws` folder.

```bash
$ source /opt/ros/noetic/setup.bash
$ cd ~/dependencies_ws/src/
$ rosdep install --from-paths . --ignore-src -y -r
$ cd ~/dependencies_ws
$ catkin clean  # remove old build files
$ catkin build
```

You’ll see a lot of colorful output, ending with a message saying how many packages were built. All `catkin build` did was compile a bunch of executables and dump build artifacts into the `build` and `devel` folders. However, your shell doesn’t know how to find and run them because they aren’t on the PATH. You will need to activate the workspace by running a special script that configures your shell’s environment variables to know about all of the build output. To activate a workspace, you run:

```bash
$ source ~/dependencies_ws/devel/setup.bash
```

### Building the Project Workspace

**Great!** We've built the dependencies workspace. Now we’re going to build and activate the main project workspace (`mushr_ws`).

First, clone your class repository into the `~/mushr_ws/src` folder:

```bash
$ cd ~/mushr_ws/src/
$ git clone git@github.com:<your-github-handle>/mushr478.git
```

**With your class repo cloned**, we can now build and activate the main project workspace (`mushr_ws`):

```bash
$ # Activate root workspace
$ source /opt/ros/noetic/setup.bash
$ # Activate dependencies underlay (built previously)
$ source ~/dependencies_ws/devel/setup.bash
$ # Build and activate projects workspace
$ cd ~/mushr_ws/
$ catkin build
$ source ~/mushr_ws/devel/setup.bash
```

Note that we sourced the dependencies workspace before we built the project workspace `mushr_ws`. This way, the packages inside `mushr_ws` have access to the packages built in the `dependencies_ws` underlay.

You can verify that your workspaces are overlaid as expected by checking the output of `catkin config` from within `mushr_ws`. The second line should read `Extending: /home/robotics/dependencies_ws/devel:/opt/ros/noetic`.

## The Simulator

This section will show you how to run the MuSHR simulator and visualize what is going on. Simulators are important because they allow you to test things quickly and without the risk of damaging a robot, the environment, or people. Our simulator uses a kinematic car model (with some noise sprinkled in), meaning it simply implements and integrates some equations that describe the ideal motion of a mechanical system. It will not perfectly reflect the real world dynamics of the car. Regardless, it is still useful for a lot of things.

We’ll run the simulator using a launch file. These are XML files that describe how different software components should be started. These files allow us to start a large number of ROS nodes with only few commands. To launch our simulator, open a terminal and run:

```bash
$ cd ~/mushr_ws/
$ source ~/mushr_ws/devel/setup.bash
$ roslaunch cse478 teleop.launch
```

Moving forward, we won’t always spell out that you’ll need to activate the `mushr_ws` workspace to do something. But remember that you can’t interact with ROS without having activated a workspace in your terminal. If you try to `roslaunch` and your terminal says “Command ‘roslaunch’ not found”, you probably forgot to activate the workspace!

Like many ROS terminal commands, the first argument is the name of a package, and the second is the name of something inside that package (in this case, a launch file).

A small gray window should appear. The simulation is running, you just can’t see it yet!

Let’s try to convince ourselves of this. Open another terminal and run `rosnode list` to see all of the nodes running. Each of those nodes is a separate process. We can see the communication between nodes by examining the topics they are publishing messages to and subscribing to messages from. Messages are packets of information in a specific format that can be routed around a ROS system. Running `rostopic list` will display the names of all the open topics. We can actually see the simulation in action if we look at some of the messages coming from the simulated hardware. Let’s use the `rostopic` helper to view these messages:

```bash
$ rostopic echo /car/odom
```

Messages will start streaming into your terminal. Now focus on the gray teleop window and use the WASD keys to send commands to the vehicle. W and A move forward and backward, S and D turn the steering wheel left and right. Notice how the numbers are changing in response to your commands?

To stop the simulation, press `<Ctrl-C>` in the `teleop.launch` terminal window.

### Visualizing the Simulator

Start the simulation again. To visualize the simulation, open up a separate terminal, run `source ~/mushr_ws/devel/setup.bash` to activate your workspace, then run:

```bash
$ rosrun rviz rviz -d ~/mushr_ws/src/mushr478/cse478/config/default.rviz
```

If you see error messages in your terminal like “Could not load model…”, remember to activate your workspace by running `source ~/mushr_ws/devel/setup.bash`. 

This will launch RViz with a configuration that has all the right topics visualized. You will see something like this:

![rviz_image]({{ site.baseurl }}/assets/lab1-assets/rviz_image.png)

To drive the car, click on the small gray window (i.e. the tele-op window you opened previously in the first terminal labeled `tk`) and use the WASD keys.

In RViz, you will notice multiple panels. The right panel is the views panel. The “Target Frame” option tells the camera what to focus on. If you don’t want it to track the car, you can change it from “base_link” to “map”. The center panel is the view panel; you can zoom, pan, and rotate using the mouse and mouse wheel.

The top panel has a few tools, like Measure and Publish Point. Publish Point is particularly useful for relocating the MuSHR car. Just click on it, then click on the map; the simulator listens to the message RViz sends and sets the car position to match.

Finally, the left panel shows all of the ROS topics that RViz is subscribed to. For example, if you toggle off the checkbox for the Map topic, you will no longer see the map visualization (although it’s still being used by the simulator). You can add additional topics by clicking the “Add” button then “By topic”. 

Remember, RViz is a visualizer, not a simulator. You can close RViz, but just like how the world continues when you close your eyes, the simulator will chug ahead. So far, we’ve been using RViz to visualize the ROS data being emitted by the simulator. In other situations, the information could come from the software running on a physical robot.

### Changing Maps

We will now change the map! Let’s take a look at the `teleop.launch` file we’ve been using. You can find it inside `cse478/launch`. A fast way to move to the package is `roscd`:

```bash
$ roscd cse478
$ pwd
/home/robotics/mushr_ws/src/mushr478/cse478
```

Now open `teleop.launch` in `launch/` using your favorite editor (e.g. `nano` or `gedit`):

```bash
$ gedit launch/teleop.launch
```

You can see a comment that explains how to configure the map:

```xml
<!-- We take a map path either as an environment variable or an argument.
     The argument takes precedence. -->
```

This means that we can either pass the map as argument each time like this,

```bash
$ roslaunch cse478 teleop.launch map:='$(find cse478)/maps/cse2_2.yaml'
```

or by putting the same string into a variable named MAP in our terminal environment:

```bash
$ export MAP='$(find cse478)/maps/cse2_2.yaml'
$ roslaunch cse478 teleop.launch
```

Using the environment variable has the benefit of being somewhat persistent; if you set the variable in your `~/.bashrc` file, you’ll launch using the chosen map every time without having to retype the command or flip back through your terminal history.

You can use whichever way you like, but note that single quotes (') are important. The terminal treats double-quoted strings differently than single-quoted strings, and interchanging them may cause errors.

> **Task:** Use whichever method you prefer to set the map to `cse2_2.yaml` from the `cse478` package. Then, save a screenshot of RViz showing the new map. (Note that the car will start in the “walls” and therefore won’t be able to drive. Use the Publish Point tool in the top panel to place the car in an open hallway before driving with WASD.)

## Running Tests in CS 7680

Each package contains a test directory with unit and integration tests built with the `rosunit` framework. The `catkin test` command provides a summary view of all test suites.

```bash
$ cd ~/mushr_ws
$ catkin test               # Run all tests
$ catkin test introduction  # Run tests for the introduction package
```

To run individual test suites, the command will differ by the type of test suite.

For tests that use ROS (to start a node, or publish/subscribe to topics from the code being tested), use `rostest` to run the corresponding test launch file (which will end in `.test`).

```bash
$ rostest introduction pose_listener.test --text
```

For pure Python unit tests (which don’t use ROS), simply run the file:

```bash
$ python3 $(rospack find introduction)/test/fibonacci.py
```

> **Missing debug prints when using rostest?**
> Simple python prints won’t work with `rostest`.
> Make sure you are using `rospy.loginfo` with `--text` flag to print information as in the above example.
> See ROS Logging and ROS test for more information.

## 💻 Coding Question 1: Fibonacci Publisher

Now that you have the simulator running and understand how to run tests, it's time to write some code! You will be working in the `introduction` folder inside your `mushr478` repository. Note that there is only one coding question on this lab- but usually we will have more!
### Your First Publisher Node!

As you may recall from above, publishers and subscribers are how ROS manages interprocess communication. Publishers send out messages to a topic, while subscribers choose a topic to receive messages from. 

We will be creating a simple ROS node that calculates the nth Fibonacci number. It will read `n` from a ROS parameter and publish the desired Fibonacci number to a ROS topic. The 0th Fibonacci number is 0, 1st is 1, 10th is 55, etc.

#### Sub-Tasks and Implementation Requirements

*   **Q1.1:** Implement the Fibonacci calculation in `src/introduction/fibonacci.py`. 
    *   **Requirement:** You will need an **O(n) implementation** to pass the larger tests.
*   **Q1.2:** Complete the ROS interface code in the provided skeleton at `scripts/fibonacci`. 
    *   **Requirement:** Follow the instructions provided inline within the script to complete the code.
*   **Q1.3:** Write a launch file for the node at `launch/fibonacci.launch`. 
    *   **Requirement:** Since the calculator requires a ROS parameter, you must pass it to your node in the launch file. Follow the inline instructions in the provided skeleton.

#### Testing Your Code

**Automated Testing:**
To verify your implementation for each sub-task, run the following commands from within your `mushr_ws` (make sure it is sourced!):

*   **For Q1.1 (Python Logic):**
    ```bash
    python3 $(rospack find introduction)/test/fibonacci.py
    ```
*   **For Q1.2 (ROS Node):**
    ```bash
    rostest introduction fibonacci_small.test
    ```
*   **For Q1.3 (Launch File):**
    ```bash
    rostest introduction fibonacci_launch.test
    ```

**Manual Verification:**
To see the output directly without the automated tests, follow these steps across three terminals. Note that you must first start `roscore`: this is the central coordinator for any ROS system. It provides the ROS Master, which allows nodes to find and communicate with each other.

1.  **Terminal 1 (Start ROS Master):**
    ```bash
    roscore
    ```
2.  **Terminal 2 (Listen to the output topic):**
    ```bash
    rostopic echo /introduction/fib_output
    ```
3.  **Terminal 3 (Launch the node):**
    ```bash
    roslaunch introduction fibonacci.launch
    # To test with a specific index (e.g., 20):
    roslaunch introduction fibonacci.launch index:=20
    ```

## 📝 Write-up

Our labs will also have a write-up portion (usually)! Create a **new file** `introduction/writeup/lab1.md`, and answer the following questions:

1.  Explain ROS nodes, topics, publishers, and subscribers in your own words.
2.  Explain what a ROS launch file is and why it's useful.

Please also include the following images in the `introduction/writeup/` directory

1. The screenshot of RViz showing the `cse2_2` map (name this `map.png`, `map.jpg`, etc).

## 🔥 Grading Breakdown

TODO!

## 🚀 Submission

When you are finished with the lab, commit all your changes and push them to your private GitHub repository. You **MUST** use a specific Git tag to signal completion so that my grading scripts can find your work. For Lab 1, this tag is exactly `submit-lab1`.

```bash
$ cd ~/mushr_ws/src/mushr478
$ git add *
$ git commit -m "Complete Lab 1" (or any fun message you like!)
$ git tag submit-lab1
$ git push origin main submit-lab1
```

### Re-submitting

If you need to make changes after you have already submitted (and before the deadline), don't panic! You can re-submit by deleting the old tag and creating a new one:

```bash
# Delete the tag locally
$ git tag -d submit-lab1
# Delete the tag on GitHub
$ git push origin :refs/tags/submit-lab1
# Create and push the new tag
$ git tag submit-lab1
$ git push origin submit-lab1
```

**Congratulations!** You've completed Lab 1 and are ready for the rest of the semester. 🏎️💨