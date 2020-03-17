# Reverse engineering the Zehnder / ComfoAir ventilation remote control

### RF protocol:
* Frequency: 868440000 Hz
* Modulation: GFSK (Gaussian Frequency Shift Keying) (nRF905 default)
* Bitrate: 100kbps (nRF905 default)
* Encoding: Manchester 1 (nRF905 default)
* Frame size: 6 bit preamble + 22 bytes (the nRF905 adds 6 bytes to the payload: 4 rx address bytes and 2 CRC bytes)
* nRF905 payload size: 16 bytes (from offset `0x04` to `0x13`)

### Frame format:

| Preamble    | Network<br>Address | Rx<br>Type | Rx<br>ID | Tx<br>Type | Tx<br>ID | TTL | Command | Parameter count | Parameters | 16-bit CRC |
|:-----------:|:------------------:|:----------:|:--------:|:----------:|:--------:|:---:|:-------:|:---------------:|:----------------:|-----------:|
| 10-bits<br>1111110101  | 4 bytes         | 1 byte | 1 byte | 1 byte | 1 byte | 1 byte | 1 byte  | 1 byte          | 9 bytes    | 2 bytes    |

##### Transmitter and receiver types:

| Value | Type               |
|:-----:|:-------------------|
| 0x01  | Main Unit          |
| 0x03  | RFZ remote control |

##### Commands:

| Value | Command                         |
|:-----:|:--------------------------------|
| 0x01  | ?                               |
| 0x02  | Set power                       |
| 0x03  | Set timer                       |
| 0x04  | ?                               |
| 0x05  | ?                               |
| 0x06  | Main unit available for linking |
| 0x07  | ?                               |
| 0x08  | ?                               |
| 0x09  | ?                               |
| 0x0A  | ?                               |
| 0x0B  | Linking successful              |
| 0x0C  | RFZ available for linking       |
| 0x0D  | ?                               |

### Commands:

#### Command 0x01: ???
I haven't captured any frames of this type yet.

#### Command 0x02: Set power
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits | 1111110101b | Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type	|
|  05   	| 1 byte	|           	| Receiver ID	|
|  06   	| 1 byte	|           	| Transmitter Type |
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	| 0x02       	| Command:<br>0x02: Set power	|
|  0A   	| 1 byte	| 0x01      	| Number of parameters:<br>1 parameter	|
|  0B   	| 1 byte	| power      	| Power:<br>`0x01`: low<br>`0x02`: medium<br>`0x03`: high |
|  0C   	| 1 byte	| 0x00      	| |
|  0D   	| 1 byte	| 0x00      	| |
|  0E   	| 1 byte	| 0x00      	| |
|  0F   	| 1 byte	| 0x00    	  | |
|  10   	| 1 byte	| 0x00      	| |
|  11   	| 1 byte	| 0x00      	| |
|  12   	| 1 byte	| 0x00      	| |
|  13   	| 1 byte	| 0x00    	  | |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

#### Command 0x03: Set timer
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits | 1111110101b | Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type |
|  05   	| 1 byte	|           	| Receiver ID |
|  06   	| 1 byte	|           	| Transmitter Type	|
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	| 0x03       	| Command:<br>0x03: Set timer	|
|  0A   	| 1 byte	| 0x02      	| Number of parameters:<br>2 parameters 	|
|  0B   	| 1 byte	| 0x03       	| Power:<br>Always `0x03`: high |
|  0C   	| 1 byte	| duration	  | Duration:<br>`0x0A`: 10 minutes<br>`0x1E`: 30 minutes |
|  0D   	| 1 byte	| 0x00      	| |
|  0E   	| 1 byte	| 0x00      	| |
|  0F   	| 1 byte	| 0x00      	| |
|  10   	| 1 byte	| 0x00    	  | |
|  11   	| 1 byte	| 0x00      	| |
|  12   	| 1 byte	| 0x00      	| |
|  13   	| 1 byte	| 0x00      	| |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

Power is always `0x03`, but my guess is that `0x01` or `0x02` should work as well. Duration is always `0x0A` (10) or `0x1E` (30), but this is probably customizable as well.

#### Command 0x04: ???
To do.
| Offset  | Size   	| Value   	  | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits | 1111110101b | Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type |
|  05   	| 1 byte	|           	| Receiver ID	|
|  06   	| 1 byte	|           	| Transmitter Type	|
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	| 0xFA        | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	| 0x04       	| Command:<br>0x04: ?????	|
|  0A   	| 1 byte	| 0x04    	  | Number of parameters:<br>4 parameters|
|  0B   	| 1 byte	|            	| Network address of main unit MSB |
|  0C   	| 1 byte	|            	| Network address of main unit |
|  0D   	| 1 byte	|             | Network address of main unit |
|  0E   	| 1 byte	|             | Network address of main unit LSB |
|  0F   	| 1 byte	| 0x00        | |
|  10   	| 1 byte	| 0x00      	| |
|  11   	| 1 byte	| 0x00       	| |
|  12   	| 1 byte	| 0x00        | |
|  13   	| 1 byte	| 0x00      	| |
|  14-15 	| 2 bytes |             | 16-bit CRC	|

Network address in parameters 1-4 (`0x0B`-`0x0E`) is the same as network address at `0x00-0x03`. This frame is only sent by the RFZ during the linking sequence.

#### Command 0x05: ???
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits | 1111110101b | Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type	|
|  05   	| 1 byte	|           	| Receiver ID	|
|  06   	| 1 byte	|           	| Transmitter Type	|
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	| 0x05       	| Command:<br>0x05: ???	|
|  0A   	| 1 byte	| 0x03      	| Number of parameters:<br>3 parameters |
|  0B   	| 1 byte	| ????      	| ????? |
|  0C   	| 1 byte	| ????      	| ????? |
|  0D   	| 1 byte	| ????      	| ????? |
|  0E   	| 1 byte	| 0x00      	| |
|  0F   	| 1 byte	| 0x00    	  | |
|  10   	| 1 byte	| 0x00      	| |
|  11   	| 1 byte	| 0x00      	| |
|  12   	| 1 byte	| 0x00      	| |
|  13   	| 1 byte	| 0x00    	  | |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

Values seen for parameter 1-3 (`0x0B`-`0x0D`) are:<br>
```
0x49, 0x02, 0x20
0x4A, 0x02, 0x20
0x4B, 0x02, 0x20
0x4C, 0x02, 0x20
0x4D, 0x02, 0x20
0x54, 0x03, 0x20
0x57, 0x03, 0x20
0x5C, 0x03, 0x20
0x5D, 0x03, 0x20
0x5E, 0x03, 0x20
```

#### Command 0x06: Main unit available for linking
When the main unit is powered on it will be available for 10 minutes for linking to remote devices. During these 10 minutes the main unit will transmit the *0x05: Main unit available for linking* frame every 420 ms.
| Offset  | Size   	| Value   	  | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits | 1111110101b | Preamble |
|  00-03  | 4 bytes | 0xA55AA55A  | Network address:<br>always `0xA55AA55A`: Default network address for linking |
|  04   	| 1 byte	|            	| Receiver Type	|
|  05   	| 1 byte	|            	| Receiver ID	|
|  06   	| 1 byte	|            	| Transmitter Type	|
|  07   	| 1 byte	|          	  | Transmitter ID |
|  08   	| 1 byte	| 0xFA        | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	| 0x06       	| Command:<br>0x06: Main unit available for linking	|
|  0A   	| 1 byte	| 0x04   	    | Number of parameters:<br>4 parameters|
|  0B   	| 1 byte	|            	| Network address of main unit MSB |
|  0C   	| 1 byte	|            	| Network address of main unit |
|  0D   	| 1 byte	|            	| Network address of main unit |
|  0E   	| 1 byte	|            	| Network address of main unit LSB |
|  0F   	| 1 byte	| 0x00       	| |
|  10   	| 1 byte	| 0x00    	  | |
|  11   	| 1 byte	| 0x00     	  | |
|  12   	| 1 byte	| 0x00       	| |
|  13   	| 1 byte	| 0x00     	  | |
|  14-15 	| 2 bytes |            	| 16-bit CRC	|

#### Command 0x07: ???
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits | 1111110101b | Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type	|
|  05   	| 1 byte	|           	| Receiver ID	|
|  06   	| 1 byte	|           	| Transmitter Type	|
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	| 0x07       	| Command:<br>0x07: ???	|
|  0A   	| 1 byte	| 0x04      	| Number of parameters:<br>4 parameters	|
|  0B   	| 1 byte	| ????      	| ????? |
|  0C   	| 1 byte	| ????      	| ????? |
|  0D   	| 1 byte	| ????      	| ????? |
|  0E   	| 1 byte	| ????      	| ????? |
|  0F   	| 1 byte	| 0x00    	  | |
|  10   	| 1 byte	| 0x00      	| |
|  11   	| 1 byte	| 0x00      	| |
|  12   	| 1 byte	| 0x00      	| |
|  13   	| 1 byte	| 0x00    	  | |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

Values seen for parameter 1-4 (`0x0B`-`0x0E`) are:<br>
```
0x01, 0x1E, 0x00, 0x05
0x02, 0x32, 0x00, 0x05
0x03, 0x5A, 0x00, 0x05
0x03, 0x5A, 0x01, 0x05
```

#### Command 0x08: ???
I haven't captured any frames of this type yet.

#### Command 0x09: ???
I haven't captured any frames of this type yet.

#### Command 0x0A: ???
I haven't captured any frames of this type yet.

#### Command 0x0B: Linking successful
This package is sent to acknowledge that a device has been successfully linked. The Main Unit will send it first, and the RFZ responds by also sending this frame.
| Offset  | Size   	| Value  	    | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits | 1111110101b | Preamble |
|  00-03  | 4 bytes | 0xA55AA55A  | Network address:<br>always `0xA55AA55A`: Default network address for linking |
|  04   	| 1 byte	|            	| Receiver Type	|
|  05   	| 1 byte	|            	| Receiver ID |
|  06   	| 1 byte	|            	| Transmitter Type |
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	| 0xFA        | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	| 0x0B       	| Command:<br>0x0B: Linking successful	|
|  0A   	| 1 byte	| 0x00    	  | Number of parameters:<br>no parameters|
|  0B   	| 1 byte	| 0x00       	| |
|  0C   	| 1 byte	| 0x00       	| |
|  0D   	| 1 byte	| 0x00      	| |
|  0E   	| 1 byte	| 0x00      	| |
|  0F   	| 1 byte	| 0x00       	| |
|  10   	| 1 byte	| 0x00        | |
|  11   	| 1 byte	| 0x00        | |
|  12   	| 1 byte	| 0x00       	| |
|  13   	| 1 byte	| 0x00        | |
|  14-15 	| 2 bytes |            	| 16-bit CRC	|

#### Command 0x0C: RFZ available for linking
When you press the *Timer* button together with one of the other buttons on the RFZ remote control unit, the RFZ will switch to linking mode. It will now transmit a *0x0C: RFZ available for linking* frame every 240ms or so.
| Offset  | Size   	| Value   	  | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits | 1111110101b | Preamble |
|  00-03  | 4 bytes |             | Network address:<br>* either the RFZ's currently linked network address<br>* or `0xA55AA55A`: default network address for linking |
|  04   	| 1 byte	|            	| Receiver Type |
|  05   	| 1 byte	|            	| Receiver ID	|
|  06   	| 1 byte	|            	| Transmitter Type |
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	| 0xFA        | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	| 0x0C       	| Command:<br>0x0C: RFZ available for linking	|
|  0A   	| 1 byte	| 0x04        | Number of parameters:<br>4 parameters	|
|  0B-0E	| 4 byte	| 0xA55AA55A 	| Network address for linking |
|  0F   	| 1 byte	| 0x00       	| |
|  10   	| 1 byte	| 0x00        | |
|  11   	| 1 byte	| 0x00     	  | |
|  12   	| 1 byte	| 0x00       	| |
|  13   	| 1 byte	| 0x00       	| |
|  14-15 	| 2 bytes |            	| 16-bit CRC	|

#### Command 0x0D: ???
To do.
| Offset  | Size   	| Value   	  | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits | 1111110101b | Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|            	| Receiver Type	|
|  05   	| 1 byte	|            	| Receiver ID	|
|  06   	| 1 byte	|            	| Transmitter Type |
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	| 0xFA        | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	| 0x0D       	| Command:<br>0x0D: ?????	|
|  0A   	| 1 byte	| 0x00    	  | Number of parameters:<br>0 parameters|
|  0B   	| 1 byte	| 0x00       	| |
|  0C   	| 1 byte	| 0x00       	| |
|  0D   	| 1 byte	| 0x00      	| |
|  0E   	| 1 byte	| 0x00       	| |
|  0F   	| 1 byte	| 0x00       	| |
|  10   	| 1 byte	| 0x00        | |
|  11   	| 1 byte	| 0x00        | |
|  12   	| 1 byte	| 0x00       	| |
|  13   	| 1 byte	| 0x00        | |
|  14-15 	| 2 bytes |            	| 16-bit CRC	|

### Analyzing RF signal with a RTL-SDR and Universal Radio Hacker
1. Install [Universal Radio Hacker](https://github.com/jopohl/urh) (URH)
2. Plug your [RTL-SDR](https://www.rtl-sdr.com/) into an USB port
3. Start URH
4. Go to *File* -> *New Project* and create a new project. And adjust the settings as shown below.
![Universal Radio Hacker - New Project](https://github.com/eelcohn/ZehnderComfoair/blob/master/images/URH-NewProject.png)
5. Go to *File* -> *Spectrum Analyzer* and adjust the settings as shown below.
![Universal Radio Hacker - Spectrum Analyzer](https://github.com/eelcohn/ZehnderComfoair/blob/master/images/URH-SpectrumAnalyzer.png)
6. Click *Start* and press a button on the Zehnder ZRF remote control. You should see a couple of pulses in the spectrum analyzer
7. Close the Spectrum Analyzer and open the recorder with *File* -> *Record Signal*. And adjust the settings as shown below.
![Universal Radio Hacker - Record Signal](https://github.com/eelcohn/ZehnderComfoair/blob/master/images/URH-Recorder.png)
8. Make a separate recording of each of the buttons on your remote control
9. Open a signal and click *Autodetect parameters*. *Samples/Symbols* should be 10 if you recorded with a sample rate of 1M (1 million samples per second at a bitrate of 100,000 bits per second makes 10 samples per bit)
![Universal Radio Hacker - Interpretation](https://github.com/eelcohn/ZehnderComfoair/blob/master/images/URH-Interpretation.png)
10. Go to the *Analysis* tab and select *Manchester I* as decoding method
![Universal Radio Hacker - Analysis](https://github.com/eelcohn/ZehnderComfoair/blob/master/images/URH-Analysis.png)
11. Enter *1111110101* (the 10-bit nRF905 preamble bits) in the *Search* box and click *Search*
12. Congratulations! You just found the start of a frame sent by your Zehnder ZRF remote control! Select the first 176 columns after the preamble (176 bits = 22 bytes), select the Hex value and copy/paste them to your favourite text editor

### Units:

#### Zehnder ComfoAir RF-CMF-0.5HEX receiver
The main ventilation unit has a seperate RF receiver which has a [PIC16F870](https://www.microchip.com/wwwproducts/en/PIC16F870) microcontroller and a [nRF905](https://infocenter.nordicsemi.com/topic/struct_nrf9/struct/nrf905.html) RF transceiver. The board has a [6-pin Molex picoblade](https://www.molex.com/molex/products/part-detail/pcb_headers/0533980671) connector, which carries the [ICSP](https://en.wikipedia.org/wiki/In-system_programming) signals from the PIC microcontroller. I've tried downloading code from the PIC using a TL866A programmer, but the PIC seems to be protected. All I got were files filled with 0x00.
There also seems to be an 11-pin expansion port on the side.
![Zenhder ZRF remote controller - receiver layout](https://github.com/eelcohn/ZehnderComfoair/blob/master/images/RF-CMF-0.5HEX.png)

#### Zenhder ZRF remote controller
The [Zehnder RFZ remote control](https://www.zehnder.nl/producten-en-systemen/comfortabele-ventilatie/zehnder-rfz) has a [PIC16F913](https://www.microchip.com/wwwproducts/en/PIC16F913) microcontroller and a [nRF905](https://infocenter.nordicsemi.com/topic/struct_nrf9/struct/nrf905.html) RF transceiver. The board has a 5-pin ICSP connector. I've also tried downloading code from the PIC on the remote control, but it's also protected. Remove the battery from the PCB if you want to try this yourself!
![Zenhder ZRF remote controller - PCB layout](https://github.com/eelcohn/ZehnderComfoair/blob/master/images/ZRF.png)

### Reference links
* [Reverse Engineering Weather Station RF Signals with an RTL-SDR](https://www.rtl-sdr.com/tag/universal-radio-hacker/)
* [Reverse Engineering - Weather Station RF signals with an SDR and URH
](https://docs.google.com/document/d/1yjAO3jTBa9lAFIuiteK_GLWh7-Xk-kSD2d0DUxQe_vU/edit)
* [Using a RTLSDR dongle to validate NRF905 configuration](https://www.embeddedrelated.com/showarticle/548.php)
* [Universal Radio Hacker](https://github.com/jopohl/urh)
* [Spektrum SV Mod: RTL-SDR Spectrum Analyzer Software Now with Improved UI ](https://www.rtl-sdr.com/tag/spektrum/)
* [NRF905 RTL-SDR Decoder](https://www.rtl-sdr.com/nrf905-rtl-sdr-decoder/)
* [tweakers.net - Zehnder comfofan s aansturen via RPi, Arduino](https://gathering.tweakers.net/forum/list_messages/1728169)
* [tweakers.net - DIY: Itho ventilatie aansturen via 868Mhz transmitter](https://gathering.tweakers.net/forum/list_messages/1690945)
* [Zehnder ComfoAir RS232-port protocol description](http://www.see-solutions.de/sonstiges/Protokollbeschreibung_ComfoAir.pdf)
