# IoT-Thingspeak
Tugas IoT menghubungkan data sensor ultrasonic dan DHT11 dengan ThingSpeak      
# Anggota Kelompok:

|Nama | NRP |
|-----|-----|
|Angga Firmansyah| 5027241062|
|Rayka Dharma Pranandita | 5027241039|

# Overview:    
Menghubungkan IoT dengan WiFI lalu mengirim data melalui protokol HTTP dengan menggunakan API ThingSpeak.
Kode:
```ccp
#include "DHTesp.h"
#include "WiFi.h" 

const char* WIFI_SSID = "<SSID>";
const char* WIFI_PASS = "<PASS>";
WiFiClient client;

const int DHT_PIN = 21;

DHTesp dhtSensor;

const int trigPin = 18;
const int echoPin = 1;

//define sound speed in cm/uS
#define SOUND_SPEED 0.034

long duration;
float distanceCm;

String thingSpeakAddress = "api.thingspeak.com";
String writeAPIKey;
String tsfield1Name;
String request_string;

void setup() {
  Serial.begin(115200);
  WiFi.disconnect();
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while ((!(WiFi.status() == WL_CONNECTED))) {
    delay(300);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  dhtSensor.setup(DHT_PIN, DHTesp::DHT11);
  pinMode(trigPin, OUTPUT); // Sets the trigPin as an Output
  pinMode(echoPin, INPUT); // Sets the echoPin as an Input
}

void loop() {
  TempAndHumidity data = dhtSensor.getTempAndHumidity();

  Serial.println("Temp: " + String(data.temperature, 2) + "°C");
  Serial.println("Humidity: " + String(data.humidity, 1) + "%");
  Serial.println("---");

  delay(1000);
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  // Sets the trigPin on HIGH state for 10 micro seconds
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(echoPin, HIGH);

  // Calculate the distance
  distanceCm = duration * SOUND_SPEED / 2;

  // Prints the distance in the Serial Monitor
  Serial.print("Distance (cm): ");
  Serial.println(distanceCm);

  kirim_thingspeak(String(data.temperature, 2).toFloat(),
                   String(data.humidity, 1).toFloat(), 
                   String(distanceCm).toFloat());

  Serial.println("--------------------");

  delay(1000);
}

void kirim_thingspeak(float suhu, float hum, float JarakCm) {
  if (client.connect("api.thingspeak.com", 80)) {
    request_string = "/update?";
    request_string += "api_key=";
    request_string += "<API_KEY>";
    request_string += "&";
    request_string += "field1";
    request_string += "=";
    request_string += suhu;
    request_string += "&";
    request_string += "field2";
    request_string += "=";
    request_string += hum;
    request_string += "&";
    request_string += "field3";
    request_string += "=";
    request_string += JarakCm;
    Serial.println(String("GET ") + request_string + " HTTP/1.1\r\n" +
                   "Host: " + thingSpeakAddress + "\r\n" +
                   "Connection: close\r\n\r\n");

    client.print(String("GET ") + request_string + " HTTP/1.1\r\n" +
                 "Host: " + thingSpeakAddress + "\r\n" +
                 "Connection: close\r\n\r\n");
    unsigned long timeout = millis();
    while (client.available() == 0) {
      if (millis() - timeout > 5000) {
        Serial.println(">>> Client Timeout !");
        client.stop();
        return;
      }
    }

    while (client.available()) {
      String line = client.readStringUntil('\r');
      Serial.print(line);
    }

    Serial.println();
    Serial.println("closing connection");
  }
}
```

Wiring:
- DHT11 (GND->GND,VCC->3.3V,DATA->GPIO 21)
- Ultrasonic Sensor (GND->GND,VCC->5V,ECHO->GPIO1,TRIGGER->GPIO18)

# Dokumentasi pengerjaan:
### Wiring:      
![Wiring](https://github.com/Rkaaaa404/IoT-Thingspeak/blob/main/assets/wiring.jpeg)
![Wiring1](https://github.com/Rkaaaa404/IoT-Thingspeak/blob/main/assets/wiring1.jpeg)
![Wiring2](https://github.com/Rkaaaa404/IoT-Thingspeak/blob/main/assets/wiring2.jpeg)     

### Dashboard:
![Dashboard ThingSpeak](https://github.com/Rkaaaa404/IoT-Thingspeak/blob/main/assets/dashboard.jpeg)
