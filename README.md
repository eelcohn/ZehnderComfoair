# Reverse engineering the Zehnder / ComfoAir ventilation remote control

#### Zehnder ComfoAir RF-CMF-0.5HEX receiver
The main ventilation unit has a seperate RF receiver which has a [PIC18F870](https://www.microchip.com/wwwproducts/en/PIC16F870) microcontroller and a [nRF905](https://infocenter.nordicsemi.com/topic/struct_nrf9/struct/nrf905.html) RF transceiver. The board has a [6-pin Molex picoblade](https://www.molex.com/molex/products/part-detail/pcb_headers/0533980671) connector, which carries the ICSP signals from the PIC microcontroller. I've tried downloading code from the PIC using a TL866A programmer, but the PIC seems to be protected. All I got were files filled with 0x00.
There also seems to be an 11-way expansion port on the side.

#### Zenhder ZRF remote control
The [Zehnder RFZ remote control](https://www.zehnder.nl/producten-en-systemen/comfortabele-ventilatie/zehnder-rfz) has a [PIC16F913](https://www.microchip.com/wwwproducts/en/PIC16F913) microcontroller and a [nRF905](https://infocenter.nordicsemi.com/topic/struct_nrf9/struct/nrf905.html) RF transceiver. The board has a 5-pin ICSP connector. I've tried downloading code from the PIC using a TL866A programmer, but the PIC seems to be protected. All I got were files filled with 0x00.

#### RF protocol:
* Frequency: 868MHz (exact frequency: tbd)
* Modulation: tbd (nRF905 uses GFSK (Gaussian Frequency Shift Keying) by default)
* Bitrate: tbd (nRF905 uses 100kbps by default)
* Encoding: tbd (nRF905 uses manchester encoding by default)
* Protocol: tbd

#### Reference links
* [Reverse Engineering Weather Station RF Signals with an RTL-SDR](https://www.rtl-sdr.com/tag/universal-radio-hacker/)
* [Universal Radio Hacker](https://github.com/jopohl/urh)
