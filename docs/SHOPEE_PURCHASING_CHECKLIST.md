# Buying list for the Hybrid Multi-Sensor Soil Reader

Start with the OLED and buttons from your LAFVIN kit. The kit lists a 0.96-inch OLED and six push buttons. Check the screen label and test it on USB before ordering a replacement. Reuse your ESP32, sensors, and MAX3485 adapter. [LAFVIN kit contents](https://lafvintech.com/products/lafvin-basic-starter-kit-for-esp32-esp-32s-wifi-i-ot-development-board-for-arduino-project-learning-kit-with-tutorials)

## Screen, storage, and clock

| Qty | Order or search for | Match before buying |
| --- | --- | --- |
| 1 if needed | DFRobot DFR0486 OLED | SSD1306, 0.96-inch, 128 × 64, I2C. Power from ESP32 `3V3`. The firmware uses address `0x3C`. If testing a kit OLED, check for `VCC`, `GND`, `SDA`, and `SCL` or `SCK` on the board. [DFRobot display](https://wiki.dfrobot.com/dfr0486/) · [I2C address](https://github.com/DFRobot/I2C_Addresses/blob/master/README.md) |
| 1 | Adafruit #4682 microSD breakout | This is the board the card plugs into. It uses 3.3 V for both power and signals. Use SPI with `CLK`, `SO`, `SI`, and `CS`. Match both the supply and signal voltage if buying another board. [Adafruit pinout](https://learn.adafruit.com/adafruit-microsd-spi-sdio/pinouts) |
| 1 | 32 GB microSDHC card, such as SanDisk Ultra 32 GB | Use FAT32. An 8 or 16 GB microSDHC card formatted FAT32 also fits the project. Add a USB microSD card reader if your PC has no suitable reader. [SD Association](https://www.sdcard.org/developers/sd-standard-overview/capacity-sd-sdhc-sdxc-sduc/) |
| 1 each | Adafruit #3013 DS3231 clock and CR1220 coin cell | Add these when you need scan dates and times offline. This board uses a CR1220, sold separately. The clock and OLED share the D21 and D22 wires. Check the required cell if buying a different DS3231 board. [Adafruit clock](https://www.adafruit.com/product/3013) · [Clock pinout](https://learn.adafruit.com/adafruit-ds3231-precision-rtc-breakout/pinouts) |
| 2 | Normally open momentary buttons | Use the kit buttons for Scan and View. Buy panel buttons if you need them for the finished enclosure. |

## Optional SMS

| Qty | Order or search for | Match before buying |
| --- | --- | --- |
| 1 | Waveshare SIM7600G-H 4G Module, global version, with LTE antenna | Choose the pre-soldered version with an SMA antenna connector and matching antenna for easier assembly. Check for a USB-C port, nano-SIM slot, and 3.3 V UART pins. Keep `VIO` open for 3.3 V signals. Modem TXD connects to D16, RXD to D17, and GND to ESP32 GND. [Waveshare module](https://www.waveshare.com/product/sim7600g-h-4g-module.htm) · [Waveshare manual](https://www.waveshare.com/wiki/SIM7600G-H_4G_Module) |
| 1 | Active LTE nano-SIM with outgoing SMS | Choose a carrier that serves the test site. Test an ordinary text to a phone using your exact SIM and modem. When a recipient is set and the modem is idle, the firmware attempts a text after a confirmed microSD save. [SMS firmware](../firmware/src/sms.cpp) |
| 1 | Separate regulated 5 V / 3 A bench supply and suitable USB-C cable | Power the Waveshare carrier through its documented 5 V USB input. The DFRobot DFR0559 output is rated at 5 V / 1 A. Keep the modem on its separate supply and test SMS before choosing a portable modem supply. [Waveshare power inputs](https://www.waveshare.com/wiki/SIM7600G-H_4G_Module) · [DFRobot DFR0559 rating](https://wiki.dfrobot.com/dfr0559) |

## Check your remaining parts

- Check for two 100 nF ceramic capacitors for the pH and moisture inputs. Have a 4.7 kΩ resistor available if the DS18B20 adapter has no built-in pull-up. Use a multimeter to check resistor values and supply voltages. [Wiring and materials](SOLAR_TO_READING_GUIDE.md#materials-check)
- You already have the NPK probe. Its label marks 5 to 30 V DC and RS485. Keep its four wires loose until you have the exact wire chart and can check the probe supply. [NPK adapter model](../models/blend-files/test.blend)
- For the enclosure, check that the OLED, buttons, microSD slot, and power switch are reachable. Allow room for the probe cables and strain relief. Keep the LTE antenna away from the pH signal wire.

Test the OLED and buttons first, then save a scan to microSD and open it in Past Scans. Add the clock if the test needs offline dates and times. Verify text receipt on a phone using a separate modem supply. The [bench test checklist](../firmware/HARDWARE_CHECKLIST.md) covers these checks.
