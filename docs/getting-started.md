---
layout: default
title: Getting Started
nav_order: 2
---

# Getting Started

This guide will help you set up ESP Dashboard in your project.

## Requirements

### Hardware
- ESP32 or ESP8266 development board
- USB cable for programming

### Software
- [PlatformIO](https://platformio.org/) (recommended) or Arduino IDE
- Python 3.x (for HTML conversion script)

### Dependencies
The library requires:
- `ESPAsyncWebServer`
- `AsyncTCP` (ESP32) or `ESPAsyncTCP` (ESP8266)
- `ArduinoJson` (v7.0.0+)

## Installation

### Using PlatformIO

1. Create a new project or open existing one
2. Add the library to `platformio.ini`:

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200

lib_deps = 
    ESP-DashboardPlus
```

3. Build the project - dependencies install automatically

### Manual Installation

1. Download the library from GitHub
2. Extract to `lib/ESP-DashboardPlus/` in your project
3. Install dependencies manually

## Project Structure

After installation, your project should look like:

```
my_project/
├── platformio.ini
├── src/
│   └── main.cpp
└── lib/
    └── ESP-DashboardPlus/
        ├── src/
        │   ├── ESPDashboardPlus.h
        │   └── dashboard_html.h
        ├── extras/
        │   └── dashboard.html
        └── scripts/
            └── html_to_header_pio.py
```

## Basic Example

Create `src/main.cpp`:

```cpp
#include <WiFi.h>
#include <ESPAsyncWebServer.h>
#include "ESPDashboardPlus.h"
#include "dashboard_html.h"

const char* ssid = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";

AsyncWebServer server(80);
ESPDashboardPlus dashboard("My ESP32 Device");

void setup() {
    Serial.begin(115200);
    
    // Connect to WiFi
    WiFi.begin(ssid, password);
    Serial.print("Connecting to WiFi");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println();
    Serial.print("Connected! IP: ");
    Serial.println(WiFi.localIP());
    
    // Initialize dashboard
    dashboard.begin(&server, DASHBOARD_HTML_DATA, DASHBOARD_HTML_SIZE);
    
    // Add a stat card
    StatCard* tempCard = dashboard.addStatCard(
        "temp",           // Unique ID
        "Temperature",    // Title
        "25.0",          // Initial value
        "°C"             // Unit
    );
    tempCard->setVariant(CardVariant::PRIMARY);
    
    // Add a toggle card
    ToggleCard* ledToggle = dashboard.addToggleCard(
        "led",           // Unique ID
        "LED Control",   // Title
        "Onboard LED",   // Label
        false            // Initial state
    );
    ledToggle->onChange = [](bool state) {
        digitalWrite(LED_BUILTIN, state);
        Serial.printf("LED: %s\n", state ? "ON" : "OFF");
    };
    
    // Start the server
    server.begin();
    Serial.printf("Dashboard ready at http://%s\n", 
                  WiFi.localIP().toString().c_str());
}

void loop() {
    // Process WebSocket events
    dashboard.loop();
    
    // Update temperature every 2 seconds
    static unsigned long lastUpdate = 0;
    if (millis() - lastUpdate > 2000) {
        lastUpdate = millis();
        
        // Simulate temperature reading
        float temp = 20.0 + random(0, 100) / 10.0;
        dashboard.updateStatCard("temp", String(temp, 1));
    }
}
```

## Build and Upload

```bash
# Build the project
pio run

# Upload to device
pio run -t upload

# Monitor serial output
pio device monitor
```

## Access the Dashboard

1. Open Serial Monitor to see the IP address
2. Open a web browser
3. Navigate to `http://<ESP_IP_ADDRESS>`

## Next Steps

- Explore all [Card Types](cards)
- Check the [API Reference](api)
- See more [Examples](examples)

## Troubleshooting

### Dashboard not loading
- Verify WiFi connection
- Check IP address in Serial Monitor
- Ensure port 80 is not blocked

### WebSocket disconnects
- Check WiFi signal strength
- Reduce update frequency if needed

### Compilation errors
- Ensure all dependencies are installed
- Check ArduinoJson version (requires v7.0.0+)
