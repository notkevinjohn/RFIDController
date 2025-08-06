# RFID Firmware #
This document covers the firmware files, written in C/processing, that are loaded onto the Arduino. 

## Main.c ##

```C
#include "SparkFun_UHF_RFID_Reader.h"
#include <SoftwareSerial.h>
#include <Adafruit_ADS1X15.h>
```
Here we import the Sparkfun RFID library for and software serial libraries for communication with the RFID shield. We also import the ADS1X15 library for communication with the external ADC we use for accurate readings from the laser height check device. 

```C
#define A 8
#define B 9

SoftwareSerial softSerial(2, 3);
RFID nano;
char inbytes[9];
char tagData[8];
Adafruit_ADS1015 ads1015;
```

We define pins A and B for controlling the antenna switcher. We then create the software serial interface to communication between the Arduino and RFID shield. We create data structures to hold the bytes for incomming data, for the data read from a tag. Finally we create the object for the external ADC. 

```C
void setup(){
        Serial.begin(115200);
        while (!Serial);
        initNano();
        initADC();
}
```
In the setup function, we start the serial interface, and then call the functions to initialize the (RFID) nano, and the ADC respectively. 

```C
void initNano(){
        if (setupNano(38400) == false)
        {
                Serial.println("Module failed to respond. Please check wiring.");
                while (1);
        }
        nano.setRegion(REGION_NORTHAMERICA);
        nano.setReadPower(2700);
        nano.setWritePower(2700);
        pinMode(A, OUTPUT);
        pinMode(B, OUTPUT);
        Antenna1();
}
```
This functions starts communication with the nano. If it fails, it prints an error message and enters an infinite loop. If it succeeds, it sets the region, read and write power, and sets the pin mode for the antenna switch pins. It defaults to selecting antenna 1. 

```C
void initADC(){
        ads1015.begin();
        ads1015.setGain(GAIN_ONE);
}
```
This function starts to ADC and set the gain to one. 

```C
void loop(){
        checkSerial();
}
```
The main loop just continually checks the serial for commands. 

```C
void processPacket() {
        byte packetType = packetInByte(0);
        switch(packetType) {
                case 0:{
                        confirmPacket('1');
                        break;
                }
                case 1:{
                        confirmPacket('1');
                        read();
                        break;
                }
                case 2:{
                        confirmPacket('1');
                        write();
                        break;
                }
                case 3:{
                        confirmPacket('1');
                        scan();
                        break;
                }
                case 4:{
                        confirmPacket('1');
                        setAntenna();
                        break;
                }
                case 5:{
                        confirmPacket('1');
                        height();
                        break;
                }
                case 255:{
                        confirmPacket('1');
                        writeLoopPacket();
                        break;
                }
                default:
                        confirmPacket('0');
                        break;

        }
	clearPacketIn();

}
```

The process packet function maps the various commands onto their respective packets.

| Packet | Function |
| -------- | ---------- |
| 0 | Empty |
| 1 | RFID read |
| 2 | RFID write |
| 3 | RFID scan |
| 4 | set ant |
| 5 | laser height |
| 255 | loop-back|

Finally, this function clears the incomming data buffer.

```C
void clearTagData(){
        for (int i=0; i<8; i++){
                tagData[i] = '\0';
        }
}
```
This functions clears any existing data from the tagData array

```C
void read(){
        clearTagData();
        byte success = readData();
        writeReadResults(success);
}

bool readData(){
        byte responseType;
        char dataOut[10];
        byte len = sizeof(dataOut);

        responseType = nano.readTagEPC(dataOut, len);
        if (responseType == RESPONSE_SUCCESS){
                for (byte x = 0; x < len; x++){
                        char c = char(dataOut[x]);
                        if (isdigit(c)){
                                tagData[x] = c;
                        }
                }
                return 1;
        }
	else {

              	return 0;
        }

}
```
The higher level **read()** function clears the tag data array, reads data from a new tag using the **readData()** function, and writes the results to the serial buffer. The lower level **readData()** function atemps to read an RFID tag using the nano, and if once is found the data from the tag is written to the tagData array. The return value specifies if the reading was successful or not. 




with the following truth table:

| A | B | Ant |
| --- | --- | --- |
| LOW | LOW | 1 |
| HIGH | LOW | 2 |
| LOW | HIGH | 3 |
| HIGH | HIGH | 4 |
