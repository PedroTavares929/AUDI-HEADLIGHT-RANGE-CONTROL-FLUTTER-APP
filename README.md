# Audi A4 B6 Headlight Range Control

A Flutter mobile app for controlling Audi A4 B6 headlight range adjustment motors via Bluetooth. Communicates with an Arduino controller that interfaces with the car's CAN bus and drives stepper motors.

---

## Overview

This project gives you manual and automatic control over the vertical range of your Audi A4 B6 headlights. The Flutter app connects to an Arduino over Bluetooth Serial, sends commands, and displays real-time motor positions and system status.

**Hardware**: Arduino + DRV8825 stepper motor drivers + MCP2515 CAN bus module
**Firmware**: Three Arduino firmware versions included (`projectv1.ino`, `projectv2.ino`, `projectv3.ino`)
**App**: Flutter (Android) with Bluetooth Serial communication

---

## Features

- **Independent motor control** — move left and right headlight motors separately
- **Preset positions** — jump to min, center, or max in one tap
- **Sweep animation** — automatic full-range sweep for both motors
- **Real-time status** — live motor positions (0–320 steps), headlight state, CAN bus status
- **CAN bus detection** — automatically detects when headlights are switched on/off
- **Persistent config** — settings saved to EEPROM (default positions, step size, animation speed, motor timeout)
- **Responsive UI** — phone and tablet layouts, light/dark/system themes

---

## Demo

<video src="https://github.com/user-attachments/assets/12023a7f-92c1-4fa7-9dd1-cb1981feacdc" controls width="100%"></video>

---

## Hardware Setup

### Components
| Component | Purpose |
|-----------|---------|
| ESP32 | Main controller with built-in Bluetooth Serial |
| MCP2515 | SPI CAN bus interface module |
| DRV8825 x2 | Stepper motor drivers (left & right headlight) |

### Pin Wiring
| Pin | Function |
|-----|----------|
| 2 | Motor enable (LOW = enabled) |
| 4 | Left motor DIR |
| 5 | Left motor STEP |
| 16 | Right motor STEP |
| 17 | Right motor DIR |
| 21 | CAN TX (to MCP2515) |
| 22 | CAN RX (from MCP2515) |

### Audi A4 B6 CAN Bus — Headlight Detection

The system listens on the Audi A4 B6 (2001–2004) body CAN bus for the instrument cluster lighting message:

| Parameter | Value |
|-----------|-------|
| CAN ID | `0x531` |
| Byte index | `0` |
| Lights ON value | `0x04` |
| Lights OFF value | `0x03` |

When `byte[0]` of frame `0x531` changes to `0x04`, the ESP32 moves both motors to the configured default position. When it changes to `0x03` (lights off), motors return to minimum.

### Motor Range
- Min: `0` steps
- Center: `160` steps
- Max: `320` steps

---

## Firmware

Three versions are included in the root of the repository:

| File | Description |
|------|-------------|
| `projectv1.ino` | Full-featured (WiFi, WebServer, Bluetooth, EEPROM) |
| `projectv2.ino` | Simplified, lighter weight |
| `projectv3.ino` | Latest optimized version |

Flash the version that matches your hardware setup using the Arduino IDE.

**Required Arduino/ESP32 libraries:**
- `ESP32-TWAI-CAN` — ESP32 CAN bus via MCP2515
- `ArduinoJson` — JSON command parsing
- `EEPROM` — persistent config/position storage
- `BluetoothSerial` — built-in ESP32 Bluetooth (no external module needed)

---

## Communication Protocol

The app communicates over Bluetooth Serial using newline-delimited JSON.

### Commands (App → Arduino)
```json
{"action": "status"}
{"action": "config_get"}
{"action": "animate"}
{"action": "center"}
{"action": "left_move", "direction": 1, "steps": 10}
{"action": "right_move", "direction": -1, "steps": 10}
{"action": "stop_motors"}
{"action": "config_set", "centerPosition": 160, "animationSpeed": 3}
```

### Status Response (Arduino → App)
```json
{
  "type": "status",
  "leftPosition": 160,
  "rightPosition": 160,
  "headlightsOn": true,
  "motorsEnabled": true,
  "isAnimating": false,
  "leftMoving": false,
  "rightMoving": false,
  "canInitialized": true,
  "frameCount": 1042
}
```

---

## App Setup

### Prerequisites
- Flutter SDK (stable channel)
- Android device with Bluetooth
- HC-05/HC-06 paired to phone as `HeadlightController`

### Install & Run
```bash
git clone <repo-url>
cd AUDI-HEADLIGHT-RANGE-CONTROL-FLUTTER-APP
flutter pub get
flutter run
```

### Build APK
```bash
flutter build apk --release
```

### Dependencies
```yaml
flutter_bluetooth_serial: 0.4.0
permission_handler: ^11.3.1
```

> **Note:** `flutter_bluetooth_serial` is pinned to exactly `0.4.0` to avoid known issues in later versions.

---

## App Usage

1. **Pair your HC-05/HC-06** Bluetooth module with your phone and name it `HeadlightController` (auto-detected) or select it manually.
2. Open the app — it scans bonded devices and attempts auto-connect.
3. Once connected, the **Control Page** shows:
   - Live motor positions with visual bars
   - Headlight and CAN bus status
   - UP/DOWN controls for each motor
   - ANIMATE and CENTER quick actions
4. Tap the settings icon to adjust: default position, step size, animation speed, motor timeout.

---

## Project Structure

```
├── lib/
│   ├── main.dart              # App entry point, theme management
│   ├── ConnectionPage.dart    # Bluetooth scan & connect UI
│   ├── ControlPage.dart       # Motor control interface
│   └── theme_constants.dart   # AppThemeMode enum
├── projectv1.ino              # Arduino firmware v1
├── projectv2.ino              # Arduino firmware v2
├── projectv3.ino              # Arduino firmware v3
└── pubspec.yaml
```

---

## Configuration Parameters

Adjustable at runtime via the in-app settings dialog and persisted to EEPROM:

| Parameter | Default | Description |
|-----------|---------|-------------|
| Center Position | 160 | Default motor position (steps) |
| Max Position | 320 | Upper limit |
| Min Position | 0 | Lower limit |
| Animation Speed | 3 ms | Delay between steps during sweep |
| Motor Timeout | 2000 ms | Delay before disabling motors after movement |
| Step Size | configurable | Steps per button press |
| Auto Center | true | Return to center on headlight off |

---

## License

MIT
