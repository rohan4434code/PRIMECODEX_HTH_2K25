#define BLYNK_TEMPLATE_ID "TMPL38ePww85_"
#define BLYNK_TEMPLATE_NAME "HARD THE HARDWARE"
#define BLYNK_AUTH_TOKEN "1yLMZ4WKHlYyHGvmIegStWlWZLJW52SX"

#include <Wire.h>
#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include "MAX30105.h"
#include "heartRate.h"
#include <Adafruit_SSD1306.h>
#include <SPI.h>
#include <SD.h>

// ---------- Blynk Credentials ----------
char auth[] =  BLYNK_AUTH_TOKEN "1yLMZ4WKHlYyHGvmIegStWlWZLJW52SX";// Paste from Blynk
char ssid[] = "Rajat";
char pass[] = "qwertyui";

// ---------- Hardware ----------
MAX30105 particleSensor;
Adafruit_SSD1306 display(128, 64, &Wire);
File dataFile;
BlynkTimer timer;

#define BUZZER_PIN 25
#define BUTTON_PIN 18
#define SD_CS 5

// ---------- Variables -------
long lastBeat = 0;
float beatsPerMinute;
int beatAvg = 0;
int spO2 = 0;
int systolicBP = 0, diastolicBP = 0;

bool measuring = false;

// ---------- Function to take vitals ----------
void takeVitals() {
  if (!measuring) return;

  long irValue = particleSensor.getIR();
  if (checkForBeat(irValue)) {
    long delta = millis() - lastBeat;
    lastBeat = millis();
    beatsPerMinute = 60 / (delta / 1000.0);
    if (beatsPerMinute < 255 && beatsPerMinute > 20) {
      beatAvg = (beatAvg * 0.9) + (beatsPerMinute * 0.1);
    }
  }

  // Fake SpO₂ & BP values for prototype demo
  spO2 = random(94, 99);
  systolicBP = random(110, 130);
  diastolicBP = random(70, 85);

  // ---------- Display on OLED ----------
  display.clearDisplay();
  display.setTextSize(1);
  display.setCursor(0,0);
  display.print("HR: "); display.print(beatAvg); display.println(" bpm");
  display.print("SpO2: "); display.print(spO2); display.println(" %");
  display.print("BP: "); display.print(systolicBP); display.print("/");
  display.println(diastolicBP);
  display.display();

  // ---------- Save to SD ----------
  dataFile = SD.open("/vitals.txt", FILE_APPEND);
  if (dataFile) {
    dataFile.print("HR: "); dataFile.print(beatAvg);
    dataFile.print(" SpO2: "); dataFile.print(spO2);
    dataFile.print(" BP: "); dataFile.print(systolicBP);
    dataFile.print("/"); dataFile.println(diastolicBP);
    dataFile.close();
  }

  // ---------- Send to Blynk ----------
  Blynk.virtualWrite(V1, beatAvg);   // HR
  Blynk.virtualWrite(V2, spO2);      // SpO₂
  Blynk.virtualWrite(V3, systolicBP); // BP Systolic
  Blynk.virtualWrite(V4, diastolicBP); // BP Diastolic

  // ---------- Alerts ----------
  if (spO2 < 92 || systolicBP > 140) {
    digitalWrite(BUZZER_PIN, HIGH);
    Blynk.logEvent("alert", "Critical Vital Detected!");
  } else {
    digitalWrite(BUZZER_PIN, LOW);
  }
}

// ---------- Button ISR ----------
void IRAM_ATTR startMeasurement() {
  measuring = !measuring;  // Toggle measurement
}

void setup() {
  Serial.begin(115200);
  Wire.begin();

  // ---------- WiFi + Blynk ----------
  Blynk.begin(auth, ssid, pass);

  // ---------- OLED ----------
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("OLED not found!");
    while (1);
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setCursor(0,0);
  display.println("Vital Monitor Init...");
  display.display();

  // ---------- MAX30102 ----------
  if (!particleSensor.begin(Wire, I2C_SPEED_STANDARD)) {
    Serial.println("MAX30102 not found!");
    while (1);
  }
  particleSensor.setup();

  // ---------- SD Card ----------
  if (!SD.begin(SD_CS)) {
    Serial.println("SD Card failed!");
  } else {
    Serial.println("SD Card OK");
  }

  // ---------- Pins ----------
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  attachInterrupt(BUTTON_PIN, startMeasurement, FALLING);

  // ---------- Timer ----------
  timer.setInterval(2000L, takeVitals);
}

void loop() {
  Blynk.run();
  timer.run();
}
