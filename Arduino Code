/*
  Bluetooth Controlled RC Car
  --------------------------------------
  Components: Arduino Uno, HC-05, L298N, 4x TT Gear Motor,
              3x 18650 (11.1V), Headlight, Taillight, Buzzer,
              Voltage Sensor

  Bluetooth Commands (send from any BT terminal app):
    F = Forward        B = Backward
    L = Left            R = Right
    S = Stop
    H = Lights ON       h = Lights OFF
    Z = Buzzer horn (200ms)
    0-9 = Speed level (0=stop speed, 9=max speed)

  Wiring:
    HC-05 TX  -> Arduino D2 (SoftwareSerial RX)
    HC-05 RX  <- Arduino D3 (through 1k+2k voltage divider!)
    L298N ENA -> D5   IN1 -> D6   IN2 -> D7
    L298N IN3 -> D8   IN4 -> D9   ENB -> D10
    Headlight -> D12   Taillight -> D13   Buzzer -> D4
    Voltage Sensor OUT -> A0
    All GNDs (Arduino, L298N, HC-05, Battery) must be common.
*/

#include <SoftwareSerial.h>

SoftwareSerial BT(2, 3);  // RX, TX

// L298N pins
const int ENA = 5;
const int IN1 = 6;
const int IN2 = 7;
const int IN3 = 8;
const int IN4 = 9;
const int ENB = 10;

// Extras
const int HEADLIGHT = 12;
const int TAILLIGHT = 13;
const int BUZZER    = 4;
const int VOLT_PIN  = A0;

// Voltage sensor divider ratio (adjust to your module, common module = 5:1)
const float VOLT_RATIO = 5.0;
const float ARDUINO_VREF = 5.0;

int carSpeed = 200;          // default PWM speed (0-255)
unsigned long lastVoltSend = 0;
const unsigned long VOLT_INTERVAL = 3000; // send voltage every 3s

void setup() {
  Serial.begin(9600);
  BT.begin(9600);

  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  pinMode(ENB, OUTPUT);

  pinMode(HEADLIGHT, OUTPUT);
  pinMode(TAILLIGHT, OUTPUT);
  pinMode(BUZZER, OUTPUT);

  stopCar();
  digitalWrite(HEADLIGHT, LOW);
  digitalWrite(TAILLIGHT, LOW);
  digitalWrite(BUZZER, LOW);

  BT.println("RC Car Ready");
}

void loop() {
  if (BT.available()) {
    char cmd = BT.read();
    handleCommand(cmd);
  }

  // periodically report battery voltage over Bluetooth
  if (millis() - lastVoltSend > VOLT_INTERVAL) {
    lastVoltSend = millis();
    sendVoltage();
  }
}

void handleCommand(char cmd) {
  switch (cmd) {
    case 'F': moveForward();  break;
    case 'B': moveBackward(); break;
    case 'L': turnLeft();     break;
    case 'R': turnRight();    break;
    case 'S': stopCar();      break;

    case 'H': digitalWrite(HEADLIGHT, HIGH); digitalWrite(TAILLIGHT, HIGH); break;
    case 'h': digitalWrite(HEADLIGHT, LOW);  digitalWrite(TAILLIGHT, LOW);  break;

    case 'Z': honk(); break;

    default:
      if (cmd >= '0' && cmd <= '9') {
        carSpeed = map(cmd - '0', 0, 9, 0, 255);
        analogWrite(ENA, carSpeed);
        analogWrite(ENB, carSpeed);
      }
      break;
  }
}

void moveForward() {
  digitalWrite(IN1, HIGH); digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH); digitalWrite(IN4, LOW);
  analogWrite(ENA, carSpeed);
  analogWrite(ENB, carSpeed);
}

void moveBackward() {
  digitalWrite(IN1, LOW); digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW); digitalWrite(IN4, HIGH);
  analogWrite(ENA, carSpeed);
  analogWrite(ENB, carSpeed);
}

void turnLeft() {
  digitalWrite(IN1, LOW); digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH); digitalWrite(IN4, LOW);
  analogWrite(ENA, carSpeed);
  analogWrite(ENB, carSpeed);
}

void turnRight() {
  digitalWrite(IN1, HIGH); digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW); digitalWrite(IN4, HIGH);
  analogWrite(ENA, carSpeed);
  analogWrite(ENB, carSpeed);
}

void stopCar() {
  digitalWrite(IN1, LOW); digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW); digitalWrite(IN4, LOW);
  analogWrite(ENA, 0);
  analogWrite(ENB, 0);
}

void honk() {
  digitalWrite(BUZZER, HIGH);
  delay(200);
  digitalWrite(BUZZER, LOW);
}

void sendVoltage() {
  int raw = analogRead(VOLT_PIN);
  float vOut = (raw / 1023.0) * ARDUINO_VREF;
  float vBattery = vOut * VOLT_RATIO;
  BT.print("Battery: ");
  BT.print(vBattery, 2);
  BT.println("V");
}
