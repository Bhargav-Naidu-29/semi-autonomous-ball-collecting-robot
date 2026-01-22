

# Semi-Autonomous Ball-Collecting Robot (SABHR)

A reactive robotics platform designed for autonomous navigation and vacuum-based object harvesting. This project integrates hardware abstraction layers, real-time sensor fusion, and power-aware motor control into a unified embedded system.

## 🚀 Overview

The SABHR platform is a 4WD mobile robot engineered to solve the "Search and Collect" problem in a confined environment. The system features a custom-built **suction-based harvesting mechanism**—utilizing a high-RPM fan mounted within an aerodynamic PVC duct—to maximize static pressure for ball collection.

Unlike traditional mechanical grippers, this vacuum-based approach increases the "effective intake area," allowing the robot to successfully harvest targets without requiring millimeter-perfect navigation precision.

## 🛠 Hardware Architecture

The system is built on a modular chassis with the following core components:

* **Compute:** Arduino Uno (Atmega328P) managing the primary control loop.
* **Actuation:** * 4x High-torque DC Geared Motors for 4WD mobility and zero-point turning.
* L293D Motor Driver Shield for high-frequency PWM-based speed control.
* SG90 Micro Servo for high-precision sensor positioning.


* **Intake System:** Custom-engineered vacuum assembly with a ducted fan for high-velocity airflow.
* **Sensing:** HC-SR04 Ultrasonic Sensor mounted on a scanning array for 180° spatial awareness.

## 💻 Software Implementation

The software is structured as a **Reactive State Machine**, prioritizing environmental awareness and hardware longevity.

### 1. Perception & Spatial Scanning

The system utilizes a "Look-Before-You-Leap" algorithm to manage navigation. Instead of a fixed sensor, the HC-SR04 is mounted on a servo to perform periodic sweeps:

* **Spatial Sampling:** When an obstacle is detected within 15cm, the robot halts and triggers `lookRight()` and `lookLeft()` to sample the environment at  and .
* **Vector Selection:** The logic performs a max-value comparison to select the path with the highest clearance (`distanceR >= distanceL`).

### 2. Power-Aware Motor Control (Soft-Start)

To prevent **Voltage Sags** (brownouts) caused by high inrush current when starting four DC motors simultaneously, I implemented a **Linear Speed Ramping** algorithm.

```cpp
// Gradual PWM ramping to protect battery chemistry and logic stability
if(!goesForward) {
    goesForward=true;
    motor1.run(FORWARD);      
    motor2.run(FORWARD);
    // ...
    for (speedSet = 0; speedSet < MAX_SPEED; speedSet +=2) {
        motor1.setSpeed(speedSet);
        motor2.setSpeed(speedSet);
        delay(5);
    }
}

```

* **Engineering Benefit:** This protects the battery's discharge rate and ensures the logic circuit (Arduino) remains stable during high-torque demands.

### 3. Sensor Noise & Error Handling

Ultrasonic sensors can return "null" or `0` readings if the pulse is absorbed or moves out of phase. The `readPing()` function includes a safety fallback to ensure system reliability.

* **Default State:** If the sensor returns `0`, the code defaults to a value of `250cm`.
* **Logic Benefit:** This prevents the robot from "hallucinating" an obstacle and performing an emergency stop due to sensor noise or signal loss.

## 🔧 Technical Key Features

* **Hardware Abstraction:** Utilizes the `AFMotor` and `NewPing` libraries to decouple high-level logic from low-level register manipulation.
* **Latency Management:** Integrated a 70ms polling delay in the sensor loop to allow ultrasonic echoes to dissipate, preventing "Cross-talk" interference.
* **State Persistence:** Uses a `goesForward` boolean flag to ensure the soft-start logic only triggers during state transitions, optimizing CPU cycles.

## 📈 Future Roadmap

* **Asynchronous Logic:** Transitioning from `delay()`-based timing to a non-blocking `millis()` architecture for true multi-tasking and parallel vacuum operation.
* **Vision Integration:** Adding an ESP32-Cam for color-based target identification (separating balls from obstacles).
* **PID Navigation:** Implementing Proportional-Integral-Derivative control for smoother distance maintenance and path correction.

---

### How to use this project

1. **Hardware Setup:** Connect the HC-SR04 to A0/A1 and the Servo to Pin 10.
2. **Library Requirements:** Install `AFMotor`, `NewPing`, and `Servo` libraries via the Arduino IDE.
3. **Deployment:** Upload the provided `.ino` file to an Arduino Uno and ensure the motor shield is powered by an external 7.4V - 12V source.

---
