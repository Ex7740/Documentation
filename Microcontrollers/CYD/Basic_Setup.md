# ESP32 Cheap Yellow Display (CYD) PlatformIO Setup Guide

Quick reference for setting up the ESP32-2432S028R (CYD) with PlatformIO.

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


build_flags =
    -D USER_SETUP_LOADED=1
    -D USER_SETUP_ID=25
    -include $PROJECT_DIR/include/User_CYD_Setup.h
```
Step 3: Sample test code

```
#include <Arduino.h>

void setup(){

  Serial.begin(115200);
  Serial.println("Hello world");
}

void loop(){
//Place code here to run repeatedly
}
```
