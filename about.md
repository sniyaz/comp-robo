---
layout: page
title: Syllabus
description: >-
    Course policies and information.
---

# Syllabus
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Basic Info

**Instructor:** Sherdil Niyaz

**Northeastern Email:** TODO AT northeastern DOT edu

**Final Exam Time:** TBD Dec 2026

**Lecture:** 5:00 - 6:30PM Wednesdays

**Lab:** 6:50 - 8:20PM Wednesdays

**Office Hours:** 10-11AM Saturdays, via Teams

## Course Description

This course explores the computational foundations of autonomous systems, organized around the **Sense, Plan, Act** paradigm: the fundamental cycle that enables robots to interact intelligently with the physical world. While robotics is inherently interdisciplinary, this class focuses **strictly** on the algorithmic and software challenges of autonomy, rather than mechanical design or hardware construction.

Our primary focus will be on the first two components of the autonomy stack:

*   **Sense (Perception):** We will investigate how robots interpret raw sensory information to build internal models of their environment.
*   **Plan (Motion Planning):** We will study the algorithms that allow robots to navigate complex environments while avoiding obstacles. This includes the study of configuration spaces, discrete search methods (e.g., A*, Dijkstra), and sampling-based motion planning (e.g., RRT, PRM).

As time permits, we will explore the **Act (Control)** component, focusing on the feedback loops required to execute planned trajectories accurately (e.g., iLQR).

By the end of the course, students will have a deep understanding of the mathematical and algorithmic tools required to transform a "dumb" machine into an autonomous agent capable of perceiving its surroundings and planning its own motions. 

**Again:** This is a software-centric course. No hardware assembly or electronics experience is expected or required.

## Prerequisites

Students should have a solid foundation in computer science and mathematics. This means the following:

- **CS 5100 (Foundations of AI)**
- **CS 5800 (Algorithms)**
- **Basic Linear Algebra** (familiarity with matrices, vectors, and coordinate transformations)

**Note:** While I will not kick you out without these prerequisites, they are **STRONGLY recommended**. If you choose to take the course without this background, you will need to learn these topics on the fly as we progress through the material, which will make the course significantly more challenging.

## Textbooks and Materials

There is no required textbook for this course. The material covered in the lecture slides will be sufficient for all assignments and exams. However, the following textbooks are excellent resources and serve as "helpful companions" if you'd like to dive deep into certain topics or would like additional practice:

- [**Probabilistic Robotics**](http://www.probabilistic-robotics.org/) by Sebastian Thrun, Wolfram Burgard, and Dieter Fox
- [**Underactuated Robotics**](https://underactuated.csail.mit.edu/) by Russ Tedrake
- [**Planning Algorithms**](https://lavalle.pl/planning/) by Steven M. LaValle

The real requirement for this course is that you have access to a personal laptop capable of running the class Virtual Machine (VM). Most modern laptops should be capable of this. Furthermore, you are **required** to bring this personal laptop with you to every class: we will be working on labs in-person during amost every session.

If you are having trouble getting access to a personal laptop and feel that it's impacting your learning in this course, send me an email: I will do my best to find a solution. My goal in this course is to give everybody an opportunity to succeed regardless of personal resources or other privileges. We will figure it out.

## Grading

Let's be honest: this is the part of the syllabus most students probably jumped to 😉. The breakdown of your grade in this course will be as follows:

**Labs (Weekly Programming Assignments)** → 45% \
**Midterm Exam** → 15% \
**Final Exam** → 40%

The percentages for each letter grades are exactly what you'd expect too, but in case you've forgotten:

**A >= 90%** \
**B >= 80%** \
**C >= 70%** \
**D >= 60%** \
**F < 60%**

**Note 1:** This means that points in different categories (Lab, Final, etc) are worth different percentages of your final grade: your percentage in each category gets reweighted according to the "recipe" above.

**Note 2:** My goal is that everybody should get 100% (or close to 100%) on the Labs.

**Do I Curve?** I reserve the right to make the bins for each grade _easier_, but I will never make them _harder_ (in case, for example, I mess up and give an exam that's way too hard).

## Late Assignments and Make-Up Exams

Programming assignments (Labs) can be submitted up to three days late with a 10% penalty of the **original** point amount each day. For example, suppose you get 9/10 on a lab assignment but submit it two days late (usually this means on a Friday). Your final score on the lab in this case would be 7/10. No submissions more than three days late will be graded unless you have an exception from me.

When it comes to exams, you usually need to let me know **72 hours** in advance if you need a make-up exam. I understand that life happens: if something unexpected happens that causes you to miss an exam, you need to let me know within **24 hours** for me to handle it somehow.

**Golden Rule:** If weird stuff happens and you aren't sure what to do, send me an email. I understand that we are all human beings with a lot going on, and I don't want this class to make your life miserable.

## Programming Assignments

All of your programming assignments in this class are composed of weekly Labs. All programming assignments can be completed alone or in groups of two: it's completely up to you. I encourage you to work in groups (both because this makes things easier and because you learn from talking to your colleagues). You are allowed to switch groups for every assignment (but you don't have to).

## Class Forum

We will have a course forum for you to discuss questions with your classmates and help each other out (I also plan to be very active and answer your questions here as well). If you're emailing me with a question, I would prefer you post it on the forum instead (assuming that it isn't super personal): that way other students can also learn from your question (and probably have the same one!)

**Danger Zone:** I expect you to behave yourself online the same way I expect you to behave in an in-person course: that means respect and patience for your fellow students. We all learn at different rates (I was personally a slow learner in college), and making others feel bad about that (or anything else) is **not** OK. I have absolutely zero tolerance for this stuff, and I **will** shut down the class forum if people are being disrespectful.

**Danger Zone Part 2:** The same collaboration/plagiarism/cheating rules that apply to the rest of this class extend to the online forum. That means no sharing of solutions, etc. You should feel comfortable asking questions about general concepts, but, when in doubt, just ask me if a question/answer is OK. Speaking of which....

## Cheating and Academic Honesty 🔥

This is the part of the syllabus I absolutely hate. But I'm also realistic and know we need to talk about this...so here goes.

Learning these ideas is challenging. I encourage you to discuss course activities with your friends and classmates as you are working on them, because you will definitely learn more in this class if you work with others than if you do not. Ask questions, answer questions, and share ideas liberally; we want a class that is open, welcoming, and collaborative, where we can help each other build the highest possible understanding of the course material.

#### Programming Assignments
The following applies to collaboration on programming assignments when interacting with students **other than your partner**.  Reminder: you are allowed to fully collaborate with your partner.

Learning collaboratively is different from sharing answers. Here are some guidelines to keep your interactions "collaborative" and not "cheating":
- You should never directly show another student your code. This means you should never send your files to another student nor should you screen share your work for other students.
  - Instead you are welcome to describe what is happening either verbally or in text.
- If you are helping another student, don't just tell them the answer; they will learn very little and run into trouble on exams. Instead, try to guide them toward discovering the solution on their own.
  - You should not be spelling out code for any other students to directly type.
- You should not be working on assignments in close collaboration with another person or group of people from start to finish. 
  - Try to collaborate on specific issues or questions as opposed to the entirety of an assignment.
- Please **NEVER** distribute solutions at any time, **even after the course is over**. This includes both solutions that you wrote and any solutions you may have obtained. "Distribution" includes uploading them to public websites (e.g. a GitHub repo), group repositories (e.g., your club's/frat's/sorority's "answer bank"), or private chats (e.g. group chat with a few friends).

#### Permitted Collaboration
Sharing answers is one form of cheating, but all forms of cheating are disallowed in this course.  **You should not claim to be responsible for work that is not yours**.  For clarity, examples of _allowed_ collaborations are listed below:

- Discussion of approaches for solving a problem.
- Giving away or receiving significant _conceptual_ ideas towards a problem solution; such help should be cited as comments in your code. For the sake of others' learning experience I ask that you try not to give away anything juicy, and instead try to lead people to such solutions.
- Using small snippets of code that you find online for solving tiny problems. For example, searching for "uppercase string" may lead you to some sample code that you copy and paste into your solution.

**Please cite any of the above collaborations as comments!**

Ultimately, **the goal of enrolling in this course is for YOU to learn this material**, so that you will be prepared for exams, for research, for job interviews, etc.  Engaging in academic misconduct does not help you towards that goal. If you are in doubt about what might constitute cheating, send me an email describing the situation and I will be happy to clarify it for you.

## Final Thoughts

I don't want this document to end with a scary section about cheating, so instead I'll end with this: I am **super excited** to be your instructor this semester. I love to teach, and I can't wait to see how you'll learn and grow this term. Welcome to CS XXXX! 🎉

Some other closing notes:

#### Inclusion
Computer Science is (as I'm sure many of you are aware) not generally a field that has been super accommodating towards underrepresented groups. With that in mind: all are welcome in this class. Making people feel unwelcome for who they are will not be tolerated. If you see or observe this, send me an email immediately.

#### Statement of Accommodations
Your success in this course is important to me. We all learn differently so if there are aspects of this course that prevent you from learning or exclude you, please let me know as soon as possible and together we will develop strategies to meet both your needs and the requirements of the course. I encourage you to visit the Accessibility Resources Centers for Students (ACCESS - formerly known as Disabled Students Programs and Services) for official accommodation services and resources to assist you in learning success. You can contact ACCESS at 274-4290.

#### Mental Health
This class can get stressful. If it gets too overwhelming at times, please feel free to send me an email and we can talk about how things are going. Do not suffer in silence: it is _literally_ my job to make sure you learn and have a good experience in this course. I would also encourage you to check out the [Student Health Center](https://www.northeastern.edu/healthcenter/mentalhealth/) for additional mental health resources. Without getting _too_ corny, I personally wish I had started on my mental health journey earlier. There's never any shame in asking for help!

#### Response Time
I will try my best to respond to all emails and forum posts within 24 hours: if I don't, please feel free to send me a reminder (I promise I will not mind). Cards on the table: I have a "day job" as a Robotics Engineer that I owe 100% of my attention to before 5PM. Teaching is something I do in my own time for personal fulfillment: but that also means that I won't always be able to reply instantly. But I also hope that my perspective as a (relatively) young person in the tech industry is more of a plus than a minus 😁