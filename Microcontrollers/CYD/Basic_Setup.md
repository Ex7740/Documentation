# ESP32 Cheap Yellow Display (CYD) PlatformIO Setup Guide

Quick reference for setting up the ESP32-2432S028R (CYD) with PlatformIO and TFT_eSPI.

---

## Step 1: Create the Project
1. Open **VS Code** with the **PlatformIO** extension.
2. Create a new project:
   - **Board:** `Espressif ESP32 Dev Module` (`esp32dev`)
   - **Framework:** `Arduino`

---

## Step 2: Configure `platformio.ini`
Replace your `platformio.ini` content with the following:

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
upload_speed = 921600

lib_deps =
    bodmer/TFT_eSPI @ ^2.5.43
    sstaub/XPT2046_Touchscreen @ ^1.4.0

build_flags =
    -D USER_SETUP_LOADED=1
    -D USER_SETUP_ID=25
    -include $PROJECT_DIR/include/User_CYD_Setup.h
```
## Step 3: Create Custom Display Setup File
```
#pragma once

#define USER_SETUP_INFO "CYD_ESP32"

// Display Driver
#define ILI9341_DRIVER     

// CYD Hardware Pin Mappings
#define TFT_MISO 12
#define TFT_MOSI 13
#define TFT_SCLK 14
#define TFT_CS   15  
#define TFT_DC    2  
#define TFT_RST  -1  

#define TFT_BL   21  
#define TFT_BACKLIGHT_ON HIGH  

#define TOUCH_CS 33  

// Fonts & Features
#define LOAD_GLCD   
#define LOAD_FONT2  
#define LOAD_FONT4  
#define LOAD_FONT6  
#define LOAD_FONT7  
#define LOAD_FONT8  
#define LOAD_GFXFF  
#define SMOOTH_FONT

// Frequencies
#define SPI_FREQUENCY  27000000
#define SPI_READ_FREQUENCY  20000000
#define SPI_TOUCH_FREQUENCY  2500000
```
Step 4: Sample test code

```
#include <Arduino.h>
#include <SPI.h>
#include <TFT_eSPI.h>

TFT_eSPI tft = TFT_eSPI();

void setup() {
  Serial.begin(115200);
  
  // Turn on backlight
  pinMode(21, OUTPUT);
  digitalWrite(21, HIGH);

  tft.init();
  tft.setRotation(1); // Landscape
  tft.fillScreen(TFT_BLACK);

  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.setTextSize(2);
  tft.setCursor(10, 10);
  tft.println("CYD Ready!");
}

void loop() {
  tft.fillScreen(TFT_RED);
  delay(1000);
  tft.fillScreen(TFT_GREEN);
  delay(1000);
  tft.fillScreen(TFT_BLUE);
  delay(1000);
}
```
