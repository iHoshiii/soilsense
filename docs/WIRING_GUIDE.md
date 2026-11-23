# Hybrid Multi-Sensor Soil Reader wiring

Use this guide with a classic ESP32-WROOM-32 or ESP-32S DevKit. Match the **GPIO labels** on your board. For example, GPIO21 means the pin marked `21` or `IO21`, not the 21st pin in a row. Check the board marking before wiring, especially if your board is from a starter kit. [Espressif DevKit reference](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32/esp32-devkitc/user_guide.html)

## 1. Decide how to power the bench setup

- Start with the ESP32 powered through its USB port. Use its **3V3** pin for the low-current 3.3 V modules and its **GND** pin for their ground. Check the voltage and keep within your board regulator's current rating, including when the SD card writes.
- Use a **regulated 5 V supply** for the DFRobot SEN0161 pH board. Connect its ground to ESP32 GND. Feed its positive wire only to the pH supply branch. Keep the ESP32's own 5 V input free while its USB cable powers the board. Start the ESP32 before powering the pH board, and turn off pH power first when shutting down. Unplug both supplies before changing the wiring. [DFRobot SEN0161 pinout](https://wiki.dfrobot.com/sen0161)
- The LTE modem and the NPK probe have their own power connections below. Run their power wires directly to their supplies. The breadboard carries the low-current connections in this setup.

## 2. Wire it with your breadboard

[Open the current wiring model](../models/blend-files/solar_to_reading.blend). See scene **02 Power and grounding** for the rails and scene **03 Moisture pH and resistors** for the analog inputs.

The middle has **two groups of five holes per row**. For row 6, A6 to E6 are one connection. F6 to J6 are a different connection across the gap. Row 7 is separate from row 6. Here, **L6** means any hole from A6 to E6 and **R6** means any hole from F6 to J6. Put wires that share a connection in different holes of the same group.

1. Put the ESP32 beside the breadboard so its width leaves all the holes free. Use jumper leads from the ESP32 headers into the board.
2. With all supplies unplugged, use a multimeter in continuity mode. Check a five-hole group, then check across the gap. Check each long `+` and `-` strip from end to end. Some strips are split halfway. Bridge segments only when they need to carry the same voltage or ground.
3. Connect ESP32 **3V3** to the left `+` rail and ESP32 **GND** to the left `-` rail. Jumper the left `-` rail to the right `-` rail so both are GND. Connect the pH supply's **5 V** to the right `+` rail and its ground to the right `-` rail. **Keep the two positive rails separate.** Label them **3.3 V** and **5 V**, even if both are printed red. [Breadboard rail and row guide](https://learn.sparkfun.com/tutorials/how-to-use-a-breadboard/anatomy-of-a-breadboard)

Use these example rows or copy the same pattern onto other free rows. For shared SDA, put the GPIO21 wire in **A6**, the OLED SDA wire in **C6**, and the RTC SDA wire in **E6**. The row numbers are for your wiring layout. The GPIO numbers come from the firmware.

| Part | Power | Wire the signals into these groups of holes |
| --- | --- | --- |
| SSD1306 I2C OLED | `VCC` to left `+`, `GND` to left `-` | `SDA` and GPIO21 to **L6**. `SCL` and GPIO22 to **L7**. Some boards mark the I2C clock `SCK`. |
| DS3231 clock (planned) | Board supply to left `+` if rated for 3.3 V, `GND` to left `-` | `SDA` to **L6** and `SCL` to **L7**. The clock and screen share the same two wires. |
| 3.3 V SPI microSD board | `3V` or documented 3.3 V supply to left `+`, `GND` to left `-` | `CLK/SCK` with GPIO18 at **L8**, `SO/MISO` with GPIO19 at **L9**, `SI/MOSI` with GPIO23 at **L10**, and `CS` with GPIO13 at **L11**. |
| DS18B20 probe and adapter | Adapter `VCC` to left `+` (3.3 V), `GND` to left `-` | Adapter `DAT` and GPIO4 to **L12**. Use the adapter's built-in **4.7 kΩ** pull-up if fitted. Otherwise add it from **L12** to left `+`. [Probe colors and adapter checks](SOLAR_TO_READING_GUIDE.md#ds18b20-probe-and-terminal-adapter) |
| DFRobot SEN0161 pH kit | Board `+` to right `+`, board `-` to right `-` | Probe plugs into the board's **BNC** socket. Board pin `A` goes to **L15** and passes through the pH divider below to GPIO34. |
| DFRobot SEN0308 moisture probe | Red to the left **3.3 V** `+` rail. Both black leads to ground. | For direct wiring, connect yellow and GPIO35 to **R15** after the voltage check below. |
| Scan button | One switched contact to GND | Other switched contact to GPIO32. Example group for the GPIO wire is **L25**. |
| View button | One switched contact to GND | Other switched contact to GPIO33. Example group for the GPIO wire is **L29**. |

For each four-leg button, use the meter to pick two legs that are **open when released and connected when pressed**. Mount it across the center gap if it fits. Use jumpers to the two identified contacts if the button has panel wires. The firmware already enables a pull-up on GPIO32 and GPIO33.

Check that OLED and RTC I2C pull-ups go to **3.3 V**. The OLED address in the firmware is `0x3C`. The DS3231 address is `0x68`. Use the backup cell type specified by your exact RTC board. Insert a FAT32 card only with power off. [Adafruit RTC pinout](https://learn.adafruit.com/adafruit-ds3231-precision-rtc-breakout/pinouts) · [Adafruit 3.3 V microSD pinout](https://learn.adafruit.com/adafruit-microsd-spi-sdio/pinouts) · [DS18B20 pull-up](https://support.analog.com/en-US/knowledgebase/article/000094969)

### Wire the pH divider and moisture input

Connect each ESP32 input at the dot shown below. Keep a **100 nF ceramic capacitor** from each input to GND, close to the ESP32 if possible. [ESP32 ADC guidance](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32/schematic-checklist.html#adc) · [DFRobot SEN0161](https://wiki.dfrobot.com/sen0161) · [DFRobot SEN0308](https://wiki.dfrobot.com/sen0308)

```text
pH board A ---- 12 kΩ ----●---- 10 kΩ ---- GND
                          ├--------------- GPIO34
                          └---- 100 nF --- GND

Moisture OUT -------------●--------------- GPIO35
                          └---- 100 nF --- GND
```

| Channel | Row-by-row placement on the breadboard |
| --- | --- |
| pH | Put `A` at **L15**. Put **12 kΩ** from **L15** to **L18**, then **10 kΩ** from **L18** to GND. GPIO34 and the capacitor's first leg also go to **L18**. If you only have kit values, build the 12 kΩ leg with 10 kΩ from L15 to L16, 1 kΩ from L16 to L17, and 1 kΩ from L17 to L18. |
| Moisture | Put yellow and the capacitor's first leg at **R15**. Add GPIO35 to **R15** after the voltage check. |

Put each capacitor's other leg on GND. Before attaching either GPIO, measure its signal point with the black meter lead at GND. With the pH divider above, a **5.0 V** pH output would give about **2.27 V** at the GPIO34 point. For direct moisture wiring, check that the dry and wet reference readings are within about **0.15 to 2.45 V**. Use the optional two-resistor moisture divider if the signal is above **2.45 V** within the sensor's **2.9 V** rating. Investigate any voltage outside the expected range. Switch off before attaching the GPIOs and calibrate with the final wiring in place. [Moisture voltage check and optional divider](SOLAR_TO_READING_GUIDE.md#dfrobot-sen0308-moisture-sensor)

## 3. Wire it without a breadboard

[Open the current pin diagram](../models/images/solar_to_reading.png).

1. Use insulated locking connectors or a soldered perfboard for permanent connections. Make one **3.3 V distribution point**, one **5 V pH distribution point**, and one **common GND point**. An insulated terminal block with linked ports can serve each point. Keep the positive terminals separate and label the wires at both ends.
2. Wire OLED and RTC `SDA` together with GPIO21 and their `SCL` together with GPIO22. Connect the other signals point to point.

   | Signal wire | Direct connection |
   | --- | --- |
   | Card `CLK/SCK`, `SO/MISO`, `SI/MOSI`, `CS` | GPIO18, GPIO19, GPIO23, GPIO13 respectively |
   | DS18B20 adapter `DAT` | **D4** (GPIO4). Adapter `VCC → 3V3` and `GND → GND`. Add **4.7 kΩ** from GPIO4 to 3.3 V only if the adapter has no pull-up. [Probe colors and adapter checks](SOLAR_TO_READING_GUIDE.md#ds18b20-probe-and-terminal-adapter) |
   | pH `A` | Through the 12 kΩ/10 kΩ divider to GPIO34 |
   | Moisture `Signal` | Yellow directly to GPIO35 after the voltage check above. Use the optional divider if the measured signal needs it. |
   | Scan, View | GPIO32 to GND through Scan. GPIO33 to GND through View. |
3. Solder or fasten each divider and capacitor close to the ESP32. Insulate the ADC junctions and exposed resistor legs. Keep probe signal wires away from the modem's power leads and antenna. Attach the cables to the case so a pull on a probe does not pull on the contacts.
4. Use the module's own labeled power pins as listed in the breadboard table. The two optional modules below use the same signal connections in both builds. Route their supply returns straight back to their supplies and connect their grounds to the common GND point for the signals.

## 4. Add the optional modules after the reader can save a scan

### SIM7600G-H SMS modem

- For the **Waveshare SIM7600G-H 4G Module carrier** with default **3.3 V UART logic**, connect modem `TXD` to ESP32 **GPIO16**, modem `RXD` to ESP32 **GPIO17**, and modem `GND` to common GND. Leave `VIO` open for the documented default. Use the exact carrier manual if the label differs. The bare SIM7600 chip and other carriers have different power and logic requirements.
- Use a **dedicated regulated 5 V supply with 3 A capacity** at the carrier's documented 5 V USB input. Use a cable suitable for the load and measure voltage during SMS. Use one input method at a time. Connect the LTE antenna and insert an active SMS-capable SIM before powering it. Follow the board's power-on instructions. The firmware uses **115200 baud** and sends after a confirmed card save. [Waveshare carrier pinout and power inputs](https://www.waveshare.com/wiki/SIM7600G-H_4G_Module)

### MAX3485 and RS485 NPK probe

- For the selected red **TTL485-V2.0 MAX3485 board (Shopee item 51161753113)**, connect `VCC` to **3V3**, `GND` to common **GND**, adapter `TXD` to ESP32 **GPIO26**, and adapter `RXD` to **GPIO27**. The back photo shows the TX/RX crossing. The pictured board handles direction automatically, so use `npkDirection = -1` and leave **GPIO25** free. Check the delivered board's chip and labels before using this map. [Selected adapter](https://shopee.ph/1PCS-MAX3485-Module-TTL-To-RS485-Module-MCU-Development-Accessories-i.293710525.51161753113) · [Front pin labels](https://down-ph.img.susercontent.com/file/7417ca5656a50bcf8e4d1faa3f25ece5) · [Back wiring diagram](https://down-ph.img.susercontent.com/file/6581a7a4387b908612651e32c7ba8b41) · [MAX3485 datasheet](https://www.analog.com/en/products/max3485.html)
- If using a different 3.3 V MAX3485 breakout with **separate `RO`, `DI`, `DE`, and `/RE` pins**, connect `RO` to **GPIO26**, `DI` to **GPIO27**, and join `DE` and `/RE` at **GPIO25**. Set `npkDirection = 25` for that board.
- The seller confirmed probe **brown → separate supply positive**, **black → supply negative**, **yellow → MAX3485 `D+/A`**, and **blue → MAX3485 `D-/B`**. Join supply negative to ESP32 GND. The adapter shares ground through its four-pin header. The probe is rated **5 to 30 V DC** and the model shows a **12 V DC** bench supply. Leave termination and bias as specified for the actual modules and cable. [Probe listing](https://makerlab.ph/products/rs485-output-3pin-probes-soil-npk-sensor-with-cable-2m) · [Complete power and pin map](SOLAR_TO_READING_GUIDE.md#optional-npk-and-sms)
- For a bench read, the factory map in the seller's manual is **4800 baud, 8N1, address `0x01`, function `0x03`, first register `0x0003`**. The ESP32 reads three consecutive registers. The manual describes them as stored or conductivity-based values. Keep NPK disabled for normal scans until you have tested the unit and validated its use. [Seller's reference manual](https://docs.google.com/document/d/1NHSvvIz9n-pFXRNgBlQDSRO91LF0v0aD/edit)

## 5. Switch to the Waveshare solar and battery supply

Bench-test on laptop USB before adding the battery and panel. Connect the **Waveshare Solar Power Manager (D)** to a verified **3.7 V nominal** battery setup, then check for 5 V at its output. Use the white connector with the factory three-slot parallel holder, or the `Bat+` and `Bat-` screws with a verified compatible pack. Keep an unverified two-slot holder disconnected. [Complete battery and solar checks](SOLAR_TO_READING_GUIDE.md#1-solar-panel-power-manager-and-18650) · [Waveshare manual](https://www.waveshare.com/wiki/Solar_Power_Manager_%28D%29)

Unplug PC USB and use the manager's `TYPE-C` in output mode to power the ESP32 through its USB port. Use the manager's `5V OUT` screws for the pH board's 5 V and common GND. Verify the shared 5 V output during a scan and save, then connect verified **6 to 24 V raw DC** from the panel to `SOLAR IN`. Give the modem and NPK probe their separate supplies as shown above.

## 6. Test one part at a time

1. With power off, check that the two positive rails are separate and that GND reaches both sides. With modules unplugged, check for shorts from either supply rail to GND. Power the rails and measure their voltage and polarity before plugging parts in.
2. Connect the OLED and buttons. Boot on USB and check the screen responds to **Scan** and **View**. Add the clock and powered-off microSD card once this works.
3. Connect the temperature sensor, then the moisture and pH sensors. In [sensor_settings.h](../firmware/include/sensor_settings.h), set each installed and checked sensor's `temperatureEnabled`, `moistureEnabled`, or `phEnabled` to `true`. Enter measured pH and moisture calibration points and upload the firmware. Use the serial monitor at **115200 baud** and `read` to check the readings before saving.
4. Take a scan with a usable reading. Confirm the OLED shows that it saved. Power down, remove the card, and open the CSV in the `scans` folder. Then test the optional modem, optional NPK probe and portable supply one at a time. Watch for supply drops or resets during an SD write or SMS attempt. [Firmware setup and operation](../firmware/README.md#configure-and-operate)
