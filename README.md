# 🤖 ARGUS V1 — Autonomous Swarm Robot

> **ARGUS V1** is an ESP32-based autonomous mobile robot developed as a modular platform for **swarm robotics, autonomous navigation, sensing, wireless communication, and object-handling applications**.

![ARGUS V1](docs/images/argus-v1.jpg)

---

## 📌 Overview

ARGUS V1 is a compact differential-drive mobile robot built around an **ESP32 38-pin development board**.

The project is being developed as a hardware and software platform for a future **multi-robot swarm system**, where multiple ARGUS robots can communicate, coordinate tasks, navigate an environment, and operate cooperatively.

The system combines:

* 🧠 ESP32-based control
* ⚙️ Dual N20 DC gear motors
* 🎛️ TB6612FNG dual motor driver
* 🧭 MPU6050 IMU
* 📏 HC-SR04 ultrasonic sensing
* 👁️ Three IR sensors
* 🔘 Limit switch
* 🧲 Electromagnet actuator
* 🔔 Buzzer
* 🚦 Status indicators
* 🔋 3S 18650 battery system with BMS
* 📡 Wireless communication capability
* 🪪 RFID identification/control
* ⚙️ Servo actuator
* 🤖 Swarm robotics software architecture

The hardware is designed around a modular architecture so that individual subsystems can be tested independently before being integrated into the complete autonomous robot.

---

# 🎯 Project Goals

The main objectives of ARGUS V1 are:

1. Develop a reliable ESP32-based mobile robot platform.
2. Implement differential-drive motion control.
3. Integrate multiple sensors for environmental awareness.
4. Implement autonomous navigation.
5. Implement wireless robot-to-controller communication.
6. Develop a scalable swarm-robot architecture.
7. Enable multiple robots to coordinate tasks.
8. Provide a modular hardware/firmware architecture.
9. Develop and validate every subsystem independently before final integration.

---

# 🧠 System Architecture

```text
                         ┌─────────────────────┐
                         │       ARGUS V1      │
                         │       ESP32         │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
        ┌───────────┐        ┌────────────┐        ┌─────────────┐
        │  Motion   │        │   Sensors  │        │ Communication│
        │  Control  │        │            │        │   / Swarm    │
        └─────┬─────┘        └──────┬─────┘        └──────┬──────┘
              │                     │                     │
              ▼                     ▼                     ▼
       ┌─────────────┐       ┌──────────────┐       ┌─────────────┐
       │ TB6612FNG   │       │ MPU6050      │       │ Wi-Fi       │
       │ Motor Driver│       │ HC-SR04      │       │ ESP-NOW*    │
       └──────┬──────┘       │ IR Sensors   │       │ Controller* │
              │              │ Limit Switch │       └─────────────┘
        ┌─────┴─────┐        └──────────────┘
        ▼           ▼
    ┌───────┐   ┌───────┐
    │ N20 L │   │ N20 R │
    │ Motor │   │ Motor │
    └───────┘   └───────┘

                     Actuation
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
          Electromagnet Servo     Buzzer
```

`*` Communication and swarm functions are under development.

---

# 🔧 Hardware

## Main Controller

| Component          | Specification          |
| ------------------ | ---------------------- |
| Microcontroller    | ESP32 38-pin           |
| Logic voltage      | 3.3 V                  |
| Motor driver       | TB6612FNG              |
| Motors             | 2 × N20 DC gear motors |
| IMU                | MPU6050                |
| Distance sensor    | HC-SR04                |
| IR sensors         | 3                      |
| Limit switch       | NO type                |
| Actuator           | Electromagnet          |
| Auxiliary actuator | Servo                  |
| Buzzer             | 1                      |
| Status LEDs        | Red / Yellow / Green   |
| Battery            | 3S 18650               |
| BMS                | 3S BMS                 |
| Power regulation   | MP1584 buck converter  |

## The ARGUS V1 schematic contains the ESP32, TB6612FNG, two N20 motors, MPU6050, HC-SR04, three IR sensors, battery/BMS and power circuitry.

# 📍 ESP32 Pin Configuration

Current ARGUS V1 firmware pin assignment:

| Function                |   GPIO |
| ----------------------- | -----: |
| Motor A PWM / PWMA      | **25** |
| Motor A IN1 / AIN1      | **26** |
| Motor A IN2 / AIN2      | **27** |
| TB6612 STBY             | **17** |
| Motor B IN1 / BIN1      | **18** |
| Motor B IN2 / BIN2      | **19** |
| Motor B PWM / PWMB      | **33** |
| MPU6050 SDA             | **21** |
| MPU6050 SCL             | **22** |
| Limit Switch            | **15** |
| Electromagnet MOSFET Q1 |  **2** |
| Auxiliary MOSFET Q2     |  **5** |
| Buzzer                  | **32** |
| Red LED                 | **13** |
| Yellow LED              | **14** |
| Green LED               | **16** |

The motor-driver pins are defined in the ARGUS V1 schematic as PWMA, AIN1, AIN2, STBY, BIN1, BIN2 and PWMB.

The schematic also specifies the traffic-light outputs, limit switch and MOSFET-controlled outputs.

> **Note:** Additional sensor and actuator GPIO assignments will be finalized during hardware bring-up and firmware integration.

---

# ⚙️ Motor Control

ARGUS V1 uses a **TB6612FNG dual H-bridge motor driver** to control two N20 DC gear motors.

```text
ESP32
  │
  ├── PWMA ──► TB6612FNG ──► Motor A
  ├── AIN1 ──►
  ├── AIN2 ──►
  │
  ├── PWMB ──► TB6612FNG ──► Motor B
  ├── BIN1 ──►
  ├── BIN2 ──►
  │
  └── STBY ──► Driver Enable
```

PWM is used to control motor speed, while the direction inputs determine forward/reverse rotation.

### Motor Bring-Up

The first hardware validation stage successfully tests:

* Motor A forward
* Motor A reverse
* Motor B forward
* Motor B reverse
* Both motors forward
* Both motors reverse

Motor control is therefore the first validated subsystem of ARGUS V1.

---

# 🧭 Sensors

## MPU6050

The MPU6050 provides:

* 3-axis accelerometer
* 3-axis gyroscope
* Orientation/motion information
* Angular velocity
* Acceleration data

Current I²C connection:

```text
MPU6050
   │
   ├── SDA → GPIO21
   ├── SCL → GPIO22
   ├── VCC
   └── GND
```

The schematic identifies the MPU-6050 sensor and its SDA/SCL interface.

---

## HC-SR04

The ultrasonic sensor is intended for:

* Front obstacle detection
* Distance measurement
* Autonomous navigation
* Collision avoidance

```text
HC-SR04
   │
   ├── VCC
   ├── TRIG
   ├── ECHO
   └── GND
```

The exact ESP32 GPIO assignment for TRIG/ECHO is maintained as a firmware configuration item until the final PCB wiring is verified. The schematic identifies the HC-SR04 interface but does not clearly establish the GPIO mapping in the parsed connection labels.

---

## IR Sensors

ARGUS V1 contains three IR sensors:

```text
IR1
IR2
IR3
```

They can be used for:

* Line/edge detection
* Obstacle detection
* Local navigation
* Position/state detection

The schematic shows three individual IR sensor modules with VCC, GND and OUT connections.

---

## Limit Switch

The mechanical limit switch is used as a safety/position input.

Intended connection:

```text
NO  → GPIO15
COM → GND
NC  → Not connected
```

The firmware uses the ESP32 internal pull-up:

```cpp
pinMode(15, INPUT_PULLUP);
```

Therefore:

```text
Released → HIGH
Pressed  → LOW
```

The schematic identifies the limit switch connection as **NO-P15** with GND on the common connection.

---

# 🧲 Electromagnet

ARGUS V1 includes an electromagnet controlled through a MOSFET switching stage.

```text
ESP32 GPIO2
     │
     ▼
 MOSFET Q1
     │
     ▼
Electromagnet
```

A MOSFET and protection diode are included in the actuator circuit.

The electromagnet is intended for object pickup/handling during autonomous missions.

---

# 🚦 Status System

ARGUS V1 contains three status outputs:

```text
GPIO13 → RED
GPIO14 → YELLOW
GPIO16 → GREEN
```

These can be used to represent robot states such as:

| LED       | Possible state              |
| --------- | --------------------------- |
| 🔴 Red    | Error / Emergency Stop      |
| 🟡 Yellow | Initialization / Processing |
| 🟢 Green  | Ready / Autonomous          |

The exact state machine will be finalized during software integration.

---

# 🔊 Buzzer

The buzzer is connected through the ARGUS V1 control circuitry.

```text
GPIO32 → Buzzer
```

Possible functions:

* Startup indication
* Warning
* Obstacle alert
* Emergency indication
* Task completion
* Communication status

The schematic identifies the buzzer control as **P32**.

---

# 🔋 Power System

ARGUS V1 uses a rechargeable **3S 18650 battery system**.

```text
        3S 18650 Battery
               │
               ▼
             3S BMS
               │
               ▼
        ┌──────────────┐
        │ Power System │
        └───────┬──────┘
                │
        ┌───────┴────────┐
        ▼                ▼
   Motor Supply      MP1584 Buck
                         │
                         ▼
                    Logic Supply
```

The schematic includes a 3S 18650 BMS and MP1584 buck converter.

> **Safety:** Battery, BMS, motor supply and charging circuitry must be verified independently before full-power operation.

---

# 🪪 RFID & Servo

RFID and servo functionality are planned as part of the robot's object/task handling system.

Potential functions include:

* Robot identification
* Object identification
* Access/task authorization
* Mechanical positioning
* Pickup/drop-off mechanisms

These subsystems will be integrated after the core motor and sensor bring-up.

---

# 🤖 Swarm Robotics

ARGUS V1 is designed as an individual node in a future multi-robot swarm.

```text
             ┌──────────────┐
             │ Swarm Master │
             └───────┬──────┘
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
       ARGUS-01   ARGUS-02   ARGUS-03
          │          │          │
          └──────────┼──────────┘
                     │
              Shared Environment
```

Each robot should eventually provide:

* Unique robot ID
* Position/state information
* Sensor status
* Battery status
* Task status
* Motor commands
* Emergency state
* Inter-robot communication

The swarm layer will be developed separately from the low-level motor and sensor firmware.

---

# 🧠 Software Architecture

The firmware is being developed in layers.

```text
┌───────────────────────────────────────┐
│           SWARM APPLICATION           │
│       Task / Formation / Mission      │
├───────────────────────────────────────┤
│         COMMUNICATION LAYER           │
│       Wi-Fi / ESP-NOW / Serial        │
├───────────────────────────────────────┤
│         NAVIGATION LAYER              │
│   Position / Heading / Obstacle Logic │
├───────────────────────────────────────┤
│          SENSOR FUSION                │
│ MPU6050 / IR / Ultrasonic / Switch   │
├───────────────────────────────────────┤
│          MOTOR CONTROL                │
│       Differential Drive / PWM        │
├───────────────────────────────────────┤
│          HARDWARE LAYER               │
│       ESP32 GPIO / Drivers / I/O      │
└───────────────────────────────────────┘
```

This modular structure allows each subsystem to be tested independently before integration.

---

# 🧪 Development & Testing Strategy

ARGUS V1 follows a staged hardware bring-up process.

### Phase 1 — Controller

* [x] ESP32 programming
* [x] Serial communication
* [x] GPIO validation

### Phase 2 — Motor System

* [x] TB6612FNG initialization
* [x] Motor A forward/reverse
* [x] Motor B forward/reverse
* [x] Both motors forward/reverse

### Phase 3 — Safety Inputs

* [ ] Limit switch
* [ ] Emergency stop behavior

### Phase 4 — Sensors

* [ ] MPU6050
* [ ] IR sensor 1
* [ ] IR sensor 2
* [ ] IR sensor 3
* [ ] HC-SR04

### Phase 5 — Actuators

* [ ] Buzzer
* [ ] Status LEDs
* [ ] Electromagnet
* [ ] Servo

### Phase 6 — Identification

* [ ] MFRC522 RFID
* [ ] RFID read test
* [ ] RFID-based task identification

### Phase 7 — Communication

* [ ] ESP32 Wi-Fi
* [ ] Robot command protocol
* [ ] Telemetry
* [ ] Robot ID
* [ ] Controller communication

### Phase 8 — Autonomous Robot

* [ ] Sensor fusion
* [ ] Heading estimation
* [ ] Obstacle detection
* [ ] Navigation
* [ ] Safety state machine

### Phase 9 — Swarm

* [ ] Multiple robot communication
* [ ] Task assignment
* [ ] Robot coordination
* [ ] Collision avoidance
* [ ] Formation/collective behavior

---

# 📁 Recommended Repository Structure

```text
ARGUS/
│
├── README.md
│
├── firmware/
│   ├── tests/
│   │   ├── 01_motor_test/
│   │   ├── 02_limit_switch_test/
│   │   ├── 03_mpu6050_test/
│   │   ├── 04_ir_sensor_test/
│   │   ├── 05_ultrasonic_test/
│   │   ├── 06_buzzer_led_test/
│   │   ├── 07_electromagnet_test/
│   │   ├── 08_rfid_test/
│   │   └── 09_servo_test/
│   │
│   ├── argus_hardware/
│   │   ├── motor_control/
│   │   ├── sensors/
│   │   ├── actuators/
│   │   └── safety/
│   │
│   └── argus_main/
│
├── swarm/
│   ├── communication/
│   ├── controller/
│   ├── task_assignment/
│   └── navigation/
│
├── hardware/
│   ├── schematic/
│   ├── pcb/
│   ├── gerber/
│   └── bom/
│
├── docs/
│   ├── architecture/
│   ├── testing/
│   └── images/
│
└── LICENSE
```

---

# 🚀 Getting Started

## 1. Install Arduino IDE

Install Arduino IDE and ESP32 board support.

Select the appropriate ESP32 development board.

## 2. Connect ARGUS V1

For initial firmware development, connect the ESP32 through USB.

For motor testing, ensure:

* Motor supply is correctly connected.
* TB6612 logic supply is connected.
* ESP32 and motor-driver grounds are common.
* Wheels are lifted from the ground during initial tests.

## 3. Upload Test Firmware

Begin with:

```text
firmware/tests/
```

Run each test individually before moving to the integrated firmware.

## 4. Verify Serial Output

Default serial communication:

```text
115200 baud
```

---

# 📊 Current Development Status

**ARGUS V1 is currently in hardware bring-up and subsystem validation.**

### Confirmed

* ✅ ESP32 programming
* ✅ TB6612FNG motor control
* ✅ Motor A control
* ✅ Motor B control
* ✅ Dual-motor operation
* ✅ GPIO testing

### In Progress

* 🔄 Limit switch
* 🔄 MPU6050
* 🔄 IR sensors
* 🔄 HC-SR04
* 🔄 Actuator testing

### Planned

* ⏳ RFID
* ⏳ Servo
* ⏳ Wireless communication
* ⏳ Autonomous navigation
* ⏳ Swarm coordination

---

# 📚 Design Reference

The software architecture and autonomous/swarm-control concepts are being developed with reference to existing robotics controller architectures, including the PARROT Capstone Robot Controller project.

PARROT's project demonstrates a separation between robot firmware, controller logic, localization, trajectory control and path planning. ARGUS V1 uses this as a **reference architecture**, while the firmware and hardware implementation are being developed specifically for ARGUS V1.

---

# 🛠️ Development Philosophy

ARGUS V1 follows a simple principle:

> **Test → Validate → Integrate → Automate → Scale**

Every hardware subsystem is first tested independently.

Only after individual tests pass are the components integrated into the main robot firmware.

The final objective is to move from:

```text
Single Robot
     ↓
Autonomous Robot
     ↓
Connected Robot
     ↓
Multi-Robot System
     ↓
ARGUS Swarm
```

---

# 📈 Future Roadmap

* [ ] Complete hardware validation
* [ ] Complete sensor integration
* [ ] Implement robust motor control
* [ ] Add IMU-based heading estimation
* [ ] Add obstacle avoidance
* [ ] Add RFID task identification
* [ ] Add servo-based mechanism
* [ ] Implement robot telemetry
* [ ] Implement wireless command protocol
* [ ] Implement robot-to-robot communication
* [ ] Implement task allocation
* [ ] Implement swarm coordination
* [ ] Develop autonomous mission execution
* [ ] Build multiple ARGUS robots
* [ ] Demonstrate multi-robot cooperation

---

# 👨‍💻 Author

**NS MAYANK**

Engineering Student
ARGUS V1 — Swarm Robotics Project

GitHub:
https://github.com/NSMAYANK

Project Repository:
https://github.com/NSMAYANK/ARGUS

---

# ⚠️ Disclaimer

ARGUS V1 is an engineering development and prototype project.

The robot should be tested in a controlled environment. Motors, batteries, electromagnetic actuators and mechanical systems can create hazards during development.

Always disconnect battery power before modifying the power circuitry or motor wiring.

---

# ⭐ ARGUS V1

**Autonomous Robotics · Intelligent Sensing · Distributed Coordination · Swarm Systems**

> **One robot can perform a task.
> A swarm can coordinate the mission.**
