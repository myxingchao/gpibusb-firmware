GPIBUSB Adapter Firmware
========================

This repository contains the source and hex files for the GPIBUSB firmware.

The associated PCB project can be found on GitHub at www.github.com/Galvant/gpibusb-pcb

Pre-assembled boards can be found at www.galvant.ca

Requires the CCS compiler.

About
-----

Many pieces of test & measurement equipment feature a connectivity port labelled "GPIB" which stands
for General Purpose Interface Bus. On some equipment this can also be labelled as "HPIB" after the 
company that originally invented it.

Commercial USB to GPIB adapters contain a lot of features and functionality, but are also very 
expensive ($500+ new). This is well out of price range for most hobbyists, as well as plenty of engineers
and scientists. A lot of people don't need all of the advanced GPIB communication functionality; some
are just looking to download multimeter readings or oscilloscope waveforms to their PC. This firmware
project aims to implement as much GPIB communication functionality as possible, while still remaining
accessable.

Another limitation of your typical USB to GPIB adapter is the development environment restriction
due to the required drivers. For example, these vendors only provide Linux drivers for Red 
Hat-based distributions, but these drivers do not include MATLAB bindings! Instead of having to deal
with all sorts of GPIB and VISA driver stuff, this GPIBUSB adapter presents itself to the attached PC
as a serial port, allowing a wide variety of operating systems and development environments to utilize
it. So although this means that this adapter is NOT a drop in replacement for convential ones, this
flexability is desired by many.

Installation
------------

Pre-assembled GPIBUSB adapters from www.galvant.ca already include the latest version of the firmware
installed. Updating your adapter to the latest version does not require any special hardware as
they all include the Tiny Bootloader (http://www.etc.ugal.ro/cchiculita/software/picbootloader.htm).

To update the firmware, you only require a single piece of PC software, as well as the updated hex file.
The hex file can be downloaded here: https://github.com/Galvant/gpibusb-firmware/releases. Be sure to check
which version is compatible with your hardware revision by checking the "Hardware Revisions Compatibility" 
section further down this page.

For Windows users, download the Tiny Bootloader package from 
http://www.etc.ugal.ro/cchiculita/software/tinyblddownload.htm and run "tinybldWin.exe".

For Linux users, download tinybldLin from http://tinybldlin.sourceforge.net/ . This may or may
not work on Macs as I have not tried.

With the adapter connected and the software downloaded, select the correct serial port and set 
the baud rate to 115200. Point the software to the hex file you have downloaded from here on 
github. Select "Write Flash" and immediately push the reset button the board. If everything was
successful, or if there are any error messages, a message will be displayed in the software.

Linux Users Note
----------------

For all Linux users, do note that since this adapter presents itself as a serial port, your account
will need permission to read and write to serial ports. This is done by adding your user account
to the "dialout" group. To do this, run the following command:

```Shell
$usermod -a -G dialout steven
```

Where steven is replaced with your user account name. Depending on your permissions you may need 
run this with sudo or as root.

Communication
---------------

Baudrate: 460800

All writes to the adapter must end with a carriage return ('\r' or dec:13).

All responses from the adapter will have a CR added.

A small delay (0.01sec) should be added between successive commands to ensure that everything
is given time to complete.

The input buffer is 1000 bytes in size. All single continuous chunks of data from the PC should be 
strictly less than 1000 bytes in length.

Command List
------------

Here is the list of all commands which are used to control the adapter itself, and are not passed
onto the GPIB bus. One can avoid having to deal with most of this by using our InstrumentKit
Python library (github.com/Galvant/InstrumentKit). These commands are implemented within 
the appropriate classes to abstract them away from your code. It is HIGHLY suggested you
use the library as much as you can (and feel free to contribute classes for additional instruments!).

```
+a:1
```
Sets the GPIB address of the device you wish to communicate with. Here the target address is 1.
GPIB address range from 1 to 31. Check the device to find what its address is.

```
+eoi:1
```
Used to toggle EOI detection on (1) and off (0). If set to on, the controller will stop reading bytes
from the device when EOI is asserted. When set to off, the controller will stop when it reads the byte
defined by the ```+eos:``` command. Default is on (1). Most devices will use EOI, some will give you
the option, and some will not. For those that give you the option, it is highly recommended that you
use EOI to signal end of data.

```
+eos:10
```
Set the character that the device uses to signal end of data. This is especially important when EOI
is off. However, some instruments will use both a termination character and use EOI. If you make sure
to set EOS even if you are using EOI data termination, the controller will suppress the last byte.
It is recommended that you disable any termination characters at the instrument and just use EOI. 
To set the termination character, put the decimal form of the character after the colon. For example, 
to set the character to 'z', you would write +eos:122

```
+read
```
Force the controller to read data from the instrument. Normally this is NOT required. When writing
commands that end with a question mark (?), the controller will automatically read the response.
However, older instruments do not include question marks at the end of query commands. In these
cases, using this command will prompt the controller to read the response data.

```
+test
```
Used to test the controller and your communication software. Controller will respond with: testing\n\r
Controller needs to be plugged into an instrument that is on (ie red LED off) for this to work.

```
+get
```
Short for “Group Execute Trigger”. This command will send the GPIB bus command to all attached
instruments. Introduced in firmware version 3.

```
+strip:0
```
Sets the number of characters to remove from the end of the instrument's response before sending 
it to the attached computer. This is useful when the instrument appends a CR as the adapter uses 
CR to terminate communication. This can prevent having to include extra serial port reads in an 
effort to clear the buffer when programming. NOTE: This command can fail if the total number of 
bytes to be sent to the computer is close to an integer multiple of 100. It is recommended that you do
not rely on this command. Introduced in firmware version 3.

```
+ver
```
Returns the firmware version. Introduced in firmware version 3.

```
+autoread:1
```
Used to toggle automatic response reading on (1) and off (0). If set to on (default) the adapter will
automatically start reading the response from the device if your data contained a question mark (?). 
When set to off, the response will not be read and you will need to send the ```+read``` command to 
do so. This is useful for when uploading small amount of binary data to your instruments as 0x3F 
(ascii for "?") will cause the adapter to attempt to read a response. Introduced in firmware version 4.

```
+reset
```
Resets the microcontroller. Can be useful for when you need settings back to power-on default, or if
and instrument/device has decided it no longer wishes to listen to the GPIB bus. Introduced in firmware
version 4.

```
+debug:0
```
Used to toggle simple debug messages on (1) and off (0). When set to on, communicaiton timeout 
error messages are sent to the host PC containing basic details as to when and what timed-out.
This includes which data line timed-out (DAV, NDAC, NRFD), if you were waiting for it to go high or low, 
as well as if the adapter was reading or writing. Default is off. Introduced in firmware version 4.

Hardware Revisions Compatibility
--------------------

Here hardware revisions (left) are listed with their compatible software versions (right). The hardware
revision is marked on the silkscreen on the adapter PCB.

revB/rev2: v1 - v3

rev3 family (ie 3.1, 3.2): v4

License
-------

This code is released under the AGPLv3 license. 
A copy of the license, as well as author information can be found in the LICENSE folder.