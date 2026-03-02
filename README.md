# 🏺 High-Precision 3D Point Cloud Scanner

An automated embedded system designed to digitize physical objects into **3D Point Cloud** data. This project leverages Infrared (IR) triangulation and synchronized motion control to generate high-fidelity spatial coordinates for 3D reconstruction in **MeshLab**.

> 🎓 **Course Project** – *Embedded System Design (CE224)*
> 
> 📍 **University of Information and Technology (UIT) – VNU-HCM**

---

## 🛠️ Technical Stack & Frameworks

* **Microcontroller:** STM32F407VETx (ARM Cortex-M4 with **Hardware FPU**)[cite: 2, 25].
* **Core Clock:** 168 MHz (HSE 25 MHz)[cite: 7, 25].
* **Framework:** STM32 Cube HAL (Hardware Abstraction Layer)[cite: 4, 11].
* **Peripheral Suite:** * **USB CDC:** Virtual COM Port for real-time data streaming[cite: 11, 25].
    * **ADC1:** 12-bit resolution for distance sensing (PA0)[cite: 25].
    * **TIM3 & TIM4:** PWM Generation for stepper motor control (PC6 & PD12)[cite: 25].
* **Software:** STM32CubeIDE[cite: 18], MeshLab (Post-processing).

---

## ⚙️ The Architecture

### 🤷‍♂️ System Overview
The scanner operates on a **Cylindrical Kinematic Model**. By rotating a turntable and incrementally raising an IR sensor along a Z-axis lead screw, the system maps the surface of an object in a helical pattern.

### 🐞 Data Processing Pipeline
1.  [cite_start]**Acquisition:** The **Sharp IR Sensor** outputs an analog voltage to **ADC1_IN0**[cite: 25].
2.  **Linearization:** Raw 12-bit ADC values are converted to voltage and then to distance using power-law regression.
3.  [cite_start]**Coordinate Transformation:** The system converts cylindrical coordinates $(r, \theta, z)$ into Cartesian coordinates $(x, y, z)$ using the **Cortex-M4 FPU**[cite: 2].
4.  [cite_start]**Transmission:** Formatted $(x, y, z)$ data is streamed via **USB OTG FS** [cite: 25] to the host PC.



---

## 🔌 Hardware Requirements

| Component | Technical Description |
| :--- | :--- |
| **STM32F407VET6** | [cite_start]High-performance MCU handling real-time control and math[cite: 2]. |
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