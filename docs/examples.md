---
layout: default
title: Examples
nav_order: 5
---

# Examples

Practical examples showing ESP DashboardPlus in action.

---

## Basic Example

A minimal example showing core functionality.

```cpp
#include <WiFi.h>
#include <ESPAsyncWebServer.h>
#include "ESPDashboardPlus.h"
#include "dashboard_html.h"

AsyncWebServer server(80);
ESPDashboardPlus dashboard("Basic Example");

void setup() {
    Serial.begin(115200);
    WiFi.begin("SSID", "PASSWORD");
    while (WiFi.status() != WL_CONNECTED) delay(500);
    
    dashboard.begin(&server, DASHBOARD_HTML_DATA, DASHBOARD_HTML_SIZE);
    
    // Temperature display
    dashboard.addStatCard("temp", "Temperature", "25.0", "°C");
    
    // LED toggle
    ToggleCard* led = dashboard.addToggleCard("led", "LED", "Onboard LED", false);
    led->onChange = [](bool state) {
        digitalWrite(LED_BUILTIN, state);
    };
    
    server.begin();
}

void loop() {
    dashboard.loop();
    
    static unsigned long lastUpdate = 0;
    if (millis() - lastUpdate > 2000) {
        lastUpdate = millis();
        float temp = 20.0 + random(0, 100) / 10.0;
        dashboard.updateStatCard("temp", String(temp, 1));
    }
}
```

---

## Sensor Monitoring

Monitor multiple sensors with charts and gauges.

```cpp
#include <WiFi.h>
#include <ESPAsyncWebServer.h>
#include "ESPDashboardPlus.h"
#include "dashboard_html.h"

AsyncWebServer server(80);
ESPDashboardPlus dashboard("Sensor Monitor");

// Simulated sensor data
float temperature = 25.0;
float humidity = 50.0;
int cpuUsage = 30;

void setup() {
    Serial.begin(115200);
    WiFi.begin("SSID", "PASSWORD");
    while (WiFi.status() != WL_CONNECTED) delay(500);
    
    dashboard.begin(&server, DASHBOARD_HTML_DATA, DASHBOARD_HTML_SIZE);
    
    // Stat cards
    StatCard* tempCard = dashboard.addStatCard("temp", "Temperature", "25.0", "°C");
    tempCard->setVariant(CardVariant::PRIMARY);
    
    StatCard* humidCard = dashboard.addStatCard("humid", "Humidity", "50", "%");
    humidCard->setVariant(CardVariant::INFO);
    
    // Gauge
    GaugeCard* cpuGauge = dashboard.addGaugeCard("cpu", "CPU Usage", 0, 100, "%");
    cpuGauge->setThresholds(60, 85);
    
    // Charts
    ChartCard* tempChart = dashboard.addChartCard("temp-chart", "Temperature History", ChartType::LINE, 30);
    ChartCard* humidChart = dashboard.addChartCard("humid-chart", "Humidity History", ChartType::AREA, 30);
    
    // Status
    StatusCard* status = dashboard.addStatusCard("status", "System Status", StatusIcon::CHECK);
    status->setStatus(StatusIcon::CHECK, CardVariant::SUCCESS, "All Systems OK", "Last update: now");
    
    // Console
    dashboard.addConsoleCard("console", "Sensor Log", 50);
    dashboard.logInfo("console", "Sensor monitoring started");
    
    server.begin();
}

void loop() {
    dashboard.loop();
    
    static unsigned long lastUpdate = 0;
    if (millis() - lastUpdate > 2000) {
        lastUpdate = millis();
        
        // Simulate readings
        temperature += random(-5, 6) / 10.0;
        temperature = constrain(temperature, 15, 35);
        
        humidity += random(-2, 3);
        humidity = constrain(humidity, 30, 70);
        
        cpuUsage += random(-5, 6);
        cpuUsage = constrain(cpuUsage, 10, 95);
        
        // Update dashboard
        dashboard.updateStatCard("temp", String(temperature, 1));
        dashboard.updateStatCard("humid", String(humidity, 0));
        dashboard.updateGaugeCard("cpu", cpuUsage);
        dashboard.updateChartCard("temp-chart", temperature);
        dashboard.updateChartCard("humid-chart", humidity);
        
        // Log readings
        dashboard.logDebug("console", "T=" + String(temperature, 1) + "°C H=" + String(humidity, 0) + "%");
        
        // Check thresholds
        if (temperature > 30) {
            dashboard.logWarning("console", "High temperature: " + String(temperature, 1) + "°C");
        }
    }
}
```

---

## LED Controller

Control RGB LED with color picker, brightness slider, and toggle.

```cpp
#include <WiFi.h>
#include <ESPAsyncWebServer.h>
#include "ESPDashboardPlus.h"
#include "dashboard_html.h"

const int LED_R = 25;
const int LED_G = 26;
const int LED_B = 27;

AsyncWebServer server(80);
ESPDashboardPlus dashboard("LED Controller");

bool ledEnabled = false;
int brightness = 100;
String currentColor = "#00D4AA";

void updateLED() {
    if (!ledEnabled) {
        analogWrite(LED_R, 0);
        analogWrite(LED_G, 0);
        analogWrite(LED_B, 0);
        return;
    }
    
    long rgb = strtol(currentColor.substring(1).c_str(), NULL, 16);
    int r = ((rgb >> 16) & 0xFF) * brightness / 100;
    int g = ((rgb >> 8) & 0xFF) * brightness / 100;
    int b = (rgb & 0xFF) * brightness / 100;
    
    analogWrite(LED_R, r);
    analogWrite(LED_G, g);
    analogWrite(LED_B, b);
}

void setup() {
    Serial.begin(115200);
    pinMode(LED_R, OUTPUT);
    pinMode(LED_G, OUTPUT);
    pinMode(LED_B, OUTPUT);
    
    WiFi.begin("SSID", "PASSWORD");
    while (WiFi.status() != WL_CONNECTED) delay(500);
    
    dashboard.begin(&server, DASHBOARD_HTML_DATA, DASHBOARD_HTML_SIZE);
    
    // Power toggle
    ToggleCard* power = dashboard.addToggleCard("power", "LED Power", "Enable LED", false);
    power->onChange = [](bool state) {
        ledEnabled = state;
        updateLED();
    };
    
    // Color picker
    ColorPickerCard* color = dashboard.addColorPickerCard("color", "LED Color", "#00D4AA");
    color->onChange = [](const String& c) {
        currentColor = c;
        updateLED();
    };
    
    // Brightness slider
    SliderCard* bright = dashboard.addSliderCard("brightness", "Brightness", 0, 100, 5, "%");
    bright->setValue(100);
    bright->onChange = [](int value) {
        brightness = value;
        updateLED();
    };
    
    server.begin();
}

void loop() {
    dashboard.loop();
}
```

---

## WiFi Configuration

Configure WiFi settings via the dashboard.

```cpp
#include <WiFi.h>
#include <ESPAsyncWebServer.h>
#include <Preferences.h>
#include "ESPDashboardPlus.h"
#include "dashboard_html.h"

Preferences prefs;
AsyncWebServer server(80);
ESPDashboardPlus dashboard("WiFi Config");

String savedSSID = "";
String savedPassword = "";

void setup() {
    Serial.begin(115200);
    
    // Load saved credentials
    prefs.begin("wifi", true);
    savedSSID = prefs.getString("ssid", "");
    savedPassword = prefs.getString("pass", "");
    prefs.end();
    
    // Try saved credentials or start AP
    if (savedSSID.length() > 0) {
        WiFi.begin(savedSSID.c_str(), savedPassword.c_str());
        int attempts = 0;
        while (WiFi.status() != WL_CONNECTED && attempts < 20) {
            delay(500);
            attempts++;
        }
    }
    
    if (WiFi.status() != WL_CONNECTED) {
        WiFi.softAP("ESP32-Setup", "password123");
    }
    
    dashboard.begin(&server, DASHBOARD_HTML_DATA, DASHBOARD_HTML_SIZE);
    
    // Status
    StatusCard* status = dashboard.addStatusCard("status", "WiFi Status", StatusIcon::WIFI);
    if (WiFi.status() == WL_CONNECTED) {
        status->setStatus(StatusIcon::WIFI, CardVariant::SUCCESS, "Connected", WiFi.localIP().toString());
    } else {
        status->setStatus(StatusIcon::WIFI, CardVariant::WARNING, "AP Mode", "192.168.4.1");
    }
    
    // SSID input
    InputCard* ssidInput = dashboard.addInputCard("ssid", "WiFi SSID", savedSSID);
    ssidInput->setValue(savedSSID);
    
    // Password input
    InputCard* passInput = dashboard.addInputCard("password", "WiFi Password", "");
    passInput->inputType = "password";
    
    // Save button
    dashboard.addButtonCard("save", "Settings", "Save & Restart", []() {
        prefs.begin("wifi", false);
        // Get values from cards
        InputCard* ssid = static_cast<InputCard*>(dashboard.getCard("ssid"));
        InputCard* pass = static_cast<InputCard*>(dashboard.getCard("password"));
        prefs.putString("ssid", ssid->value);
        prefs.putString("pass", pass->value);
        prefs.end();
        
        delay(1000);
        ESP.restart();
    });
    
    server.begin();
}

void loop() {
    dashboard.loop();
}
```

---

## OTA Firmware Update

Enable over-the-air firmware updates.

```cpp
#include <WiFi.h>
#include <ESPAsyncWebServer.h>
#include "ESPDashboardPlus.h"
#include "dashboard_html.h"

AsyncWebServer server(80);
ESPDashboardPlus dashboard("OTA Example");

void setup() {
    Serial.begin(115200);
    WiFi.begin("SSID", "PASSWORD");
    while (WiFi.status() != WL_CONNECTED) delay(500);
    
    dashboard.begin(&server, DASHBOARD_HTML_DATA, DASHBOARD_HTML_SIZE);
    
    // Firmware info
    StatCard* version = dashboard.addStatCard("version", "Firmware Version", "1.0.0", "");
    
    // OTA card
    OTACard* ota = dashboard.addOTACard("ota", "Firmware Update", 4);
    
    ota->onProgress = [](size_t current, size_t total) {
        int percent = (current * 100) / total;
        Serial.printf("OTA Progress: %d%%\n", percent);
    };
    
    ota->onComplete = [](bool success) {
        if (success) {
            Serial.println("OTA Success! Restarting...");
        } else {
            Serial.println("OTA Failed!");
        }
    };
    
    // Console for OTA logs
    ConsoleCard* console = dashboard.addConsoleCard("console", "Update Log", 20);
    dashboard.logInfo("console", "Ready for firmware update");
    
    server.begin();
}

void loop() {
    dashboard.loop();
}
```

---

## Complete All-Cards Example

See the `examples/basic/main.cpp` in the library for a complete example demonstrating all 16 card types.
