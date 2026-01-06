---
layout: default
title: API Reference
nav_order: 4
---

# API Reference

Complete API documentation for ESP-DashboardPlus.

---

## ESPDashboardPlus Class

The main dashboard class that manages cards and WebSocket communication.

### Constructor

```cpp
ESPDashboardPlus(const String& title = "ESP32 Dashboard")
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `title` | `String` | "ESP32-DashboardPlus" | Dashboard title |

### Methods

#### begin()

Initialize the dashboard with PROGMEM HTML data.

```cpp
void begin(AsyncWebServer* server, const uint8_t* htmlData, size_t htmlSize, const String& wsPath = "/ws")
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `server` | `AsyncWebServer*` | - | Pointer to web server |
| `htmlData` | `const uint8_t*` | - | PROGMEM HTML data |
| `htmlSize` | `size_t` | - | HTML data size |
| `wsPath` | `String` | "/ws" | WebSocket path |

#### loop()

Process WebSocket events. Call this in your main loop.

```cpp
void loop()
```

#### setTitle()

Change the dashboard title.

```cpp
void setTitle(const String& title)
```

---

## Card Creation Methods

All methods return a pointer to the created card for further configuration.

### addStatCard()

```cpp
StatCard* addStatCard(const String& id, const String& title, 
                      const String& value = "", const String& unit = "")
```

### addGaugeCard()

```cpp
GaugeCard* addGaugeCard(const String& id, const String& title, 
                        float min = 0, float max = 100, const String& unit = "%")
```

### addChartCard()

```cpp
ChartCard* addChartCard(const String& id, const String& title, 
                        ChartType type = ChartType::LINE, int maxPoints = 20)
```

### addToggleCard()

```cpp
ToggleCard* addToggleCard(const String& id, const String& title, 
                          const String& label = "", bool defaultValue = false)
```

### addSliderCard()

```cpp
SliderCard* addSliderCard(const String& id, const String& title, 
                          int min = 0, int max = 100, int step = 1, 
                          const String& unit = "")
```

### addButtonCard()

```cpp
ButtonCardImpl* addButtonCard(const String& id, const String& title, 
                              const String& label, ButtonCallback cb)
```

### addActionButton()

```cpp
ActionButton* addActionButton(const String& id, const String& title, 
                              const String& label, const String& confirmTitle, 
                              const String& confirmMsg, ButtonCallback cb)
```

### addLinkCard()

```cpp
LinkCard* addLinkCard(const String& id, const String& title, 
                      const String& label, const String& url)
```

### addInputCard()

```cpp
InputCard* addInputCard(const String& id, const String& title, 
                        const String& placeholder = "")
```

### addDropdownCard()

```cpp
DropdownCardImpl* addDropdownCard(const String& id, const String& title, 
                                  const String& placeholder = "Select...")
```

### addColorPickerCard()

```cpp
ColorPickerCard* addColorPickerCard(const String& id, const String& title, 
                                    const String& defaultColor = "#00D4AA")
```

### addDateCard()

```cpp
DateCard* addDateCard(const String& id, const String& title, 
                      bool includeTime = false)
```

### addTimezoneCard()

```cpp
TimezoneCard* addTimezoneCard(const String& id, const String& title, 
                              const String& label = "Get Browser Timezone")
```

### addStatusCard()

```cpp
StatusCard* addStatusCard(const String& id, const String& title, 
                          StatusIcon icon = StatusIcon::INFO)
```

### addConsoleCard()

```cpp
ConsoleCard* addConsoleCard(const String& id, const String& title = "Console Log", 
                            int maxEntries = 100)
```

### addOTACard()

```cpp
OTACard* addOTACard(const String& id = "ota", 
                    const String& title = "Firmware Update (OTA)", 
                    int maxSize = 4)
```

---

## Update Methods

Methods to update card values and broadcast changes to all connected clients.

### updateStatCard()

```cpp
void updateStatCard(const String& id, const String& value)
```

### updateGaugeCard()

```cpp
void updateGaugeCard(const String& id, float value)
```

### updateChartCard()

```cpp
void updateChartCard(const String& id, float value)
```

### updateToggleCard()

```cpp
void updateToggleCard(const String& id, bool value)
```

### updateSliderCard()

```cpp
void updateSliderCard(const String& id, int value)
```

### updateColorCard()

```cpp
void updateColorCard(const String& id, const String& color)
```

### updateDropdownCard()

```cpp
void updateDropdownCard(const String& id, const String& value)
```

### updateDateCard()

```cpp
void updateDateCard(const String& id, const String& value)
```

### updateLinkCard()

```cpp
void updateLinkCard(const String& id, const String& url)
```

### updateStatusCard()

```cpp
void updateStatusCard(const String& id, StatusIcon icon, CardVariant variant, 
                      const String& label, const String& message)
```

---

## Console Logging Methods

Serial.println-like API for logging to ConsoleCard.

### logDebug()

```cpp
void logDebug(const String& consoleId, const String& message)
```

### logInfo()

```cpp
void logInfo(const String& consoleId, const String& message)
```

### logWarning()

```cpp
void logWarning(const String& consoleId, const String& message)
```

### logError()

```cpp
void logError(const String& consoleId, const String& message)
```

### log()

```cpp
void log(const String& consoleId, LogLevel level, const String& message)
```

### clearConsole()

```cpp
void clearConsole(const String& consoleId)
```

---

## Card Management

### getCard()

Get a card by its ID.

```cpp
DashboardCard* getCard(const String& id)
```

### removeCard()

Remove a card from the dashboard.

```cpp
void removeCard(const String& id)
```

---

## Enums

### CardVariant

```cpp
enum class CardVariant {
    PRIMARY,    // Teal/cyan
    SUCCESS,    // Green
    WARNING,    // Yellow/amber
    DANGER,     // Red
    INFO,       // Blue
    SECONDARY   // Gray
};
```

### ChartType

```cpp
enum class ChartType {
    LINE,      // Connected line with points
    AREA,      // Filled area
    BAR,       // Vertical bars
    SCATTER,   // Unconnected points
    STEP       // Staircase line
};
```

### StatusIcon

```cpp
enum class StatusIcon {
    CHECK,     // ✓
    ERROR,     // ✗
    WARNING,   // ⚠
    INFO,      // ℹ
    WIFI,      // 📶
    POWER,     // ⚡
    SYNC,      // 🔄
    CLOUD,     // ☁
    LOCK,      // 🔒
    UNLOCK     // 🔓
};
```

### LogLevel

```cpp
enum class LogLevel {
    DEBUG,     // Gray
    INFO,      // Blue
    WARNING,   // Yellow
    ERROR      // Red
};
```

---

## Callback Types

```cpp
typedef std::function<void()> ButtonCallback;
typedef std::function<void(const String&)> InputCallback;
typedef std::function<void(bool)> ToggleCallback;
typedef std::function<void(int)> SliderCallback;
typedef std::function<void(const String&)> ColorCallback;
typedef std::function<void(const String&)> DropdownCallback;
typedef std::function<void(const String&, int, const String&)> TimezoneCallback;
typedef std::function<void(const String&)> DateCallback;
```
