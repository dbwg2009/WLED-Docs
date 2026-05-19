---
title: Heltec WiFi LoRa 32 V3
hide:
  # - navigation
  # - toc
---

# Heltec WiFi LoRa 32 V3 Board Guide

The Heltec WiFi LoRa 32 V3 is a powerful compact IoT development board featuring the ESP32-S3 microcontroller with integrated OLED display and LoRa capabilities.

## Hardware Specifications

| Specification | Details |
|---|---|
| **Microcontroller** | ESP32-S3FN8 (dual-core, 240MHz) |
| **Flash Memory** | 8MB |
| **RAM** | 320KB SRAM (**NO PSRAM**) |
| **Display** | 0.96" SSD1306 OLED (128x64) |
| **Connectivity** | WiFi 802.11 b/g/n, Bluetooth 5.0, LoRa 915MHz |
| **USB** | USB Type-C (flashing & power) |
| **Power** | 3.3V logic, USB 5V input |
| **GPIO Pins** | 26 (many are multiplexed with special functions) |

### Key Pinout for WLED
| Function | GPIO Pin | Notes |
|---|---|---|
| I2C SDA (OLED) | GPIO 17 | |
| I2C SCL (OLED) | GPIO 18 | |
| Display Reset | GPIO 21 | Hardware reset for display |
| Vext Control | GPIO 36 | Enable/disable power to external devices |
| Data Output | GPIO 39 | Suggested for LED data line |

**Important:** The Heltec V3 has a **switchable power rail (Vext)** controlled via GPIO 36. This allows you to power down the OLED display and other peripherals to save power.

## Getting Started

### Where to Buy
- [Heltec Official Store](https://heltec.org/project/wifi-lora-32-v3/)
- Major electronics distributors (AliExpress, Banggood, etc.)
- [Amazon](https://www.amazon.com/s?k=Heltec+WiFi+LoRa+32+V3)

### Flashing WLED

#### Method 1: Web Installer (Easiest)
1. Visit [WLED Web Installer](https://install.wled.me/)
2. Click "Install" and select the build for your device type
3. Follow the on-screen prompts
4. Connect via USB Type-C and power-cycle after flashing

#### Method 2: PlatformIO (Custom Build)
See [Compiling WLED](/advanced/compiling-wled/) for detailed instructions. When building for Heltec V3:
1. Use `heltec_wifi_lora_32_v3` as your build environment
2. Add display and GPIO flags to `platformio_override.ini` (see Display Setup section below)

### First Boot
After flashing:
1. Device will create a WiFi hotspot `WLED-XXXXXX`
2. Connect to this hotspot and open `http://4.3.2.1` in your browser
3. Configure WiFi credentials for your network
4. Access WLED at the assigned IP address

## OLED Display Setup

### Prerequisites
The OLED display uses the **Four Line Display (FLD) usermod** for WLED. This usermod provides a simple interface showing network status, brightness, and effect information.

### Wiring
No external wiring needed! The display is built-in:
- **SDA:** GPIO 17 (I2C)
- **SCL:** GPIO 18 (I2C)
- **RST:** GPIO 21 (Hardware reset)
- **Vext:** GPIO 36 (Power control)

### Compilation with Display Support

1. **Copy the build environment:**
   - Open `platformio.ini` in the WLED source
   - Find the `[env:heltec_wifi_lora_32_v3]` section and copy it

2. **Create `platformio_override.ini`:**
   ```ini
   [env:heltec_wifi_lora_32_v3]
   extends = env:heltec_wifi_lora_32_v3
   custom_usermods = 
     four_line_display
   build_flags =
     ${env.build_flags}
     -D USE_ALT_DMA_DESC=1
     -D FLD_PIN_RST=21
     -D HELTEC_VEXT_PIN=36
     -D I2C_SDA=17
     -D I2C_SCL=18
   ```

3. **Compile and upload** using PlatformIO

### Configuration via Web UI

After flashing with display support enabled:

1. Go to **Settings → User Interface → 4-Line Display**
2. Configure:
   - **Display Type:** SSD1306 (I2C)
   - **I2C Address:** 0x3C (default for Heltec)
   - **I2C SDA Pin:** 17
   - **I2C SCL Pin:** 18
   - **Rotation:** As desired (0° or 180°)
   - **Clock:** Set to your timezone for accurate time display

3. **Enable brightness control:** Go to **Settings → LED Preferences → Brightness Limiter** to set max brightness (prevents excessive power consumption)

### Troubleshooting Display Issues

**Black screen after flashing:**
- Ensure GPIO 36 (Vext) is controlled properly. The display won't turn on if Vext is not enabled.
- Try power cycling the device
- Check that the correct I2C pins (17 & 18) are configured

**Display shows garbage/corrupted data:**
- Verify I2C address is 0x3C in settings
- Ensure `FLD_PIN_RST=21` is set in build flags
- Try a different rotation setting

**Display works but very dim:**
- Check GPIO 36 voltage level
- Ensure Vext power control is not pulling the pin low
- Adjust contrast in display settings if available

## Special Build Flags

When compiling WLED for Heltec V3, use these important flags:

| Flag | Value | Purpose |
|---|---|---|
| `FLD_PIN_RST` | 21 | Hardware reset pulse for SSD1306 display initialization |
| `HELTEC_VEXT_PIN` | 36 | Enable switchable power rail (Vext) for peripherals |
| `I2C_SDA` | 17 | I2C data line for OLED |
| `I2C_SCL` | 18 | I2C clock line for OLED |

**Note:** Always use the dedicated `heltec_wifi_lora_32_v3` build environment, NOT the generic `esp32_s3_dev` or standard `esp32dev`. This ensures proper PSRAM handling (the V3 has none) and peripheral setup.

## LoRa Module

The Heltec WiFi LoRa 32 V3 includes a built-in LoRa 915MHz module (SX1262). Currently, **WLED does not have native support for LoRa functionality**. The radio is available for advanced users who wish to:
- Build custom sketches alongside WLED
- Use dedicated LoRa libraries (e.g., Arduino-LoRa, RadioLib)
- Create hybrid applications

For LoRa projects, refer to:
- [Heltec LoRa Examples](https://heltec.org/project/wifi-lora-32-v3/)
- [SX1262 Module Documentation](https://www.semtech.com/uploads/documents/DS_SX1261-2_V1.2.pdf)

## Power Considerations

- **USB Power:** Limited to ~500mA (sufficient for controller + OLED)
- **LED Power:** For extensive LED strips, use a separate 5V/12V/24V power supply with proper level shifters
- **Sleep Modes:** The Heltec V3 supports deep sleep (GPIO 36 can cut power to Vext peripherals)
- **Current Draw:** WLED alone: ~80-150mA; with LEDs running: depends on effect complexity and LED count

## Common Use Cases with WLED

1. **Small desk ambience:** 10-30 LEDs powered from USB
2. **Room lighting controller:** Use separate PSU for 5V addressable strips
3. **Portable project:** Compact form factor ideal for battery-powered applications
4. **Audio-reactive setup:** Consider adding a microphone via I2S or analog pins

## Additional Resources

- [Official Heltec Documentation](https://heltec.org/project/wifi-lora-32-v3/)
- [WLED Compiling Guide](/advanced/compiling-wled/)
- [Four Line Display Usermod](https://github.com/wled/WLED/tree/master/usermods/usermod_v2_four_line_display)
- [ESP32-S3 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf)

## Support & Community

- [WLED Discord](https://discord.gg/wled)
- [WLED Forum](https://wled.discourse.group/)
- [GitHub Issues](https://github.com/wled/WLED/issues)
