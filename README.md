# YACK - Yet Another CW Keyer - User Manual

## Notice

Original YACK is at [https://sourceforge.net/projects/yack/](https://sourceforge.net/projects/yack/), written for ATtiny45 by DK3LJ.

**This is a fork, Arduino UNO (ATmega328P) port.**

## License

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.
    

## Introduction

YACK is a universal CW keyer library and application for the AVR architecture that is designed to be
reusable. It consists of a set of functions to read, play, record and decode CW input from a paddle
keyer (single or double lever). The library can be used to easily create keyers, CW decoders and trainers, beacons,
door openers, alarm clocks and many more applications at very moderate cost. It can also be mixed with other
applications in the same chip.

Components of the application can be left out or included on as needed, reducing the memory footprint. The library 
can be found in the file yack.c 

To demonstrate the library, a standard keyer application has been written which can be found in main.c

This document describes the operation of the keyer from a user perspective

Version: 0.7

## Hardware

Original YACK is configured to run on a stock ATTINY45 cpu with its internal oscillator at 8MHz, prescaled to 1MHz.

This fork is intended for Arduino UNO (ATmega328P or compatible processor such as LGT8F328P).

Hardware connections are defined in yack.h 

- PB4 - TX key line (polarity configurable)
- PB5 - Sidetone (Connect a piezo disc)
- PB6 - Command button (towards GND)
- PB7 - DIT (towards GND, buffer with 10nF cap)
- PB8 - DAH (towards GND, buffer with 10nF cap)

## Usage

After reset in default mode, the keyer operates as regular IAMBIC keyer in IAMBIC B at 12 WPM
(words per minute = 60 CPM). The sidetone generator operates at 800 Hz.

### Speed Change

Speed can be changed py pressing and holding the command key while operating the DIT and DAH paddles.
DIT reduces speed while DAH increases speed. The keyer plays an alternating sequence of dit and dah while
changing speed without keying the transmitter.

### Command mode

Pressing the command button without changing speed will switch the keyer into command mode. This will be 
confirmed with the '?' character. Another press of the same button takes the keyer back into regular
keyer mode and will be confirmed by a single dot.

During command mode the transceiver is never keyed and sidetone is always activated. Further
functions can be accessed by keying one-letter commands as listed below.

#### V - Version

The keyer responds with the current keyer software version number

#### P - Pitch

Allows modifying the sidetone pitch to a higher or lower level. A sequence of dits will be played
and the pitch can be modified with the dit and dah paddles. If no paddle is touched for 5 seconds, 
OK is sounded and the mode terminates, leaving the user in command mode.

#### R - Reset

All settings are returned to their default values except for the stored messages in the message buffers. 
Restored settings include speed and pitch, Paddle Swap, TX level inversion, sidetone and TX keyer settings.

#### U - Tune mode

The transceiver is keyed for a duration of 20 seconds for tuning purposes. Tuning mode is aborted once either
DIT or DAH paddles are touched or the control key is pressed.

#### A - IAMBIC A

Keyer sets IAMBIC A as permanent keying mode. Request is answered with "A OK".

#### B - IAMBIC B

Keyer sets IAMBIC B as permanent keying mode. Request is answered with "B OK".

#### L - Ultimatic

Sets the keyer into ULTIMATIC mode. In Ultimatic mode always tha last paddle to be touched is repeated indefinitely
when paddles are squeezed

#### D - DAH priority mode. In squeezed state a sequence of DAHs is sent.

Some of the first generation keyers exhibited this behaviour so the chip can simulate that

#### X - Paddle swapping

DIT and DAH paddles are swapped. Request is answered with "OK".

#### S - Sidetone toggle

The builtin sidetone generator setting is toggled (ON -> OFF or OFF -> ON). NOTE: This setting is only of relevance 
for regular keying mode. Sidetone is always on in command mode. Request is answered with "OK".

#### K - TX Keying toggle

Toggles the setting of the TX keyer output. In default state the keyer switches the output line when it  is in keyer mode. 
Toggling this setting enables or disables that function. NOTE: Keying is always off in Command mode. Request is answered 
with "OK".

#### Z - Set Farnsworth pause

Allows setting of an extended inter-character Pause in all sending modes which makes fast keying easier to understand. 
Note that this of course only influences RECEPTION, not TRANSMISSION. If you desire farnsworth mode in transmission, please 
manually pause during characters.
 
#### I - TX level inverter toggle

This function toggles wether the "active" level on the keyer output is VCC or GND. This setting is dependent on the 
attached keying circuit. Request is answered with "OK".

#### W - Query current WPM speed

Keyer responds with current keying speed in WPM.

#### 1 and 2 - Record internal messages 1 or 2

The keyer immediately responds with "1" or "2" after which a message up to 100 characters can be keyed at current WPM speed.
After 5 seconds of inactivity the message is played back once and then stored in EEPROM. Choosing "1" or "2" but not keying 
a new message deletes the chosen message buffer content.

#### E and T - Play back internal messages 1 or 2

The stored messages 1 or 2 are played back with keying enabled (if configured). A press of the command key immediately
returns the keyer to keyer mode so a QSO can be started.

#### N - Automatic Beacon

The keyer responds with "N" after which a number between 0 and 9999 can be keyed. After a 5 second timeout the keyer
responds by repeating the number and "OK". Once the keyer returns to keyer mode, the content of message buffer 2 is
repeated in intervals of n seconds. The setting is preserved in EEPROM so the chip can be used as a fox hunt keyer.

Returning to command mode and entering an interval of 0 (or none at all) stops beacon mode.

#### 0 - Lock configuration

The 0 command locks or unlocks the main configuration items but not speed, pitch and playback functions.

#### C - Callsign trainer

The keyer plays a generated callsign (sidetone only) and the user must repeat it. If it was repeated correctly, "R" is 
played and the next callsign is given. If a mistake was sensed, the error prosign (8 dots) is sounded and
the current callsign is repeated again for the user to try once more. If nothing is keyed for 10 seconds, the keyer returns
to command mode.
