---
layout: page
title: Virtual Machine Setup
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