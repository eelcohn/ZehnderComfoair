# Reverse engineering the Zehnder / ComfoAir ventilation remote control

Looking for background info about this project? Check out the [Wiki](https://github.com/eelcohn/ZehnderComfoair/wiki)!

### Commands:

#### Command 0x01: ???
I haven't captured any frames of this type yet.

#### Command 0x02: Set power
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type	|
|  05   	| 1 byte	|           	| Receiver ID	|
|  06   	| 1 byte	|           	| Transmitter Type |
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	|`0x02`      	| Command:<br>`0x02`: Set power	|
|  0A   	| 1 byte	|`0x01`     	| Number of parameters:<br>1 parameter	|
|  0B   	| 1 byte	| power      	| Power:<br>`0x01`: low<br>`0x02`: medium<br>`0x03`: high |
|  0C   	| 1 byte	|`0x00`     	| |
|  0D   	| 1 byte	|`0x00`     	| |
|  0E   	| 1 byte	|`0x00`     	| |
|  0F   	| 1 byte	|`0x00`   	  | |
|  10   	| 1 byte	|`0x00`     	| |
|  11   	| 1 byte	|`0x00`     	| |
|  12   	| 1 byte	|`0x00`     	| |
|  13   	| 1 byte	|`0x00`   	  | |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

#### Command 0x03: Set timer
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type |
|  05   	| 1 byte	|           	| Receiver ID |
|  06   	| 1 byte	|           	| Transmitter Type	|
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	|`0x03`      	| Command:<br>`0x03`: Set timer	|
|  0A   	| 1 byte	|`0x02`     	| Number of parameters:<br>2 parameters 	|
|  0B   	| 1 byte	|`0x03`      	| Power:<br>Always `0x03`: high |
|  0C   	| 1 byte	| duration	  | Duration:<br>`0x0A`: 10 minutes<br>`0x1E`: 30 minutes |
|  0D   	| 1 byte	|`0x00`     	| |
|  0E   	| 1 byte	|`0x00`     	| |
|  0F   	| 1 byte	|`0x00`     	| |
|  10   	| 1 byte	|`0x00`   	  | |
|  11   	| 1 byte	|`0x00`     	| |
|  12   	| 1 byte	|`0x00`     	| |
|  13   	| 1 byte	|`0x00`     	| |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

Power is always `0x03`, but my guess is that `0x01` or `0x02` should work as well. Duration is always `0x0A` (10) or `0x1E` (30), but this is probably customizable as well.

#### Command 0x04: ???
To do.
| Offset  | Size   	| Value   	  | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type |
|  05   	| 1 byte	|           	| Receiver ID	|
|  06   	| 1 byte	|           	| Transmitter Type	|
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	|`0xFA`       | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	|`0x04`      	| Command:<br>`0x04`: ?????	|
|  0A   	| 1 byte	|`0x04`   	  | Number of parameters:<br>4 parameters|
|  0B   	| 1 byte	|            	| Network address of main unit MSB |
|  0C   	| 1 byte	|            	| Network address of main unit |
|  0D   	| 1 byte	|             | Network address of main unit |
|  0E   	| 1 byte	|             | Network address of main unit LSB |
|  0F   	| 1 byte	|`0x00`       | |
|  10   	| 1 byte	|`0x00`     	| |
|  11   	| 1 byte	|`0x00`      	| |
|  12   	| 1 byte	|`0x00`       | |
|  13   	| 1 byte	|`0x00`     	| |
|  14-15 	| 2 bytes |             | 16-bit CRC	|

Network address in parameters 1-4 (`0x0B`-`0x0E`) is the same as network address at `0x00-0x03`. This frame is only sent by the RFZ during the linking sequence.

#### Command 0x05: ???
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type	|
|  05   	| 1 byte	|           	| Receiver ID	|
|  06   	| 1 byte	|           	| Transmitter Type	|
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	|`0x05`      	| Command:<br>`0x05`: ???	|
|  0A   	| 1 byte	|`0x03`     	| Number of parameters:<br>3 parameters |
|  0B   	| 1 byte	| ????      	| ????? |
|  0C   	| 1 byte	| ????      	| ????? |
|  0D   	| 1 byte	| ????      	| ????? |
|  0E   	| 1 byte	|`0x00`     	| |
|  0F   	| 1 byte	|`0x00`   	  | |
|  10   	| 1 byte	|`0x00`     	| |
|  11   	| 1 byte	|`0x00`     	| |
|  12   	| 1 byte	|`0x00`     	| |
|  13   	| 1 byte	|`0x00`   	  | |
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
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |`0xA55A5AA5` | Network address:<br>always `0xA55A5AA5`: Default network address for linking |
|  04   	| 1 byte	|            	| Receiver Type	|
|  05   	| 1 byte	|            	| Receiver ID	|
|  06   	| 1 byte	|            	| Transmitter Type	|
|  07   	| 1 byte	|          	  | Transmitter ID |
|  08   	| 1 byte	|`0xFA`       | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	|`0x06`     	| Command:<br>`0x06`: Main unit available for linking	|
|  0A   	| 1 byte	|`0x04` 	    | Number of parameters:<br>4 parameters|
|  0B   	| 1 byte	|            	| Network address of main unit MSB |
|  0C   	| 1 byte	|            	| Network address of main unit |
|  0D   	| 1 byte	|            	| Network address of main unit |
|  0E   	| 1 byte	|            	| Network address of main unit LSB |
|  0F   	| 1 byte	|`0x00`     	| |
|  10   	| 1 byte	|`0x00`   	  | |
|  11   	| 1 byte	|`0x00`    	  | |
|  12   	| 1 byte	|`0x00`      	| |
|  13   	| 1 byte	|`0x00`    	  | |
|  14-15 	| 2 bytes |            	| 16-bit CRC	|

#### Command 0x07: Reply/acknowledge to 0x02 (Set Power) and 0x03 (Set Timer)
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|           	| Receiver Type	|
|  05   	| 1 byte	|           	| Receiver ID	|
|  06   	| 1 byte	|           	| Transmitter Type	|
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	|`0x07`      	| Command:<br>`0x07`: Reply/acknowledge	|
|  0A   	| 1 byte	|`0x04`     	| Number of parameters:<br>4 parameters	|
|  0B   	| 1 byte	| Power      	| Power (same as in command `0x02`/`0x03`)<br>`0x01`: low<br>`0x02`: medium<br>`0x03`: high |
|  0C   	| 1 byte	| ????      	| ????? (% Power?) |
|  0D   	| 1 byte	| TimerFlag  	| Timer flag:<br>`0x00`: Reply to `0x02`: Set Power<br>`0x01`: Reply to `0x03`: Set Timer |
|  0E   	| 1 byte	| ????      	| ????? (Next command = `0x05` ?) |
|  0F   	| 1 byte	|`0x00`   	  | |
|  10   	| 1 byte	|`0x00`     	| |
|  11   	| 1 byte	|`0x00`     	| |
|  12   	| 1 byte	|`0x00`     	| |
|  13   	| 1 byte	|`0x00`   	  | |
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
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |`0xA55A5AA5` | Network address:<br>always `0xA55A5AA5`: Default network address for linking |
|  04   	| 1 byte	|            	| Receiver Type	|
|  05   	| 1 byte	|            	| Receiver ID |
|  06   	| 1 byte	|            	| Transmitter Type |
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	|`0xFA`       | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	|`0x0B`      	| Command:<br>`0x0B`: Linking successful	|
|  0A   	| 1 byte	|`0x00`   	  | Number of parameters:<br>no parameters|
|  0B   	| 1 byte	|`0x00`      	| |
|  0C   	| 1 byte	|`0x00`      	| |
|  0D   	| 1 byte	|`0x00`     	| |
|  0E   	| 1 byte	|`0x00`     	| |
|  0F   	| 1 byte	|`0x00`      	| |
|  10   	| 1 byte	|`0x00`       | |
|  11   	| 1 byte	|`0x00`       | |
|  12   	| 1 byte	|`0x00`      	| |
|  13   	| 1 byte	|`0x00`       | |
|  14-15 	| 2 bytes |            	| 16-bit CRC	|

#### Command 0x0C: RFZ available for linking
When you press the *Timer* button together with one of the other buttons on the RFZ remote control unit, the RFZ will switch to linking mode. It will now transmit a *0x0C: RFZ available for linking* frame every 240ms or so.
| Offset  | Size   	| Value   	  | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address:<br>* either the RFZ's currently linked network address<br>* or `0xA55A5AA5`: default network address for linking |
|  04   	| 1 byte	|            	| Receiver Type |
|  05   	| 1 byte	|            	| Receiver ID	|
|  06   	| 1 byte	|            	| Transmitter Type |
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	|`0xFA`       | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	|`0x0C`      	| Command:<br>`0x0C`: RFZ available for linking	|
|  0A   	| 1 byte	|`0x04`       | Number of parameters:<br>4 parameters	|
|  0B-0E	| 4 byte	|`0xA55A5AA5`	| Network address for linking |
|  0F   	| 1 byte	|`0x00`      	| |
|  10   	| 1 byte	|`0x00`       | |
|  11   	| 1 byte	|`0x00`    	  | |
|  12   	| 1 byte	|`0x00`      	| |
|  13   	| 1 byte	|`0x00`      	| |
|  14-15 	| 2 bytes |            	| 16-bit CRC	|

#### Command 0x0D: ???
To do.
| Offset  | Size   	| Value   	  | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|            	| Receiver Type	|
|  05   	| 1 byte	|            	| Receiver ID	|
|  06   	| 1 byte	|            	| Transmitter Type |
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	|`0xFA`       | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	|`0x0D`      	| Command:<br>`0x0D`: ?????	|
|  0A   	| 1 byte	|`0x00`   	  | Number of parameters:<br>0 parameters|
|  0B   	| 1 byte	|`0x00`      	| |
|  0C   	| 1 byte	|`0x00`      	| |
|  0D   	| 1 byte	|`0x00`     	| |
|  0E   	| 1 byte	|`0x00`      	| |
|  0F   	| 1 byte	|`0x00`      	| |
|  10   	| 1 byte	|`0x00`       | |
|  11   	| 1 byte	|`0x00`       | |
|  12   	| 1 byte	|`0x00`      	| |
|  13   	| 1 byte	|`0x00`       | |
|  14-15 	| 2 bytes |            	| 16-bit CRC	|

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
