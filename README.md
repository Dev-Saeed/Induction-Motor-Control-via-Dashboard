# 3-Phase Induction Motor Control via Dashboard using ESP32

This repository provides a detailed guide to controlling a **3-phase induction motor** using an ESP32 board and a **Variable Frequency Drive (VFD)**. The project allows control over the motor's speed and direction through a web-based dashboard on **Arduino Cloud**.  

---

## Table of Contents

1. [Overview](##overview)  
2. [Hardware Requirements](#hardware-requirements)  
3. [Project Workflow](#project-workflow)  
4. [Wiring Setup](#wiring-setup)  
5. [Arduino Cloud Setup](#arduino-cloud-setup)  
6. [ESP32 Code](#esp32-code)  
7. [How to Upload the Code](#how-to-upload-the-code)  
8. [Testing the Project](#testing-the-project)  
9. [Troubleshooting and Tips](#troubleshooting-and-tips)  

---

## Overview
[![Watch the video](https://img.youtube.com/vi/7XRjqi22TJw/0.jpg)](https://www.youtube.com/watch?v=7XRjqi22TJw)

This project demonstrates how to control a **3-phase induction motor** using a **VFD** and an **ESP32 microcontroller**. The VFD adjusts the motor’s speed and direction by varying the **input voltage and frequency**. Using the **Arduino Cloud dashboard**, the motor's speed and direction can be controlled remotely.

---

## Hardware Requirements

1. **ESP32 Board**  
2. **Variable Frequency Drive (VFD)** compatible with your motor  
3. **3-phase induction motor**  
4. **Relays** (for switching P1 and P2 pins)  
5. **24V power supply** for VFD control pins  
6. **Jumper wires**  
7. **Arduino Agent** installed on your computer  
8. Optional: **Temperature sensor** and **LED** (if needed for additional features)

---

## Project Workflow

The VFD controls the motor using:  
- **P1 Pin**: Moves the motor in one direction when connected to 24V.  
- **P2 Pin**: Moves the motor in the reverse direction when connected to 24V.  
- **V1 Pin**: Controls the motor speed based on the input voltage.  

Since the ESP32 outputs a maximum of **3.3V**, you will need to configure the VFD to work with a voltage range of **0–3.3V** instead of the default **0–10V**.

---

## Wiring Setup

Follow the wiring diagram to correctly connect the ESP32, VFD, and relays:

![Induction Motor Control Diagram](https://github.com/Dev-Saeed/Induction-Motor-Control-via-Dashboard/blob/main/Induction-Motor-Control-via-Dashboard.drawio.png.png)

1. **P1 (ESP32 GPIO 18)** → Controls motor forward direction (connected to relay).  
2. **P2 (ESP32 GPIO 19)** → Controls motor reverse direction (connected to relay).  
3. **V1 (ESP32 GPIO 5)** → Controls motor speed by varying PWM output.  
4. Ensure the 24V power supply is connected to both P1 and P2 relays.  
5. Verify that the VFD is properly configured to accept inputs from these pins.

---

## Arduino Cloud Setup

To use Arduino Cloud for remote control, follow these steps:

1. **Create an Arduino Account**  
   - Visit [Arduino Cloud](https://create.arduino.cc/) and **log in** or **sign up** for a new account.  

2. **Set Up the ESP32 Device**  
   - Go to **Devices** → **Add Device**.  
   - Select the correct ESP32 board.  
   - Provide **Wi-Fi credentials** so the ESP32 can connect to your network.

3. **Define Variables in Arduino Cloud**  
   - Create three variables:  
     - **forward**: Boolean (for P1 control)  
     - **reverse**: Boolean (for P2 control)  
     - **m1_Speed**: Float (for V1 control)  
   - (Optional) Add variables for **temperature sensors** or **LED control**.

4. **Build the Dashboard**  
   - Go to **Dashboards** → **Create Dashboard**.  
   - Add:
     - **Two buttons**: One for **Forward** and one for **Reverse**.  
     - **One slider**: To control motor speed.  
   - Link these widgets to the variables you defined in the **Things** section.

5. **Generate the Arduino Cloud Sketch**  
   - Go to **Sketch** → **New Sketch**.  
   - Use the code below, replacing placeholders with your specific device configurations.

---

## ESP32 Code

Below is the complete code for controlling the motor. Ensure all wiring is correct before uploading the code.

```cpp
#include "arduino_secrets.h"  // Include file for your Wi-Fi credentials (automatically generated).
#include "thingProperties.h"  // Include the file managing Arduino Cloud properties and variables.

// Define GPIO pins for controlling relays and motor speed
int myLed = 2;                // Onboard LED pin (used for status indicator)
int P1 = 18, P2 = 19, V1 = 5; // P1 and P2 for direction, V1 for motor speed control

// PWM configuration
int PWM_Frq = 5000;           // PWM frequency set to 5000 Hz
int PWM_Chanel = 0;           // PWM channel (ESP32 allows multiple PWM channels)
int PWM_Res = 8;              // PWM resolution set to 8 bits (0-255 for duty cycle)
int speed;                    // Variable to store the mapped motor speed

void setup() {
  // Initialize serial communication to monitor output in the Serial Monitor
  Serial.begin(9600);
  delay(1500);  // Short delay to ensure everything is initialized correctly

  // Initialize cloud properties (variables defined in the Arduino Cloud dashboard)
  initProperties();  

  // Configure pins as outputs
  pinMode(myLed, OUTPUT);  // Configure LED pin as output
  pinMode(P1, OUTPUT);     // Configure P1 pin (forward direction control)
  pinMode(P2, OUTPUT);     // Configure P2 pin (reverse direction control)
  pinMode(V1, OUTPUT);     // Configure V1 pin (PWM signal for speed control)

  // Set up PWM functionality on the V1 pin
  ledcSetup(PWM_Chanel, PWM_Frq, PWM_Res);  // Initialize PWM channel with frequency and resolution
  ledcAttachPin(V1, PWM_Chanel);            // Attach the PWM channel to the V1 pin

  // Connect to the Arduino Cloud
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);

  // Set debug message level to 2 for more detailed output
  setDebugMessageLevel(2);
  ArduinoCloud.printDebugInfo();  // Print connection status to the Serial Monitor
}

void loop() {
  // Continuously update the cloud connection and synchronize variables
  ArduinoCloud.update();

  // Control the forward direction (P1) based on cloud variable
  if (forward) digitalWrite(P1, HIGH);  // Activate forward direction
  else digitalWrite(P1, LOW);           // Deactivate forward direction

  // Control the reverse direction (P2) based on cloud variable
  if (reverse) digitalWrite(P2, HIGH);  // Activate reverse direction
  else digitalWrite(P2, LOW);           // Deactivate reverse direction
}

// Function triggered when the LED variable is changed from the cloud
void onLedChange() {
  // Print the LED status to the Serial Monitor for debugging
  Serial.println(Led);
  
  // Control the onboard LED based on the cloud variable
  if (Led) digitalWrite(myLed, HIGH);  // Turn LED on
  else digitalWrite(myLed, LOW);       // Turn LED off
}

// Function triggered when the motor speed (m1_Speed) is changed from the cloud
void onM1SpeedChange() {
  // Map the speed from 0-60 Hz to 0-255 (PWM duty cycle)
  speed = map(m1_Speed, 0, 60, 0, 255);
  
  // Write the mapped speed to the PWM channel (controls motor speed)
  ledcWrite(PWM_Chanel, speed);
}

// Function triggered when the Forward button is changed from the cloud
void onForwardChange() {
  reverse = false;             // Disable reverse direction when forward is active
  digitalWrite(P2, LOW);       // Ensure P2 is LOW (reverse direction off)

  // Activate or deactivate forward direction based on cloud input
  if (forward) digitalWrite(P1, HIGH);  // Move motor forward
  else digitalWrite(P1, LOW);           // Stop forward motion
}

// Function triggered when the Reverse button is changed from the cloud
void onReverseChange() {
  forward = false;             // Disable forward direction when reverse is active
  digitalWrite(P1, LOW);       // Ensure P1 is LOW (forward direction off)

  // Activate or deactivate reverse direction based on cloud input
  if (reverse) digitalWrite(P2, HIGH);  // Move motor in reverse
  else digitalWrite(P2, LOW);           // Stop reverse motion
}

// Function triggered when the Right variable is changed (can be used for further customization)
void onRightChange() {
  // Add your custom code here if using the Right variable for additional control
}

// Function triggered when the Left variable is changed (can be used for further customization)
void onLeftChange() {
  // Add your custom code here if using the Left variable for additional control
}
```

---

## How to Upload the Code

1. **Install Arduino Agent**  
   - You will be prompted to install the Arduino Agent when you try to upload the code.  

2. **Connect the ESP32** to your computer using a USB cable.  

3. **Upload the code** from the Arduino Cloud interface.

---

## Testing the Project

1. Open the Arduino Cloud dashboard and connect the ESP32 to power.  
2. Use the **Forward** and **Reverse** buttons to test motor direction control.  
3. Adjust the slider to change motor speed.  

---

## Troubleshooting and Tips

1. **Wi-Fi Connection Issues**: Ensure that the Wi-Fi credentials are correct and the ESP32 is within range.  
2. **VFD Configuration**: Double-check that the VFD is set to operate with **P1/P2 for direction** and **V1 for speed** control.  
3. **PWM Configuration**: If the motor speed does not change, verify the PWM frequency and resolution settings.  
4. **Relays Not Working**: Ensure that the relays are wired properly and receiving 24V input.

---

This repository provides everything you need to build and operate the motor control system using an ESP32 and Arduino Cloud. If you encounter any issues or have questions, feel free to open an issue on this repository.
