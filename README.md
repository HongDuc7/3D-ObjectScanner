# 🏺 High-Precision 3D Point Scanner

This project is an Automated 3D Scanner designed to bridge the gap between the physical and digital worlds. It transforms real-life objects—like a small plant or a clay model—into precise digital 3D models by capturing thousands of individual spatial points.

> 🎓 **Course Project** – *Electronic Devices and Circuits (CE124)*
> 
> 📍 **University of Information and Technology (UIT) – VNU-HCM**

---

## 🛠️ Technical Stack & Frameworks

* **Microcontroller:** STM32F407VETx (ARM Cortex-M4 with **Hardware FPU**)
* **Core Clock:** 168 MHz (HSE 25 MHz)
* **Framework:** STM32 Cube HAL (Hardware Abstraction Layer)
* **Peripheral Suite:** * **USB CDC:** Virtual COM Port for real-time data streaming
    * **ADC1:** 12-bit resolution for distance sensing (PA0)
    * **TIM3 & TIM4:** PWM Generation for stepper motor control (PC6 & PD12)
* **Software:** STM32CubeIDE, MeshLab (Post-processing).

---

## ⚙️ The Architecture

### 🤷‍♂️ System Overview
The scanner operates on a **Cylindrical Kinematic Model**. By rotating a turntable and incrementally raising an IR sensor along a Z-axis lead screw, the system maps the surface of an object in a helical pattern.

### 🐞 Data Processing Pipeline
1.  **Acquisition:** The **Sharp IR Sensor** outputs an analog voltage to **ADC1_IN0**.
2.  **Linearization:** Raw 12-bit ADC values are converted to voltage and then to distance using power-law regression.
3.  **Coordinate Transformation:** The system converts cylindrical coordinates $(r, \theta, z)$ into Cartesian coordinates $(x, y, z)$ using the **Cortex-M4 FPU**.
4.  **Transmission:** Formatted $(x, y, z)$ data is streamed via **USB OTG FS**  to the host PC.

---

## 🔌 Hardware Requirements

| Component | Technical Description |
| :--- | :--- |
| **STM32F407VET6** | High-performance MCU handling real-time control and math. |
| **Stepper 17HS4401** | NEMA 17 motor providing precise rotation and linear elevation. |
| **Sharp IR Sensor** | Triangulation-based sensor for non-contact distance measurement. |
| **LM2596 Buck Converter** | DC-DC step-down providing a stable 5V rail. |
| **Lead Screw** | Converts rotational motion of the Z-motor into linear height ($z$). |
| **3D Printed Chassis** | Custom structural components for the turntable and sensor mount. |
| **Object Sample** | Physical targets like small plants or 3D models. |

---

## 📐 Kinematic Formulas

For every sample point, the firmware computes:

* **Radius ($r$):** Distance from center to the object surface.
* **$X, Y$ Coordinates:** $x = r \cdot \cos(\theta)$, $y = r \cdot \sin(\theta)$.
* **$Z$ Coordinate:** Current elevation layer.

---

## 💻 Firmware Implementation

The following logic handles the spatial transformation and data transmission. 

```c
void Process_Scanning_Data(float distance, float angle_deg, float z_height)
{
    float r = DISTANCE_TO_CENTER - distance;
    float angle_rad = angle_deg * (3.14159265f / 180.0f);
    
    float x = r * cosf(angle_rad);
    float y = r * sinf(angle_rad);
    
    char usb_buffer[64];
    int msg_len = sprintf(usb_buffer, "%.3f %.3f %.3f\r\n", x, y, z_height);
    
    CDC_Transmit_FS((uint8_t*)usb_buffer, msg_len);
}