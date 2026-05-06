# Automatic-seed-sowing-and-irritating-robot-
An automatic seed sowing and irrigation robot is an agricultural machine designed to reduce manual labor and improve farming efficiency. This robot can automatically plant seeds at proper depth and spacing in the soil, ensuring uniform crop growth. It uses programmed mechanisms and sensors to move across the field and drop seeds accurately.

#include <Servo.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <SoftwareSerial.h>

// LCD I2C Address
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Servo and Bluetooth
Servo servoMotor;
SoftwareSerial BT(0,1); // RX, TX

// Motor Driver Pins (L298N)
#define IN1 A5
#define IN2 A4
#define IN3 A3
#define IN4 A2
#define ENA 9
#define ENB 3

// Relay, Servo, IR Sensor
#define RELAY 4
#define SERVO_PIN 2
#define IR 5
#define BZ 3

// Variables
bool forwardState = false;
unsigned long previousServoMillis = 0;
unsigned long previousPumpMillis = 0;
unsigned long pumpStartTime = 0;
bool pumpActive = false;
int servoAngle = 0;

void setup() {
  // Pin Modes
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);
  pinMode(RELAY, OUTPUT);
  pinMode(IR, INPUT);
   pinMode(BZ, OUTPUT);

  // Servo & Relay Init
  servoMotor.attach(SERVO_PIN);
  servoMotor.write(0);
  digitalWrite(RELAY, LOW);
  digitalWrite(BZ, LOW);

  // LCD Init
  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Bluetooth Bot");
  lcd.setCursor(0, 1);
  lcd.print("Initializing...");
  delay(2000);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Ready to Connect");

  // Bluetooth Start
  BT.begin(9600);

  // Serial Monitor Start
  Serial.begin(9600);
  Serial.println("System Ready");
}

void loop() {
  unsigned long currentMillis = millis();

  // === IR Obstacle Detection ===
  if (digitalRead(IR) == LOW) {
    stopAll();
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Obstacle Ahead");
    lcd.setCursor(0, 1);
    lcd.print("Stopped!");
    Serial.println("Obstacle Detected");
    return;
  }

  // === Bluetooth Commands ===
  if (BT.available()) {
    char command = BT.read();
    while (BT.available()) BT.read(); // clear buffer

    Serial.print("Command: ");
    Serial.println(command);

    switch (command) {
      case 'F':
        forwardState = true;
        moveForward();
        displayLCD("Moving Forward", "");
        break;

      case 'B':
        forwardState = false;
        moveBackward();
        displayLCD("Moving Backward", "");
        break;

      case 'L':
        forwardState = false;
        turnLeft();
        displayLCD("Turning Left", "");
        break;

      case 'R':
        forwardState = false;
        turnRight();
        displayLCD("Turning Right", "");
        break;

      case 'S':
        forwardState = false;
        stopAll();
        displayLCD("Stopped", "");
        break;
    }
  }

  // === Forward Mode Actions ===
  if (forwardState) {
    // Servo flip every 1 second
    if (currentMillis - previousServoMillis >= 1000) {
      previousServoMillis = currentMillis;
      servoAngle = (servoAngle == 0) ? 90 : 0;
      servoMotor.write(servoAngle);
    }

    // Pump ON every 1.5 sec
    if (currentMillis - previousPumpMillis >= 1500) {
      previousPumpMillis = currentMillis;
      digitalWrite(RELAY, HIGH);
      pumpStartTime = currentMillis;
      pumpActive = true;
      Serial.println("Pump ON");
    }

    // Pump OFF after 300 ms
    if (pumpActive && currentMillis - pumpStartTime >= 300) {
      digitalWrite(RELAY, LOW);
      pumpActive = false;
      Serial.println("Pump OFF");
    }
  }
}

// ================= Helper Functions =================

void displayLCD(String line1, String line2) {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(line1);
  lcd.setCursor(0, 1);
  lcd.print(line2);
}

void moveForward() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  analogWrite(ENA, 200);
  analogWrite(ENB, 200);
  digitalWrite(BZ, HIGH);
}

void moveBackward() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  analogWrite(ENA, 200);
  analogWrite(ENB, 200);
  digitalWrite(BZ, HIGH);
}

void turnLeft() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  analogWrite(ENA, 180);
  analogWrite(ENB, 180);
  digitalWrite(BZ, HIGH);
}

void turnRight() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  analogWrite(ENA, 180);
  analogWrite(ENB, 180);
  digitalWrite(BZ, HIGH);
}

void stopAll() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  analogWrite(ENA, 0);
  analogWrite(ENB, 0);
  digitalWrite(RELAY, LOW);
  servoMotor.write(0);
  pumpActive = false;
  forwardState = false;
}

