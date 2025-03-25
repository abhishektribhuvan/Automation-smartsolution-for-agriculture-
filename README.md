# Automation-smartsolution-for-agriculture-
//The Smart Agriculture System is an loT-based project designed to automate farming processes and optimize resource utilization using an ESP32 microcontroller. The system continuously monitors soil moisture, humidity, and light intensity to make real-time decisions

#include <WiFi.h>
#include <WebServer.h>
#include <DHT.h>

// Pin Definitions
#define DHTPIN 4
#define DHTTYPE DHT11
#define MOISTURE_SENSOR_PIN 34
#define LIGHT_SENSOR_PIN 35
#define BUZZER_PIN 5 // Buzzer control pin

// Create Objects
DHT dht(DHTPIN, DHTTYPE);
WebServer server(80);

// Sensor Thresholds (Default Values)
int moistureThreshold = 500;
int lightThreshold = 200;
int humidityThreshold = 50;

// Variables
int moisture = 0;
int light = 0;
float humidity = 0.0;
String emergencyState = "OFF";

// WiFi Credentials
const char* ssid = "Your_SSID";
const char* password = "Your_PASSWORD";

void setup() {
  // Initialize Serial Monitor
  Serial.begin(115200);

  // Initialize Sensors and Buzzer
  dht.begin();
  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(BUZZER_PIN, LOW); // Turn buzzer OFF initially

  // Connect to WiFi
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConnected to WiFi");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  // Setup Web Server Routes
  server.on("/", handleRoot);
  server.on("/setThresholds", handleSetThresholds);
  server.on("/toggleBuzzer", handleToggleBuzzer);

  // Start Server
  server.begin();
}

void loop() {
  // Read Sensor Values
  moisture = analogRead(MOISTURE_SENSOR_PIN);
  light = analogRead(LIGHT_SENSOR_PIN);
  humidity = dht.readHumidity();

  // Control Buzzer Based on Sensor Values
  if ((moisture < moistureThreshold) || (light < lightThreshold) || (humidity > humidityThreshold)) {
    digitalWrite(BUZZER_PIN, HIGH); // Turn buzzer ON (emit sound)
  } else {
    digitalWrite(BUZZER_PIN, LOW); // Turn buzzer OFF
  }

  // Handle Web Server Requests
  server.handleClient();
}

// Web Page for Displaying Sensor Data and Changing Thresholds
void handleRoot() {
  String page = "<!DOCTYPE html><html><head><title>Smart Farm Monitoring</title>";
  page += "<style>";
  page += "body { font-family: Arial, sans-serif; margin: 20px; }";
  page += "table { border-collapse: collapse; width: 100%; margin-bottom: 20px; }";
  page += "th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }";
  page += "th { background-color: #4CAF50; color: white; }";
  page += "tr:nth-child(even) { background-color: #f2f2f2; }";
  page += "tr:hover { background-color: #ddd; }";
  page += ".button { background-color: #4CAF50; color: white; border: none; padding: 10px 20px; text-align: center; text-decoration: none; font-size: 16px; margin: 4px 2px; cursor: pointer; }";
  page += ".button:hover { background-color: #45a049; }";
  page += "</style></head><body>";

  page += "<h1>Smart Farm Monitoring System</h1>";
  
  // Sensor Data Table
  page += "<table>";
  page += "<tr><th>Sensor</th><th>Current Value</th><th>Threshold</th></tr>";
  page += "<tr><td>Moisture</td><td>" + String(moisture) + "</td><td>" + String(moistureThreshold) + "</td></tr>";
  page += "<tr><td>Light Intensity</td><td>" + String(light) + "</td><td>" + String(lightThreshold) + "</td></tr>";
  page += "<tr><td>Humidity</td><td>" + String(humidity) + "%</td><td>" + String(humidityThreshold) + "</td></tr>";
  page += "</table>";

  // Threshold Update Form
  page += "<form action='/setThresholds' method='GET'>";
  page += "<h3>Set Thresholds</h3>";
  page += "Moisture: <input type='number' name='moisture' value='" + String(moistureThreshold) + "'><br><br>";
  page += "Light: <input type='number' name='light' value='" + String(lightThreshold) + "'><br><br>";
  page += "Humidity: <input type='number' name='humidity' value='" + String(humidityThreshold) + "'><br><br>";
  page += "<input class='button' type='submit' value='Update Thresholds'>";
  page += "</form>";

  // Emergency Buzzer Control Button
  page += "<h3>Emergency Buzzer Control</h3>";
  page += "<p>Current Buzzer State: <b>" + emergencyState + "</b></p>";
  page += "<form action='/toggleBuzzer' method='GET'>";
  page += "<button class='button' type='submit'>Toggle Buzzer</button>";
  page += "</form>";

  page += "</body></html>";
  server.send(200, "text/html", page);
}

// Handle Threshold Updates
void handleSetThresholds() {
  if (server.hasArg("moisture")) {
    moistureThreshold = server.arg("moisture").toInt();
  }
  if (server.hasArg("light")) {
    lightThreshold = server.arg("light").toInt();
  }
  if (server.hasArg("humidity")) {
    humidityThreshold = server.arg("humidity").toInt();
  }

  server.send(200, "text/html", "<h1>Thresholds Updated</h1><a href='/'>Go Back</a>");
}

// Handle Emergency Buzzer Toggle
void handleToggleBuzzer() {
  if (emergencyState == "OFF") {
    emergencyState = "ON";
    digitalWrite(BUZZER_PIN, HIGH); // Turn buzzer ON
  } else {
    emergencyState = "OFF";
    digitalWrite(BUZZER_PIN, LOW); // Turn buzzer OFF
  }
  server.send(200, "text/html", "<h1>Buzzer Toggled</h1><a href='/'>Go Back</a>");
}
