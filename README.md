CODING :  
#include <Servo.h> 
// --------- PINS ---------- 
#define IR_SENSOR A3 
#define BUZZER 4 
#define METAL_PIN A2 
#define MOISTURE_PIN A0 
#define RELAY_PIN 7 
Servo wasteServo;   // sorting servo 
Servo lidServo;     // NEW: open/close lid servo 
// --------- VARIABLES ---------- 
int moistureValue; 
int metalDetected; 
int currentAngle = 0; 
// --------- LID ANGLES ---------- 
#define LID_OPEN  180 
#define LID_CLOSE 0 
// --------- SERVO SMOOTH FUNCTION ---------- 
void moveServoSmooth(int toAngle) { 
if (currentAngle < toAngle) { 
for (int i = currentAngle; i <= toAngle; i++) { 
wasteServo.write(i); 
delay(5); 
} 
} else { 
for (int i = currentAngle; i >= toAngle; i--) { 
wasteServo.write(i); 
delay(5); 
} 
} 
currentAngle = toAngle; 
} 
// --------- LID CONTROL ---------- 
void openLid() { 
for (int i = LID_CLOSE; i <= LID_OPEN; i++) { 
lidServo.write(i); 
delay(5); 
} 
} 
void closeLid() { 
for (int i = LID_OPEN; i >= LID_CLOSE; i--) { 
lidServo.write(i); 
delay(5); 
} 
} 
void setup() { 
Serial.begin(9600); 
pinMode(IR_SENSOR, INPUT); 
pinMode(BUZZER, OUTPUT); 
pinMode(METAL_PIN, INPUT); 
pinMode(RELAY_PIN, OUTPUT); 
wasteServo.attach(10); 
lidServo.attach(9);   // NEW SERVO PIN 
wasteServo.write(0); 
lidServo.write(LID_CLOSE); 
currentAngle = 0; 
digitalWrite(BUZZER, LOW); 
digitalWrite(RELAY_PIN, LOW); 
Serial.println("SMART WASTE SYSTEM READY"); 
} 
void loop() { 
 
  int objectDetected = digitalRead(IR_SENSOR); 
 
  // --------- WAIT FOR OBJECT ---------- 
  if (objectDetected == HIGH) { 
    digitalWrite(BUZZER, LOW); 
    delay(200); 
    return; 
  } 
 
  // --------- OBJECT DETECTED ---------- 
  Serial.println("OBJECT DETECTED"); 
 
digitalWrite(BUZZER, HIGH); 
delay(300); 
digitalWrite(BUZZER, LOW); 
 
 delay(300); 
 
  // --------- READ SENSORS ---------- 
  moistureValue = analogRead(MOISTURE_PIN); 
  metalDetected = digitalRead(METAL_PIN); 
 
  Serial.print("Moisture: "); 
  Serial.print(moistureValue); 
  Serial.print(" | Metal: "); 
  Serial.println(metalDetected); 
 
  // --------- FOOD WASTE ---------- 
  if (moistureValue < 800) { 
    Serial.println("FOOD WASTE DETECTED"); 
 
    moveServoSmooth(180); 
  delay(1500); 
    openLid();      // OPEN DROP 
    delay(1500);    // wait for waste to fall 
    closeLid();     // CLOSE 
      delay(1500); 
 
 
 
 
 
 
 
 
 
 
// --------- RELAY MOTOR ---------- 
digitalWrite(RELAY_PIN, HIGH); 
delay(3500); 
digitalWrite(RELAY_PIN, LOW); 
delay(1000); 
} 
// --------- METAL WASTE ---------- 
else if (metalDetected == LOW) { 
Serial.println("METAL WASTE DETECTED"); 
moveServoSmooth(90); 
delay(1500); 
openLid(); 
delay(1500); 
closeLid(); 
delay(1500); 
// --------- RELAY MOTOR ---------- 
digitalWrite(RELAY_PIN, HIGH); 
delay(3500); 
digitalWrite(RELAY_PIN, LOW); 
delay(1000); 
} 
// --------- PLASTIC WASTE ---------- 
else { 
Serial.println("PLASTIC WASTE DETECTED"); 
moveServoSmooth(0); 
delay(1500); 
openLid(); 
delay(1500); 
closeLid(); 
delay(1500); 
// --------- RELAY MOTOR ---------- 
digitalWrite(RELAY_PIN, HIGH); 
delay(3500); 
digitalWrite(RELAY_PIN, LOW); 
delay(1000); 
} 
}
