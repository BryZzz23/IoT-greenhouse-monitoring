#include <WiFi.h>
#include <HTTPClient.h>
#include <DHT.h>
#define DHTPIN 25
#define DHTTYPE DHT11
const char* ssid = "NETWORK_SSID";
const char* password = "NETWORK_PASSWORD";
const char* thingSpeakServer = "http://api.thingspeak.com/update";
const char* writeAPIKey = "TZVCA6VUGKCKB04X";
DHT dht(DHTPIN, DHTTYPE);
const int redPin = 23;
const int greenPin = 22;
const int bluePin = 21;
const int soilMoisturePin = 32;
void setup()
{
Serial.begin(9600);
WiFi.begin(ssid, password);
pinMode(redPin, OUTPUT);
pinMode(greenPin, OUTPUT);
pinMode(bluePin, OUTPUT);
pinMode(soilMoisturePin, INPUT);
while (WiFi.status() != WL_CONNECTED) {
delay(500);
Serial.print(".");
}
Serial.println("WiFi connected");
dht.begin();
}
void loop() {
float humidity = dht.readHumidity();
float temperature = dht.readTemperature();
int soilMoistureValue = analogRead(soilMoisturePin);
float soilMoisturePercent = map(soilMoistureValue, 0, 4095, 100, 0);
if (isnan(humidity) || isnan(temperature)) {
Serial.println("Failed to read from DHT sensor!");
} else {
Serial.print("Humidity: ");
Serial.print(humidity);
Serial.print("%\t");
Serial.print("Temperature: ");
Serial.print(temperature);
Serial.println("°C");
updateRGBLED(temperature, humidity);
Serial.print("Soil Moisture (%): ");
Serial.println(soilMoisturePercent);
// Construct API request URL
String httpRequestData = String(thingSpeakServer) + "?api_key=" + writeAPIKey +
"&field1=" + String(temperature) + "&field2=" + String(humidity);
if (WiFi.status() == WL_CONNECTED) {
String httpRequestData = String(thingSpeakServer) + "?api_key=" + writeAPIKey +
"&field1=" + String(temperature) +
"&field2=" + String(humidity) +
"&field3=" + String(soilMoisturePercent); // Add soil moisture as field3
sendDataToThingSpeak(httpRequestData);
}
}
delay(5000); // ThingSpeak update interval is at least 15 seconds
}
void updateRGBLED(float temperature, float humidity) {
if (temperature > 30 && humidity < 30) {
// Conditions met: Turn LED red
digitalWrite(redPin, HIGH);
digitalWrite(greenPin, LOW);
digitalWrite(bluePin, LOW);
Serial.println("I am red");
} else {
// Conditions not met: Turn LED yellow
digitalWrite(redPin, LOW);
digitalWrite(greenPin, HIGH);
digitalWrite(bluePin, LOW);
Serial.println("I am green");
}
}
void sendDataToThingSpeak(String httpRequestData) {
HTTPClient http;
http.begin(httpRequestData);
int httpResponseCode = http.GET();
if (httpResponseCode == 200) {
Serial.println("Data sent to ThingSpeak");
} else {
Serial.println("Error sending data to ThingSpeak");
Serial.println(httpResponseCode);
}
http.
