# 📋 Smart RFID Attendance System using ESP32

An IoT-based **Smart Attendance System** using an ESP32 and RC522 RFID reader. The system identifies students using their unique RFID card UID and records their name, date, time, and attendance status.

The project is developed and simulated using **Wokwi**, with NTP used for real-time date and time synchronization.

---

## 📌 Project Overview

The system automates student attendance using RFID cards. When a student taps their RFID card on the RC522 reader, the ESP32 reads the card's unique UID and compares it with the registered student list.

If the UID is recognized, the system displays the student's name, current date, time, and attendance status as **PRESENT**.

### Working Flow

```text
RFID Card
    ↓
RC522 RFID Reader
    ↓
ESP32
    ↓
Read RFID UID
    ↓
Compare with Registered Students
    ↓
Identify Student
    ↓
Get Date & Time using NTP
    ↓
Display Attendance
```

---

## 🚀 Features

* RFID-based student identification
* Unique UID-based authentication
* Student name identification
* Automatic date and time capture
* Present/unknown card detection
* ESP32-based embedded implementation
* SPI communication with RC522
* NTP-based time synchronization
* Wokwi simulation and testing

---

## 🛠️ Components Required

### Hardware

| Component         |    Quantity |
| ----------------- | ----------: |
| ESP32 DevKit      |           1 |
| RC522 RFID Reader |           1 |
| RFID Card/Tag     | As required |

### Software

* Wokwi
* Arduino/C++
* ESP32 Arduino Core
* MFRC522 Library

---

## 🔌 RC522 to ESP32 Connections

| RC522 Pin | ESP32 Pin |
| --------- | --------- |
| 3.3V      | 3.3V      |
| GND       | GND       |
| SDA / SS  | GPIO 5    |
| SCK       | GPIO 18   |
| MOSI      | GPIO 23   |
| MISO      | GPIO 19   |
| RST       | GPIO 21   |

> **Note:** The RC522 `SDA` pin is used as the SPI chip-select (SS) pin in this project. It is not being used as an I²C SDA connection.

---

## 📚 Required Library

Add the following library to the Wokwi `libraries.txt` file:

```text
MFRC522
```

---

## 💻 Source Code

```cpp
#include <WiFi.h>
#include <SPI.h>
#include <MFRC522.h>
#include <time.h>

#define SS_PIN 5
#define RST_PIN 21

const char* ssid = "Wokwi-GUEST";
const char* password = "";

// India Standard Time: UTC + 5:30
const long GMT_OFFSET_SEC = 19800;
const int DAYLIGHT_OFFSET_SEC = 0;

MFRC522 rfid(SS_PIN, RST_PIN);

void setup() {

  Serial.begin(115200);

  SPI.begin();
  rfid.PCD_Init();

  WiFi.begin(ssid, password);

  Serial.print("Connecting to WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("WiFi Connected!");

  configTime(
    GMT_OFFSET_SEC,
    DAYLIGHT_OFFSET_SEC,
    "pool.ntp.org",
    "time.nist.gov"
  );

  Serial.println("Synchronizing time...");

  struct tm timeinfo;

  while (!getLocalTime(&timeinfo)) {
    Serial.print(".");
    delay(500);
  }

  Serial.println();
  Serial.println("Time synchronized!");

  Serial.println();
  Serial.println("================================");
  Serial.println("     SMART ATTENDANCE SYSTEM");
  Serial.println("================================");
  Serial.println("Scan your RFID card...");
}

String getUID() {

  String uid = "";

  for (byte i = 0; i < rfid.uid.size; i++) {

    if (rfid.uid.uidByte[i] < 0x10) {
      uid += "0";
    }

    uid += String(rfid.uid.uidByte[i], HEX);

    if (i < rfid.uid.size - 1) {
      uid += ":";
    }
  }

  uid.toUpperCase();

  return uid;
}

void checkAttendance(String uid) {

  struct tm timeinfo;

  if (!getLocalTime(&timeinfo)) {
    Serial.println("Could not get time!");
    return;
  }

  char dateString[20];

  strftime(
    dateString,
    sizeof(dateString),
    "%d-%m-%Y",
    &timeinfo
  );

  char timeString[20];

  strftime(
    timeString,
    sizeof(timeString),
    "%H:%M:%S",
    &timeinfo
  );

  String name = "UNKNOWN";

  if (uid == "11:22:33:44") {
    name = "Loki";
  }
  else if (uid == "55:66:77:88") {
    name = "Arun";
  }
  else if (uid == "01:02:03:04") {
    name = "Kumar";
  }
  else if (uid == "AA:BB:CC:DD") {
    name = "Priya";
  }
  else if (uid == "04:11:22:33") {
    name = "Ravi";
  }
  else if (uid == "C0:FF:EE:99") {
    name = "Suresh";
  }

  Serial.println();
  Serial.println("================================");

  Serial.print("Card UID : ");
  Serial.println(uid);

  Serial.print("Name     : ");
  Serial.println(name);

  Serial.print("Date     : ");
  Serial.println(dateString);

  Serial.print("Time     : ");
  Serial.println(timeString);

  if (name != "UNKNOWN") {
    Serial.println("Status   : PRESENT");
  }
  else {
    Serial.println("Status   : UNKNOWN CARD");
  }

  Serial.println("================================");
}

void loop() {

  if (!rfid.PICC_IsNewCardPresent()) {
    return;
  }

  if (!rfid.PICC_ReadCardSerial()) {
    return;
  }

  String uid = getUID();

  checkAttendance(uid);

  rfid.PICC_HaltA();
  rfid.PCD_StopCrypto1();

  delay(2000);
}
```

> **Security note:** If you publish this code publicly, replace any personal student names and RFID UIDs with generic examples.

---

## 🧪 Example Output

When a registered RFID card is scanned:

```text
================================
Card UID : C0:FF:EE:99
Name     : Suresh
Date     : 16-08-2026
Time     : 10:15:32
Status   : PRESENT
================================
```

For an unregistered card:

```text
================================
Card UID : XX:XX:XX:XX
Name     : UNKNOWN
Date     : 16-08-2026
Time     : 10:20:15
Status   : UNKNOWN CARD
================================
```

---

## 🧠 Technologies Used

* **ESP32** — Main microcontroller
* **RC522** — RFID card reader
* **SPI** — Communication between ESP32 and RC522
* **Wi-Fi** — Internet connectivity
* **NTP** — Date and time synchronization
* **Wokwi** — Hardware simulation
* **Arduino C++** — Programming language

---

## 📊 System Architecture

```text
                 RFID CARD
                     │
                     ▼
              ┌─────────────┐
              │    RC522    │
              │ RFID Reader │
              └──────┬──────┘
                     │
                    SPI
                     │
                     ▼
              ┌─────────────┐
              │    ESP32    │
              └──────┬──────┘
                     │
              ┌──────┴──────┐
              │             │
              ▼             ▼
          UID Check      NTP Server
              │             │
              │       Date & Time
              └──────┬──────┘
                     │
                     ▼
             Attendance Result
                     │
          ┌──────────┴──────────┐
          │                     │
       PRESENT            UNKNOWN CARD
```

---

## 🔮 Future Improvements

The current version demonstrates RFID identification and attendance recording in the Wokwi Serial Monitor.

Future versions can include:

* Firebase cloud database integration
* Web-based attendance dashboard
* Automatic absent status
* Multiple classroom support
* Daily/monthly attendance reports
* Duplicate-scan prevention
* Attendance percentage calculation
* Student registration through a web interface
* Cloud-based attendance history

---

## 📁 Project Structure

```text
Smart-RFID-Attendance/
│
├── README.md
├── sketch.ino
├── diagram.json
└── libraries.txt
```

---

## 📌 Project Status

**Status: Completed – Wokwi Simulation**

The current version successfully demonstrates RFID-based student identification with automatic date/time capture and attendance status.

**Firebase cloud storage and dashboard integration are planned as future improvements.**

---

## 📜 License

This project is created for educational and learning purposes.
