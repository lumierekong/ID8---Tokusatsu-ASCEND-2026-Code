# ID8---Tokusatsu-ASCEND-2026-Code
Code for ASCEND 2026, by TOKUSATSU team (ID8), please read readme.
# Satellite Telemetry & Ground Station System

## Project Overview
This system implements a LoRa communication link between a Sender (TokuSat) and a Receiver (Ground Station). It is designed for monitoring soil and road sinking and motion data using an ESP32.

### Key Features:
* **LZ4 Compression:** Compressing telemetry structs to minimize power usage and LoRa use time.
* **Deep Sleep Optimization:** Satellite node utilizes ESP32 Deep Sleep (30s cycles) with RTC memory persistence.
* **Dual-Layer Integrity:** Uses both inner (struct-level) and outer (packet-level) XOR checksums.
* **Persistent Baseline:** Automatically sets a baseline distance on first boot to calculate subsidence.

---

## Hardware Pinout (ESP32)

| Component       | Pin Assignment          | Protocol |
|-----------------|-------------------------|----------|
| **LoRa (SS)** | GPIO 18                 | SPI      |
| **LoRa (RST)** | GPIO 14                 | Reset    |
| **LoRa (DIO0)** | GPIO 26                 | Interrupt|
| **SD Card CS** | GPIO 5                  | SPI      |
| **Ultrasonic** | Trig: 16, Echo: 17      | Digital  |
| **MPU6050** | SDA: 21, SCL: 22        | I2C      |
| **Slider/Pot** | GPIO 34                 | Analog   |

---

## Technical Specifications

### Data Packet Structure
The system wraps compressed data in a 5-byte header for synchronization and validation:
1. **Sync Bytes:** `0xAB`, `0xCD`
2. **Packet Type:** `0x02` (Compressed Telemetry)
3. **Payload Length:** 2-byte integer (Little Endian)
4. **Compressed Data:** LZ4 encoded `TelemetryPacket`
5. **Checksum:** Final byte (XOR validation)

### Telemetry Struct
```cpp
struct TelemetryPacket {
  uint16_t sync;      // 0xABCD
  uint8_t packetID;   // 0x01
  uint32_t timestamp; // Seconds since wake
  float distance;     // Ultrasonic reading
  float subsidence;   // Δ from baseline
  int16_t ax, ay, az; // Accelerometer data
  uint16_t slider;    // Analog sensor value
  uint8_t checksum;   // Inner CRC
};
