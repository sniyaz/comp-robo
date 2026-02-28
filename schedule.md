---
layout: page
title: Office Hours + Schedule
description: The weekly event schedule.
---

## Office Hours Instructions

**Teams link** for office hours: click [here]().

### How to Queue

To queue for office hours, please follow these steps:

1.  **Find the Thread:** Go to Piazza and find the **pinned post** for the current week's office hours (e.g., "Week 1 Office Hours Queue").
2.  **Post Your Question:** Add a comment to that thread with a short description of what you're working on and the specific issue you're facing. For example:
    *   `Lab 1: Compilation error when using the robot's sensor API.`
3.  **Wait for Assistance:** I will help students in the order they post. When it is your turn, I'll call you into a breakout room on Teams.
4.  **Completion:** Once I've finished helping you, I will reply to your comment on Piazza with "Done" so you know you've been checked off the queue.

**Don't worry** if you don't see the instructor in the main Teams call: they are helping another student in a breakout room.

## Weekly Schedule

(Make sure to **scroll** to view the entire page!)

{% for schedule in site.schedules %}
{{ schedule }}
{% endfor %}
