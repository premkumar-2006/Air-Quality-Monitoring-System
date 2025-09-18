# 🌍 Air Quality & Weather Monitoring System (ESP32 + MQ135 + DHT22 + LCD + ThingSpeak)

This project monitors **air quality (CO₂, smoke, and other gases)** and **weather conditions (temperature & humidity)** using an **ESP32 microcontroller**, and sends real-time data to **ThingSpeak** for IoT visualization.  

It also provides **local feedback** via an **I2C LCD** display and an **alert buzzer** for poor air quality.

---

## 🚀 Features
- 📡 **WiFi-enabled ESP32** sends live data to ThingSpeak Cloud.  
- 🌡️ **DHT22 sensor** for temperature & humidity measurement.  
- 🏭 **MQ135 sensor** for air quality detection (PPM calculation).  
- 📟 **16x2 I2C LCD** displays sensor readings in real-time.  
- 🔔 **Buzzer alerts**:  
  - Slow beeps = MQ135 sensor error.  
  - Fast beeps = Poor air quality detected.  
- ☁️ **ThingSpeak Integration** for remote monitoring and data logging.  

---

## 🛠️ Hardware Components
- ESP32 Development Board  
- MQ135 Gas Sensor  
- DHT22 Temperature & Humidity Sensor  
- 16x2 LCD with I2C module  
- Buzzer  
- Jumper wires & breadboard  

---

## 📋 Circuit Connections
| Component | Pin (ESP32) |
|-----------|-------------|
| DHT22 Data | GPIO 4 |
| MQ135 Analog | GPIO 34 |
| Buzzer | GPIO 15 |
| LCD (I2C) | SDA → GPIO 21, SCL → GPIO 22 |

---

## ⚡ Software & Libraries
Install the following libraries from Arduino IDE Library Manager:
- `Wire.h`
- `LiquidCrystal_I2C.h`
- `Adafruit_Sensor.h`
- `DHT.h`
- `DHT_U.h`
- `WiFi.h`
- `ThingSpeak.h`

---

## 🌐 ThingSpeak Setup
1. Create a [ThingSpeak](https://thingspeak.com/) account.  
2. Create a new channel with **3 fields**:  
   - Field 1 → Air Quality (PPM)  
   - Field 2 → Temperature (°C)  
   - Field 3 → Humidity (%)  
3. Copy your **Channel Number** and **Write API Key**.  
4. Replace them in the code:  
   ```cpp
   unsigned long myChannelNumber = YOUR_CHANNEL_NUMBER;
   const char* myWriteAPIKey = "YOUR_WRITE_API_KEY";
