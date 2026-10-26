# KOUDRUPED – Quadruped Kinematics, Gait & Balance Control

<p align="center">
  <img src="media/robot_real.jpg" width="48%" />
  <img src="media/robot_cad.png" width="48%" />
</p>
<p align="center"><i>Left: the built robot. Right: CAD model.</i></p>

**KOUDRUPED** is a 12-DOF mini quadruped robot built as our graduation project at Kocaeli University Mechatronics Engineering, supported by **TÜBİTAK 2209-B** with **Altınay Defense Technologies** as industry partner.

This repository contains **my part of the project**: the leg kinematics, gait simulations and body-balance controller, built in MATLAB/Simulink and Simscape Multibody.

## What's inside

| Folder | Content |
|---|---|
| `matlab/` | Inverse kinematics scripts and leg workspace plots |
| `simulink/` | Simscape Multibody model of the robot, gait and balance controller |
| `media/` | Images used in this README |

## 1. Kinematics

Each leg is a 3-DOF chain: shoulder (θ1), hip (θ2) and knee (θ3). I defined the kinematic chain, which the team then used as the basis for the mechanical design.

<p align="center">
  <img src="media/ik_front.png" height="380" />
  <img src="media/ik_side.png" height="380" />
</p>
<p align="center"><i>Link definitions: hip offset (top view), upper leg / femur and lower leg / tibia (side view).</i></p>

<p align="center">
  <img src="media/leg_kinematics.png" width="45%" />
</p>
<p align="center"><i>Joint coordinate frames on the leg.</i></p>

I solved the inverse kinematics **geometrically**, using closed-form trigonometric relations instead of matrix inversion or iteration. This keeps the solution fast enough for an embedded controller and easy to debug, because each equation maps directly onto the leg's geometry.

1. **Front view → θ1:** The target foot position is first expressed in the shoulder frame. Seen from the front, the upper and lower leg are treated as one line, and θ1 comes from `atan2` and `asin` relations, mirrored for the left and right legs.
2. **Side view → θ2, θ3:** After θ1 is applied, the target is moved into the hip frame. The upper leg, lower leg and hip-to-foot distance form a triangle: θ3 follows from the law of cosines, and θ2 from the triangle angles plus the direction of the target.

## 2. Physical simulation (Simscape Multibody)

<p align="center">
  <img src="media/standup.png" width="70%" />
</p>
<p align="center"><i>The robot in Simscape Multibody.</i></p>

- The CAD model was exported from SolidWorks with Simscape Multibody Link, keeping masses and inertias.
- A 6-DOF joint connects the body to the world, and the hip and knee joints are revolute.
- Foot–ground contact uses Spatial Contact Force blocks. I tuned the stiffness and damping so the feet don't bounce off or sink into the ground.
- A 2-second **stand-up phase** moves the legs smoothly from the start position to the nominal posture. This avoids the initialisation shock that otherwise locks the joints.
- I evaluated trot and step gait patterns in this model before manufacturing.

## 3. Body-balance controller

The controller keeps the body parallel to the ground on slopes and under disturbances:

1. **Attitude:** orientation is read as a quaternion to avoid gimbal lock, then converted to pitch and roll (ZYX). In simulation, the IMU signal is fused with a complementary filter.
2. **Control:** separate PI controllers for pitch and roll, with **back-calculation anti-windup** because the actuator limits saturate the output.
3. **Mixing:** the pitch and roll corrections are added with opposite signs to the target height of each leg, around a nominal body height of –130 mm.
4. A time-triggered switch keeps the controller off until the stand-up phase is finished.

## 4. On the real robot

The team built the robot from 3D-printed parts, with RDS3235 servos and a Raspberry Pi 5. I helped with assembly and integration, and transferred the controller logic from this model to the robot. In hardware tests:

- After calibrating joint offsets, the foot-tip error was within **5 mm**.
- With IMU feedback, the body stayed level on **10° slopes**.

Walking on the real robot was not as smooth as in simulation. Limited time and battery constraints kept us from fully tuning the gaits on hardware, and this is the main next step.

## Team

- **Emirhan Keşkek:** kinematics, Simulink / Simscape simulation and control, assembly and testing
- **Ali Haydar Yıldırım** and **Fatih İnan:** mechanical design, electronics and robot software

Supervisor: Dr. Haluk Özakyol, Kocaeli University

## Requirements

MATLAB / Simulink with Simscape Multibody.
