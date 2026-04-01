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

Welcome to Lab 0! Here, we'll set up the class VM for the rest of the semester and learn a bit about ROS. We'll also get familiar with the GitHub Classroom system you'll use to submit your assignments in CS 7680.

## Virtual Machine Setup

Note that the real requirement for this course is that you have access to a personal laptop capable of running the class Virtual Machine (VM). Most modern laptops should be capable of this. Furthermore, you are **required** to bring this personal laptop with you to every class: we will be working on labs in-person during almost every session.

Of course, that means the first thing we'll do in Lab 1 is **set up** this fabled VM!

### 0. Requirements

Before you begin, ensure your disk has at least **14GB** of free space.

### Download VM Image

Download the appropriate virtual machine image for your hardware. Use the Google Drive folder [here](https://drive.google.com/drive/folders/1NS3p4AtmgWsnuty7PZSxREThAKnUy_sC?usp=drive_link).

*   **For Windows / Linux / Intel Macs:** You will need both the `.vmdk` and the `.ovf` files.
*   **For Apple Silicon (M1/M2/M3):** Download the `.utm` virtual machine image.

### Install Hypervisor

Depending on your operating system, you will need to install a "hypervisor" to run the VM:

*   **Windows / Linux:** Use **VMware Workstation Pro**. You will need to register for a free account on the Broadcom Support portal to download it [here](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion). Search in the "Free Downloads" area for **VMware Workstation Pro**. Once installed, import the `.ovf` file.
*   **Intel Macs (pre-Apple Silicon):** Use **VMware Fusion**. You will need to register for a free account on the Broadcom Support portal to download it [here](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion). Search in the "Free Downloads" area for **VMware Fusion**. Once installed, import the `.ovf` file.
*   **Apple Silicon Macs:** Use the [**UTM hypervisor**](https://mac.getutm.app/). Unzip the downloaded VM file and double-click the `.utm` image to open it in UTM.

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

### VM Troubleshooting

#### Not enough physical memory
If you see an error stating "Not enough physical memory is available," navigate to **Virtual Machine > Settings > Processors & Memory** and reduce the allocated RAM from the default (typically 4096MB).

#### 3D Graphics Acceleration
It is highly recommended to enable **3D Graphics Acceleration** in the VM settings to improve the frame rate in RViz and other simulation tools. This usually requires a full VM shutdown before the setting can be changed.
