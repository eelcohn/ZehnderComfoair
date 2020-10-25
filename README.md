# Reverse engineering the ComfoFan ventilation RF-protocol

ComfoFan is a house ventilation model sold under several brands like Zehnder, StorkAir, BUVA and others.

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

| Value | Type                    |
|:-----:|:------------------------|
|`0x00` | Broadcast ?             |
|`0x01` | Main Unit               |
|`0x03` | RFZ remote control      |
|`0x04` | ???                     |
|`0x0B` | ? sensor like the CO2 ? |
|`0x16` | Timer RF remote control |
|`0x18` | CO2 RF sensor           |
|`0x19` | CO2 slave monitor?      |
|`0x1C` |                         |

##### Commands:

| Value | Command                         |
|:-----:|:--------------------------------|
|`0x01` | Set voltage                     |
|`0x02` | Set speed                       |
|`0x03` | Set timer                       |
|`0x04` | Current network address ?       |
|`0x05` | Reply to set speed / timer ?    |
|`0x06` | Main unit available for linking |
|`0x07` | Current fan settings            |
|`0x0B` | Linking successful              |
|`0x0C` | RFZ available for linking       |
|`0x0D` | Query device with broadcast ?   |
|`0x10` | Query device ?                  |
|`0x1D` | Reply to set voltage ?          |

### Commands:

#### Command 0x01: Set voltage
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|`0x01`     	| Receiver Type:<br>`0x01`set voltage on a fan unit |
|  05   	| 1 byte	|`0x00`     	| Receiver ID:<br>`0x00`: broadcast to all fans<br>`ID`: target one specific fan |
|  06   	| 1 byte	|           	| Transmitter Type |
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	|`0x01`      	| Command:<br>`0x01`: Set voltage	|
|  0A   	| 1 byte	|`0x01`     	| Number of parameters:<br>1 parameter	|
|  0B   	| 1 byte	| Voltage   	| Fan voltage:<br>`0x00`: 0.0 volt (0x00 is 0d)<br>`0x1E`: 3.0 volt (0x1E is 30d)<br>`0x32`: 5.0 volt (0x32 is 50d)<br>`0x5A`: 9.0 volt (0x5A is 90d)<br>`0x64`: 10.0 volt (0x64 is 100d) |
|  0C   	| 1 byte	|`0x00`     	| |
|  0D   	| 1 byte	|`0x00`     	| |
|  0E   	| 1 byte	|`0x00`     	| |
|  0F   	| 1 byte	|`0x00`   	  | |
|  10   	| 1 byte	|`0x00`     	| |
|  11   	| 1 byte	|`0x00`     	| |
|  12   	| 1 byte	|`0x00`     	| |
|  13   	| 1 byte	|`0x00`   	  | |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

Receiver ID at offset `0x05` is always `0x00`: broadcast, but you can also target a specific fan.

Voltage at offset 0B is in the range from 0x00 - 0x7F, which is 000 - 127 decimal, and represents 0.0V to 12.7V. The maximum output voltage to the fan seems to be 10.5V, so all values from 106 to 127 result in an output voltage of 10.5V. Values from 0x80 to 0xFF (128 - 255 decimal) result in the same values from 0x00-0x7F. The MSB bit 7 therefore seems to be ignored.

#### Command 0x02: Set speed
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|`0x01`     	| Receiver Type:<br>`0x01`set speed on a fan unit |
|  05   	| 1 byte	|`0x00`     	| Receiver ID:<br>`0x00`: broadcast to all fans<br>`ID`: target one specific fan |
|  06   	| 1 byte	|           	| Transmitter Type |
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	|`0x02`       | Command:<br>`0x02`: Set speed |
|  0A   	| 1 byte	|`0x01`     	| Number of parameters:<br>1 parameter	|
|  0B   	| 1 byte	| speed      	| Fan speed setting:<br>`0x01`: low<br>`0x02`: medium<br>`0x03`: high<br>`0x04`: max |
|  0C   	| 1 byte	|`0x00`     	| |
|  0D   	| 1 byte	|`0x00`     	| |
|  0E   	| 1 byte	|`0x00`     	| |
|  0F   	| 1 byte	|`0x00`   	  | |
|  10   	| 1 byte	|`0x00`     	| |
|  11   	| 1 byte	|`0x00`     	| |
|  12   	| 1 byte	|`0x00`     	| |
|  13   	| 1 byte	|`0x00`   	  | |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

Receiver ID at offset `0x05` is always `0x00`: broadcast, but you can also target a specific fan.

#### Command 0x03: Set timer
To do.
| Offset  | Size   	| Value     	| Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|`0x01`     	| Receiver Type:<br>`0x01`set timer on a fan unit |
|  05   	| 1 byte	|`0x00`     	| Receiver ID:<br>`0x00`: broadcast to all fans<br>`ID`: target one specific fan |
|  06   	| 1 byte	|           	| Transmitter Type	|
|  07   	| 1 byte	|           	| Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	|`0x03`      	| Command:<br>`0x03`: Set timer	|
|  0A   	| 1 byte	|`0x02`     	| Number of parameters:<br>2 parameters 	|
|  0B   	| 1 byte	| speed      	| Fan speed setting:<br>`0x01`: low<br>`0x02`: medium<br>`0x03`: high<br>`0x04`: max |
|  0C   	| 1 byte	| duration	  | Duration:<br>`0x00`: Reset timer<br>`0x0A`: 10 minutes<br>`0x1E`: 30 minutes<br>`0x3C`: 60 minuten |
|  0D   	| 1 byte	|`0x00`     	| |
|  0E   	| 1 byte	|`0x00`     	| |
|  0F   	| 1 byte	|`0x00`     	| |
|  10   	| 1 byte	|`0x00`   	  | |
|  11   	| 1 byte	|`0x00`     	| |
|  12   	| 1 byte	|`0x00`     	| |
|  13   	| 1 byte	|`0x00`     	| |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

Receiver ID at offset `0x05` is always `0x00`: broadcast, but you can also target a specific fan.

For the official RF remote control the speed is always `0x03`, but you can use custom values like `0x01` or `0x02` as well. Or `0x04` in case of a Timer RF control. 
Duration is always `0x0A` (10) or `0x1E` (30) for the official ZRF, but this is customizable as well.

#### Command 0x04: Current network address ?
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

#### Command 0x05: Reply to set speed / timer ?
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
0x50, 0x02, 0x20
0x52, 0x03, 0x20
0x51, 0x03, 0x20
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

#### Command 0x07: Current fan settings
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
|  0B   	| 1 byte	| speed      	| Current speed setting:<br>`0x01`: low<br>`0x02`: medium<br>`0x03`: high<br>`0x04`: max |
|  0C   	| 1 byte	| voltage   	| Current fan voltage:<br>`0x00`: 0.0 volt<br>`0x1E`: 3.0 volt<br>`0x32`: 5.0 volt<br>`0x5A`: 9.0 volt<br>`0x64`: 10.0 volt |
|  0D   	| 1 byte	| flags     	| Timer flag:<br>`0x00`: timer is off (set after `0x02`: Set speed)<br>`0x01`: timer is on (set after `0x03`: Set Timer)<br>`0x02`: ??? |
|  0E   	| 1 byte	| ????      	| ????? |
|  0F   	| 1 byte	|`0x00`   	  | |
|  10   	| 1 byte	|`0x00`     	| |
|  11   	| 1 byte	|`0x00`     	| |
|  12   	| 1 byte	|`0x00`     	| |
|  13   	| 1 byte	|`0x00`   	  | |
|  14-15 	| 2 bytes |         	  | 16-bit CRC	|

Parameter 3 at offset `0x0D` seems to be a flag byte. Values seen are: `0x00`, `0x01`, `0x02`<br>
bit 0: 0=timer is off / 1 = timer is on<br>
bit 1: ???<br>
bit 2: ???<br>
bit 3: ???<br>
bit 4: ???<br>
bit 5: ???<br>
bit 6: ???<br>
bit 7: ???<br>

Parameter 4 at offset `0x0E` is still unknown. It seems to be always the same value. I have seen different values on different ComfoFan's, so it might indicate the type of Comfofan,  software version, the dip-switch settings or maybe some other settings? Values seen are:<br>
`0x03`: ?<br>
`0x05`: seen on Comfofan CFMe R<br>
`0x06`: ?<br>

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

#### Command 0x10: Query device
Query device (fan) for last known settings. The fan unit will reply with `0x07`
| Offset  | Size   	| Value   	  | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|            	| Receiver Type	|
|  05   	| 1 byte	|            	| Receiver ID	|
|  06   	| 1 byte	|            	| Transmitter Type |
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	|`0xFA`       | Time-To-Live (always `0xFA`) |
|  09   	| 1 byte	|`0x10`      	| Command:<br>`0x10`: Query device	|
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

#### Command 0x1D: Reply to set voltage ?
To do.
| Offset  | Size   	| Value   	  | Description 	|
|:------: |:------:	|:-----------:|-------------	|
|         | 10 bits |`1111110101b`| Preamble |
|  00-03  | 4 bytes |             | Network address |
|  04   	| 1 byte	|`0x00`       | Receiver Type: always `0x00`: broadcast |
|  05   	| 1 byte	|`0x00`       | Receiver ID: always `0x00`: broadcast	|
|  06   	| 1 byte	|            	| Transmitter Type |
|  07   	| 1 byte	|             | Transmitter ID |
|  08   	| 1 byte	|             | Time-To-Live |
|  09   	| 1 byte	|`0x1D`      	| Command:<br>`0x1D`: Last known configuration?	|
|  0A   	| 1 byte	|`0x03`   	  | Number of parameters:<br>3 parameters|
|  0B   	| 1 byte	|`0x76`      	| ??? |
|  0C   	| 1 byte	| voltage   	| Fan voltage:<br>`0x00`: 0.0 volt (0x00 is 0d)<br>`0x1E`: 3.0 volt (0x1E is 30d)<br>`0x32`: 5.0 volt (0x32 is 50d)<br>`0x5A`: 9.0 volt (0x5A is 90d)<br>`0x64`: 10.0 volt (0x64 is 100d) |
|  0D   	| 1 byte	|`0x28`     	| ??? |
|  0E   	| 1 byte	|`0x00`      	| |
|  0F   	| 1 byte	|`0x00`      	| |
|  10   	| 1 byte	|`0x00`       | |
|  11   	| 1 byte	|`0x00`       | |
|  12   	| 1 byte	|`0x00`      	| |
|  13   	| 1 byte	|`0x00`       | |
|  14-15 	| 2 bytes |            	| 16-bit CRC	|

Parameter 1 at offset `0x0B` is still unknown. Values seen: `0x05`, `0x11`, `0x76`<br>
Parameter 3 at offset `0x0D` is still unknown. It seems to be almost always the same value. Only 1 user has seen different values for the same setup (`0x33` and `0x34`). Different ComfoFans report different values, so it might indicate the type of Comfofan, software version, the dip-switch settings or maybe some other settings? It could also mean the CO2-level?<br>
When parameter 1 is 0x05 the values seen so far  are: `0x4C`, `0x4E`, `0x4F`, `0x50`, `0x51`, `0x52`, `0x53`, `0x54`, `0x55`, `0x56`, `0x57`, `0x58`, `0x59`, `0x5D`<br>
When parameter 1 is 0x11 the values seen so far  are: `0x2D`, `0x2E`<br>
When parameter 1 is 0x76 the values seen so far  are: `0x23`, `0x25`, `0x26`, `0x27`, `0x28`, `0x2C`<br>
`0x00`, `0x2C`, `0x33`, `0x34`: unsure if these relate to a specific value for parameter 1<br>
This message differs from other message types because the `0x1D` message is always a broadcast to all (`0x00` `0x00`).<br>

## Capturing and analyzing RF signals
There are currently 3 known methods for capturing and analyzing RF signals from your ComfoFan system:
1. Use the [nRF905-API](https://github.com/eelcohn/nRF905-API/)
2. With rtl_433 and a RTL-SDR
3. With URH and a RTL-SDR

### Analyzing RF signal with nRF905-API
See the [nRF905-API](https://github.com/eelcohn/nRF905-API/) page for building/installing. Use the `/api/v1/receive.json` API endpoint to capture data.

### Analyzing RF signal with a RTL-SDR and rtl_433
1. Install [rtl_433](https://github.com/merbanan/rtl_433)
2. Plug your [RTL-SDR](https://www.rtl-sdr.com/) into an USB port
3. Start rtl_433 using the following command:
```
rtl_433 -f 868400000 -g 10000 -S known -R 0 -X n=zehnder,m=FSK_MC_ZEROBIT,s=10,r=100,preamble={10}fd4,invert -F csv:Zehnder-log.csv
```
The readable data will be written to the `Zehnder-log.csv` file.

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
