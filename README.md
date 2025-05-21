# FF-VL-PP-RE
FlashForge and VoxeLab 3D printer Network Protocol

A reverse engineering of Flash Forge and Voxelab 3D printers WIFI protocol

Capture is done with Wireshark over Ethernet

## Work in Progress:
ESP32 based Simulator: Fully simulate M-Code Responses for these Printers:
| Printer | M-CODE | G-CODE | G-CODE DOWNLOAD | Discovery | Comment |
|-|-|-|-|-|-|
|Voxelab Aries|FULL|NOT SUPPORTED|Simulated (Working)|Working|Printer Owned|
|Voxelab Aquila Pro|FULL|NOT SUPPORTED|Simulated (Working)|Working|FW Partially Reverse Engeniered|
|Flashforge Finder 3|PARTIAL|NOT SUPPORTED|Simulated (Working)|Not Working|FW Partially Reverse Engeniered|
|FlashForge Adventurer III|PARTIAL|NOT SUPPORTED|Simulated (Working)|Not Working|FW Partially Reverse Engeniered|
|Flashforge Adventurer 5M Pro|PARTIAL|NOT SUPPORTED|Simulated (Working)|Not Working|FW Partially Reverse Engeniered|

Windows Simulator: Early stage, Made with C++ builder

Widows App Controler: Early stage, Made with C++ builder, Simulate the look and feel of Flashprint control window and multiple machines control


## How it Works:
Automatic Mode:

The slicer make a broadcast with an UDP Packet and the printer responds to it with a UDP broadcast packet (My Printer (voxelab Aries) fail most of the time the automatic search, and the wifi is not stable)

UDP Port on printer: 19000

UDP Port on PC: dynamic 

PC send broadcast on IP 225.0.0.9 on port 19000
```
AABBCCDDEEEE0000
```
the data transmited is the IP in hexadecial value (AA.BB.CC.DD) and the port as a 16bit Hex Value (EEEE)

Fox Voxelab Printers:

printer is not busy (HEX value)
```
0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000e100000922c32b7110010000
```

printer is busy (HEX value)
```
0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000e100000922c32b7110010002
```
PS: some research shows that the first 32 bytes can be replace by the name of the printer in flashforge, since VoxelMaker is a fork of an older version of flashprint, a simulation with the name transposed in the 32 bytes, shows the in the printer Finder

For FlashForge Printers:

To be determined

Manual Mode:

It's a simple TCP socket on port 8899 and the Gcode is sent with a tilde "~"at the start of the command

there is some proprietary commands like M601 (~M601 S1) and M602 (~M602) that are needed to send other Gcodes via the TCP socket
As soon as the M-code M601 is sent you can talk both G and M code directly to the printer. 

There is a quirk as you need to send keep alive codes or the printer forcefully closses the printer after some times (under a minute, the slicer sends a command every few seconds)
the Keep alive used by the Voxelab Slicer is a mix of M27 M105 and M119 


### Example:
Client:<br>
```~M27\r\n```

Printer:
```
CMD M27 Received.\r\n
SD printing byte 0/100\r\n
ok\r\n
```

Client:<br>
```~M119\r\n```

Printer:
```
CMD M119 Received.\r\n
Endstop: X-max: 1 Y-max: 1 Z-max: 1\r\n
MachineStatus: READY\r\n
MoveMode: READY\r\n
Status: S:1 L:0 J:0 F:1\r\n
ok\r\n
```


## Command List

When Veified is false, only the bare command has been sent, but the reaction hasn't been checked on the printer or because no parameters ha been sent to either have a modified response or a change in the Printer behaviour

| M-Code | Response | Verified | Comment | Marlin or RepRap Command name |
|-|-|-|-|-|
| ~M17 |CMD M17 Received.<br>ok|true| Enable Steppers|Enable Steppers|
| ~M18 |CMD M18 Received.<br>ok|true| Disable Steppers|Disable steppers|
| ~M23 |CMD M23 Received.<br>File opened:  Size: xxxxxxxxxxx<br>File selected<br>ok|true| Select and print SD File|Select SD file|
| ~M24 |CMD M24 Received.<br>ok|true| Resume SD print|Start or Resume SD print|
| ~M25 |CMD M25 Received.<br>ok|true| Pause SD print|Pause SD print|
| ~M26 |CMD M26 Received.<br>ok|true| Stops the Print|Set SD position|
| ~M27 |CMD M27 Received.<br>SD printing byte 0/100<br>ok|true| Printing Status report in % or bytes|Report SD print status|
| ~M29 |CMD M29 Received.<br>ok|true| Start SD Write|Start SD write|
| ~M29 |CMD M29 Received.<br>ok|true| Stop SD Write|Stop SD write|
| ~M104 SXXX |CMD M104 Received.<br>ok|true| Set Hotend Temperature witn XXX °C|Set Hotend Temperature|
| ~M105 |CMD M105 Received.<br>T0:20 /0 B:21/0<br>ok|true| Hot end and print bed Temperature report|Report Temperatures|
| ~M106 |CMD M106 Received.<br>ok|true| Enable cooling fan, no S parametter|Set Fan Speed|
| ~M107 |CMD M107 Received.<br>ok|true| Stop cooling fan|Fan Off|
| ~M108 |CMD M108 Received.<br>ok|false| Command to be determined|Break and Continue|
| ~M112 |CMD M112 Received.<br>ok|true| Stop the printer|Full Shutdown|
| ~M114 |CMD M114 Received.<br>X:0 Y:0 Z:0 A:0 B:0<br>ok|true| ToolHead Postion report|Get Current Position|
| ~M115 |CMD M115 Received.<br>Machine Type: Voxelab Aries<br>Machine Name: Aries<br>Firmware: v1.1.3<br>SN: ABCDEF1234567<br>X: 200 Y: 200 Z: 200<br>Tool Count: 1<br>ok|true| 3D Printer Information|Firmware Info|
| ~M119 |CMD M119 Received.<br>Endstop: X-max:0 Y-max:0 Z-max:1<br>MachineStatus: READY<br>MoveMode: READY<br>Status: S:0 L:0 J:0 F:0<<br>ok|true| Printer Status Report|Endstop States|
| ~M140 SXXX |CMD M140 Received.<br>ok|true| Set Bed Temperature witn XXX °C|Set Bed Temperature|
| ~M146 rXXX gXXX bXXX F0|CMD M146 Received.<br>ok|true| Set hotend LED On or Off (set XXX as 0 for Off and 255 for ON)|-|
| ~M601 S1 |CMD M601 Received.<br>Control Success.<br>ok<br>|true| Login Command|-|
| ~M602 |CMD M602 Received.<br>Control Release.<br>ok|true| Logout Command|-|
| ~M610 |CMD M610 Received.<br>ok|true| Change Printer Name|-|
| ~M611 |CMD M611 Received.<br>ok|false| Command to be determined|-|
| ~M612 |CMD M612 Received.<br>ok|false| Command to be determined|-|
| ~M640 |CMD M640 Received.<br>ok|false| Command to be determined|-|
| ~M650 |CMD M650 Received.<br>X: 1.0 Y: 0.5<br>ok|false| Command to be determined|-|


| M-Code | Response | Verified | Comment                                          |
|-|-|-|-|
|~G1|CMD G1 Received.|true|Linear Move|
|~G28|CMD G28 Received.|true|Homing|
|~G90|CMD G90 Received.|true|Put Printer in absolute coordinates|
|~G91|CMD G91 Received.|true|Put Printer in Relative coordinates|
|~G92|CMD G92 Received.|false|Set position|

## M119 : Machine Status

| Reported String | Machine Mode is checked on Slicer | Verified | Comment                                          |
|-|-|-|-|
|READY|Yes|true|Printer is ready|
|ERROR|No|true|Printer is Halted|
|BUILDING_FROM_SD|Yes|true|Printer is Printing|

##  M119 : Machine Mode

| Reported String | Slicer Reports | Machine Status | Comment                                          |
|-|-|-|-|
|READY|Printing|ALL|Printer is Printing|
|PAUSED|Paused|BUILDING_FROM_SD|Printer is Paused|
|WAIT_ON_TOOL|Preheating|BUILDING_FROM_SD|Printer is heating Extruder|
|WAIT_ON_PLATFORM|Preheating|BUILDING_FROM_SD|Printer is heating Bed|
|HOMING|Homming|READY|Printer is Homming it's Axes|

