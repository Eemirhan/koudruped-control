# KOUDRUPED – Quadruped Kinematics, Gait & Balance Control in Simulink

![Simscape simulation](media/simscape_walk.gif)

**KOUDRUPED** is a 12-DOF mini quadruped robot built as our graduation project at Kocaeli University Mechatronics Engineering, supported by **TÜBİTAK 2209-B** with **Altınay Defense Technologies** as industry partner.

This repository contains **my part of the project**: the kinematic model, gait simulations and body-balance controller, built in MATLAB/Simulink and Simscape Multibody.

<p align="center">
  <img src=<img width="3024" height="3024" alt="WhatsApp Image 2026-10-26 at 15 17 54" src="https://github.com/user-attachments/assets/24dab15b-5335-48ca-b1c3-d0d203fc6006" />
 width="48%" />
  <img src=<img width="690" height="527" alt="image" src="https://github.com/user-attachments/assets/8f82665e-acaa-46dd-989b-b4c20a635dc2" />
 width="48%" />
</p>

## What's inside

| Folder | Content |
|---|---|
| `matlab/` | D-H parameters, forward / inverse kinematics scripts, workspace plots |
| `simulink/` | Simscape Multibody model of the robot, gait and balance controller |
| `media/` | Images and GIFs used in this README |

## 1. Kinematics

Each leg is a 3-DOF chain (hip, upper leg, lower leg). I defined the kinematic chain, which the team then used as the basis for the mechanical design. I also derived the D-H table and the analytical inverse kinematics.

![Leg kinematic model](media/leg_kinematics.png)

## 2. Physical simulation (Simscape Multibody)

- The CAD model was exported from SolidWorks with Simscape Multibody Link, keeping masses and inertias.
- A 6-DOF joint connects the body to the world, and the hip and knee joints are revolute.
- Foot–ground contact uses Spatial Contact Force blocks. I tuned the stiffness and damping so the feet don't bounce or sink into the ground.
- A 2-second **stand-up phase** moves the legs smoothly from the start position to the nominal posture. This avoids the initialisation shock that otherwise locks the joints.

![Stand-up phase](media/standup.png)

## 3. Gaits

I evaluated trot and step gait patterns in simulation before manufacturing.

![Trot gait](media/trot.gif)

## 4. Body-balance controller

The controller keeps the body parallel to the ground on slopes and under disturbances:

1. **Attitude:** orientation is read as a quaternion to avoid gimbal lock and converted to pitch and roll (ZYX). In my simulation the IMU signal is fused with a complementary filter.
2. **Control:** separate PI controllers for pitch and roll, with **back-calculation anti-windup** because the actuator limits saturate the output.
3. **Mixing:** the pitch and roll corrections are added with opposite signs to the target height of each leg, around a nominal body height of –130 mm.
4. A time-triggered switch keeps the controller off until the stand-up phase is finished.

![Balance controller](media/balance_controller.png)
![Pitch / roll response](media/pitch_roll_response.png)

## 5. On the real robot

We built the robot from 3D-printed parts, with RDS3235 servos and a Raspberry Pi 5. I helped with assembly and integration, and transferred the controller logic from this model to the robot. In hardware tests:

- After calibrating joint offsets, the foot-tip error was within **5 mm**.
- With IMU feedback, the body stayed level on **10° slopes**.

Walking on the real robot was not as smooth as in simulation. Limited tuning time and distributions of masses constraints kept us from fully tuning the gaits on hardware.

![Real robot](media/robot_real_2.jpg)

## Team

- **Emirhan Keşkek:** kinematics, Simulink / Simscape simulation and control, assembly and testing
- **Ali Haydar Yıldırım** and **Fatih İnan:** mechanical design, electronics and various software

Supervisor: Dr. Haluk Özakyol, Kocaeli University

## Requirements

MATLAB / Simulink with Simscape Multibody (Robotics System Toolbox recommended for the kinematics scripts).
