# Self-Balancing Robot & TinyML Gesture Controller

## Architecture Overview
This engineering-grade repository integrates a **FreeRTOS Self-Balancing Robot** with a **TinyML Wearable Gesture Controller** using ESP-NOW wireless transmission. It includes real-time PID tuning via WebSocket dashboard and sub-50ms embedded ML gesture inference.

## Repository Structure
```text
├── /robot_firmware        # FreeRTOS-based PID balancing and WebSockets dashboard code
├── /wearable_firmware     # TinyML inference pipeline (TFLite Micro) and ESP-NOW integration
├── /hardware              # Onshape exported STLs, Wiring Schematics, and v5 BOM
└── /docs                  # Build logs, verification plans, and OTA checklists
```

## FreeRTOS Task Architecture
The firmware utilizes deterministic FreeRTOS multi-tasking to ensure sensor read stability and balance recovery without CPU starvation.

### Priority Table
| Task          | Rate/Trigger | Core Function |
|---------------|--------------|---------------|
| `sensor_task` | 500Hz        | Interfacing with MPU6050 & Complementary Filter execution for tilt angle. |
| `pid_task`    | 200Hz        | Calculating PID response and modulating L298N PWM (5kHz: `MOTOR_PWM_FREQ_HZ 5000`). |
| `wifi_task`   | Asynchronous | Hosting the ESPAsyncWebServer and real-time WebSocket dashboard for PID adjustments. |
| `oled_task`   | 10Hz         | Updating the SSD1306 OLED interface displaying angles and connection state. |
| `ota_task`    | Asynchronous | Listening for Over-The-Air updates with motor-safe flash guards. |

## Mandatory Engineering & Safety Specifications
The following hardware standards must be maintained over the lifecycle of this repository to prevent device failure and thermal risk. **(Ref: Engineering Review v5)**

### 1. E-07 Power Gate (Switching)
The primary slide switch on the wearable LiPo circuit **MUST** be rated for ≥1A continuous load (e.g., `MK-12C02`). The standard kit `SS-12D00` slide switch is only rated for 100-300mA and will **arc and fail** under the 815mA peak current of the ESP32.

### 2. Charging Constraints (TP4056)
The TP4056 modules driving the 500mAh LiPo must run the DW01 protection variant. Before soldering, verify that the **PROG Resistor is swapped to 3kΩ**. This forces charging to cap at 400mA (0.8C), preventing rapid thermal runaway of the WLY752530 cell.

### 3. I2C Bus Safety Mutex
FreeRTOS context-switching mid I2C-transfer corrupts the state machine and stalls the robot. Both ESP32s rely on an I2C Mutex implementation:
```c
SemaphoreHandle_t i2c_mutex = xSemaphoreCreateMutex();
// Must wrap EVERY I2C read/write:
if (xSemaphoreTake(i2c_mutex, portMAX_DELAY)) {
    // Perform Transfer
    xSemaphoreGive(i2c_mutex);
}
```

### 4. TinyML Tensor Specs
The wearable leverages TensorFlow Lite Micro for gesture inference. Model specs:
- **INT8 Quantization** is strictly enforced.
- Tensor workspace defined: `#define TFLITE_ARENA_SIZE (96 * 1024)`.
- Must assert `AllocateTensors() == kTfLiteOk` on boot to prevent hard-faults.
- Inference task requires a 16KB stack allocation in `xTaskCreate`.

### 5. OTA Lockouts
OTA flashes during active balancing **WILL** drop the robot. The firmware must disable `MOTOR_PWM` and halt the `pid_task` upon receiving the first OTA callback ping.
