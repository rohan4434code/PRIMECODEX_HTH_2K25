# PRIMECODEX_HTH_2K25

🏥 HTH-25: Vital Monitoring Kit for Rural Clinics
📖 Project Documentation
Access to healthcare in rural areas is often limited due to the lack of diagnostic tools.
This project provides a low-cost, portable vital monitoring kit that can measure Blood Pressure (BP), Heart Rate (HR), and SpO₂ (Oxygen Saturation).
The device works offline with SD card logging and online with Blynk IoT for real-time monitoring and alerts.

📝 Problem Statement Title & ID
HTH-25: Vital Monitoring Kit for Rural Clinics
Problem Statement:
Rural areas lack diagnostic tools; build a portable BP, SpO₂, HR monitor with offline sync.

💡 Solution Approach
Sensing:

Use the MAX30102 sensor to measure HR and SpO₂.
Simulated BP values are currently used (future scope: digital BP sensor integration).
Display & Alerts:

OLED screen shows real-time values.
Buzzer and Blynk notifications provide emergency alerts when abnormal vitals are detected.
Data Logging:

An SD card module logs vitals for offline access in rural clinics without internet.
Connectivity:

Blynk IoT platform syncs data online for remote monitoring by doctors when WiFi is available.
Control:

A push button toggles measurement ON/OFF to save power in field conditions.
🛠️ Required Components
ESP32 – Main microcontroller
MAX30102 – Heart rate and SpO₂ sensor
OLED Display (0.96” I2C) – For live readings
SD Card Module – For offline data storage
Buzzer – Alerts for abnormal vitals
Push Button – Start/Stop measurement
Power Supply / Battery
(Optional: Digital BP sensor for real BP readings)
🏗️ Overall Structure
Initialization:

ESP32 connects to WiFi and Blynk.
OLED and SD card modules are initialized.
Measurement Cycle:

On button press, ESP32 starts reading from MAX30102.
HR & SpO₂ are calculated; BP is generated (simulated).
Data Handling:

Values are displayed on OLED.
Logged into SD card (vitals.txt).
Sent to Blynk app for remote monitoring.
Alert Mechanism:

If SpO₂ < 92% or Systolic BP > 140, buzzer rings and alert is triggered in Blynk.
Future Scope:

Integration with real BP sensor.
Cloud health database sync.
Solar power for remote clinics.
