# ESP32 firmware

The firmware builds for a classic ESP32-WROOM-32 / ESP-32S development board with exposed GPIOs. Flashing the firmware onto the ESP32 installs the interface in `src/display.cpp`. The ESP32 draws each screen on the SSD1306 over I2C and reads the physical Scan and View buttons. USB serial is available for setup and diagnostics.

## Windows setup and build

Run from the repository root in PowerShell:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\firmware\firmware.ps1 build
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\firmware\firmware.ps1 ports
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\firmware\firmware.ps1 upload -Port COM5
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\firmware\firmware.ps1 monitor -Port COM5
```

Replace COM5 with the detected ESP32 port. Close the serial monitor before uploading. Use a USB data cable. The script uses PlatformIO on PATH or the standard user install at `%USERPROFILE%\.platformio\penv\Scripts\platformio.exe`. The build installs the pinned ESP32 platform and Arduino libraries on first use.

## Preview the OLED on your monitor

From the repository root, open the desktop preview:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\firmware\firmware.ps1 preview
```

Press **Scan** to see a sample measurement and **View** to switch between measurements and the NPK screen. Reopen the preview to see the startup screen. To change the screen text or layout, save your edits in `src/display.cpp`. The preview rebuilds from that file using the same Adafruit font and drawing code as the firmware.

The desktop preview uses simulated sensor and module results. It runs locally and requires no ESP32 or OLED. The launcher looks for Python with Tkinter. If you need a C++ compiler, install the `ziglang` package in the same Python installation as Tkinter. Run the firmware build once to install the local PlatformIO libraries used by the preview.

## Proposed pin map

Use the [step-by-step wiring guide](../docs/WIRING_GUIDE.md) and [current pin diagram](../models/images/solar_to_reading.png) with a classic ESP32-WROOM-32 DevKit. With power off, match the **GPIO numbers** to the labels on the actual board.

| Module | ESP32 connections | Bench notes |
| --- | --- | --- |
| SSD1306 128×64 I2C OLED | SDA 21, SCL 22 | 3.3 V supply and pull-ups. Default address 0x3C. Check the board marking and change the address in `include/config.h` if needed. |
| DS3231 RTC, for offline timestamps | SDA 21, SCL 22 | 3.3 V I2C, address 0x68. Use the cell specified for the chosen breakout. |
| 3.3 V SPI microSD breakout | SCK 18, MISO 19, MOSI 23, CS 13 | Use a 3.3 V-compatible breakout and an 8–32 GB FAT32 card. |
| Scan / View buttons | Scan 32, View 33 | Momentary buttons connect to GND. Internal pull-ups are used. |
| LTE modem UART2 | ESP32 RX 16 ← modem TXD. ESP32 TX 17 → modem RXD. | Use a documented 3.3 V logic carrier. Power modem from a separate regulated supply suitable for bursts, with common ground. |
| DS18B20 | Data 4 | Three-wire mode. Use a 4.7 kΩ pull-up from data to 3.3 V. |
| DFRobot SEN0161 pH interface | ADC1 34 | Power the board at 5 V. Use the pH divider in the wiring guide and measure its output before connecting the ESP32. |
| DFRobot SEN0308 moisture sensor | ADC1 35 | Power at 3.3 V. Connect yellow directly after the voltage check. Use the optional divider if needed. Calibrate with the final wiring. [Moisture wiring](../docs/SOLAR_TO_READING_GUIDE.md#dfrobot-sen0308-moisture-sensor) |
| Provisional RS485 NPK | UART1 RX 26, TX 27 | Red 3.3 V TTL485-V2.0 MAX3485: adapter TXD to ESP32 RX, adapter RXD to ESP32 TX. Uses automatic direction (`npkDirection = -1`). Set `npkDirection = 25` for a manual-direction `DE` and `/RE` breakout wired to GPIO25. |

The OLED and RTC share SDA and SCL. Verify that the I2C pull-ups go to 3.3 V and check the breakouts for installed pull-ups before adding any. For the referenced SPI microSD breakout, `SO` connects to MISO and `SI` to MOSI. [Adafruit DS3231 breakout](https://learn.adafruit.com/adafruit-ds3231-precision-rtc-breakout/pinouts), [Adafruit microSD pinout](https://learn.adafruit.com/adafruit-microsd-spi-sdio/pinouts)

Measure the pH and moisture voltages at their respective ADC inputs. Set the calibration points with the final wiring in place. Check the [optional modem and NPK power connections](../docs/WIRING_GUIDE.md#4-add-the-optional-modules-after-the-reader-can-save-a-scan) before adding those modules.

## Configure and operate

1. Wire and verify sensors individually. Enable the installed inputs in `include/sensor_settings.h`. Enter measured pH calibration points and moisture endpoints. All sensor inputs are disabled in the stock build so an unconnected board reports unavailable readings.
2. The selected probe's preset is **4800 baud, 8N1, address `0x01`, function `0x03`, first register `0x0003`** from the seller's reference. Set `npkEnabled` only when ready for a bench read and verify the settings if the unit has been reconfigured. The registers can return stored or conductivity-based values. The firmware displays and saves the raw values. Validate their meaning before using them as soil nutrient measurements. [Seller's reference manual](https://docs.google.com/document/d/1NHSvvIz9n-pFXRNgBlQDSRO91LF0v0aD/edit)
3. Power on with a FAT32 card present. Set the DS3231 using the serial command `rtc-set YYYY-MM-DDTHH:MM:SSZ` with the actual UTC time. Philippine time is UTC+8. An unset or failed clock produces a blank timestamp and `unknown` quality.
4. Let the probes stabilize under the approved procedure. Press **Scan** to capture once. Press **View** to switch between measurements and raw NPK. The OLED shows the latest attempt from the current session. Use **Scan** for a new attempt after a storage error.
5. To enable SMS, create `firmware/include/secrets.h` from `firmware/include/secrets.example.h` if it is missing. Set `SOIL_SMS_RECIPIENT` to a consenting recipient's international number. Use a provisioned SMS-capable SIM in the powered modem. The OLED reports modem submission status. Verify phone receipt separately.

At 115200 baud with newline, `read` previews sensors without saving or sending SMS, `scan SAMPLE|LOCATION` saves with optional location, and `status` reports clock, storage mount and pH offset. A button scan uses its record UUID as the sample ID. Match the first eight displayed ID characters with the full ID in the exported file and field log. `ph-offset DEVICE_ID OFFSET` preserves the original NVS calibration workflow and applies only to future scans.

## Local files and SMS

The firmware writes one `/scans/<record-id>.csv` per capture with at least one usable channel. It verifies a temporary `.part` file and the renamed CSV before reporting success. Partial files may remain after an interrupted write. Keep them for inspection and import committed CSVs. Turn the device off before swapping or removing the card. No capture erases existing records. Power interruption and card capacity limits still need bench verification.

For each confirmed save, a configured and idle modem gets one SMS submission attempt with the short scan ID, time, pH, moisture index and temperature. A busy modem skips the new text while the new scan remains on the card. A timeout after submission reports unconfirmed and pauses future SMS until restart to avoid misattributing late responses. A modem acknowledgement means submitted. Measure handset receipt and latency separately.

Keep the original `SOIL_DEVICE_ID` to retain a previously saved pH offset.
