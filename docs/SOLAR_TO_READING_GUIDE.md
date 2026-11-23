# Hybrid Multi-Sensor Soil Reader

[Open the wiring model](../models/blend-files/solar_to_reading.blend) · [View the full diagram](../models/images/solar_to_reading.png)

![Solar input, power manager terminals, moisture and pH wiring, ESP32 pin map, and scan outputs](../models/images/solar_to_reading.png)

## Materials check

Use this list with the diagram. The LAFVIN kit already lists the ESP32 board, a 0.96-inch OLED, an 830-point breadboard, a micro-USB cable, jumper wires, buttons, and an assortment of 220 Ω, 1 kΩ, and 10 kΩ resistors. Check the OLED in your kit against the wiring table before buying another one. [LAFVIN kit contents](https://lafvintech.com/products/lafvin-basic-starter-kit-for-esp32-esp-32s-wifi-i-ot-development-board-for-arduino-project-learning-kit-with-tutorials)

| Group | Parts or tools to have |
| --- | --- |
| ESP32 bench setup | 1 classic ESP32 DevKit, 1 USB data cable, 1 breadboard, and jumper wires that fit the headers you have. |
| Moisture and pH | 1 SEN0308, 1 SEN0161 interface board and its BNC probe, **two 10 kΩ resistors, one 1 kΩ resistor, and two 100 nF ceramic capacitors**. Use a verified regulated 5 V supply for the SEN0161. Allow two extra 10 kΩ resistors if the moisture signal needs a divider. |
| Soil temperature | 1 waterproof DS18B20 probe, its `DAT` / `VCC` / `GND` terminal adapter, and three jumper wires. Have a **4.7 kΩ** resistor if your adapter has no built-in pull-up. |
| Display and controls | 1 four-pin 128×64 I2C SSD1306 and 2 normally open momentary buttons for Scan and View. |
| Storage and time | 1 compatible 3.3 V SPI microSD breakout and 1 FAT32 microSD card. Add 1 DS3231 and its board-specific backup cell for offline timestamps. Have a PC card reader if needed. |
| Solar and battery | 1 Waveshare Solar Power Manager (D), compatible 6 to 24 V raw DC panel, and a compatible 3.7 V battery setup. The factory option is a three-slot parallel 18650 holder. Verify your holder and cells before charging. |
| Optional NPK | 1 MLE05243 probe, 1 red `TTL485-V2.0` MAX3485 adapter (Shopee item **51161753113**), and a separate regulated supply for the probe. The seller confirmed the four probe wire colors below. [Selected adapter](https://shopee.ph/1PCS-MAX3485-Module-TTL-To-RS485-Module-MCU-Development-Accessories-i.293710525.51161753113) |
| Optional SMS | 1 Waveshare SIM7600G-H carrier, its matching LTE antenna, active SMS-capable nano-SIM, and a separate regulated 5 V / 3 A bench supply with a suitable cable. |
| Final assembly | Enclosure, mounts, insulated leads or terminals, and cable strain relief. Select any master switch or fuse for the actual load and wiring. Make the screen, buttons, microSD, and power connections accessible. |
| Calibration and checks | Multimeter, two pH calibration buffers or powders prepared as labeled, distilled water, pH electrode storage solution, and a clean soil-water sample cup. Use the matching reference instruments when doing thesis validation. |

The portable wiring plan uses the **Waveshare Solar Power Manager (D)** from Shopee item **51516350811**. The listing offers board-only and holder options. Match the board and battery connector you received to the photos. [Selected module](https://shopee.ph/Waveshare-Solar-Power-Manager-Module-%28D%29-Supports-6V~24V-Solar-Panel-and-Type-C-Power-Adapter-5V-3A-for-Raspberry-Pi-Arduino-i.345394400.51516350811) · [Parts and testing list](COMPLETE_MATERIALS_AND_TESTING_BUDGET.md)

## 1. Solar panel, power manager, and 18650

The Waveshare (D) accepts **6 to 24 V DC** at its solar input. It charges a **3.7 V nominal Li-ion** battery with a **4.2 V** charge cutoff. The USB-C port and output screw terminal use the same power manager. Budget **up to 3 A total at 5 V** for this build and verify the output under load. [Waveshare manual](https://www.waveshare.com/wiki/Solar_Power_Manager_%28D%29)

| Waveshare (D) connection | What goes there |
| --- | --- |
| `SOLAR IN` green screws `IN+` and `IN-` | Panel raw DC positive and negative after the voltage check. The DC-002 barrel jack is an alternative for a correctly wired plug. Use one solar input. |
| White `Battery 3.7V` PH2.0 4-pin connector | Plug for the matching factory parallel 18650 holder. Check the holder's polarity and use compatible cells. |
| Green `Bat+` and `Bat-` screws | Alternative battery input for a verified 3.7 V nominal, 4.2 V full pack. Use one battery input. |
| `TYPE-C` | Use an ordinary USB cable to the **ESP32 USB port** in output mode. The same port can instead accept a compatible 5 V USB charger. Unplug PC USB before switching the ESP32 to this supply. |
| Green `5V OUT` screws marked `5V` and `GND` | **pH red `+` → `5V`**, **pH black `−` → `GND`**. Share ground with the ESP32. Verify 5 V at the screw terminal before connecting the pH board. |

**Panel check.** Your panel was listed as **6 V / 10 W**, at the low end of the manager's range. Identify its raw DC leads and check polarity and open-circuit voltage before connecting. Confirm the input stays within **6 to 24 V** under charging load. If it falls below 6 V, use USB-C charging until the panel setup is verified. The panel's USB ports are separate outputs. [Waveshare solar input](https://www.waveshare.com/wiki/Solar_Power_Manager_%28D%29)

**Battery check.** The factory three-slot holder connects the cells in **parallel**, so the voltage stays **3.7 V nominal**. Follow the polarity markings and use matched, undamaged rechargeable cells of the same type and charge state. Keep your separate two-slot holder disconnected until its wiring, polarity, and cell ratings are verified for this charger. A two-cell series holder is **7.4 V nominal** and can damage this input. [Waveshare battery notes](https://www.waveshare.com/wiki/Solar_Power_Manager_%28D%29)

**Shut down before moving wires.** Disconnect the battery, panel, USB charger, PC USB, and any separate supplies. Use ESP32 `3V3` for the 3.3 V modules. Keep that positive rail separate from the pH board's 5 V rail. Waveshare specifies that fast charging and discharging are unavailable when charging through USB-C while drawing power from the screw terminals. [Waveshare usage notes](https://www.waveshare.com/wiki/Solar_Power_Manager_%28D%29) · [ESP32 power options](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32/esp32-devkitc/user_guide.html#power-supply-options)

## 2. Resistors you already have

`b` = brown, `r` = red, `bl` = black. Your ESP32 kit lists **220 Ω, 1 kΩ, and 10 kΩ**. Read each resistor with its tolerance band last. Use a multimeter with power off and the resistor out of the circuit to confirm its value. [Vishay color chart](https://www.vishay.com/docs/49411/resistor_color_code_calculator.pdf)

| Your resistor | Full band colors | Rated value |
| --- | --- | ---: |
| First, `b bl bl r b` | Brown, black, black, red, brown | **10 kΩ = 10,000 Ω**, ±1% |
| Second, `b bl bl b b` | Brown, black, black, brown, brown | **1 kΩ = 1,000 Ω**, ±1% |
| Third, `b bl bl r r` | Read from the other end as red, red, black, black, brown | **Likely 220 Ω**, ±1%. Confirm with the meter. |

**Check the third resistor before using it in a divider.** Its bands can also be read as 10 kΩ ±2% in the original direction. If it is already wired in, unplug power, remove it, and measure it. If it reads near **220 Ω (0.22 kΩ)**, replace it with a verified **10 kΩ** resistor before testing.

**For the pH divider, find two 10 kΩ resistors and one 1 kΩ resistor.** Select each by its measured value. Look for about **10,000 Ω** or **1,000 Ω** on the meter.

## 3. Moisture and pH signal wiring

### DFRobot SEN0308 moisture sensor

**Red → ESP32 `3V3`. Both black wires → ESP32 `GND`. Yellow → `D35` for direct wiring.** With yellow disconnected from `D35`, measure yellow to GND in your dry and wet reference samples. DFRobot rates the output up to **2.9 V**. If your readings stay in about **0.15 to 2.45 V**, use this direct circuit. Espressif recommends the **100 nF** filter from the signal pin to GND. [DFRobot SEN0308 specifications](https://wiki.dfrobot.com/sen0308) · [DFRobot wiring example](https://wiki.dfrobot.com/sen0308/docs/18941) · [ESP32 ADC range and filter](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32/schematic-checklist.html#adc)

```text
Yellow ──────────●──────────── D35
                 └── 100 nF ── GND
```

If the SEN0308's measured output goes above **2.45 V** within its rated range, use the optional divider for better accuracy. Connect yellow through **10 kΩ** to `D35`, and a second **10 kΩ** from `D35` to GND. Keep the capacitor on the `D35` side. The divider halves the voltage. Recheck the voltage across the sample range before calibrating. Switch off before changing the wiring.

Calibrate using stable readings in **millivolts** from your ESP32 with the final wiring. Use the same probe depth and similar soil packing for the dry and wet references. Keep the warning line above the sample. DFRobot's example numbers are raw counts from an Arduino Uno. [DFRobot calibration procedure](https://wiki.dfrobot.com/sen0308/docs/18941#calibration)

### DFRobot SEN0161 pH sensor

Plug the probe into the round BNC socket. **Red (`+`) → regulated 5 V. Black (`−`) → common GND. Blue (`A`) → first 10 kΩ resistor.** `D34` is GPIO34. On the bench, power the ESP32 from PC USB and use a separate 5 V source for the pH board. Join their grounds. Turn the ESP32 on first and the pH supply off first. [DFRobot pH pinout](https://wiki.dfrobot.com/sen0161)

```text
Blue A ── 10 kΩ ── 1 kΩ ──●── 10 kΩ ── GND
                          ├──────────── D34
                          └── 100 nF ── GND
```

**Measure before attaching `D34`.** Leave the GPIO jumper off and power the pH board. With the meter in DC volts, put black on GND and red on the `●` junction. If the board's `A` signal reaches 5.0 V, the junction would read about **2.38 V** with nominal resistors. Investigate a negative reading or a junction above about **2.45 V** before connecting the signal. Turn all power off before attaching the jumper. The **100 nF ceramic** filter is often marked `104`. [ESP32 analog-input guidance](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32/schematic-checklist.html#adc)

## 4. ESP32 pins for the rest of the reader

Use the GPIO number printed on the board, for example `D21`, `21`, or `IO21`. This map follows the classic ESP32 DevKit and the current firmware. Match each actual module's voltage and labels before powering it. [ESP32 pin reference](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32/esp32-devkitc/user_guide.html#header-block) · [Firmware device pins](../firmware/include/config.h) · [Firmware sensor pins](../firmware/include/sensor_settings.h)

| Part | ESP32 signal connections | Power or other connection |
| --- | --- | --- |
| SSD1306 I2C OLED | `SDA → D21`, `SCL` or I2C `SCK → D22` | Matching `VCC → 3V3`, `GND → GND`. Firmware address `0x3C`. |
| DS3231 clock | `SDA → D21`, `SCL → D22` | Use a board rated for `3V3`, including its I2C pull-ups. `GND → GND`. Clock address `0x68`. Use only the backup cell specified for that breakout. |
| 3.3 V SPI microSD board | `SCK → D18`, `MISO/DO → D19`, `MOSI/DI → D23`, `CS → D13` | Use the module's documented 3.3 V power input and GND. Insert the FAT32 card while power is off. |
| DS18B20 temperature probe and terminal adapter | Adapter `DAT → D4` (GPIO4) | Adapter `VCC → 3V3`, `GND → GND`. Follow the wire-color and resistor checks below. |
| Scan and View buttons | Scan `D32 → switch → GND`, View `D33 → switch → GND` | Pick contacts that close when pressed. The firmware enables internal pull-ups. |
| Red TTL485-V2.0 MAX3485 breakout (item 51161753113) | Adapter `TXD → D26`, adapter `RXD → D27` | `VCC → 3V3`, `GND → GND`. The pictured board handles direction automatically. Leave D25 free. |
| Waveshare SIM7600G-H carrier | Modem `TXD → D16/RX2`, modem `RXD → D17/TX2`, `GND → GND` | Separate regulated **5 V / 3 A** bench supply at its documented USB input. Use its default 3.3 V logic with `VIO` open. Insert the SIM and attach the antenna before power-on. |

### DS18B20 probe and terminal adapter

Unplug USB and any other power before wiring. Put the probe's bare ends into the matching green screw terminals and tighten them. Then use the three jumpers from the adapter's pin header to the ESP32. Read the printed labels on both sides of the adapter.

| Probe wire | Green screw terminal | Adapter header pin | ESP32 pin |
| --- | --- | --- | --- |
| Red | `VCC` | `VCC` | `3V3` |
| Black | `GND` | `GND` | `GND` |
| Yellow | `DAT` | `DAT` | `D4` or `GPIO4` |

**Violet, gray, and white are the jumper wires.** Find which color you plugged into each adapter header pin, then connect the other end to the ESP32 pin in the same row. Use a continuity meter to trace the wires if you cannot follow them by eye. Use **3.3 V** for this adapter so its data line is also pulled up to 3.3 V. [Seller's adapter photo](https://down-ph.img.susercontent.com/file/sg-11134201-7rcee-lttejticwthj6e) · [Matching adapter wiring reference](https://www.auscomtech.com.au/support/guides/ds18b20-waterproof-temperature-sensor-probe-with-terminal-adapter-arduino-wiring-code-troubleshooting/)

**Check the resistor on the adapter.** The seller's photo shows `472`, which means **4.7 kΩ**. With the adapter unplugged from both the ESP32 and the probe, set your meter to resistance and measure between `DAT` and `VCC`. If it reads about **4.7 kΩ**, use the adapter as it is. If the adapter has no pull-up, put one **4.7 kΩ** resistor between `DAT` and `3V3`. A four-band 4.7 kΩ resistor is yellow, violet, red, gold. The resistor connects across those two points. The `DAT` wire goes directly to `D4`. [DS18B20 datasheet, Figure 7](https://www.analog.com/media/en/technical-documentation/data-sheets/ds18b20.pdf#page=7)

### Optional NPK and SMS

**NPK adapter.** The selected Shopee listing shows a red `TTL485-V2.0` board with a four-pin header and a blue three-screw block. With the header on your left and the blue block on your right, the front labels read top to bottom as shown below. Check that your delivered board matches these labels and uses a 3.3 V MAX3485 before powering it. [Selected adapter](https://shopee.ph/1PCS-MAX3485-Module-TTL-To-RS485-Module-MCU-Development-Accessories-i.293710525.51161753113) · [Front pin labels](https://down-ph.img.susercontent.com/file/7417ca5656a50bcf8e4d1faa3f25ece5) · [Back wiring diagram](https://down-ph.img.susercontent.com/file/6581a7a4387b908612651e32c7ba8b41) · [MAX3485 datasheet](https://www.analog.com/en/products/max3485.html)

| Side | Printed labels, top to bottom | Connections in the same order |
| --- | --- | --- |
| ESP32 header | `VCC`, `TXD`, `RXD`, `GND` | ESP32 `3V3`, `D26` (RX), `D27` (TX), common `GND`. |
| Blue screw block | `GND`, `D-/B`, `D+/A` | Common ground is already shared through the header. Connect probe blue to `D-/B` and probe yellow to `D+/A`. |

**NPK probe wiring.** The seller confirmed these wire colors. Use a separate regulated DC supply within the probe's marked **5 to 30 V DC** range. The model shows a **12 V DC** bench supply. Switch off both supplies while wiring. [Probe listing](https://makerlab.ph/products/rs485-output-3pin-probes-soil-npk-sensor-with-cable-2m) · [NPK scene in the Blender model](../models/blend-files/test.blend)

| Probe wire | Function | Connection |
| --- | --- | --- |
| Brown | Power positive | Separate regulated supply `+` |
| Black | Ground | Supply `−`. Jumper supply `−` to ESP32 `GND`. |
| Yellow | RS485 A | MAX3485 blue screw terminal `D+/A` |
| Blue | RS485 B | MAX3485 blue screw terminal `D-/B` |

Power the MAX3485 `VCC` from ESP32 `3V3`. The seller's VMS-3002-TR-ECTH NPK-N01 manual specifies **4800 baud, 8N1, address `0x01`, and read function `0x03`** as the factory settings for this read. The three zero-based registers are `0x0003` for N, `0x0004` for P, and `0x0005` for K. The project presets these values in `sensor_settings.h` for a bench check. Check the settings on the unit if it has been reconfigured. [Seller's reference manual, sections 5.1 to 5.4](https://docs.google.com/document/d/1NHSvvIz9n-pFXRNgBlQDSRO91LF0v0aD/edit) · [Firmware settings](../firmware/include/sensor_settings.h)

The probe label says **SOIL NPK STORAGE**. The manual describes the three registers as temporary storage. It says they return conductivity-based values until a value is written, and return the stored value afterward. A valid reply confirms communication. Verify any nutrient values against an independent soil test before using them as soil measurements. [Seller's reference manual, section 5.3](https://docs.google.com/document/d/1NHSvvIz9n-pFXRNgBlQDSRO91LF0v0aD/edit)

**Give the modem its own regulated 5 V / 3 A bench supply.** Test it separately so its current peaks leave the reader's 5 V supply available for the ESP32 and pH board. Validate a portable modem supply under load before field use. [Waveshare carrier manual](https://www.waveshare.com/wiki/SIM7600G-H_4G_Module) · [Project power and signal connections](WIRING_GUIDE.md#4-add-the-optional-modules-after-the-reader-can-save-a-scan)

## 5. Test in this order

1. **PC USB and moisture.** Measure the moisture signal with `D35` disconnected, then switch off before connecting it. With `moistureEnabled = true`, send `read` in Serial Monitor. Record stable **Moisture input** readings in mV for dry and wet references at the same probe depth. Enter both readings and upload for your moisture index. Measure both again if you add or remove the divider. [Sensor settings](../firmware/include/sensor_settings.h)
2. **pH by itself.** Run the powered meter test with `D34` disconnected. In two labeled calibration buffers, record each exact pH and stable junction voltage. Rinse the probe tip between buffers. Connect `D34` with power off once the voltage check passes. Enable `phEnabled`, fill in the measured pH and mV pairs, upload, and verify `read`. Use the final wiring when calibrating. [DFRobot pH procedure](https://wiki.dfrobot.com/sen0161/docs/19898) · [Upload and monitor commands](../command.md)
3. **Temperature, OLED, clock, and storage.** After checking the DS18B20 wiring, set `temperatureEnabled = true` in [sensor_settings.h](../firmware/include/sensor_settings.h), upload, and send `read` in Serial Monitor. Check for a room-temperature reading in °C, then warm the metal tip with your hand and check that the reading goes up. Add the remaining parts one at a time. If installed, set the clock in UTC with `rtc-set`. Use a soil-water sample cup for the liquid pH probe and use the moisture probe in soil. Press **Scan**, check the OLED, then hold **View** and open **Past Scans** to confirm the record is readable. Shut the reader down before removing the microSD card, then check the CSV in its `scans` folder on the PC. [Serial commands](../command.md)
4. **Optional NPK and SMS.** Wire the NPK probe, then enable it only for a read test once the connection and settings are ready. Interpret its reply using the storage note above. Add SMS after the device can save CSVs. Confirm handset receipt separately from modem submission.
5. **Portable power.** With the panel disconnected, verify the Waveshare's 5 V output on a compatible battery. Switch off before moving wires. Replace PC USB with the manager's `TYPE-C` output and move pH power to the green `5V OUT` screws. Reconnect the battery. Check that the output stays at 5 V during a scan and SD write. Budget up to **3 A total** and watch for resets. Add the panel after its input checks pass.

## 6. What happens when you press Scan

**Solar charging or battery → Waveshare Solar Power Manager (D) → ESP32 and sensors → Scan → OLED results and microSD CSV → Past Scans.** A scan saves when it has at least one usable reading and the SD write succeeds. A configured, idle modem may submit an SMS after a successful save. OLED readings and microSD storage work offline. Wi-Fi is optional and Bluetooth pairing is still pending in the firmware. [Firmware capture flow](../firmware/src/main.cpp) · [OLED menu](../firmware/src/display.cpp)

The NPK data path is **probe ↔ MAX3485 (item 51161753113) ↔ ESP32 D26/D27**. The probe uses its separate supply and the ESP32 powers the adapter through `3V3`. [NPK firmware](../firmware/src/npk.cpp)
