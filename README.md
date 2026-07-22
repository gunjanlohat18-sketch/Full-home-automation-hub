# Full-home-automation-hub
A complete IoT-based Smart Home Automation System built using the ESP32 that combines multiple sensors, automation rules, Wi-Fi connectivity, MQTT communication, and a web dashboard into a single intelligent home monitoring solution.
<img width="1193" height="1600" alt="1000052610" src="https://github.com/user-attachments/assets/8f5fddc1-ca63-4bf4-9007-734334524498" />


https://github.com/user-attachments/assets/733d6581-e66e-411e-aace-f7a781a36216

#include "BluetoothSerial.h"

BluetoothSerial SerialBT;

// Pin Definitions
#define RELAY1 26
#define RELAY2 27
#define BUTTON1 0
#define BUTTON2 35
#define BUZZER 32
#define LED_BT 2

bool relay1State = false;
bool relay2State = false;

bool lastButton1 = HIGH;
bool lastButton2 = LOW;

void updateRelays() {
  // Active LOW Relay Module
  digitalWrite(RELAY1, relay1State ? LOW : HIGH);
  digitalWrite(RELAY2, relay2State ? LOW : HIGH);
}

void beep() {
  digitalWrite(BUZZER, HIGH);
  delay(50);
  digitalWrite(BUZZER, LOW);
}

void sendStatus() {
  SerialBT.print("L1:");
  SerialBT.print(relay1State ? "ON " : "OFF ");
  SerialBT.print("L2:");
  SerialBT.println(relay2State ? "ON" : "OFF");
}

void setup() {

  Serial.begin(115200);
  delay(1000);

  Serial.println("ESP32 Started");

  if (!SerialBT.begin("IIT_IoT_HomeCtrl")) {
    Serial.println("Bluetooth Failed!");
    while (1);
  }

  Serial.println("Bluetooth Started");
  Serial.println("Device Name: IIT_IoT_HomeCtrl");

  pinMode(RELAY1, OUTPUT);
  pinMode(RELAY2, OUTPUT);

  pinMode(BUTTON1, INPUT_PULLUP);
  pinMode(BUTTON2, INPUT);

  pinMode(BUZZER, OUTPUT);
  pinMode(LED_BT, OUTPUT);

  digitalWrite(BUZZER, LOW);

  relay1State = false;
  relay2State = false;
  updateRelays();
}

void loop() {

  // Bluetooth Connected Indicator
  digitalWrite(LED_BT, SerialBT.hasClient());

  // Bluetooth Commands
  if (SerialBT.available()) {

    char cmd = SerialBT.read();

    switch (cmd) {

      case '1':
        relay1State = true;
        break;

      case '2':
        relay1State = false;
        break;

      case '3':
        relay2State = true;
        break;

      case '4':
        relay2State = false;
        break;

      case '5':
        relay1State = true;
        relay2State = true;
        break;

      case '6':
        relay1State = false;
        relay2State = false;
        break;

      case '?':
        sendStatus();
        break;
    }

    updateRelays();
    beep();
  }

  // Button 1 (GPIO0)
  bool button1 = digitalRead(BUTTON1);

  if (lastButton1 == HIGH && button1 == LOW) {
    relay1State = !relay1State;
    updateRelays();
    beep();
    sendStatus();
    delay(200);
  }

  lastButton1 = button1;

  // Button 2 (GPIO35)
  bool button2 = digitalRead(BUTTON2);

  if (lastButton2 == LOW && button2 == HIGH) {
    relay2State = !relay2State;
    updateRelays();
    beep();
    sendStatus();
    delay(200);
  }

  lastButton2 = button2;
}
