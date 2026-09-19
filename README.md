# ROS 2 Roll Call

A small ROS 2 exercise where students publish their names to a shared topic and have them appear on the lecturer's screen.

## Overview

This project introduces the basics of ROS 2 communication:

- connecting to a remote machine with SSH
- publishing a message to a ROS topic
- creating a simple Python node
- building and running a ROS package
- understanding how nodes communicate through topics and message types

## Goal

The goal is to get your name to appear on the lecturer's screen by publishing a message on the `/roll_call` topic.

## Prerequisites

- Access to the lecturer's laptop over SSH
- ROS 2 installed on the remote machine
- A valid student username and password

## Basic command

```bash
ros2 topic pub /roll_call std_msgs/msg/String "data: 'Maria'" -r 1
```

Replace `Maria` with your own name.

## Project concept

This exercise demonstrates the core ROS 2 pattern:

- one node publishes data
- another node subscribes to a topic
- both agree on the topic name and message type

## Files

- `ros2_roll_call_student_guide.md` — full student instructions

## Notes

This is a learning activity designed for classroom use. It helps students understand how topics, publishers, and message passing work in ROS 2.
