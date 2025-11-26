# GeekMagic S3 Display (No LVGL)

ESPHome configuration for the GeekMagic SmallTV S3 that renders Home Assistant sensor data on the built-in 240×240 ST7789 display without depending on LVGL. A bit more info: [Building a Touch-Controlled Home Assistant Dashboard on an ESP32-S3 (with LVGL & ESPHome)](https://www.vahac.com/how-to-display-home-assistant-sensor-data-on-the-geekmagic-smalltv-s3?utm_source=github)

## Highlights
- Draws a custom dashboard with temperature, humidity, barometric pressure, battery level, and the Home Assistant logo using pure `display.lambda` code.
- Virtual `Text Color` RGB light lets you recolor the interface (and syncs with the real backlight brightness) directly from Home Assistant.
- OTA updates, encrypted ESPHome API, web server v3, and debug sensors are enabled out of the box.
- Uses Google Fonts (`Montserrat` 18/30/48/80pt, 4 bpp) for crisp typography and custom Material Design Icons for weather glyphs.
- Backlight controlled via a 20 kHz LEDC PWM output so it stays silent and flicker-free even at low brightness.

## Hardware & Wiring
- MCU: `esp32-s3-devkitm-1` (the GeekMagic S3 uses this module internally).
- Display: ST7789V (SPI, 240×240, `clk=12`, `mosi=11`, `dc=7`, `reset=6`, `MODE3`, 40 MHz bus, inverted colors).
- Backlight: GPIO14 (PWM, inverted, tied to `Display Backlight` light entity).
- No physical RGB LEDs are required—the template outputs exist purely so Home Assistant can expose a color picker.

## Home Assistant Entities
| Purpose | Entity ID in YAML | Notes |
| --- | --- | --- |
| Temperature | `sensor.outdoor_sensor_temperature` | Displayed with icon + large digits + “°C”. |
| Humidity | `sensor.outdoor_sensor_humidity` | Shows percentage with droplet icon. |
| Pressure | `sensor.outdoor_pressure_mmhg` | Rendered bottom-left in mmHg. |
| Battery | `sensor.outdoor_sensor_battery` | Value + bar graph at bottom right. |

Each sensor’s `on_value` trigger calls `component.update: lcd_display`, so the screen refreshes only when data changes, keeping the ESP32 mostly idle.

## Lights & Outputs
- `Display Backlight` (monochromatic) — physical PWM backlight with `RESTORE_DEFAULT_ON` so the unit wakes up lit.
- `Text Color` (virtual RGB) — choose UI color + brightness in Home Assistant. When it turns on/off or changes brightness, the lambda keeps the backlight in sync and redraws the screen immediately.
- `Current Brightness` template sensor — publishes the backlight level in %, filtered to 1 % steps for clean graphs/automations.

## Secrets & Network
Create the following entries in `secrets.yaml` before compiling:

```yaml
wifi_ssid: "..."
wifi_password: "..."
geekmagic-s3-display-without-lvgl_encryption_key: "api encryption key"
geekmagic-s3-display-without-lvgl_ota_password: "ota password"
```

The config enables a fallback AP called `Geekmagics3 Fallback Hotspot` (password `SnS1zf9rX2i0`) plus the standard ESPHome captive portal for first-time provisioning.

## Build & Flash
1. Install ESPHome (CLI or dashboard) and clone/download this repo.
2. Update `secrets.yaml` with your Wi-Fi/API/OTA credentials and verify the Home Assistant entity IDs match your system.
3. Connect the GeekMagic S3 to USB and run `esphome run geekmagic-s3-display-without-lvgl.yaml` to compile and upload. Future updates can be done OTA.

## Customizing the UI
- Adjust fonts by tweaking the `font:` block (different Google Font or point size).
- Change which sensors are shown by editing their `entity_id` values or adding new ones; remember to update the `display.lambda` drawing code accordingly.
- Modify colors or add theming logic in the lambda (e.g., color temperature text red below 0 °C).
- Because `auto_clear_enabled` is `false`, the lambda redraws every pixel each update—feel free to draw additional widgets without flicker.

## Troubleshooting
- If the screen stays black, confirm the `Text Color` light is ON and the backlight GPIO14 PWM is set to a non-zero brightness.
- OTA/API issues usually stem from a wrong encryption key—double-check the value you pasted into Home Assistant.
- Use the built-in web server or `logger` output (set `logger.level` to `DEBUG`) to inspect incoming sensor states and redraw timing.

For step-by-step photos, enclosure notes, and automation ideas, read the accompanying blog post: https://www.vahac.com/how-to-display-home-assistant-sensor-data-on-the-geekmagic-smalltv-s3?utm_source=github
