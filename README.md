#include <SPI.h>
#include <MFRC522.h>
#include <ESP32Servo.h>
// ---------------- PIN DEFINITIONS ----------------
#define SS_PIN       5
#define RST_PIN      22


#define GREEN_LED    32
#define RED_LED      25
#define BUZZER_PIN   33
#define SERVO_PIN    4


// ---------------- OBJECTS ----------------
MFRC522 mfrc522(SS_PIN, RST_PIN);
Servo doorServo;


// ---------------- USER STRUCTURE ----------------
struct User {
  String uid;
  String name;
  String rollNumber;
  bool isPresent;
};


// ---------------- USER DATABASE ----------------
// Replace the UIDs with your actual RFID card UIDs
User users[] = {
  {"FE 9C FA 03", "KB", "CS101", false},
  {"YY YY YY YY", "Bob Jones", "CS102", false}
};


const int numUsers = sizeof(users) / sizeof(users[0]);


// ---------------- SETUP ----------------
void setup() {
  Serial.begin(115200);


  // Start SPI communication
  SPI.begin();


  // Initialize RFID reader
  mfrc522.PCD_Init();


  // Configure output pins
  pinMode(GREEN_LED, OUTPUT);
  pinMode(RED_LED, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);


  // Turn everything OFF initially
  digitalWrite(GREEN_LED, LOW);
  digitalWrite(RED_LED, LOW);
  digitalWrite(BUZZER_PIN, LOW);


  // Initialize servo
  doorServo.attach(SERVO_PIN);


  // Door initially closed
  doorServo.write(0);


  Serial.println("=================================");
  Serial.println("RFID Door Access System");
  Serial.println("System Ready.");
  Serial.println("Tap an RFID card...");
  Serial.println("=================================");
}


// ---------------- MAIN LOOP ----------------
void loop() {


  // Check if a new RFID card is present
  if (!mfrc522.PICC_IsNewCardPresent()) {
    return;
  }


  // Read the RFID card
  if (!mfrc522.PICC_ReadCardSerial()) {
    return;
  }


  // ---------------- READ UID ----------------
  String scannedUID = "";


  for (byte i = 0; i < mfrc522.uid.size; i++) {


    if (mfrc522.uid.uidByte[i] < 0x10) {
      scannedUID += "0";
    }


    scannedUID += String(mfrc522.uid.uidByte[i], HEX);


    if (i < mfrc522.uid.size - 1) {
      scannedUID += " ";
    }
  }


  scannedUID.toUpperCase();


  Serial.println();
  Serial.println("-------------------------------");
  Serial.print("Scanned Card UID: ");
  Serial.println(scannedUID);


  // ---------------- CHECK USER ----------------
  bool accessGranted = false;


  for (int i = 0; i < numUsers; i++) {


    if (scannedUID == users[i].uid) {


      accessGranted = true;


      Serial.println("Access Granted!");
      Serial.print("Name: ");
      Serial.println(users[i].name);


      Serial.print("Roll No: ");
      Serial.println(users[i].rollNumber);


      // ---------------- CHECK-IN / CHECK-OUT ----------------
      if (users[i].isPresent == false) {


        Serial.println("Status: CHECK-IN SUCCESSFUL");
        Serial.println("Welcome!");


        users[i].isPresent = true;


      } else {


        Serial.println("Status: CHECK-OUT SUCCESSFUL");
        Serial.println("Goodbye!");


        users[i].isPresent = false;
      }


      // ---------------- GREEN LED + BUZZER ----------------
      digitalWrite(GREEN_LED, HIGH);


      digitalWrite(BUZZER_PIN, HIGH);
      delay(200);
      digitalWrite(BUZZER_PIN, LOW);


      // ---------------- OPEN DOOR ----------------
      Serial.println("Door Opening...");
      doorServo.write(90);


      // Keep door open for 3 seconds
      delay(3000);


      // ---------------- CLOSE DOOR ----------------
      Serial.println("Door Closing...");
      doorServo.write(0);


      digitalWrite(GREEN_LED, LOW);


      break;
    }
  }


  // ---------------- ACCESS DENIED ----------------
  if (!accessGranted) {


    Serial.println("Access Denied!");
    Serial.println("Unregistered RFID Card.");


    // Make sure door remains closed
    doorServo.write(0);


    // Red LED + buzzer
    digitalWrite(RED_LED, HIGH);
    digitalWrite(BUZZER_PIN, HIGH);


    // Keep alarm ON for 1.5 seconds
    delay(1500);


    // Turn OFF alarm
    digitalWrite(RED_LED, LOW);
    digitalWrite(BUZZER_PIN, LOW);
  }


  // Stop communication with the current card
  mfrc522.PICC_HaltA();


  // Stop encryption/authentication if applicable
  mfrc522.PCD_StopCrypto1();


  Serial.println("-------------------------------");
  Serial.println("Ready for next card...");
}







Component


Pin


ESP32
MFRC522
SDA/SS
GPIO 5


SCK
GPIO 18


MOSI
GPIO 23


MISO
GPIO 19


RST
GPIO 22


3.3V
3.3V


GND
GND
Green LED
+
GPIO 32 + 220Ω


-
GND
Red LED
+
GPIO 25 + 220Ω


-
GND
Buzzer
+
GPIO 33


-
GND
Servo
Signal
GPIO 4


VCC
5V


GND
GND

																																																																					
