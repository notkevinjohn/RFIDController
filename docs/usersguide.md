# RFID Controllers User's Guide #

## Hardware ##
This device includes the hardware for reading and writing to RFID tags. It includes an antenna switcher that allows it to select between up to 4 different atennas for reading from multiple locations. It also includes the hardware to connect an optional laser height measurement tool. 

### Bottom Ports ###
![bottom ports](https://raw.githubusercontent.com/notkevinjohn/RFIDController/refs/heads/master/images/Hardware_Bottom.jpg)
1. Top USB Port is for RFID Power and Data. It should be connected to a USB 3.0 source that can provide 3A of power, which will usually mean a powered USB 3.0 hub.
2. Bottom USB Port is for fan power. It should be connected to a USB source that can provide ~0.5A, and needs not have a data connection.
3. Bottom barrell jack adapter is used to power the laser if one is connected. It should provide a 12V 2A power source. 

### Top Ports ###
![top ports](https://raw.githubusercontent.com/notkevinjohn/RFIDController/refs/heads/master/images/Hardware_Top.jpg)
1. SMA connected for antennas should be populated as needed from lowest to highest number, starting with the port that is furtherst from the 6 pin circular connector. At least one is required for RFID communications.
2. The 6pin circular connector goes out to the laser height meaurement tool, if it is included. 

## Udev ##
The RFID device is driven by an Arduino Uno, and it requires a udev rule proving some kind of hardware path for this device. A rule like the following should be included in the udev path:
``` bash
SUBSYSTEM=="tty", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0043", SYMLINK+="arduino-uno", MODE="0666"
```
If multiple arduino-uno devices are used on the test computer, it may be needed to provide a more specific hardware path like the following:

``` bash
SUBSYSTEM=="tty", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0043", ATTRS{serial}=="44231313430351703140", SYMLINK+="arduino-rfid", MODE="0666"
```

## Firmware ##
The firmware can be loaded from the Arduino directory of this repository. The firmware can be loaded using the **arduino-load** script. 
```bash
> ./arduino-load
Setting Parameters
device: /dev/arduino-uno
rate: 115200
fqbn: arduino:avr:uno
source: Main
/dev/arduino-uno found

Moving files
Main.c -> Main/Main.ino
Serial.c -> Main/Serial.ino

Compiling
Sketch uses 15066 bytes (46%) of program storage space. Maximum is 32256 bytes.
Global variables use 1024 bytes (50%) of dynamic memory, leaving 1024 bytes for local variables. Maximum is 2048 bytes.

Used library                                  Version Path                                                                               
SparkFun_Simultaneous_RFID_Tag_Reader_Library 1.2.0   /home/kevin/Arduino/libraries/SparkFun_Simultaneous_RFID_Tag_Reader_Library        
SoftwareSerial                                1.0     /home/kevin/.arduino15/packages/arduino/hardware/avr/1.8.5/libraries/SoftwareSerial
Adafruit_ADS1X15                              2.5.0   /home/kevin/Arduino/libraries/Adafruit_ADS1X15                                     
Adafruit_BusIO                                1.16.1  /home/kevin/Arduino/libraries/Adafruit_BusIO                                       
Wire                                          1.0     /home/kevin/.arduino15/packages/arduino/hardware/avr/1.8.5/libraries/Wire          
SPI                                           1.0     /home/kevin/.arduino15/packages/arduino/hardware/avr/1.8.5/libraries/SPI           

Used platform Version Path                                                      
arduino:avr   1.8.5   /home/kevin/.arduino15/packages/arduino/hardware/avr/1.8.5


Uploading

Monitoring
Monitor port settings:
baudrate=115200
Connected to /dev/arduino-uno! Press CTRL-C to exit.
```
If you are using a different hardware path than /dev/arduino-uno, you will need to update the path specified in the arduino-load script
``` bash
#####################
###### setup ########
#####################
source colors.sh
source arduino.sh

device=/dev/arduino-rfid
rate=115200
fqbn=arduino:avr:uno
src=Main
```

Note: Building the hardware requires that the Arduino Command Line Interface and releveant libraries are installed on the test computer. If that's not the case, see the stand-alone software documentation. 

## Software ##
Control of the device is provided by the **RFIDController.py** class. Note that this file depends on the **SerialManager.py** and **PacketList.py** classes. 
This can be instantiated in an interactive shell to test the fucntions of the device in real time, and the RFID Controller can be accessed through a variable named **rfc**

### Open Interactive Shell ###
``` bash
python3 -i RFIDController.py 
>>>
```
### Select Antenna ###
```bash
>>> rfc.setAnt(1)
True
```
This will return true if the Arduino successfully recieves the antenna change command

### Scan for Tags ###
``` bash
>>> rfc.scan()
[[-51, 1071560], [-50, 1071560], [-50, 1071560], [-50, 1071560], [-50, 1071560], [-51, 1071560]]
```
This will return a list of valued pairs, the first item is the RSSI and the second is the serial number

### Check Height ###
``` bash
>>> rfc.getHeight()
-0.5
```
This will return a single value representing the variance from the target height, expressed in millimeters. 


