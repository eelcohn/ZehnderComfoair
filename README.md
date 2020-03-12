# Reverse engineering the Zehnder / ComfoAir ventilation remote control

#### Zehnder ComfoAir RF-CMF-0.5HEX receiver
The main ventilation unit has a seperate RF receiver which has a [PIC18F870](https://www.microchip.com/wwwproducts/en/PIC16F870) microcontroller and a [nRF905](https://infocenter.nordicsemi.com/topic/struct_nrf9/struct/nrf905.html) RF transceiver. The board has a [6-pin Molex picoblade](https://www.molex.com/molex/products/part-detail/pcb_headers/0533980671) connector, which carries the ICSP signals from the PIC microcontroller. I've tried downloading code from the PIC using a TL866A programmer, but the PIC seems to be protected. All I got were files filled with 0x00.
There also seems to be an 11-pin expansion port on the side.

#### Zenhder ZRF remote controller
The [Zehnder RFZ remote control](https://www.zehnder.nl/producten-en-systemen/comfortabele-ventilatie/zehnder-rfz) has a [PIC16F913](https://www.microchip.com/wwwproducts/en/PIC16F913) microcontroller and a [nRF905](https://infocenter.nordicsemi.com/topic/struct_nrf9/struct/nrf905.html) RF transceiver. The board has a 5-pin ICSP connector. I've also tried downloading code from the PIC on the remote control, but it's also protected.

#### RF protocol:
* Frequency: 868433000 Hz
* Modulation: GFSK (Gaussian Frequency Shift Keying) (nRF905 default)
* Bitrate: 100kpbs (nRF905 default)
* Encoding: Manchester 1 (nRF905 default)
* Frame format: preliminary results:
```
| 00 | remote controller address
| 01 | remote controller address
| 02 | remote controller address
| 03 | remote controller address
| 04 | receiver address
| 05 | receiver address
| 06 | receiver address
| 07 | receiver address
| 08 | 0xFA
| 09 | ??? (Number of parameters?)
| 0A | Parameter 1: Command:
                    0x01: Set power
                    0x02: Set timer (parameter 2 is always 0x03)
                    0x03: ??? (Reply from fan after 0x01 Set Power?)
                    0x04: ??? (Reply?)
| 0B | Parameter 2: Power:
                    0x01: low
                    0x02: medium
                    0x03: high (always 0x03 if command was 0x02: Timer)
| 0C | Parameter 3: Timer: number of minutes. Always 0x00 if command was 0x01.
| 0D | Parameter 4: ??? (always 0x00 if command was 0x01 or 0x02)
| 0E | 0x00
| 0F | 0x00
| 10 | 0x00
| 11 | 0x00
| 12 | 0x00
| 13 | 0x00
| 14 | CRC
| 15 | CRC
```

#### Analyzing RF signal with a RTL-SDR and Universal Radio Hacker
1. Install [Universal Radio Hacker](https://github.com/jopohl/urh) (URH)
2. Plug your [RTL-SDR](https://www.rtl-sdr.com/) into an USB port
3. Start URH
4. Go to *File* -> *New Project* and create a new project. And adjust the settings as shown below.
![Universal Radio Hacker - New Project](https://github.com/eelcohn/ZehnderComfoair/blob/master/screenshots/URH-NewProject.png)
5. Go to *File* -> *Spectrum Analyzer* and adjust the settings as shown below.
![Universal Radio Hacker - Spectrum Analyzer](https://github.com/eelcohn/ZehnderComfoair/blob/master/screenshots/URH-SpectrumAnalyzer.png)
6. Click *Start* and press a button on the Zehnder ZRF remote control. You should see a couple of pulses in the spectrum analyzer
7. Close the Spectrum Analyzer and open the recorder with *File* -> *Record Signal*. And adjust the settings as shown below.
![Universal Radio Hacker - Record Signal](https://github.com/eelcohn/ZehnderComfoair/blob/master/screenshots/URH-Recorder.png)
8. Make a separate recording of each of the buttons on your remote control
9. Open a signal and click *Autodetect parameters*. *Samples/Symbols* should be 10 if you recorded with a sample rate of 1M
![Universal Radio Hacker - Interpretation](https://github.com/eelcohn/ZehnderComfoair/blob/master/screenshots/URH-Interpretation.png)
10. Go to the *Analysis* tab and select *Manchester I* as decoding method
![Universal Radio Hacker - Analysis](https://github.com/eelcohn/ZehnderComfoair/blob/master/screenshots/URH-Analysis.png)
11. Enter *1111110101* (the 10-bit nRF905 preamble bits) in the *Search* box and click *Search*
12. Congratulations! You just found the start of a frame sent by your Zehnder ZRF remote control! Select the first 176 columns after the preamble (176 bits = 22 bytes), select the Hex value and copy/paste them to your favourite text editor

#### Reference links
* [Reverse Engineering Weather Station RF Signals with an RTL-SDR](https://www.rtl-sdr.com/tag/universal-radio-hacker/)
* [Reverse Engineering - Weather Station RF signals with an SDR and URH
](https://docs.google.com/document/d/1yjAO3jTBa9lAFIuiteK_GLWh7-Xk-kSD2d0DUxQe_vU/edit)
* [Using a RTLSDR dongle to validate NRF905 configuration](https://www.embeddedrelated.com/showarticle/548.php)
* [Universal Radio Hacker](https://github.com/jopohl/urh)
* [Spektrum SV Mod: RTL-SDR Spectrum Analyzer Software Now with Improved UI ](https://www.rtl-sdr.com/tag/spektrum/)
* [NRF905 RTL-SDR Decoder](https://www.rtl-sdr.com/nrf905-rtl-sdr-decoder/)
* [tweakers.net - Zehnder comfofan s aansturen via RPi, Arduino](https://gathering.tweakers.net/forum/list_messages/1728169)
* [tweakers.net - DIY: Itho ventilatie aansturen via 868Mhz transmitter](https://gathering.tweakers.net/forum/list_messages/1690945)
