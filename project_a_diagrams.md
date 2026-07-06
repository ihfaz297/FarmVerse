# Project A: Vision-Guided Object Sorting Arm - Hardware & Architecture Overview

> [!NOTE]
> This guide is designed specifically for the Hardware Engineer to understand the physical setup, power requirements, and the functional logic of the system.

## 1. Physical Layout & Component Setup

![Robot Arm Concept](./assets/robot_arm_setup.png)

### Hardware Specifications to Ensure:
* **Microcontroller:** ESP32 or Arduino Uno.
* **Servos:**
  * 2x **MG996R metal-gear servos** for the Base and Shoulder (High torque is required here).
  * 2x **SG90 micro servos** for the Wrist and Gripper.
* **Servo Driver:** **PCA9685 PWM Driver** is strongly recommended for 4 servos to ensure stable signal distribution.
* **Power Supply:** **External 5V 2–3A power supply**. 
* **Webcam:** A standard USB webcam mounted directly above the sorting tray.

> [!CAUTION]
> **POWER RAILS:** Never power the servos directly off the Microcontroller's 5V pin. This will cause brownouts and random resets. Use the separate external power supply for the PCA9685 driver, and ensure there is a **common ground** between the external power supply, the PCA9685, and the microcontroller.

> [!IMPORTANT]
> **GRASP RELIABILITY:** Cheap servos have backlash. The gripper must close with consistent force. Consider adding a rubber or foam pad to the gripper fingers to improve friction and ensure items aren't dropped. Confirm every joint moves freely by hand before applying power.

## 2. Functional Architecture & Control Flow

The following diagram illustrates how the software and hardware interact. The heavy lifting (Computer Vision) happens on the laptop, which then simply tells the microcontroller "which predefined motion sequence" to run. No complex inverse kinematics are computed on the hardware.

```mermaid
flowchart TD
    %% Define Styles
    classDef hardware fill:#f9f0ff,stroke:#8e44ad,stroke-width:2px;
    classDef software fill:#eaf2ff,stroke:#2980b9,stroke-width:2px;
    classDef power fill:#fff2e6,stroke:#d35400,stroke-width:2px;
    
    subgraph Laptop ["Laptop (Software Layer)"]
        A[Webcam Video Stream] -->|Frames| B(YOLOv8n Inference Python)
        B -->|Detects Cup/Bottle/Phone| C{Stable Detection for 10+ frames?}
        C -->|Yes| D[Map Centroid to 1 of 3 Zones]
        D -->|Send Single Byte over Serial '0', '1', or '2'| E[Wait for 'Done-Ack' from MCU]
    end

    subgraph Hardware ["Microcontroller & Actuators (Hardware Layer)"]
        F(ESP32 / Arduino Uno)
        G[PCA9685 PWM Driver]
        H((MG996R: Base))
        I((MG996R: Shoulder))
        J((SG90: Wrist))
        K((SG90: Gripper))
        
        F -->|I2C Signals| G
        G -->|PWM| H
        G -->|PWM| I
        G -->|PWM| J
        G -->|PWM| K
        
        F -->|Send 'Done-Ack' over Serial 'D'| E
    end

    subgraph Power ["Power Distribution"]
        L[External 5V 2-3A Supply]
        M[Laptop USB]
    end

    L ==>|Power| G
    L -.->|Common Ground| F
    M ==>|Power & Serial Data| F

    class A,B,C,D,E software;
    class F,G,H,I,J,K hardware;
    class L,M power;
```

### Communication Protocol
- The **laptop acts as the brain**. When an object is stable in a specific zone for ~10 frames, it sends a single ASCII digit (e.g., `0`, `1`, or `2`) over USB Serial.
- The **microcontroller receives the digit** and runs a pre-programmed, hardcoded series of waypoints: `Home -> Hover -> Lower -> Grip -> Lift -> Carry to Bin -> Release -> Home`.
- Once the physical sequence completes, the **microcontroller sends a `"D"` (Done-Ack)** back to the laptop to indicate it is ready for the next command. This handshaking prevents race conditions.
