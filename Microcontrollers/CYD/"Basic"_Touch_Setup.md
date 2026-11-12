# ESP32 Cheap Yellow Display Touchscreen Test

This example initializes the CYD display and XPT2046 touchscreen. When the
screen is touched, the touch coordinates are printed to the serial monitor and
the touch location is shown as a red marker.

## `platformio.ini`

```ini
[platformio]
src_dir = .
default_envs = cyd

[env]
platform = espressif32
board = esp32dev
framework = arduino

lib_deps =
    TFT_eSPI
    XPT2046_Touchscreen

monitor_speed = 115200
monitor_filters = esp32_exception_decoder
upload_speed = 921600
board_build.partitions = min_spiffs.csv

build_flags =
    -DUSER_SETUP_LOADED
    -DUSE_HSPI_PORT
    -DTFT_MISO=12
    -DTFT_MOSI=13
    -DTFT_SCLK=14
    -DTFT_CS=15
    -DTFT_DC=2
    -DTFT_RST=-1
    -DTFT_BL=21
    -DTFT_BACKLIGHT_ON=HIGH
    -DSPI_FREQUENCY=55000000
    -DSPI_READ_FREQUENCY=20000000
    -DSPI_TOUCH_FREQUENCY=2500000
    -DLOAD_GLCD
    -DLOAD_FONT2
    -DLOAD_FONT4
    -DLOAD_FONT6
    -DLOAD_FONT7
    -DLOAD_FONT8
    -DLOAD_GFXFF

[env:cyd]
build_flags =
    ${env.build_flags}
    -DILI9341_2_DRIVER

[env:cyd2usb]
build_flags =
    ${env.build_flags}
    -DST7789_DRIVER
    -DTFT_RGB_ORDER=TFT_BGR
    -DTFT_INVERSION_OFF
```

## `main.cpp`

```cpp
#include <SPI.h>
#include <TFT_eSPI.h>
#include <XPT2046_Touchscreen.h>

namespace {
constexpr uint8_t TOUCH_IRQ = 36;
constexpr uint8_t TOUCH_MOSI = 32;
constexpr uint8_t TOUCH_MISO = 39;
constexpr uint8_t TOUCH_CLK = 25;
constexpr uint8_t TOUCH_CS = 33;

constexpr int16_t TOUCH_X_MIN = 200;
constexpr int16_t TOUCH_X_MAX = 3700;
constexpr int16_t TOUCH_Y_MIN = 240;
constexpr int16_t TOUCH_Y_MAX = 3800;
constexpr int16_t MARKER_RADIUS = 10;

XPT2046_Touchscreen touchscreen(TOUCH_CS, TOUCH_IRQ);
TFT_eSPI display;
}

void printTouch(TS_Point point) {
    Serial.printf(
        "Pressure = %d, x = %d, y = %d\n",
        point.z,
        point.x,
        point.y
    );
}

void showTouch(TS_Point point) {
    const int16_t x = map(
        constrain(point.x, TOUCH_X_MIN, TOUCH_X_MAX),
        TOUCH_X_MIN,
        TOUCH_X_MAX,
        0,
        display.width() - 1
    );
    const int16_t y = map(
        constrain(point.y, TOUCH_Y_MIN, TOUCH_Y_MAX),
        TOUCH_Y_MIN,
        TOUCH_Y_MAX,
        0,
        display.height() - 1
    );

    display.fillScreen(TFT_BLACK);
    display.fillCircle(x, y, MARKER_RADIUS, TFT_RED);
}

void setup() {
    Serial.begin(115200);

    SPI.begin(TOUCH_CLK, TOUCH_MISO, TOUCH_MOSI, TOUCH_CS);
    touchscreen.begin();
    touchscreen.setRotation(1);

    display.init();
    display.setRotation(1);
    display.fillScreen(TFT_BLACK);
    display.drawCentreString(
        "Touch Screen to Start",
        display.width() / 2,
        100,
        2
    );
}

void loop() {
    if (!touchscreen.tirqTouched() || !touchscreen.touched()) {
        return;
    }

    const TS_Point point = touchscreen.getPoint();
    printTouch(point);
    showTouch(point);
    delay(100);
}
```

## Notes

- Install the `TFT_eSPI` and `XPT2046_Touchscreen` libraries through
  PlatformIO.
- Select `cyd` for the ILI9341 display variant.
- Select `cyd2usb` for the ST7789 display variant.
- Open the serial monitor at `115200` baud to view touch coordinates.
