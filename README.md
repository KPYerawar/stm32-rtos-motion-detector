# Real-Time Motion Detection & Response System

A multi-tasking, event-driven embedded system designed to detect and classify motion states (Stable, Tilted, Shock/Vibration) in real-time. The project is implemented on an **STM32F410RBTx (ARM Cortex-M4)** microcontroller using **FreeRTOS** via the **CMSIS-RTOS v2 API** to ensure deterministic execution and precise task synchronization.

## 🛠️ Hardware Stack
* **MCU:** STM32F410RBTx (ARM Cortex-M4 @ 100 MHz, 128 KB Flash)
* **Sensor:** MPU6050 6-axis IMU (Accelerometer + Gyroscope)
* **Protocol:** I2C (for sensor data acquisition) & UART (for terminal logging)
* **Actuator:** Onboard LED (PA5)

## 💻 Software & Architecture
* **IDE:** STM32CubeIDE
* **Configuration:** STM32CubeMX
* **RTOS Kernel:** FreeRTOS (CMSIS-RTOS v2 Wrapper)

### Thread & Task Design
The application is split into three concurrent tasks with distinct preemptive priorities to isolate concerns cleanly:

1.  **SensorTask (`osPriorityHigh`):** Periodically wakes up every 25 ms (40 Hz sampling rate) to read raw accelerometer X/Y/Z data from the MPU6050 over I2C, packages it into a custom structure, and pushes it into a message queue.
2.  **LogicTask (`osPriorityNormal`):** Dequeues the structure, processes the acceleration vectors using an exponential low-pass filter ($\alpha = 0.15$), and determines the physical state. If a tilt or sudden shock threshold is violated, it releases a binary semaphore and prints status packets over UART.
3.  **ControlTask (`osPriorityBelowNormal`):** Blocks on the binary semaphore. Upon acquisition (triggered by a motion event), it drives the onboard LED high for a non-blocking 200 ms duration before returning to a blocked state.

---
---

## 📈 Inter-Task Communication Primitives
* **`osMessageQueue`:** Safely streams data structures across thread boundaries from `SensorTask` to `LogicTask` without data corruption.
* **`osSemaphore` (Binary):** Handshakes `LogicTask` and `ControlTask` asynchronously, eliminating polling overhead and maintaining low power.
* **`osDelay`:** Provides non-blocking execution intervals, allowing lower priority threads to utilize CPU time efficiently.

## 🚀 How to Run

### 1. Hardware Connections
| MPU6050 Pin | STM32F410 Pin |
|:---:|:---:|
| VCC | 3.3V |
| GND | GND |
| SCL | I2C1_SCL |
| SDA | I2C1_SDA |

### 2. Verification
* Clone this repository into your STM32CubeIDE workspace.
* Build and flash the project onto your board.
* Open a serial terminal (e.g., PuTTY, Minicom, or CuteCOM) configured to **115200 baud, 8-N-1** on your serial port (`/dev/ttyACM0` or `COMx`).
* Observe real-time telemetry filters and state evaluations.
