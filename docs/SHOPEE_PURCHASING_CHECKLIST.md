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
| 1 | Separate regulated 5 V / 3 A bench supply and suitable USB-C cable | Power the modem through its documented 5 V USB input. Use its own supply during the SMS test and validate a portable modem supply under load. [Waveshare modem power inputs](https://www.waveshare.com/wiki/SIM7600G-H_4G_Module) |

## Check your remaining parts

- The portable reader uses the **Waveshare Solar Power Manager (D)**, Shopee item **51516350811**. Confirm the matching three-slot parallel holder or a compatible 3.7 V pack and check the solar panel's raw DC output. [Selected manager](https://shopee.ph/Waveshare-Solar-Power-Manager-Module-%28D%29-Supports-6V~24V-Solar-Panel-and-Type-C-Power-Adapter-5V-3A-for-Raspberry-Pi-Arduino-i.345394400.51516350811) · [Waveshare manual](https://www.waveshare.com/wiki/Solar_Power_Manager_%28D%29)

- Check for two 100 nF ceramic capacitors for the pH and moisture inputs. Have a 4.7 kΩ resistor available if the DS18B20 adapter has no built-in pull-up. Use a multimeter to check resistor values and supply voltages. [Wiring and materials](SOLAR_TO_READING_GUIDE.md#materials-check)
- The selected NPK adapter is the red **TTL485-V2.0 MAX3485, Shopee item 51161753113**. The separate NPK probe is marked 5 to 30 V DC and RS485. Use the seller-confirmed wire colors and a separate regulated supply for the probe. [Selected adapter](https://shopee.ph/1PCS-MAX3485-Module-TTL-To-RS485-Module-MCU-Development-Accessories-i.293710525.51161753113) · [NPK power and wiring](SOLAR_TO_READING_GUIDE.md#optional-npk-and-sms)
- For the enclosure, check that the OLED, buttons, microSD slot, and power switch are reachable. Allow room for the probe cables and strain relief. Keep the LTE antenna away from the pH signal wire.

Test the OLED and buttons first, then save a scan to microSD and open it in Past Scans. Add the clock if the test needs offline dates and times. Verify text receipt on a phone using a separate modem supply. The [bench test checklist](../firmware/HARDWARE_CHECKLIST.md) covers these checks.
