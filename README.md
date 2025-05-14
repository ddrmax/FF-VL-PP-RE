# FF-VL-PP-RE
FlashForge and VoxeLab 3D printer Network Protocol
A  reverse engineering of Flash Forge and Voxelab 3D printers WIFI protocol

Capture is done with Wireshark over Ethernet




## How it Works:
Automatic Mode:
The slicer make a broadcast with an UDP Packet and the printer responds to it with a UDP broadcast packet (needs more work as my Printer (voxelab Aries) fail most of the time the automatic search, and the wifi is not stable
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
| ~M601 S1 |CMD M601 Received.<br>Control Success.<br>ok<br>|true| Login Command|-|
| ~M602 |CMD M602 Received.<br>Control Release.<br>ok|true| Logout Command|-|
| ~M17 |CMD M17 Received.<br>ok|false| Command to be determined|Enable Steppers|
| ~M18 |CMD M18 Received.<br>ok|false| Command to be determined|Disable steppers|
| ~M23 |CMD M23 Received.<br>File opened:  Size: xxxxxxxxxxx<br>File selected<br>ok|false| Command to be determined|Select SD file|
| ~M24 |CMD M24 Received.<br>ok|false| Command to be determined|Start or Resume SD print|
| ~M25 |CMD M25 Received.<br>ok|false| Command to be determined|Pause SD print|
| ~M26 |CMD M26 Received.<br>ok|false| Command to be determined|Set SD position|
| ~M27 |CMD M27 Received.<br>SD printing byte 0/100<br>ok|true| Printing Status report in % or bytes|Report SD print status|
| ~M29 |CMD M29 Received.<br>ok|false| Command to be determined|Stop SD write|
| ~M104 |CMD M104 Received.<br>ok|false| Command to be determined|Set Hotend Temperature|
| ~M105 |CMD M105 Received.<br>T0:20 /0 B:21/0<br>ok|true| Hot end and print bed Temperature report|Report Temperatures|
| ~M106 |CMD M106 Received.<br>ok|false| Command to be determined|Set Fan Speed|
| ~M107 |CMD M107 Received.<br>ok|false| Command to be determined|Fan Off|
| ~M108 |CMD M108 Received.<br>ok|false| Command to be determined|Break and Continue|
| ~M112 |CMD M112 Received.<br>ok|false| Command to be determined|Full Shutdown|
| ~M114 |CMD M114 Received.<br>X:0 Y:0 Z:0 A:0 B:0<br>ok|true| ToolHead Postion report|Get Current Position|
| ~M115 |CMD M115 Received.<br>Machine Type: Voxelab Aries<br>Machine Name: Aries<br>Firmware: v1.1.3<br>SN: ABCDEF1234567<br>X: 200 Y: 200 Z: 200<br>Tool Count: 1<br>ok|true| 3D Printer Information|Firmware Info|
| ~M119 |CMD M119 Received.<br>Endstop: X-max:0 Y-max:0 Z-max:1<br>MachineStatus: READY<br>MoveMode: READY<br>Status: S:0 L:0 J:0 F:0<<br>ok|true| Printer Status Report|Endstop States|
| ~M140 |CMD M140 Received.<br>ok|false| Command to be determined|Set Bed Temperature|
| ~M146 |CMD M146 Received.<br>ok|false| Command to be determined|-|


| M-Code | Response | Verified | Comment                                          |
|-|-|-|-|
|~G1|CMD G1 Received.|true|Linear Move|
|~G28|CMD G28 Received.|true|Homing|
|~G90|CMD G90 Received.|true|Put Printer in absolute coordinates|
|~G91|CMD G91 Received.|true|Put Printer in Relative coordinates|
|~G92|CMD G92 Received.|false|Set position|
