# 💧 Smart Water Level Monitoring System

An ESP32-based smart water tank monitoring and automatic pump control system using an HC-SR04 ultrasonic sensor. It measures the water level, displays live status on a 16×2 I2C LCD, indicates the level with LEDs, alerts users with a buzzer, and controls a pump using threshold-based hysteresis.

> **Wokwi Simulation:** https://wokwi.com/projects/478147041187580929

## 📌 Overview

Traditional water tanks are often monitored manually, which can lead to water overflow, unnecessary pump operation, and empty tanks. This project automates tank-level monitoring and pump control using an ESP32.

The tank height can be adjusted from **10 cm to 100 cm** using a potentiometer, allowing the same design to be configured for different tank depths. The project also includes sensor-failure protection, a pump run-time timeout, a manual-fill button, and remote status reporting over Wi-Fi using MQTT.

## ✨ Features

- **Real-time water-level measurement** using a non-contact ultrasonic sensor.
- **Adjustable tank height** from 10 cm to 100 cm using a potentiometer.
- **16×2 I2C LCD** displays the water-level percentage, tank height, status, and pump state.
- **Three-level LED indication**:
  - 🟢 Green: 70%–100%
  - 🟡 Yellow: 30%–69%
  - 🔴 Red: 0%–29%
- **Low-water alert** using a buzzer.
- **Automatic pump control**: pump ON below 30%; pump OFF at 90% or above.
- **Hysteresis control** helps prevent rapid pump switching.
- **Filtered sensor readings**: takes five readings and uses the median to reduce the effect of noisy measurements.
- **Safety handling**: stops the pump if the sensor fails or the pump exceeds its run-time timeout.
- **Manual-fill button** to start filling and clear a fault.
- **Wi-Fi/MQTT status reporting** approximately every five seconds.

## 🧰 Hardware Components

| Component | Purpose |
|---|---|
| ESP32 development board | Main controller and Wi-Fi communication |
| HC-SR04 ultrasonic sensor | Measures distance to the water surface |
| 16×2 I2C LCD | Displays level, tank height, status, and pump state |
| 1-channel relay module | Controls the pump |
| Green, yellow, and red LEDs | Indicate the water-level band |
| Buzzer | Alerts when the water level is low |
| Potentiometer | Sets tank height from 10 cm to 100 cm |
| Push button | Manual fill and fault clearing |
| 220 Ω resistors | Current limiting for LEDs |
| Breadboard, jumper wires, and power supply | Prototype wiring |

## 🔌 ESP32 Pin Connections

| Component | ESP32 GPIO |
|---|---:|
| HC-SR04 TRIG | GPIO 5 |
| HC-SR04 ECHO | GPIO 18 |
| Potentiometer signal | GPIO 34 |
| Manual-fill push button | GPIO 14 |
| Green LED | GPIO 19 |
| Yellow LED | GPIO 25 |
| Red LED | GPIO 26 |
| Buzzer | GPIO 23 |
| Pump relay | GPIO 27 |
| LCD SDA | GPIO 21 |
| LCD SCL | GPIO 22 |

**Wiring notes**
- Connect all circuit grounds to a common ESP32 GND.
- Use a 220 Ω series resistor with each LED.
- The project specification lists the HC-SR04 and LCD as 5 V components and the potentiometer as a 3.3 V component.
- **Hardware safety:** ESP32 GPIOs are 3.3 V logic and are not 5 V tolerant. Check the HC-SR04 ECHO signal voltage and use a suitable level shifter or voltage divider before connecting it to GPIO 18. Do not power a pump directly from an ESP32 GPIO; use a correctly rated relay/driver and suitable separate pump power. Take care with mains-powered pumps.

## ⚙️ How It Works

1. The HC-SR04 sends an ultrasonic pulse toward the water surface.
2. The reflected echo returns to the sensor.
3. The ESP32 measures the echo time and calculates the distance to the water.
4. The potentiometer reading is mapped to a tank height between 10 cm and 100 cm.
5. The controller takes five distance readings and uses their median to reduce noise.
6. The water-level percentage is calculated.
7. The LCD, LEDs, and buzzer are updated according to the current level.
8. The pump relay is controlled using the low-level and full-level thresholds.
9. The system publishes status over Wi-Fi using MQTT.

### Water-Level Formula

The sensor measures the distance from the sensor to the water surface.

```text
Water Level (%) = (1 - Distance / Tank Height) × 100
```

For example, if the tank height is 50 cm and the measured distance is 25 cm:

```text
Water Level = (1 - 25 / 50) × 100 = 50%
```

### Level Indication and Pump Logic

| Water level | Indicator | Status | Automatic pump behaviour |
|---|---|---|---|
| 70%–100% | Green LED | High/full band | Pump turns OFF at 90% or above; otherwise it retains its previous state |
| 30%–69% | Yellow LED | Medium | Retains the previous pump state |
| 0%–29% | Red LED + buzzer | Low/critical | Pump turns ON |

The pump uses two thresholds: it starts below 30% and stops at 90% or above. This **hysteresis** avoids rapid ON/OFF switching near a single threshold.

### Safety Behaviour

- If the ultrasonic sensor fails to return a valid reading, the pump is switched OFF and an error is displayed.
- If the pump runs too long without filling the tank, the system stops it and reports a pump fault. The case study uses a 60-second timeout in the demonstration.
- The manual-fill button can start the pump; the full-level cutoff still applies. The button can also clear a fault.

## 🖥️ Simulation

The project is simulated in Wokwi using an ESP32. In the simulation, a distance slider represents the distance between the sensor and the water surface, and a potentiometer sets the tank height.

**Try these tests:**
1. Move the distance slider to simulate a changing water level.
2. Adjust the potentiometer and observe the tank height and percentage on the LCD.
3. Simulate a level below 30% and check that the red LED, buzzer, and pump turn ON.
4. Raise the level and check that the pump remains in its previous state through the medium band.
5. Raise the level to 90% or above and check that the pump turns OFF.
6. If supported by the simulation setup, test sensor failure and pump-timeout handling.

Open the simulation: https://wokwi.com/projects/478147041187580929

## 🛠️ Software and Techniques

- ESP32 firmware
- Ultrasonic time-of-flight distance measurement
- Median filtering using five samples
- Analog input mapping for adjustable tank height
- Threshold-based control with hysteresis
- Fault detection and pump run-time timeout
- Wi-Fi and MQTT for remote status reporting
- Wokwi for circuit simulation

> This version uses conventional embedded control logic; it does not use machine learning, deep learning, NLP, or computer vision.

## 🚀 Getting Started

1. Open the [Wokwi simulation](https://wokwi.com/projects/478147041187580929).
2. Start the simulation.
3. Use the ultrasonic distance control to simulate the water surface position.
4. Adjust the tank-height potentiometer.
5. Observe the LCD, LEDs, buzzer, and pump relay response.
6. To run on physical hardware, assemble the circuit using the pin table above, review the electrical safety notes, and configure Wi-Fi/MQTT credentials in the firmware as required.

The case study provides the design and pin mapping. If you are publishing the source code separately, add the actual firmware and any required dependency/setup instructions to this repository.

## 💰 Approximate Prototype Cost

The case study estimates the prototype at approximately **₹700**, excluding the pump. Prices vary by seller and location.

| Component | Approximate cost |
|---|---:|
| ESP32 development board | ₹350 |
| HC-SR04 ultrasonic sensor | ₹70 |
| 16×2 I2C LCD | ₹165 |
| 1-channel relay module | ₹57 |
| LEDs, resistors, buzzer, potentiometer, and button | ₹60 |
| **Total (pump excluded)** | **Approximately ₹700** |

## 🌱 Applications

- Residential overhead and underground water tanks
- Apartments and housing complexes
- Agricultural irrigation
- Industrial water storage
- Water treatment plants
- Smart water-management systems

## 🔮 Future Improvements

- Mobile application for remote monitoring
- Cloud dashboard and historical data storage
- SMS or email alerts for low water, overflow, and pump faults
- Multi-tank monitoring
- Solar-powered operation
- Water-usage analytics and leak detection

## 🤝 Contributing

Contributions and suggestions are welcome. You can improve the sensor filtering, add new monitoring interfaces, extend the system to multiple tanks, or improve fault handling. Please document changes and test them in simulation before using them with physical hardware.

## 📄 Project Information

**Project:** Smart Water Level Monitoring System Using ESP32 and Ultrasonic Sensor  
**Simulation:** [Wokwi project](https://wokwi.com/projects/478147041187580929)

## 📜 License

No license has been specified yet. If you want others to use, modify, and distribute this project, add a suitable open-source license file (for example, MIT) after deciding the terms.
