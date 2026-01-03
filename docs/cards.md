---
layout: default
title: Cards Reference
nav_order: 3
---

# Cards Reference

ESP DashboardPlus provides 16 different card types for building your dashboard.

---

## StatCard

Displays a numeric value with optional unit and trend indicator.

### Visual Representation

```
┌─────────────────────────────────────┐
│  Temperature                        │
│                                     │
│         23.5 °C                     │
│           ↑ +0.5°C                  │
└─────────────────────────────────────┘
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | `String` | Unique identifier for the card |
| `title` | `String` | Card title displayed at the top |
| `value` | `String` | The main value to display |
| `unit` | `String` | Unit suffix (e.g., "°C", "%", "V") |

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `variant` | `CardVariant` | Color theme (PRIMARY, SUCCESS, WARNING, etc.) |
| `trend` | `String` | Trend direction: "up", "down", or "" |
| `trendValue` | `String` | Trend value text (e.g., "+0.5°C") |

### Methods

| Method | Parameters | Description |
|--------|------------|-------------|
| `setValue()` | `const String& val` | Update the displayed value |
| `setTrend()` | `const String& t, const String& val` | Set trend direction and value |
| `setVariant()` | `CardVariant v` | Change the color variant |

### Example

```cpp
StatCard* tempCard = dashboard.addStatCard(
    "temp",           // ID
    "Temperature",    // Title
    "23.5",          // Initial value
    "°C"             // Unit
);

tempCard->setVariant(CardVariant::PRIMARY);
tempCard->setTrend("up", "+0.5°C");

// Update value
dashboard.updateStatCard("temp", "24.0");
```

---

## GaugeCard

Displays a circular gauge with configurable thresholds.

### Visual Representation

```
┌─────────────────────────────────────┐
│  CPU Usage                          │
│                                     │
│          ╭─────╮                    │
│        ╱    65%  ╲                  │
│       ╱           ╲                 │
│      ●─────────────●                │
│      0%          100%               │
└─────────────────────────────────────┘
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `id` | `String` | - | Unique identifier |
| `title` | `String` | - | Card title |
| `min` | `float` | 0 | Minimum value |
| `max` | `float` | 100 | Maximum value |
| `unit` | `String` | "%" | Unit suffix |

### Example

```cpp
GaugeCard* cpuGauge = dashboard.addGaugeCard(
    "cpu", "CPU Usage", 0, 100, "%"
);
cpuGauge->setThresholds(60, 85);  // Warning at 60%, danger at 85%
cpuGauge->setValue(45);

// Update
dashboard.updateGaugeCard("cpu", 72);
```

---

## ChartCard

Displays a time-series chart with multiple chart type options.

### Visual Representation

```
┌─────────────────────────────────────┐
│  Temperature History                │
│                                     │
│     25 ┤      ●                     │
│     24 ┤   ●     ●    ●             │
│     23 ┤ ●         ●     ●          │
│     22 ┤                    ●       │
│        └────────────────────────    │
└─────────────────────────────────────┘
```

### Chart Types

| Type | Description |
|------|-------------|
| `LINE` | Connected line chart with round points |
| `AREA` | Filled area chart |
| `BAR` | Vertical bar chart |
| `SCATTER` | Scatter plot with round points |
| `STEP` | Step/staircase line chart |

### Example

```cpp
ChartCard* tempChart = dashboard.addChartCard(
    "temp-chart",           // ID
    "Temperature History",  // Title
    ChartType::AREA,        // Chart type
    30                      // Keep 30 data points
);
tempChart->setVariant(CardVariant::PRIMARY);

// Add data points
dashboard.updateChartCard("temp-chart", 23.5);
dashboard.updateChartCard("temp-chart", 24.0);
```

---

## ToggleCard

On/off switch for controlling boolean states.

### Visual Representation

```
┌─────────────────────────────────────┐
│  LED Control                        │
│                                     │
│  Onboard LED               [═══●]   │
│                               ON    │
└─────────────────────────────────────┘
```

### Example

```cpp
ToggleCard* ledToggle = dashboard.addToggleCard(
    "led-toggle", "LED Control", "Onboard LED", false
);

ledToggle->onChange = [](bool value) {
    digitalWrite(LED_BUILTIN, value);
    Serial.printf("LED: %s\n", value ? "ON" : "OFF");
};

// Programmatic update
dashboard.updateToggleCard("led-toggle", true);
```

---

## SliderCard

Range slider for numeric value input.

### Visual Representation

```
┌─────────────────────────────────────┐
│  Brightness                         │
│                                     │
│  ○────────────●──────────○    75%   │
│  0           50         100         │
└─────────────────────────────────────┘
```

### Example

```cpp
SliderCard* brightness = dashboard.addSliderCard(
    "brightness", "Brightness", 0, 100, 5, "%"
);
brightness->setValue(75);

brightness->onChange = [](int value) {
    analogWrite(LED_PIN, map(value, 0, 100, 0, 255));
};
```

---

## ButtonCard

Simple clickable button that triggers an action.

### Example

```cpp
ButtonCardImpl* saveBtn = dashboard.addButtonCard(
    "save", "Settings", "Save Configuration",
    []() {
        saveSettingsToEEPROM();
        Serial.println("Settings saved!");
    }
);
saveBtn->setVariant(CardVariant::PRIMARY);
```

---

## ActionButton

Button with a confirmation popup before executing.

### Visual Representation

```
┌─────────────────────────────────────┐
│  ⚠ Restart Device?                  │
│                                     │
│  The device will restart and        │
│  temporarily lose connection.       │
│                                     │
│    [Cancel]          [Confirm]      │
└─────────────────────────────────────┘
```

### Example

```cpp
ActionButton* restartBtn = dashboard.addActionButton(
    "restart",
    "Device Control",
    "Restart Device",
    "Restart Device?",
    "The device will restart and temporarily lose connection.",
    []() {
        ESP.restart();
    }
);
restartBtn->setVariant(CardVariant::WARNING);
```

---

## LinkCard

Button that redirects to an external URL.

### Example

```cpp
LinkCard* docsLink = dashboard.addLinkCard(
    "docs", "Documentation", "View Docs", "https://github.com/example/docs"
);
docsLink->setTarget("_blank");
docsLink->setVariant(CardVariant::INFO);
```

---

## InputCard

Text or number input field with submit functionality.

### Example

```cpp
// Text input
InputCard* ssidInput = dashboard.addInputCard(
    "wifi-ssid", "WiFi SSID", "Enter network name..."
);
ssidInput->onSubmit = [](const String& value) {
    Serial.printf("New SSID: %s\n", value.c_str());
};

// Number input
InputCard* intervalInput = dashboard.addInputCard(
    "interval", "Update Interval", "1000"
);
intervalInput->setNumberInput(100, 60000, 100, "ms");
```

---

## DropdownCard

Dropdown/select menu for choosing from predefined options.

### Example

```cpp
DropdownCardImpl* modeDropdown = dashboard.addDropdownCard(
    "wifi-mode", "WiFi Mode", "Select mode..."
);
modeDropdown->addOption("sta", "Station (Client)");
modeDropdown->addOption("ap", "Access Point");
modeDropdown->addOption("apsta", "AP + Station");
modeDropdown->setValue("sta");

modeDropdown->onChange = [](const String& value) {
    Serial.printf("Mode: %s\n", value.c_str());
};
```

---

## ColorPickerCard

Color picker with preset colors and custom hex input.

### Example

```cpp
ColorPickerCard* colorPicker = dashboard.addColorPickerCard(
    "led-color", "LED Color", "#00D4AA"
);

colorPicker->onChange = [](const String& color) {
    // Parse hex color and apply to RGB LED
    long rgb = strtol(color.substring(1).c_str(), NULL, 16);
    int r = (rgb >> 16) & 0xFF;
    int g = (rgb >> 8) & 0xFF;
    int b = rgb & 0xFF;
    setRGB(r, g, b);
};
```

---

## DateCard

Date and optional time picker.

### Example

```cpp
// Date only
DateCard* scheduleDate = dashboard.addDateCard(
    "schedule", "Schedule Date", false
);

// With time
DateCard* alarmTime = dashboard.addDateCard(
    "alarm", "Alarm Time", true
);
alarmTime->setRange("2024-01-01", "2024-12-31");
alarmTime->setCallback([](const String& value) {
    Serial.printf("Selected: %s\n", value.c_str());
});
```

---

## TimezoneCard

Button to detect and retrieve browser timezone.

### Example

```cpp
TimezoneCard* tzCard = dashboard.addTimezoneCard(
    "timezone", "Browser Timezone", "Detect Timezone"
);

tzCard->setCallback([](const String& tz, int offset, const String& offsetStr) {
    Serial.printf("Timezone: %s (%s)\n", tz.c_str(), offsetStr.c_str());
});
```

---

## StatusCard

Displays a status icon with label and optional message.

### Status Icons

| Icon | Constant | Use |
|------|----------|-----|
| ✓ | `CHECK` | Success, operational |
| ✗ | `ERROR` | Error, failure |
| ⚠ | `WARNING` | Warning, caution |
| ℹ | `INFO` | Information |
| 📶 | `WIFI` | WiFi status |
| ⚡ | `POWER` | Power status |
| 🔄 | `SYNC` | Syncing |
| ☁ | `CLOUD` | Cloud connection |
| 🔒 | `LOCK` | Locked |
| 🔓 | `UNLOCK` | Unlocked |

### Example

```cpp
StatusCard* wifiStatus = dashboard.addStatusCard(
    "wifi-status", "WiFi", StatusIcon::WIFI
);

wifiStatus->setStatus(
    StatusIcon::WIFI, CardVariant::SUCCESS,
    "Connected", WiFi.localIP().toString()
);
```

---

## ConsoleCard

Displays timestamped log entries with debug, info, warning, and error levels.

### Visual Representation

```
┌─────────────────────────────────────────────────────────────────┐
│  System Console                                                 │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ 00:00:01.234  DEBUG  Sensor data collected               │  │
│  │ 00:00:03.456  INFO   System heartbeat #1                 │  │
│  │ 00:00:05.678  WARN   High CPU usage: 75%                 │  │
│  │ 00:00:07.890  ERROR  Temperature exceeded                │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Example

```cpp
ConsoleCard* console = dashboard.addConsoleCard(
    "console", "System Console", 50
);

// Log messages (Serial.println-like API)
dashboard.logDebug("console", "Initializing sensors...");
dashboard.logInfo("console", "WiFi connected");
dashboard.logWarning("console", "Battery low: 15%");
dashboard.logError("console", "Sensor read failed!");

// Clear console
dashboard.clearConsole("console");
```

---

## OTACard

Over-the-Air firmware update with drag-and-drop support.

### Example

```cpp
OTACard* otaCard = dashboard.addOTACard(
    "ota", "Firmware Update", 4  // Max 4MB
);

otaCard->onProgress = [](size_t current, size_t total) {
    Serial.printf("OTA: %d%%\n", (current * 100) / total);
};

otaCard->onComplete = [](bool success) {
    Serial.printf("OTA %s\n", success ? "Success!" : "Failed!");
};
```
