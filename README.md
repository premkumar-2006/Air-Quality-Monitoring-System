#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>
#include <WiFi.h>
#include <ThingSpeak.h>

// Pins
#define DHTPIN 4
#define DHTTYPE DHT22
#define MQ135PIN 34
#define BUZZER 15

// WiFi and ThingSpeak settings
const char* ssid = "LAVA AGNI 2";
const char* password = "98765432";
WiFiClient client;
unsigned long myChannelNumber = 2981759;
const char* myWriteAPIKey = "0N1W2H9NXO70HOIF";

// Sensor & LCD
DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal_I2C lcd(0x27, 16, 2);

// MQ135 calibration constants
#define RLOAD 10.0
#define RZERO 76.63
#define PARA 116.6020682
#define PARB -2.769034857

float getResistance(int rawADC) {
  float voltage = rawADC * (3.3 / 4095.0);
  if (voltage == 0) return -1;
  float resistance = ((3.3 - voltage) * RLOAD) / voltage;
  return resistance;
}

float getPPM(int rawADC) {
  if (rawADC <= 50 || rawADC >= 4090) return -1;
  float resistance = getResistance(rawADC);
  if (resistance < 1 || resistance > 10000) return -1;
  float ratio = resistance / RZERO;
  float ppm = PARA * pow(ratio, PARB);
  return ppm;
}

void setup() {
  Serial.begin(115200);
  dht.begin();
  lcd.init();
  lcd.backlight();
  pinMode(BUZZER, OUTPUT);
  noTone(BUZZER);

  lcd.setCursor(0, 0);
  lcd.print("Connecting WiFi");
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("WiFi Connected");
  Serial.println("\nWiFi connected");
  delay(2000);
  lcd.clear();

  ThingSpeak.begin(client);
}

void loop() {
  float temp = dht.readTemperature();
  float hum = dht.readHumidity();
  int mq135Raw = analogRead(MQ135PIN);
  float ppm = getPPM(mq135Raw);

  lcd.clear();

  if (ppm == -1) {
    Serial.println("Error: MQ135 not detected!");
    lcd.setCursor(0, 0);
    lcd.print("Sensor Error!");
    lcd.setCursor(0, 1);
    lcd.print("Check MQ135");

    // Slow low beeps for sensor error
    for (int i = 0; i < 2; i++) {
      tone(BUZZER, 300); // Low tone
      delay(500);
      noTone(BUZZER);
      delay(500);
    }
    return;
  }

  // Air quality decision
  String airStatus = (ppm <= 100) ? "GOOD AIR" : "BAD AIR";
  bool alert = (ppm > 100);

  // Serial Output
  Serial.print("Temp: ");
  Serial.print(temp);
  Serial.print("°C | Hum: ");
  Serial.print(hum);
  Serial.print("% | PPM: ");
  Serial.print(ppm);
  Serial.print(" | ");
  Serial.println(airStatus);

  // LCD Output
  lcd.setCursor(0, 0);
  lcd.print("PPM:");
  lcd.print(ppm, 0);
  lcd.print(" ");
  lcd.print(airStatus);

  lcd.setCursor(0, 1);
  lcd.print("T:");
  lcd.print(temp, 1);
  lcd.print(" H:");
  lcd.print(hum, 0);
  lcd.print("%");

  // Send to ThingSpeak
  ThingSpeak.setField(1, ppm);
  ThingSpeak.setField(2, temp);
  ThingSpeak.setField(3, hum);
  int response = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);

  if (response == 200) {
    Serial.println("Data sent to ThingSpeak.");
  } else {
    Serial.print("ThingSpeak Error: ");
    Serial.println(response);
  }

  // Custom beep for BAD AIR
  if (alert) {
    for (int i = 0; i < 3; i++) {
      tone(BUZZER, 1200, 200); // High-pitched short beep
      delay(250);
    }
  } else {
    noTone(BUZZER); // No beep for good air
  }

  delay(5000); // Wait 5 seconds for next reading
}
