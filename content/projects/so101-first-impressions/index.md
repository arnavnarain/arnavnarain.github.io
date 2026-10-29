---
title: "SO-101 Arm: First Impressions"
date: 2026-10-29
draft: false
tags: ["robotics", "so-101", "lerobot", "teleoperation"]
summary: "I chose the SO-101 arm because it's fairly cheap and beginner friendly. It's a 6-axis robot arm built to work natively with the Hugging Face LeRobot ecosystem for recording datasets and training policies."
---

{{< figure src="hero.jpg" alt="The orange SO-101 follower arm with its gripper next to the leader arm with its trigger handle, both folded on a desk" caption="Follower (left, with the gripper) and leader (right, with the handle)." >}}

> *A warning for the reader: if you expect to see the solution to dexterity here, expect to be disappointed. We're just running smoke tests today.*

## Background

I chose the SO-101 arm because it's fairly cheap and beginner friendly. It's a **6-axis robot arm** built to work natively with the [Hugging Face LeRobot](https://huggingface.co/docs/lerobot) ecosystem for recording datasets and training policies.

We have 2 robots - a **leader** and a **follower**. In teleop mode, the follower receives a constant stream of joint positions from the leader and aims to replicate it. This allows you to record a series of actions and see it replicated on the follower.

## Setup

Part of what makes the SO-101 so popular is it's completely open-source. Anyone can 3D print the components and assemble one. It's a fantastic lesson in the basics of robotics: mechanical assembly, motor wiring, and patience. I'd highly recommend it.

So anyway, I bought mine pre-assembled. I made a conscious decision to prioritize using the robot over building it. Besides, I don't consider myself very hardware-oriented, nor do I want to be. If that means future repairs/improvements on the arm are tougher, so be it.

On the software side, I set up a Python virtual environment, complete with the `lerobot` library and its Feetech motor drivers.

After connecting the two arms to my laptop and to power, I was all set. I ran a quick command to find the ports the leader & follower live on and ran basic calibration to find the complete range of motion present in each arm.

## The Smoke Test

The problem for today is getting the follower to pick up a piece of measuring tape. Hardly an unsolved problem in robotics, especially when using the leader to direct its actions. Here's a clip of me doing it:

{{< video src="teleop.mp4" >}}

I wrote a simple script that records the stream of positions coming from each arm. It's visualized in the graph below. The **red line** shows the error (calculated as the difference between both positions).

{{< figure src="tracking_error.png" alt="Six stacked plots of leader command vs follower position for each joint over 28 seconds, with the error in red on a right-hand axis" caption="Leader (blue) vs follower (orange) for each joint over ~28s of teleop. Red is command minus actual, on its own right-hand axis." >}}

One interesting thing - the gripper shows a **constant error of about 7.5** from roughly **14s to 21.5s**. That's because the leader is fully closed, whereas the follower is unable to reach that position, blocked by the measuring tape.

{{< figure src="gripper_tracking.png" alt="Gripper command vs actual position, with the gap shaded red while the gripper holds the measuring tape" caption="Gripper only. The shaded band is the follower stopping short of fully closed because the tape is in the way." >}}

All the computation ran on my laptop, while each servo handled moving itself to the target position. My laptop did not lag very much during the process, although there were a few alerts that it was not able to maintain a refresh rate of 60Hz.

```text
Teleop loop time: 16.69ms (60 Hz)
WARNING 2026-10-25 16:25:51 le_timer.py:407 Control loop is running slower (29.8 Hz)
  than the target FPS (60 Hz). Robot control might be unstable.
Teleop loop time: 16.68ms (60 Hz)
...
Cadence summary — whole run · target 60 Hz (16.7 ms budget per tick): 29904 ticks
```

*Mostly a steady 60 Hz, with a dip roughly once a minute over ~30,000 ticks.*

If computations were to scale, that could cause a more significant delay between the follower and the leader. The graph above shows a *slight* delay between follower and leader action, under 100ms per joint. Part of that is the round trip from the leader, to my laptop, to the follower, but most of it is the motors physically catching up. That's also why the error spikes during fast swings. Worth paying attention to, since it will be more relevant when we mix in simulation.

## What's next

Next up: mounting a camera, recording demos, and training a policy so the follower can do basic tasks without me. I'll start with ACT, a standard imitation learning model, then (likely) move to VLAs (vision-language-action models).
