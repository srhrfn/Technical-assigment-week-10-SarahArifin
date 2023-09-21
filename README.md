# Technical-assigment-week-10-SarahArifin
#code python

#include <WiFi.h>
#include "DHT.h"
#include "ThingSpeak.h"

// Network credentials
char* ssid = "ini hotspot"; // Ganti dengan SSID WiFi kamu
char* passphrase = "24681012"; // Ganti dengan kata sandi WiFi kamu

WiFiServer server(80);
WiFiClient client;

// ThingSpeak channel details
unsigned long myChannelNumber = 3;
const char * myWriteAPIKey = "0IS1580HNIS6XGTB";

// Timer variables
unsigned long lastTime = 0;
unsigned long timerDelay = 1000;

// DHT declarations
#define DHTPIN 4 // Digital pin connected to the DHT sensor
#define DHTTYPE DHT11 // DHT 11
DHT dht(DHTPIN, DHTTYPE);

// MQ-9 Gas Sensor
const int MQ9_SENSOR_PIN = 35; // Ganti dengan pin analog yang sesuai

void setup() {
  Serial.begin(115200);
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, passphrase);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  server.begin();

  // Initialize DHT11
  dht.begin();
  ThingSpeak.begin(client); // Initialize ThingSpeak

  // Setup for MQ-9 Gas Sensor
  pinMode(MQ9_SENSOR_PIN, INPUT);

  // put your setup code here, to run once:
}

void loop() {
  if ((millis() - lastTime) > timerDelay) {
    delay(2500);

    // Reading temperature or humidity takes about 250 milliseconds!
    float h = dht.readHumidity();
    float t = dht.readTemperature();
    float f = dht.readTemperature(true);

    // Reading MQ-9 Gas Sensor
    int mq9_value = analogRead(MQ9_SENSOR_PIN);

    if (isnan(h) || isnan(t) || isnan(f)) {
      Serial.println(F("Failed to read from DHT sensor!"));
      return;
    }

    Serial.print("Temperature (ºC): ");
    Serial.print(t);
    Serial.println("ºC");

    Serial.print("Humidity");
    Serial.println(h);

    Serial.print("Gas MQ-9: ");
    Serial.println(mq9_value);

    ThingSpeak.setField(1, h); // Humidity read
    ThingSpeak.setField(2, t); // Temperature read
    ThingSpeak.setField(3, mq9_value); // MQ-9 Gas Sensor read

    // Write to ThingSpeak
    int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);

    if(x == 200){
      Serial.println("Channel update successful.");
    }
    else{
      Serial.println("Problem updating channel. HTTP error code " + String(x));
    }

    lastTime = millis();
  }


}
