# Zehnder RF protocol: research plan

Sep 28, 2026 · @Skygod

## Summary

The fastest route to the remaining unknowns is the ComfoAir 350 / WHR930 serial port: its RS-232 protocol carries every RF frame the unit receives and sends, decoded, with Zehnder's own field names. Manuals add three further leads: three different RFZ linking procedures, the RF input's 0-100 scale with installer settings, and the device family (Timer RF, Easy RF, Chrono RF, Hygro RF, CO2 main control and extension sensor, RF repeater) behind the unidentified device types.

- **Best new method:** the RS-232 RF bridge (commands `0x0039`, `0x0040`, `0x003E`, `0x00E5`), see below.
- **Biggest new risk:** after "timer OFF", a fan used with an RFZ goes to setting 1 (low), not its previous speed. The component's timer cancel may do the same.
- **Candidates for `0x0B` and `0x1C`:** RF repeater, Hygro sensor RF, Easy RF, Chrono RF, Hygro-Presence sensor RF.
- **Not the same protocol:** BUVA Q-Stream uses KNX RF ("KNX LINK" button, "KNX Repeater", "KNX Serial Number"). BUVA BoxStream RF does use the Zehnder protocol.

## Sources found

The most useful sources are the ComfoAir RS-232 protocol and the RFZ and Timer RF manuals; the rest confirm device roles and settings.

| Source | What it contributes |
| --- | --- |
| [ComfoAir RS-232 protocol (see-solutions)](http://www.see-solutions.de/sonstiges/Protokollbeschreibung_ComfoAir.pdf) | RF bridge commands with field names ("Lifetime", "Datentyp"), RF status incl. pairing mode, RF timer durations, RF input settings |
| [RFZ manual](https://www.notice-facile.com/nl/handleiding/382213/zehnder+rfz) | Three linking procedures (1, 2 or 3 + timer button, 6 s); LED meanings; timer 10 min short, 30 min long press |
| [Timer RF manual](https://static.webshopapp.com/shops/087300/files/156010934/handleiding-zehnder-stork-timer-bediening-rf.pdf) | 10/30/60 min at highest setting; timer OFF behaviour; "same RF address" procedure; LED confirms reception |
| [Chrono RF installer manual](https://www.lueftungsland.de/static/uploads/pictures/original/other/installateurshandleiding-zehnder-chrono-rf-afstandsbediening_2545011307143.pdf) | WHR menus P850-P856 (RF input 0-100); ComfoFan DIP switches for Chrono; percentages per setting |
| [CO2 RF main control and extension sensor manual](https://space-s.nl/wp-content/uploads/2022/09/asset-handleiding-zehnder-hoofdbediening-co2-rf.pdf) | Master = first paired sensor; output "0%RF-100%RF" proportional-integral around 1050 ppm; 400-2000 ppm range; combination table with Hygro-Presence sensor RF |
| [CO2 sensor RF product sheet](https://intovent.nl/uploads/product_downloads/productblad-bedieningen-zehnder-co2-sensor-rf-039005500-1689595611.pdf) | Sensors talk to each other and to the unit; RF repeater exists |
| ComfoFan S installer manuals V0418 and V1119 (yours) | DIP switch table; RF print is a separate daughterboard; pairing one RF device per power cycle; RF accessory list |
| BUVA Q-Stream installation manual (yours) | Different protocol (KNX RF); useful only as contrast |
| [JoooostB/esphome-zehnder-comfoair-rf](https://github.com/JoooostB/esphome-zehnder-comfoair-rf), [Sanderhuisman/ESPHome-Zehnder-RF](https://raw.githubusercontent.com/Sanderhuisman/ESPHome-Zehnder-RF/master/components/zehnder/zehnder.h) | Same constants as the protocol description; ComfoAir 350/550 users as a source of captures |
| [eelcohn/ZehnderComfoair](https://github.com/eelcohn/ZehnderComfoair) | The protocol description itself; observed value lists for `0x05`, `0x1D`, byte 10 |

## What the sources tell us

Eight of the open questions now have a concrete hypothesis, each testable with the methods below.

| Open question | Hypothesis from the sources | How to test |
| --- | --- | --- |
| Command `0x04` "Current network address?" | RFZ link variant 2 (button 2 + timer, 6 s): "the ventilation unit gets the address of the switch"; Timer RF: 30 min + timer OFF, 6 s | Record each RFZ link variant with an SDR |
| Types `0x0B`, `0x1C` | RF repeater, Hygro sensor RF, Hygro-Presence sensor RF, Easy RF or Chrono RF | Pair or power each device and note its sender type |
| Types `0x18` / `0x19` | Main control CO2 RF (master, first paired) / extension sensor CO2 RF | Pair a second CO2 device and compare |
| Set voltage (`0x01`) from sensors | The unit's RF input, 0-100 %, like a 0-10 V input; WHR menus P850-P856 set mode, setpoint, min, max, polarity | Read `0x009D` and P856 on a WHR while sending `0x01` |
| `0x1D` parameter 3 | A sensor value: CO2 level, or the CO2 controller's output (0-100 % RF, PI around 1050 ppm) | Log against the sensor's LEDs (green < 1200 ppm, orange 1200-1500, red > 1500) |
| `0x05` parameters 1-2 | Link quality (RSSI-like) or battery voltage of the remote | Vary distance; power the RFZ from an adjustable supply |
| Byte 10 of `0x07` (`0x03`, `0x05`, `0x06` seen) | DIP switch settings, read at power-up; or unit type (S R, S CO2, Hygro) | Change one DIP switch per power cycle, query |
| TTL ("Lifetime" in Zehnder's naming) | Hop limit for repeaters (RF repeater, BoxStream "repeater" DIP) | Send low TTLs, watch repeats with an SDR |

Two behaviours matter for the component too:

- **Highest demand wins.** With several controls, "the ventilation system follows the highest ventilation setting".
- **The unit has its own RF timer durations.** WHR units store "RF hoch Zeit kurz / lang" (RF high time short / long); whether the timer frame's minutes override them is untested.

## Corrections needed in the ESPHome component

The protocol findings added on 2026-09-28 would report documented, normal values as unknown; the timer cancel needs a hardware check.

- [ ] **Acknowledgement rule (finding 4):** accept `0x05` parameter 1 `0x49`-`0x5E` with parameter 2 `0x02`/`0x03` and parameter 3 `0x20`; accept `0x1D` parameter 1 `0x05`, `0x11`, `0x76`. Report only values outside these ranges.
- [ ] **Byte 10 rule (finding 2):** known values `0x03`, `0x05`, `0x06`.
- [ ] **Output rule (finding 5):** report only above 105 (10.5 V); raise the Voltage limit to 10.5 V, since values 106-127 are capped at 10.5 V and bit 7 is ignored.
- [ ] **Types `0x0B`, `0x1C`:** seen before; keep reporting, but say "seen before, meaning unknown" in the message.
- [ ] **Timer cancel:** test whether the 0-minute timer with speed 1 leaves the fan at low (see Active experiments, test A1). If it does, send the previous speed instead of 1, or follow the cancel with a Set speed.
- [ ] **Sender rule (finding 7):** if tests show fans sending `0x1D`, allow it.

## Method: the RS-232 RF bridge on ComfoAir 350 / WHR930

On these units the main board runs the RF protocol and an RF module only relays frames over RS-232, so a serial tap shows every frame with the unit's own interpretation. This needs a ComfoAir 350/550, WHR930 or compatible unit: your own, or a volunteer's.

| Command | Direction | Content |
| --- | --- | --- |
| `0x00 0x39` Empfangenes RF Kommando | module → unit | receiver type/ID, sender type/ID, Lifetime (TTL), Datentyp (command), data bytes 1-10 |
| `0x00 0x40` RF Kommando senden | unit → module | the same fields + RF address + control bits: `0x01` repeat previous packet first, `0x02` 250 ms pause before sending, `0x04` receive on sender address |
| `0x00 0x3E` RF Adresse setzen | unit → module | the RF (network) address |
| `0x00 0xE5` RF Status abrufen | PC → unit | RF address, RF ID, module present, self-learning (pairing) mode active |
| `0x00 0xC9` Verzögerung abrufen | PC → unit | includes RF high time short and long (minutes) |
| `0x00 0x9D` Analogwerte abrufen | PC → unit | RF present, control/regulate, polarity, RF min/max/setpoint |

Serial settings: 9600 baud, 8N1, frames start with `07 F0` and end with `07 0F`; the RJ45 on the control board carries 12 V, RX, TX and GND.

What it answers:

1. How the unit builds its replies (`0x07`, `0x05`, `0x1D`) and pairing frames, including the timing flags.
2. Which RF value the unit applies for each `0x01` Set voltage (read `0x009D`, P856 on the display).
3. Whether the timer frame's minutes override the unit's RF high times.

Caution: the [FHEM wiki](https://wiki.fhem.de/wiki/ComfoAir) warns against a PC and a CC Ease both talking to the unit; use the "PC log mode" (`0x00 0x9B` data `0x04`) or listen only.

## Passive experiments (listening only)

These need no transmitting and each can settle one documented unknown in an afternoon.

| # | Experiment | Setup | Settles |
| --- | --- | --- | --- |
| P1 | Long-term SDR log | RTL-SDR + rtl\_433 (`-f 868400000 ... preamble={10}fd4,invert`), days of CSV with timestamps | Timing, repeats, TTL, other networks, pairing traffic |
| P2 | `0x05` parameters vs distance | Press an RFZ button at 1 m, 5 m and behind metal; log the `0x05` frame the RFZ sends after the fan's reply | Link quality vs battery hypothesis (with P3) |
| P3 | `0x05` parameters vs battery | RFZ on an adjustable supply, 2.4-3.2 V | Battery hypothesis |
| P4 | `0x1D` parameter 3 vs CO2 | CO2 sensor in a closed box with a person; note LED colour changes | CO2 level or controller output |
| P5 | Flag bit 1 | Query the fan within 10 min of power-on (pairing window), after it, with/without CO2 sensor, at speed 0 | Pairing window, CO2 control or override |
| P6 | Byte 10 vs DIP switches | Unit unplugged: change one DIP switch, power up, query; restore | Byte 10 meaning |
| P7 | Timer durations | Press Timer RF 10/30/60 and RFZ short/long; log the minutes byte and the fan's countdown | Whether units override the minutes |

The controller already helps here: raw frame logging, Last RX frame, and the protocol-finding notifications.

## Active experiments (sending, own network only)

A1 comes first because it affects the component's timer cancel; the linking captures need a spare fan or a planned re-pairing.

| # | Experiment | Procedure | Settles |
| --- | --- | --- | --- |
| A1 | Timer OFF result | Fan at high via RFZ; start a timer; send the 0-minute timer with speed 1; query. Repeat with speed 3 in the cancel frame | Whether cancel restores the previous speed or applies the speed byte |
| A2 | RFZ linking variants | SDR capture of RFZ "1 + timer", "2 + timer" and "3 + timer" (6 s each) during the fan's pairing window | `0x04`, network-address push, random address |
| A3 | Timer RF "same RF address" | SDR capture of "30 min + timer OFF" (6 s) with fans in pairing mode | Multi-fan address sync |
| A4 | Command scan | Send `0x08`-`0x0A`, `0x0E`, `0x0F`, `0x11`-`0x1C` to your fan's ID, no parameters, one per minute; log replies | Hidden commands |
| A5 | Query with a parameter | `0x10` and `0x0D` with one parameter (0, 1, 2...) | Paged information (hours, filter, version) |
| A6 | Sender type | Same Set speed as type `0x03`, `0x16` and `0x18` | Priority between controls |
| A7 | TTL | Frames with TTL `0x01` and `0x00`; watch repeats on the SDR | Mesh forwarding |
| A8 | Voltage edge values | 101-105, 106-127, 128+ via raw frame; measure the 0-10 V output | Confirms the 10.5 V cap and bit 7 |

Before each session: note the fan's settings and keep the RFZ at hand to restore them.

## Hardware methods

The ComfoFan's RF print is a separate daughterboard (installer manual, "Toegang tot de RF print"), so its connector to the main board is the ComfoFan equivalent of the WHR serial tap.

- **RF print connector.** Identify its pins with the unit unplugged; then log with a logic analyzer (PulseView/sigrok). If it is a UART like the WHR's, the frames may match `0x0039`/`0x0040`. If it is an analogue or PWM signal, the RF print does the protocol itself.
- **SPI between MCU and nRF905** on the RF print or in an RFZ: gives the original configuration (address width, CRC, TX power, payload width) and when the device listens. No firmware readout needed.
- **0-10 V output:** log the fan's control voltage with a multimeter or an ESP32 ADC input, to check presets, the 10.5 V cap and what a cancel really does.
- **Firmware readout:** possible on your own hardware, but often blocked by read protection; the taps above give more for less effort.

Mains safety: the ComfoFan main board carries 230 V. Only work on it unplugged, and log through isolated or battery-powered equipment.

## Crowdsourcing and reporting

The controller's notifications already send users to the issue tracker; an issue template makes their reports comparable.

- **Issue template fields:** frame hex, timestamp, unit model and type (S R, S CO2, Hygro, WHR...), DIP switch or P8xx settings, paired devices and their types, firmware version of the controller.
- **Where to ask for captures:** the issue trackers of the three ESPHome projects above, the tweakers.net threads linked in the protocol description, and WHR/ComfoAir owners with an RS-232 setup (FHEM, openHAB, Home Assistant users).
- **Prefilled report link:** the notifications could link to a new issue with these fields already filled.

## Ground rules

Work only on your own devices and network, and keep transmissions sparse.

- **Own network only.** Never send to network IDs you did not create or own; neighbours' fans may be in range.
- **Duty cycle.** EU rules for this 868 MHz sub-band typically allow about 1 % transmit time (roughly 36 s per hour); rate-limit scans to well below that.
- **Recoverability.** Note settings before each session; linking tests (A2, A3) can change the fan's network address, so plan a re-pairing.
- **Mains.** Unplug before opening any unit (see Hardware methods).
- **Reverse engineering.** Studying your own devices for interoperability is generally accepted in the EU; publishing extracted firmware is not. When in doubt, share observations, not code.

## Suggested order

Fix the component first, then answer the questions that need no transmitting, then the linking and scanning work.

- [ ] Correct the findings rules and the Voltage limit (Corrections)
- [ ] A1: timer OFF result, then fix the cancel if needed
- [ ] P1: start a long-term SDR log
- [ ] P5 and P6: flag bit 1 and byte 10
- [ ] P2-P4 and P7: `0x05`, `0x1D`, timer durations
- [ ] Find a WHR/ComfoAir owner for the RS-232 RF bridge tap
- [ ] Identify the ComfoFan RF print connector
- [ ] A2 and A3: linking captures (plan a re-pairing)
- [ ] A4-A8: command scan, query parameters, sender types, TTL, voltage edges
- [ ] Update the protocol description with each result
