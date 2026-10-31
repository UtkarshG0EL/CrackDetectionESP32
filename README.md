# Crack Detection Drone Using ESP32-CAM & ML-Based Vision System

An ultra-low-cost, decentralized Unmanned Aerial Vehicle (UAV) vision system designed for automated structural health monitoring and non-destructive testing (NDT) of civil infrastructure such as buildings, bridges, flyovers, and road surfaces. 

By pairing a lightweight **ESP32-CAM** wireless video acquisition payload on an aerodynamically optimized drone chassis with a **browser-based WebGL Machine Learning pipeline**, this system performs real-time surface crack detection at $\sim 22$ FPS with zero cloud latency.

---

## Table of Contents
1. [Overview & Problem Statement](#overview--problem-statement)
2. [Key Features](#key-features)
3. [System Architecture](#system-architecture)
4. [Hardware Specifications](#hardware-specifications)
5. [CAD Design & CFD Analysis](#cad-design--cfd-analysis)
6. [Software & Environment Requirements](#software--environment-requirements)
7. [Installation & Execution Guide](#installation--execution-guide)
8. [Experimental Results & Performance](#experimental-results--performance)
9. [Real-World Applications](#real-world-applications)
10. [Scope of Future Work](#scope-of-future-work)
11. [Project Team & Guidance](#project-team--guidance)

---

## Overview & Problem Statement
Traditional structural inspection relies heavily on manual visual checks using scaffolding, heavy machinery, or industrial rope access. These methods expose engineers to severe occupational hazards, suffer from human subjectivity and fatigue, and incur massive indirect economic costs due to traffic closures and facility downtime.

Conversely, high-end commercial inspection UAVs equipped with heavy onboard GPUs or continuous high-bandwidth cloud dependencies are prohibitively expensive for routine, scalable deployment. 

**The Solution:** This project bridges the gap by splitting the computational workload. The drone acts strictly as an ultra-lightweight wireless video broadcaster using an **ESP32-CAM** and **OV2640** sensor, while heavy Convolutional Neural Network (CNN) inference is offloaded over a local Wi-Fi network to a ground-station web browser utilizing **WebGL** and **HTML5 Web Workers**.

---

## Key Features
* **Decentralized Edge AI:** Offloads deep learning computations from the resource-constrained microcontroller to the ground-station GPU via a browser interface.
* **Zero Cloud Latency:** Downloads the trained CNN model (`crack-detection-kjvhd`, v3) locally into the browser via the Roboflow `inference.js` SDK, eliminating REST API round-trip delays.
* **HTML5 Canvas Bridge & `CVImage` Wrapper:** Continuously paints incoming MJPEG stream frames onto an HTML5 `<canvas>` and converts raw pixels into transferable array buffers to run inside background Web Workers without triggering `DataCloneError` exceptions or freezing the UI.
* **Aerodynamic & Mechanical Optimization:** Custom-modeled CAD payload mount featuring anti-vibration rubber dampers to prevent rolling-shutter artifacts from the $1000\text{ KV}$ BLDC motors, validated via SolidWorks Flow Simulation (CFD).
* **Extended Flight Endurance:** Stripping onboard ML inference from the ESP32-CAM reduces active payload power draw to $\sim 0.60\text{ W}$, preventing thermal throttling and preserving battery life.

---

## System Architecture

```text
+-----------------------+       +------------------------+
|     OV2640 Sensor     |       |      Power Module      |
|    (Visual Input)     |       |   (LiPo + Buck 5V)     |
+-----------+-----------+       +-----------+------------+
            |                               |
            +---------------+---------------+
                            |
                            v
            +-------------------------------+
            |           ESP32-CAM           |
            |   (Wireless Video Streamer)   |
            +---------------+---------------+
                            |
                            | Wi-Fi MJPEG Stream (http://192.168.137.121/stream)
                            v
            +-------------------------------+
            |         Ground Station        |
            | (Local Python HTTP Server)    |
            +---------------+---------------+
                            |
                            v
            +-------------------------------+
            |   Browser Inference Pipeline  |
            |  Hidden <img> -> <canvas> ->  |
            |  CVImage -> Web Worker (GPU)  |
            +---------------+---------------+
                            |
                            v
            +-------------------------------+
            |   Real-Time UI Bounding Box   |
            |       Overlay Rendering       |
            +-------------------------------+
```

---

## Hardware Specifications

### 1. Microcontroller (ESP32-CAM)
Programmed via an **FT232RL FTDI USB-to-TTL Serial Adapter**.

| Parameter | Specification |
| :--- | :--- |
| **Processor Core** | Tensilica LX6 dual-core 32-bit |
| **Clock Frequency** | Up to $240\text{ MHz}$ |
| **Internal SRAM** | $520\text{ KB}$ |
| **External PSRAM** | $4\text{ MB}$ (Used for high-res frame buffering) |
| **Wireless Connectivity** | $802.11\text{ b/g/n}$ Wi-Fi, Bluetooth 4.2 / BLE |
| **Storage** | MicroSD Card Slot (up to $4\text{ GB}$) |
| **Operating Voltage** | $3.3\text{ V}$ (powered via $5\text{ V}$ input pin) |

#### ESP32-CAM Pin Configuration
| Pin Function | ESP32-CAM Pin | Description |
| :--- | :--- | :--- |
| **Power Supply** | `5V`, `GND` | Main power from voltage regulator |
| **UART Transmit** | `UOT` (`GPIO 1`) | Serial TX for firmware flashing/debugging |
| **UART Receive** | `UOR` (`GPIO 3`) | Serial RX for firmware flashing/debugging |
| **Flash Mode** | `GPIO 0` | Pulled to `GND` during firmware upload |
| **Onboard Flash LED** | `GPIO 4` | High-power white LED for dark environments |
| **Indicator LED** | `GPIO 33` | Built-in red LED for status signaling |

### 2. Camera Sensor (OV2640)
| Feature | Specification |
| :--- | :--- |
| **Active Array Size** | $1622 \times 1200$ ($2\text{ Megapixels}$) |
| **Lens Size** | $\frac{1}{4}\text{ inch}$ |
| **Output Formats** | YUV422, RGB565, JPEG compressed |
| **Maximum Frame Rate** | UXGA/SXGA @ $15\text{ fps}$, SVGA @ $30\text{ fps}$ |
| **Sensitivity** | $0.6\text{ V/Lux-sec}$ |
| **Power Consumption** | $125\text{ mW}$ (Active), $600\ \mu\text{W}$ (Standby) |

### 3. Unmanned Aerial Vehicle (UAV) Platform
| Feature | Specification |
| :--- | :--- |
| **Base Platform** | Primus X5 / Custom Multirotor Chassis |
| **Dimensions** | $16 \times 16\text{ cm}$ |
| **Total Weight** | $60\text{ g}$ |
| **Max Speed** | $3\text{ m/s}$ |
| **Max Range** | $60\text{ m}$ |
| **Flight Time** | $8+\text{ minutes}$ |
| **Battery** | $600\text{ mAh}$ Lithium Polymer (LiPo), $25\text{C}$ Discharge |

---

## CAD Design & CFD Analysis
The drone chassis and specialized ESP32-CAM payload cradle were modeled using **Dassault Systèmes SOLIDWORKS Premium 2023 SP0.1**. 
* **Mechanical Dampening:** The camera cradle is positioned centrally beneath the frame to optimize the center of gravity and integrates anti-vibration rubber dampers to isolate the CMOS sensor from harmonic oscillations.
* **Aerodynamic Profiling:** Computational Fluid Dynamics (CFD) simulations were conducted using the **SolidWorks Flow Simulation** add-in to evaluate velocity ($\text{m/s}$), static pressure ($\text{Pa}$), and fluid temperature ($\text{K}$) across the canopy and propeller blades along the $Z$-axis.
* **Surface Force Results:**
  * **Normal Force:** $31.375\text{ N}$
  * **Resultant Force:** $31.181\text{ N}$

---

## Software & Environment Requirements

### Embedded Development (Drone Node)
* **OS:** FreeRTOS (built into ESP32 Arduino Core)
* **IDE:** Visual Studio Code (v1.95+) with **PlatformIO IDE** extension (Core 6.1.x)
* **Framework:** Arduino Core for ESP32 (v2.0.14+)
* **Libraries:** `esp_camera.h`, `WiFi.h`

### Ground Station (Frontend & ML Engine)
* **OS:** Microsoft Windows 11 (Build 25H2+) or macOS (26.5+)
* **CPU:** Multi-core processor (Intel Core i5 / AMD Ryzen 5 or better)
* **GPU:** DirectX 12 or Metal-compatible GPU (Intel Iris Xe, Apple M-Series, or dedicated NVIDIA/AMD GPU) required for **WebGL** acceleration
* **RAM:** Minimum $8\text{ GB}$ DDR4 ($16\text{ GB}$ recommended)
* **Local Server Backend:** Python 3.12.x or newer (`http.server` module)
* **Web Browser:** Google Chrome (v140.0+)
* **Machine Learning SDK:** Roboflow `inference.js` (YOLO-based CNN compiled for TensorFlow.js / WebGL, trained on $2,000$ augmented concrete defect images)

---

## Installation & Execution Guide

### Step 1: Flash the ESP32-CAM Firmware
1. Open the firmware project in **VS Code** with the **PlatformIO** extension.
2. Connect the ESP32-CAM to your computer using the **FT232RL programmer** (ensure `GPIO 0` is shorted to `GND` for flashing mode).
3. Update the Wi-Fi credentials in the firmware source code:
   ```cpp
   const char* ssid = "DEVICE4903"; // Replace with your local Wi-Fi SSID
   const char* password = "YOUR_PASSWORD";
   ```
4. Compile and upload the code. Once uploaded, disconnect `GPIO 0` from `GND` and press the Reset button.
5. Open the Serial Monitor ($115200\text{ baud}$) to verify PSRAM initialization and note the local IP address printed (e.g., `http://192.168.137.121/stream`).

### Step 2: Start the Ground Station Local HTTP Server
Modern browsers restrict local files (`file:///`) from executing Web Workers and cross-origin scripts. Host the frontend directory using Python's built-in HTTP server:

```bash
cd path/to/project/Arduino
python -m http.server 8000
```

### Step 3: Launch Unsecured Chrome Instance (CORS Bypass)
Because the lightweight ESP32-CAM MJPEG stream does not natively attach Cross-Origin Resource Sharing (`CORS`) headers, standard browsers will taint the HTML5 `<canvas>` and block pixel reading. 

1. Completely close all running instances of Google Chrome.
2. Launch an isolated Chrome instance with web security disabled:
   * **Windows (Command Prompt / Run):**
     ```cmd
     chrome.exe --user-data-dir="C:/Chrome dev session" --disable-web-security
     ```
   * **macOS (Terminal):**
     ```bash
     open -n -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --args --user-data-dir="/tmp/chrome_dev_test" --disable-web-security
     ```

### Step 4: Run Real-Time Crack Inference
1. Inside the unsecured Chrome window, navigate to:
   ```text
   http://localhost:8000/ESPCAMtest1.html
   ```
2. Open the Developer Console (`F12`) to monitor the initialization of `inference.js` and model weight downloading.
3. Once loaded, live video from the drone will render on the canvas with real-time bounding boxes and confidence scores overlaid on detected structural cracks.

---

## Experimental Results & Performance

### Machine Learning & Latency Metrics
Evaluated on the custom-trained Roboflow CNN model (`crack-detection-kjvhd`, v3) across 20 epochs:

| Metric | Formula / Definition | Result Achieved |
| :--- | :--- | :--- |
| **Mean Average Precision (mAP)** | Area under the Precision-Recall curve | $88.5\%$ |
| **Precision ($P$)** | $\frac{TP}{TP + FP}$ | $91.2\%$ |
| **Recall / Sensitivity ($R$)** | $\frac{TP}{TP + FN}$ | $87.0\%$ |
| **F1-Score** | $\frac{2 \cdot (P \cdot R)}{P + R}$ | $89.0\%$ |
| **Stream Transmission Latency** | MJPEG feed delay over local Wi-Fi | $\sim 150\text{ ms}$ |
| **Inference Latency (PC GPU)** | Time per image forward pass via WebGL | $\sim 45\text{ ms}$ ($\sim 22\text{ FPS}$) |

### Payload Power Consumption Analysis
Offloading ML inference to the ground station reduces continuous current draw on the drone:

| Operational Mode | ESP32-CAM Current Draw (at $5\text{ V}$) | Estimated Payload Power |
| :--- | :--- | :--- |
| **Deep Sleep** | $\sim 6\text{ mA}$ | $0.03\text{ W}$ |
| **Idle (No Wi-Fi)** | $\sim 40\text{ mA}$ | $0.20\text{ W}$ |
| **Active Image Capture (Selected Mode)** | $\sim 120\text{ mA}$ | $0.60\text{ W}$ |
| **Active Onboard ML Inference** | $\sim 160\text{ mA}$ | $0.80\text{ W}$ |
| **Inference + Flash LED** | $\sim 280\text{ mA}$ | $1.40\text{ W}$ |

---

## Real-World Applications
* **Municipal Infrastructure Audits (e.g., BBMP):** Rapid, monthly aerial sweeps of aging urban flyovers, underpasses, and retaining walls without halting city traffic.
* **Highway & Bridge Monitoring (e.g., NHAI):** Automated concrete surface inspection across elevated expressways and suspension bridges.
* **Metro & Public Works (e.g., BMRCL & PWD):** Proactive digital logging of metro rail pillars and high-rise public facades.
* **Post-Disaster Assessment:** Immediate deployment into seismically damaged or flooded zones to assess structural integrity without risking human inspectors.
* **Versatile Payload Integration:** Beyond UAVs, the lightweight ESP32-CAM pod can be mounted on **rock-climbing robots with biomimetic grippers** (micro-spine arrays or gecko-inspired adhesives) to scale concrete dams and masonry pillars in high-wind environments.

---

## Scope of Future Work
1. **Semantic Segmentation:** Transitioning from bounding-box detection to lightweight **U-Net** architectures to isolate pixel-level crack boundaries and measure exact defect length, width, and surface area.
2. **Autonomous Wall-Following:** Integrating lightweight **Time-of-Flight (ToF) / LiDAR** and optical flow sensors into the flight controller to maintain a locked standoff distance from vertical walls regardless of wind gusts.
3. **Indoor Mapping & Swarm Intelligence:** Utilizing **ESP-MESH** networking to coordinate fleets of ultra-low-cost drones for simultaneous scanning of massive structures (e.g., dam faces) and GPS-denied indoor voids (e.g., bridge box-girders) linked to a 3D Building Information Model (BIM) Digital Twin.
4. **Standalone Cross-Platform Apps:** Packaging the HTML5/JS inference engine into native desktop apps via **Electron** and mobile field apps (Android/iOS) using **React Native** or **Flutter**, eliminating the manual Python server and Chrome flag setup.

---

## Project Team & Guidance

**Department of Mechanical Engineering**  
**Ramaiah Institute of Technology (RIT), Bengaluru – 560054**  
*(Affiliated to Visvesvaraya Technological University, Belagavi)*

### Authors
* **Prateek Ghale** (`1MS23ME086`)
* **Samarth Sharma** (`1MS23ME098`)
* **Shashanka C** (`1MS23ME106`)
* **Utkarsh Goel** (`1MS23ME123`)

### Project Guide
* **Dr. Aruna Kumara P.C.** – *Assistant Professor, Department of Mechanical Engineering, RIT Bengaluru*
