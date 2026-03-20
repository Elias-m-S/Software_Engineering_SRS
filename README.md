# Emergency Stop System — Software Requirements Specification

> Autonomous Miniature Truck · ROS · Arduino · LiDAR · DHBW Friedrichshafen

---

## Overview

This repository contains the **Software Requirements Specification (SRS)** for the emergency stop system of an autonomously driving miniature truck, developed as a project submission for the *Software Engineering (Program Design)* module at DHBW Ravensburg, Campus Friedrichshafen.

**Team:** Vier Muske(l)tiere — Dominik Borowski, Niklas Grunewald, Carla Klammt, Elias Sehmsdorf  
**Supervisor:** Daniel Stimac  
**Version:** 0.4 — December 2025

> **Note:** This repository contains the SRS document only. No implementation code is included — the specification serves as the basis for a future implementation phase.

---

## System Description

The emergency stop system is a safety-critical subsystem running in parallel to the autonomous driving logic. It continuously monitors sensor data and operator commands, and overrides all other driving functions when a hazardous situation is detected.

The system is implemented as a **ROS node** on the main computer (Linux / Jetson / NUC) and communicates with the hardware layer via CAN bus through a Master/Slave Arduino setup.

### Stop Modes (Priority Order)

| Priority | Mode | Trigger | System Lock |
|---|---|---|---|
| 1 | **Hardstop** | Physical mushroom button (Pilztaster) | Yes — requires manual reset |
| 2 | **E-Stop** | Automatic obstacle detection via LiDAR | No |
| 3 | **Softstop** | Side button or UI/RViz command | No |
| 4 | **Warning Mode** | Sensor fault detected | No |

### Post-Stop Behavior

After a Softstop or E-Stop, the vehicle can resume via two modes: **Auto-Resume** (if the path is clear) or **Manual Resume** (operator confirmation required). Both lead into **Creep Mode** — a slow, controlled movement for safe situation assessment.

---

## Key Requirements Summary

**Functional:** Hardstop locks the system and cuts motor PWM immediately. Softstop and E-Stop allow restricted movement (reverse always permitted, forward only in creep mode if obstacle distance > 10 cm). Obstacle detection uses both 2D LaserScan and 3D PointCloud2 with OR logic. All stop events are logged with timestamp, trigger source, sensor data, vehicle pose and control mode.

**Non-functional:** Max 50 ms latency from obstacle detection to stop command. Vehicle must stop within a 20 cm safety margin at up to 0.55 m/s. False positive rate ≤ 5%, false negative rate ≤ 1%. Logs stored as ROS Bags, auto-deleted after 14 days.

---

## Architecture

```
Operator / Sensors
       |
  NothaltNode (ROS)         ← Central safety logic
       |
  TruckBridgeNode           ← ROS → CAN bridge
       |
  MasterArduino             ← CAN command processing
       |
  SlaveArduino              ← PWM signal generation
       |
  MotorController           ← Physical braking
       |
  Logger / ROS-Bags         ← Event recording
```

ROS topics published: `/emergency_stop_state`, `/fault_state`, `/sensor_health`

---

## Document Structure

```
Projektarbeit.pdf           ← Full SRS document
```

The SRS covers: system overview & context, functional requirements (Hardstop, Softstop, E-Stop, Warning Mode, Resume, Logging, Warning Lights), non-functional requirements (timing, safety, reliability, architecture), UML models (state diagrams, activity diagram, sequence diagrams), system architecture, and requirements quality assurance with full traceability.