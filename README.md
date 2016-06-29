# DYNAUDIO CONNECT API (preliminary)

This document describes the Dynaudio - Connect API. This API gives control and feedback of devices linked with the Connect (e.g. Xeo 4/6, Focus-XD).

This document is mostly accurate. However, I might have mixed up states (on/off).

#### Contents

- [Overview](#1-overview)
- [Commands](#2-control)
  - [shortcuts](#21-shortcuts)
- [Feedback](#3-feedback)
  - [shortcuts](#31-shortcuts)
- [Checksum](#4-checksum)
- [Connection](#5-connection)
- [Questions and Requests](#6-questions-and-requests)
- [Glossary](#7-glossary)

## 1. Overview

All command and feedback messages have the following structure:

<pre>
0xFF 0x55 0x'payload size (PS)' [payload of PS bytes] 0x'checksum'
</pre>


## 2. Control

The API allows for control of the volume level and the functions mute, input, and power. The payload size (PS) has a length of 5 bytes.

Most commands carry a status value (SV). This value carries current zone (Z) and input (I) information. This byte is calculated as follows: 0x'I'*16+0x'Z'.

The following commands can be identified:
&nbsp;

type | Command Code (CC) | Command Value (CV) | payload structure | payload example
---- | ----------------- | ------------------ | ----------------- | ---------------
volume | 0x13 / 0x14 |  0x00 - 0x11 | 0x2F 0x0A 0x'CC' 0x'CV' 0x'SV' | 0x2F 0x0A 0x13 0x04 0x42
mute (off/on) |0x12 | 0x00 / 0x01 | 0x2F 0x0A 0x'CC' 0x'CV' 0x'SV' | 0x2F 0x0A 0x12 0x01 0x42
input | 0x15 | 0x01 - 0x07 | 0x2F 0x0A 0x'CC' 0x'CV' 0x'SV' | 0x2F 0x0A 0x15 0x04 0x42
power on | 0x01 | 0x00 | 0x2F 0x0A 0x01 0x'CV' 0xF'Z' | 0x2F 0x0A 0x01 0x00 0xF2
power off | 0x02 | 0x00 | 0x2F 0x0A 0x02 0x'CV' 0xF'Z' | 0x2F 0x0A 0x02 0x00 0xF2

Notice that volume has two commands codes. The CC for volume up is 0x13 and for volume down 0x14.

&nbsp;

The Dynaudio Connect has 7 inputs. Their 'Command Values' are as follows:
&nbsp;

input type | Command Value (CV)
---------- | ------------------
Minijack | 0x01
Line | 0x02
Optical | 0x03
Coax | 0x04
USB | 0x05
Bluetooth | 0x06
Stream | 0x07

&nbsp;

Devices linked with the Dynaudio Connect can be set to one of three zones. Their numeric values are:
&nbsp;

zone colour | zone value
----------- | ----------
Red | 0x01
Green | 0x02
Blue | 0x03


### 2.1. Shortcuts

Not all that is specified is necessary. While volume commands have a specific command code for volume up and down, when specifying absolute volume commands (is there another way?), this distinction can be ignored.

The status value supplied with most commands is only partly processed. It is necessary to specify the correct zone. However, the input part of the status value is ignored.


## 3. Feedback

Feedback is slightly different and carries a payload of 8 bytes instead of 5. A number of examples follow:
&nbsp;

type | Command Code (CC) | Command Value (CV) | payload structure | payload example
---- | ----------------- | ------------------ | ----------------- | ---------------
volume | 0x04 / 0x05 |  0x00 - 0x11 | 0x2E 0x0A 0x'CC' 0x'CV' 0x'SV' 0x00 0x00 0xD9 | 0x2E 0x0A 0x05 0x04 0x42 0x00 0x00 0xD9
mute |0x03 | 0x00 / 0x01 | 0x2E 0x0A 0x'CC' 0x'CV' 0x'SV' 0x00 0x00 0xD9 | 0x2F 0x0A 0x12 0x01 0x42 0x00 0x00 0xD9
input | 0x?? | ?? | ?? | ??
power on | 0x01 | 0x00 | 0x2E 0x0A 0x01 0x'CV' 0x'SV' 0x00 0x00 0xD9 | 0x2F 0x0A 0x01 0x00 0xF2 0x00 0x00 0xD9
power off | 0x02 | 0x00 | 0x2E 0x0A 0x02 0x'CV' 0x'SV' 0x00 0x00 0xD9 | 0x2F 0x0A 0x02 0x00 0xF2 0x00 0x00 0xD9

Notice again the differential CC for volume up and down. While the feedback on input changes is still uncertain, it can be extracted from other commands, notice the SV on the commands.

## 3.1 Shortcuts

idem to 2.1

## 4. Checksum

The checksum can be calculated as follows:

ROUNDUP(SUM(payload)/255)*255-SUM(payload)-([Payload N]-ROUNDUP(SUM(payload/255)))

If the result is negative add 256 or alternatively extract the 8 least significant bits.

## 5. Connection

Connect through port 1901. Only one connection is allowed at a time.


## 6. Questions and Requests

Questions and request can be submitted via GitHub.

For an example in C check [this file](https://github.com/therealmuffin/synchronator/blob/master/src/modCommandDynaudio.c#L90).


## 7. Glossary

- PS - Payload Size [in bytes]
- Z - current Zone
- I - Input number
- CC - Command Code
- CV - Command Value
- SV - Status Value [zone and input]
- Ch - Checksum