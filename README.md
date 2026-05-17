# Embedded-Project-
Team 14 Sec1
# CSE211s: Autonomous Object-Following Robot Car

An embedded systems project that implements a real-time, object-tracking robot vehicle using an ARM Cortex-M based STM32 Nucleo board, an HC-SR04 ultrasonic sensor, and an L298N H-bridge motor driver. 

Developed as a core project requirement for **CSE211s Introduction to Embedded Systems** | Spring 2026.
**Department of Mechatronics, Faculty of Engineering, Ain Shams University.**

---

## 📋 Project Requirements & Objectives

The goal of the assignment was to design an embedded system mounted on a 2WD robot chassis that can:
1. **Track an object** moving in a straight line ahead of it.
2. **Maintain a safe following distance** dynamically without colliding or lagging too far behind.
3. **Execute a safe stop** immediately if the target object halts.

### Key Evaluation Criteria:
* **Hardware Efficiency:** Direct use of low-level microcontroller hardware modules (Timers, Input Capture, PWM) rather than processor-blocking delay routines.
* **Code Modularity:** Logical separation of hardware configurations, background execution tasks, and time-critical interrupt systems.
* **Documentation:** Clear presentation, comprehensive documentation, and well-commented code.

---

## 🛠️ What We Did (Implementation Details)

To fulfill the requirements efficiently, we developed an **interrupt-driven, non-blocking software architecture** using the Mbed OS framework. 

### 1. Hardware Interfacing & Pin Mapping
The physical wiring links the Nucleo microcontroller directly to the sensor and actuator drivers using the following pin macro definitions:

* **HC-SR04 Ultrasonic Sensor:**
  * `TRIG_PIN` --> **D8** (Digital Output used to fire the acoustic burst)
  * `ECHO_PIN` --> **D9** (Interrupt Input used to time the returning pulse)
* **L298N Motor Driver Logic:**
  * `ENA_PIN` --> **D3** (PWM Output controlling Left Motor speed)
  * `ENB_PIN` --> **D5** (PWM Output controlling Right Motor speed)
  * `IN1_PIN` / `IN2_PIN` --> **D4** / **D7** (Left Motor direction control pins)
  * `IN3_PIN` / `IN4_PIN` --> **D10** / **D11** (Right Motor direction control pins)

### 2. Interrupt Service Routines (ISRs) for Sensor Efficiency
Instead of traditional blocking functions like `pulseIn()` which halt the CPU during measurement, we utilized the Nucleo's hardware interrupts (`InterruptIn`):
* **`echo_rise_ISR()`**: Triggers exactly when the Echo pin goes HIGH, resetting and starting an internal hardware timer (`Timer echoTimer`).
* **`echo_fall_ISR()`**: Triggers when the Echo pin drops LOW. It captures the elapsed microsecond count and calculates the precise distance using the speed of sound formula:
  $$\text{Distance (cm)} = \frac{\text{Time } (\mu\text{s}) \times 0.0343}{2}$$

### 3. State Control Logic
The main execution loop triggers a sensor measurement every 60ms and evaluates the target distance based on strict operational zones:
* **distance_cm < STOP_DISTANCE_MIN (Too Close):** Immediate motor brake to prevent collision.
* **distance_cm >= STOP_DISTANCE_MIN && distance_cm <= STOP_DISTANCE_MAX (Safe Zone):** Target has stopped at a suitable distance; motors are halted safely.
* **distance_cm > STOP_DISTANCE_MAX && distance_cm < MAX_FOLLOW_DIS (Follow Zone):** Target is moving away; the car engages forward propulsion at a calibrated 20% base duty cycle to track smoothly in a straight line.
* **distance_cm > MAX_FOLLOW_DIS (Lost Target):** Out of bounds range; the car halts automatically as a safety measure.

*(Note: Motor speed balance adjustment factors are built into the software logic to correct physical chassis drift and guarantee straight-line tracking).*

---

## 📁 Repository Structure

```text
├── src/
│   └── main.cpp           # Complete commented source code with modular helper functions
├── docs/
│   ├── Report.pdf         # Final technical report containing software layout and structures
│   └── Wiring_Diagram.png # Hardware schematic and pin-to-pin connections
└── README.md              # Project overview and documentation
