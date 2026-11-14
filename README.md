# 🌱 Smart Agri Spray Robot

Wi-Fi smart farming robot on ESP32: drive it from any phone, and spray crops at 3 intensity levels with LED and LCD feedback. No app needed.

![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-ESP32-blue)
![Framework](https://img.shields.io/badge/framework-Arduino-teal)
![Build](https://img.shields.io/badge/build-PlatformIO-orange)

> 📹 **Demo video / GIF goes here** (robot driving, then spraying with the LED changing color)

---

## Features

- Built-in web dashboard served by the ESP32, usable from any phone or laptop browser
- Drive control: forward, backward, left, right, stop
- 3 spray levels with matching LED indicators:

| Level | LED | Pump time |
|---|---|---|
| Normal | 🟢 Green | 2 seconds |
| Moderate | 🟡 Yellow | 4 seconds |
| High | 🔴 Red | 6 seconds |

- 16x2 I2C LCD status display (address auto-detected by an I2C scan)
- Non-blocking spray timer using `millis()`
- Motors and pump forced OFF at boot for safety

> **Note:** the LCD messages "LEAF DETECTED" and "PHOTO CAPTURED" are a presentation sequence. The spray level is chosen by the button pressed on the web page. Real leaf-health detection (Raspberry Pi camera) is on the roadmap.

---

## Hardware

| Part | Qty | Notes |
|---|---|---|
| ESP32 development board | 1 | 30-pin DevKit |
| Dual H-bridge motor driver (L298N or similar) | 3 | 2 for drive motors, 1 for the pump |
| DC gear motors | 4 | 2 per side |
| Small DC water pump | 1 | 5-12 V, matching your supply |
| 16x2 LCD with I2C backpack | 1 | Usually address 0x27 or 0x3F |
| LEDs (red, yellow, green) | 3 | |
| 220 Ω resistors | 3 | One per LED |
| Battery pack | 1 | Sized for motors and pump |
| Chassis, wheels, tubing, nozzle, water tank | | |
| Jumper wires | | |

---

## Wiring / Connections

### Left motor driver (Driver 1)

| Driver pin | ESP32 GPIO | Function |
|---|---|---|
| IN1 | 23 | Left motor A, direction |
| IN2 | 19 | Left motor A, direction |
| IN3 | 13 | Left motor B, direction |
| IN4 | 27 | Left motor B, direction |

### Right motor driver (Driver 2)

| Driver pin | ESP32 GPIO | Function |
|---|---|---|
| IN1 | 25 | Right motor A, direction |
| IN2 | 26 | Right motor A, direction |
| IN3 | 32 | Right motor B, direction |
| IN4 | 33 | Right motor B, direction |

### Pump motor driver (Driver 3)

| Driver pin | ESP32 GPIO | Function |
|---|---|---|
| IN3 | 18 | Pump control |
| IN4 | 17 | Pump control (kept LOW) |

The pump connects to the **OUT3 / OUT4** terminals of this driver.

### Spray indicator LEDs

| LED | ESP32 GPIO | Wiring |
|---|---|---|
| Red (HIGH spray) | 4 | GPIO → 220 Ω → LED anode, cathode → GND |
| Yellow (MODERATE spray) | 2 | GPIO → 220 Ω → LED anode, cathode → GND |
| Green (NORMAL spray) | 15 | GPIO → 220 Ω → LED anode, cathode → GND |

### I2C LCD (16x2)

| LCD pin | ESP32 | 
|---|---|
| SDA | GPIO 21 |
| SCL | GPIO 22 |
| VCC | 5 V (VIN) |
| GND | GND |

### Power

| Connection | Details |
|---|---|
| Motor drivers' +12 V / VS input | Battery positive |
| Motor drivers' GND | Battery negative **and** ESP32 GND (common ground is required) |
| ESP32 | Powered by USB, or from the driver's 5 V output into VIN |
| Driver ENA / ENB pins | Leave the jumper caps on (always enabled), or tie to 5 V. The code uses no PWM speed control. |

### Wiring overview

```
                    ┌───────────────────────┐
   Phone/Browser ──►│         ESP32         │
      (Wi-Fi)       │                       │
                    │ 23,19,13,27 ──► LEFT driver  ──► Left motors
                    │ 25,26,32,33 ──► RIGHT driver ──► Right motors
                    │ 18,17       ──► PUMP driver  ──► Water pump
                    │ 4  ──► Red LED    (HIGH)
                    │ 2  ──► Yellow LED (MODERATE)
                    │ 15 ──► Green LED  (NORMAL)
                    │ 21 (SDA), 22 (SCL) ──► I2C LCD
                    └───────────────────────┘
        All grounds (battery, drivers, ESP32) must be connected together.
```

### Wiring notes

- **Common ground** between the battery, all drivers, and the ESP32 is mandatory, or the motors will behave randomly.
- **Power the pump and motors from the battery**, never from the ESP32 3.3 V pin.
- GPIO **2, 4, 12, 15** are ESP32 strapping pins. LEDs with resistors are normally fine, but if the board fails to boot, disconnect the LEDs on GPIO 2 and 15 while uploading.
- If a motor spins the wrong way, swap that motor's two wires at the driver output.

---

## Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/YOUR-USERNAME/smart-agri-spray-robot.git
cd smart-agri-spray-robot
```

### 2. Set your Wi-Fi

Copy `include/secrets.example.h` to `include/secrets.h` and edit it:

```cpp
#define WIFI_SSID "your-wifi-name"
#define WIFI_PASSWORD "your-wifi-password"
```

`secrets.h` is listed in `.gitignore`, so your password is never uploaded.

### 3. Build and upload

With **PlatformIO**:

```bash
pio run --target upload
pio device monitor
```

With **Arduino IDE**: install the *ESP32* board package and the *LiquidCrystal_I2C* library, select your ESP32 board, then upload.

### 4. Open the dashboard

1. Open the Serial Monitor at **115200 baud**.
2. Wait for `Wi-Fi CONNECTED!` and note the printed **IP address**.
3. On a phone or laptop on the **same Wi-Fi**, open `http://<that-ip-address>/`.

---

## Web Endpoints

| URL | Action |
|---|---|
| `/` | Dashboard |
| `/forward` `/backward` `/left` `/right` `/stop` | Drive control |
| `/spray_normal` | Green LED, pump 2 s |
| `/spray_moderate` | Yellow LED, pump 4 s |
| `/spray_high` | Red LED, pump 6 s |
| `/spray_stop` | Stop spraying |
| `/test` | Check that the server responds |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| LCD stays blank | Check SDA/SCL wiring. The Serial Monitor shows the I2C scan result. Adjust the contrast screw on the backpack. |
| `Wi-Fi CONNECTION FAILED` | Check SSID and password. The ESP32 supports 2.4 GHz only, not 5 GHz. |
| Motors don't move | Check battery power, common ground, and the ENA/ENB jumpers. |
| Pump doesn't run | Check the pump driver's power and the OUT3/OUT4 connections. |
| Upload fails or board won't boot | Unplug the LEDs on GPIO 2 and 15 during upload. |

---

## Roadmap

- [ ] Real leaf-health detection with a Raspberry Pi camera
- [ ] Non-blocking spray state machine (web server stays responsive during spraying)
- [ ] Wi-Fi loss failsafe (auto-stop motors)
- [ ] Wi-Fi auto-reconnect
- [ ] PWM speed control
- [ ] Autonomous row-following mode

---

## Contributing

Issues and pull requests are welcome. If you build one, share a photo in Discussions.

## License

MIT License. See [LICENSE](LICENSE).

## Author

Built by **YOUR NAME**. Feel free to connect on [LinkedIn](https://linkedin.com/in/your-profile).
