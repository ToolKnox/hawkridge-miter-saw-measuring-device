# Miter Saw Measuring Device with Phone Readout (Wemos D1 Mini) - Build Guide

Remix credit: based on the Miter Saw Measuring Device by WarrenW17 (https://www.instructables.com/Miter-Saw-Measuring-Device/, CC BY 4.0), updated/refurbished for modern use with a current parts list, two firmware fixes and a rewritten build guide.

## What you are building

A rubber RC-car wheel rides on top of the board you are cutting. The wheel turns a 600 pulse-per-revolution rotary encoder, a Wemos D1 mini (ESP8266) counts the pulses, and your phone shows the distance live in a web page: inches with 1/16 fractions, or millimetres. A LiPo cell and a charger board make it cordless. The unit hangs on two copper pipes so it can lift and drop with the stock.

Printed parts (PLA or PETG; only the collar needs supports):

- `Miter Measure Case.stl` - main case, holds the encoder, electronics and the flange bearing
- `Miter Measure Cover.stl` - back cover
- `Miter Measure Wheel Hub.stl` - adapter from the encoder shaft to the wheel's 12 mm hex
- `Miter Measure Collar.stl` - collar on one copper pipe
- `Miter Measure XY Mount.stl` - joins the vertical and horizontal pipes
- `Miter Measure.step` - full assembly for reference and edits

## Parts you buy separately

The shop parts are in the Bill of Material PDF. Buy these separately:

- **Adafruit USB LiIon/LiPoly charger, product 259.** The case and wiring follow this board (DCIN, BATT and LOAD pads, on-board charge LEDs). It is an Adafruit product; buy it from Adafruit or an Adafruit distributor.
- **Copper pipe, two pieces**, from a hardware store. The collar and the XY mount are sized for plumbing copper pipe. Take the printed collar and XY mount to the store and choose the pipe that fits them snugly. Cut the pieces to suit your saw.

## Step 1 - Install the software and flash the board first

1. Install the Arduino IDE (2.x).
2. In **File > Preferences > Additional boards manager URLs** add `https://arduino.esp8266.com/stable/package_esp8266com_index.json` and install **esp8266 by ESP8266 Community** in the Boards Manager.
3. Select the board **LOLIN(WEMOS) D1 R2 & mini**.
4. Install the library **WebSockets** by Markus Sattler with the Library Manager. EEPROM, ESP8266WiFi and ESP8266WebServer come with the board package.
5. Open `miter_measure.ino` from the project repository.
6. Edit before upload:
   - Lines 66-67: `ssid` and `password` of your workshop Wi-Fi (2.4 GHz).
   - Lines 69-70: the device's own access point is called `miterwifi` with the password `measureit`. Change the password to your own (8 characters or more) so nobody else can join it.
7. Apply two small fixes to the original sketch. Without them the millimetre readout is wrong and calibration is lost at every power-off:
   - **Line 51:** change `int mmConversionFactor = 3;` to `float mmConversionFactor = 3;`. The sketch divides the inch factor by 25.4 (line 97); stored in an `int`, 2.9 becomes 2 and every millimetre reading comes out about 40 % too large.
   - **After line 267** (`Serial.println(saveconversion);`) add the line `if (saveconversion > 0) inchConversionFactor = saveconversion;`. The original reads the saved calibration from EEPROM at start-up but never uses it, so the device falls back to the built-in value 71 after each power cycle.
8. Upload. Open the serial monitor at **9600 baud** to see the start-up messages and the Wi-Fi connection.

## Step 2 - Wiring

There are two separate circuits.

**Measuring circuit**

| From | To |
|---|---|
| Encoder red | Wemos 5V |
| Encoder black | Wemos G (GND) |
| Encoder green (A) | Wemos D5 (GPIO 14) |
| Encoder white (B) | Wemos D6 (GPIO 12) |
| Blue LED anode (long leg) through a 1 kOhm resistor | Wemos D4 (GPIO 2) |
| Blue LED cathode | Wemos G |

The sketch turns on the internal pull-ups on D5 and D6 and counts on the rising edge of D6. The LPD3806-600BM encoder has NPN open-collector outputs, so the pull-ups hold the signals at 3.3 V and the Wemos pins are safe. Do not use an encoder version with 5 V push-pull (voltage) outputs on these pins without a level shifter.

**Charging and power circuit**

| From | To |
|---|---|
| Micro-USB breakout VCC | Charger DCIN + |
| Micro-USB breakout GND | Charger DCIN - |
| LiPo cell (JST plug) | Charger BATT JST socket |
| Charger LOAD + | one leg of the latching push switch |
| Other switch leg | Wemos 5V |
| Charger LOAD - | Wemos G |

Check the plug polarity of your LiPo cell against the + and - marks next to the charger's JST socket **before** you plug it in. Cells are sold with both polarities; a reversed plug destroys the charger. Swap the pins in the housing if needed.

When the switch is on, the Wemos runs from the cell through its own 3.3 V regulator. The encoder then gets the cell voltage (about 3.7-4.2 V) instead of 5 V. The original build runs this way; if counts drop out when the cell is low, charge it.

Keep wires short (26-24 AWG silicone wire) so everything closes up inside the case.

## Step 3 - Assembly

1. Melt the heat-set inserts in with a soldering iron: four M3 inserts in the case, one M3 insert in the wheel hub and one M4 insert in the bottom of the collar.
2. Push the collar onto one copper pipe. It should be tight; a drop of super glue keeps it from turning.
3. Press the wheel hub into the wheel's 12 mm hex socket, insert first, and fix it from the front of the wheel with an M3 x 6 screw and the M3 washer. Slide the hub onto the encoder shaft.
4. Press the F608ZZ flange bearing into its seat in the case. A couple of drops of super glue are optional.
5. Fit the encoder, the Wemos, the charger, the micro-USB breakout, the LED in its 5 mm holder and the switch into the case.
6. Build the pipe mount: the XY mount joins the vertical and the horizontal pipe. For each pipe it has a hole for an M3 screw and a slot for an M3 nut; drop the nut into the slot and clamp the pipe with the screw. Fit the vertical pipe first, but put the nut and screw for the horizontal pipe in place now. The original parts list has six M3 x 6 screws in total; if 6 mm does not reach the captured nut in your print, use a longer M3 screw there. The collar's M4 insert takes an M4 screw. The original write-up does not describe where each M4 part goes, so place the three M4 x 30 screws, the two M4 nuts and the M4 washer as the `Miter Measure.step` assembly shows them, and tighten only until the pipes cannot slip.
7. Close the case with the cover and the remaining M3 x 6 screws.

## Step 4 - First run

1. Charge the cell through the micro-USB port with the switch off. The charger's own LEDs show charging and done.
2. Switch on. The blue LED blinks while the Wemos connects to your Wi-Fi and stays on once it is connected.
3. Find the device's address: join the Wi-Fi network `miterwifi` with your phone, open `http://10.10.10.1/wifi.htm` and note the IP address shown there. Rejoin your normal Wi-Fi and open `http://<that address>/`. Bookmark it.
4. The page shows the live distance and has three buttons: Reset Distance, Change Unit of Measure and Calibrate Device.

## Step 5 - Calibrate

1. Tap **Calibrate Device**, put a board under the wheel and tap **Next**.
2. Move the board exactly **10 inches (254 mm)** along the fence, measured with a ruler, and tap **Next**.
3. The sketch stores the new counts-per-inch value in EEPROM. With the fix from Step 1 it is reloaded at every start.

Repeat the calibration if you change the wheel or if readings drift. Reset the distance with the board's end against the blade before each measurement.

## Troubleshooting

- Distance counts the wrong way: swap the green and white encoder wires (D5 and D6).
- Reading jumps or misses counts: the wheel is slipping. Check that the unit can drop freely on the pipe so the wheel presses on the stock.
- Blue LED never stops blinking: wrong Wi-Fi name or password, or a 5 GHz-only network.
- Millimetres look about 40 % too large: the `float` fix in Step 1 is missing.

## Files in this project

- Print: the five STL files listed above; `Miter Measure.step` for reference
- Firmware: `miter_measure.ino` in the project repository (apply the two fixes in Step 1)
- Bill of Material PDF with shop links for each part
