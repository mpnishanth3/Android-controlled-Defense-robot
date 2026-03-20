# 🤖 Arduino Line-Following & Obstacle-Avoiding Robot Car (with Bluetooth)

An Arduino-based 4-wheel drive robot car that operates in two modes:
- **Auto Mode** — follows a line using IR sensors and stops for obstacles via ultrasonic sensors
- **Manual Mode** — full directional control from an Android device over Bluetooth (HC-05)

Both modes respect obstacle detection — the robot will not drive into an object even when commanded via Bluetooth.

---

## 📋 Table of Contents

- [Features](#features)
- [Hardware Requirements](#hardware-requirements)
- [Pin Configuration](#pin-configuration)
- [Circuit Diagram Overview](#circuit-diagram-overview)
- [HC-05 Bluetooth Wiring](#hc-05-bluetooth-wiring)
- [Software Requirements](#software-requirements)
- [Installation & Upload](#installation--upload)
- [Control Modes](#control-modes)
- [Bluetooth Command Reference](#bluetooth-command-reference)
- [Android App Setup](#android-app-setup)
- [How Auto Mode Works](#how-auto-mode-works)
- [Function Reference](#function-reference)
- [Known Limitations](#known-limitations)
- [Future Improvements](#future-improvements)

---

## ✨ Features

- **Line Following** — 4 IR sensors (front-left, front-right, back-left, back-right) detect and follow a line in both forward and backward directions
- **Obstacle Avoidance** — dual ultrasonic sensors (front and back) halt the robot if an obstacle is within 50 cm — active in **both** Auto and Bluetooth modes
- **Bluetooth Manual Control** — drive forward, backward, left, right, and stop from any Android app that supports serial Bluetooth
- **Adjustable Motor Speed** — send digits `0`–`9` over Bluetooth to set speed from low (80/255) to full (255/255)
- **Dual Control Source** — seamlessly switch between Auto (IR) and Manual (Bluetooth) modes at runtime
- **Physical Switch Override** — the two push-buttons always take precedence and return the robot to Auto mode
- **Serial Feedback** — the robot echoes status strings back over Bluetooth (e.g. `MOVE:FWD`, `BLOCKED:FWD`, `SPEED:200`)

---

## 🛒 Hardware Requirements

| Component | Quantity |
|---|---|
| Arduino Uno (or compatible) | 1 |
| Adafruit Motor Shield V1 | 1 |
| DC Motors (for 4WD chassis) | 4 |
| IR Obstacle / Line Sensors | 4 |
| HC-SR04 Ultrasonic Sensors | 2 |
| HC-05 Bluetooth Module | 1 |
| Push-button Switches | 2 |
| 4WD Robot Car Chassis | 1 |
| 1 kΩ & 2 kΩ resistors (voltage divider for HC-05 RX) | 1 each |
| Power Supply (7–12 V) | 1 |
| Jumper Wires | As needed |

---

## 📌 Pin Configuration

### IR Sensors

| Sensor | Arduino Pin |
|---|---|
| Forward Left IR | A0 |
| Forward Right IR | A1 |
| Backward Left IR | A2 |
| Backward Right IR | A3 |

### Push-button Switches (INPUT_PULLUP — pressed = LOW)

| Switch | Arduino Pin |
|---|---|
| Forward Mode Switch | A4 |
| Backward Mode Switch | A5 |

### Ultrasonic Sensors

| Signal | Arduino Pin |
|---|---|
| Forward US Trigger | 9 |
| Forward US Echo | 10 |
| Backward US Trigger | 8 |
| Backward US Echo | 13 |

### HC-05 Bluetooth Module

| HC-05 Pin | Arduino Pin | Notes |
|---|---|---|
| VCC | 5 V | |
| GND | GND | |
| TX | D2 | SoftwareSerial RX |
| RX | D3 | Via 1 kΩ / 2 kΩ voltage divider — see below |

### Motors (via Adafruit Motor Shield)

| Motor | Shield Port |
|---|---|
| motor1 (Left Front) | M3 |
| motor2 (Right Front) | M4 |
| motor3 (Left Rear) | M1 |
| motor4 (Right Rear) | M2 |

---

## 🔌 Circuit Diagram Overview

```
Arduino + Motor Shield
    ├── IR Sensors         → A0, A1, A2, A3
    ├── Push Buttons       → A4 (Forward), A5 (Backward)  [INPUT_PULLUP]
    ├── Front HC-SR04      → Trig: D9,  Echo: D10
    ├── Back HC-SR04       → Trig: D8,  Echo: D13
    ├── HC-05 Bluetooth    → TX→D2,  RX→D3 (via divider)
    └── DC Motors          → Motor Shield M1–M4
```

---

## 📡 HC-05 Bluetooth Wiring

The HC-05 RX pin operates at 3.3 V logic. Connecting it directly to the Arduino's 5 V TX pin will damage the module over time. Use a simple resistor voltage divider:

```
Arduino D3 (TX) ──┤ 1 kΩ ├──┬── HC-05 RX
                             │
                          2 kΩ
                             │
                            GND
```

This divides 5 V down to ~3.3 V, which is safe for the HC-05 RX pin.  
The HC-05 TX → Arduino D2 connection requires no divider (HC-05 outputs 3.3 V, which Arduino reads as HIGH).

> **Default HC-05 baud rate:** 9600. If yours is different, update `BTSerial.begin(9600)` in the sketch to match.

---

## 💻 Software Requirements

- [Arduino IDE](https://www.arduino.cc/en/software) 1.8.x or later
- [Adafruit Motor Shield Library V1](https://github.com/adafruit/Adafruit-Motor-Shield-library)
- `SoftwareSerial` — built into the Arduino IDE (no separate install needed)

### Installing the Motor Shield Library

1. Open Arduino IDE
2. Go to **Sketch → Include Library → Manage Libraries**
3. Search for **"Adafruit Motor Shield"** and install the **V1** version

---

## 🚀 Installation & Upload

1. Clone this repository:
   ```bash
   git clone https://github.com/your-username/your-repo-name.git
   ```

2. Open `robot_car_bluetooth.ino` in the Arduino IDE.

3. **Disconnect the HC-05 TX/RX wires** before uploading — `SoftwareSerial` shares pins with the programmer and will cause upload failures if the module is connected.

4. Select the correct board and port under **Tools → Board** and **Tools → Port**.

5. Click **Upload**, then reconnect the HC-05 wires.

6. Open **Serial Monitor** (9600 baud) to confirm `Robot ready. Waiting for commands...`

---

## 🕹️ Control Modes

| Mode | How to activate | Behaviour |
|---|---|---|
| **Auto** | Press the Forward or Backward physical switch | IR line following with ultrasonic obstacle avoidance |
| **Manual (Bluetooth)** | Send `M` from the Android app | Full directional control via Bluetooth commands |

Pressing a physical switch while in Bluetooth mode immediately returns the robot to Auto mode. Obstacle avoidance is always active — sending `F` or `B` over Bluetooth will stop the robot if an obstacle is detected within 50 cm and reply with `BLOCKED:FWD` or `BLOCKED:BWD`.

---

## 📟 Bluetooth Command Reference

| Command (ASCII char) | Action |
|---|---|
| `F` | Move forward |
| `B` | Move backward |
| `L` | Turn left (spin in place) |
| `R` | Turn right (spin in place) |
| `S` | Stop all motors |
| `0` – `9` | Set speed (0 = 80/255 … 9 = 255/255) |
| `M` | Switch to Manual (Bluetooth) mode |
| `A` | Switch to Auto (IR) mode |

### Feedback strings sent back to Android

| String | Meaning |
|---|---|
| `MOVE:FWD` | Moving forward |
| `MOVE:BWD` | Moving backward |
| `MOVE:LEFT` | Turning left |
| `MOVE:RIGHT` | Turning right |
| `MOVE:STOP` | Motors stopped |
| `BLOCKED:FWD` | Obstacle detected — forward move cancelled |
| `BLOCKED:BWD` | Obstacle detected — backward move cancelled |
| `SPEED:<value>` | Current speed value acknowledged |
| `MODE:AUTO` | Switched to Auto mode |
| `MODE:MANUAL` | Switched to Manual mode |
| `ERR:UNKNOWN_CMD` | Unrecognised command received |

---

## 📱 Android App Setup

Any Android Bluetooth terminal app that can send single ASCII characters will work. Recommended options:

| App | Notes |
|---|---|
| **Bluetooth RC Controller** (Play Store) | Pre-mapped joystick buttons — configure to send `F`, `B`, `L`, `R`, `S` |
| **Serial Bluetooth Terminal** (Play Store) | Manual text input; good for testing individual commands |
| **MIT App Inventor** | Build a custom controller with your own button layout |

### Pairing the HC-05

1. Power on the robot — the HC-05 LED will blink rapidly.
2. On your Android device go to **Settings → Bluetooth** and scan for devices.
3. Select **HC-05** and enter the pairing PIN: `1234` (default).
4. The LED will blink slowly once paired.
5. Open your Bluetooth app, connect to HC-05, and send commands.

---

## ⚙️ How Auto Mode Works

### Line Following Logic (Forward)

| Front-Left IR | Front-Right IR | Action |
|---|---|---|
| HIGH | HIGH | Stop (both sensors detect edge / obstacle) |
| HIGH | LOW | Correct right → left motors reverse, right forward |
| LOW | HIGH | Correct left → left motors forward, right reverse |
| LOW | LOW | Drive straight forward |

The same mirrored logic applies in backward mode using the rear IR sensors.

### Obstacle Avoidance

Before executing any movement (in both Auto and Bluetooth modes), `checkObstacle()` fires the appropriate ultrasonic sensor. If the measured distance is less than 50 cm, all motors are released.

---

## 📖 Function Reference

### `void handleBluetooth(char cmd)`
Parses a single ASCII character from the HC-05 and executes the corresponding motor command or mode switch. Sends a feedback string back over `BTSerial`.

### `void runAutoMode()`
Reads all four IR sensors and both ultrasonic sensors, then drives the motors according to the line-following logic for the active direction.

### `void setMotorDirection(int leftDirection, int rightDirection)`
Sets all four motors to `motorSpeed` with the specified directions (`FORWARD`, `BACKWARD`, or `RELEASE`).

### `void stopMotors()`
Releases all four motors immediately.

### `bool checkObstacle(int trigPin, int echoPin)`
Fires an ultrasonic pulse and returns `true` if an obstacle is detected within 50 cm.

### `long microsecondsToCentimeters(long microseconds)`
Converts a raw `pulseIn` duration to centimetres.

---

## ⚠️ Known Limitations

- `SoftwareSerial` on pins D2/D3 cannot receive data while the motor shield is actively switching (PWM interference may cause occasional dropped bytes).
- Motor speed is controlled globally — no per-side speed tuning.
- The 50 cm obstacle threshold is fixed; adjust `cm < 50` in `checkObstacle()` as needed.
- Only one Bluetooth client can connect at a time (HC-05 limitation).

---

## 🔮 Future Improvements

- [ ] Replace HC-05 with HC-06 (simpler slave-only module) or ESP32 for Wi-Fi control
- [ ] Custom MIT App Inventor Android UI with live sensor readback
- [ ] PID control for smoother line following
- [ ] Dynamic obstacle threshold configurable via Bluetooth
- [ ] Battery voltage monitoring sent to Android app
- [ ] OLED display showing current mode and speed

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).

---

## 🙌 Acknowledgements

- [Adafruit Industries](https://www.adafruit.com/) for the Motor Shield library
- Arduino community for `SoftwareSerial` documentation and examples
