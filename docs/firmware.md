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

```C
int write(){
        long serial = packetInLong(2);
        char sbytes[8];
        sprintf(sbytes, "%ld",serial);
        appendCharToPacketOut('0');
        byte success = writeData(sbytes);
        writeWriteResults(success);
}

bool writeData(char dataIn[]){
        byte responseType = nano.writeTagEPC(dataIn, 8);
        if (responseType == RESPONSE_SUCCESS) {
                return 1;
        }
        else {
                return 0;
        }
}
```
The higher levle **write()** function extracts the supplied serial number from the packet bytes, writes it into a string, and then passes that string to the lower level **writeData()** function to to call the nano functions to write the data to the RFID tag. The success of this operation is passed back up to the higher level function, and written to the packet that will be returned to the test computer. 

```C
void scan(){
        char serial[8];
        nano.startReading();
        for(int i =0; i<50; i++) {
                if (nano.check() == true){
                        byte responseType = nano.parseResponse();
                        if (responseType == RESPONSE_IS_TAGFOUND){
                                int rssi = nano.getTagRSSI();
                                for(int i=0; i<8; i++){
                                        serial[i] = '\0';
                                }
                                for (byte x = 0 ; x < 8 ; x++){
                                        char c = char(nano.msg[31 + x]);
                                        if (isdigit(c)){
                                                serial[x] = c;
                                        }
                                }
                                Serial.println(rssi);
                                char *endptr;
                                long num = strtod(serial, &endptr);
                                writeScanPacket(rssi, num);
                        }
                }
                delay(10);
        }
        nano.stopReading();
        writeScanComplete();
}
```
The **scan()** function searches for any and all tags in the field of view of the reader. It does this for around 500ms (50 cycles at 10ms each). For each value it finds, it sends a packet that includes the serial and RSSI of the tag that it read. This allows for reading multiple tags in the field of view. Finally, when the process is completed, a final packet is written indicating scanning is complete.   

```C
void height(){
        float height = getHeight();
        int height1000 = int(height *1000);
        writeHeightPacket(height1000);
}

float getHeight(){
  int16_t adc = 0;
  for(int i =0; i<10; i++){
        adc += ads1015.readADC_SingleEnded(0);
  }
  adc /= 10;
  return adu_to_mm(adc);
}

float adu_to_mm (int adu){
        return 0.06394*adu - 80.05;
}
```
This is the code that operates the laser height check tool. The system reads from the external (higher resolution) ADC, averaging ten samples to get a less noisy reading expressed in ADU. This is then passed to a function that converts ADU to mm using a formula that was empirically derived. The top level heigh function multiples this value by 1000 to express it as an int instead of a float with sufficient accuracy (it will be divided by 1000 on the other side of the serial interface). 

```C
void Antenna1(){
        digitalWrite(A, LOW);
        digitalWrite(B, LOW);
        writeAntennaSet(1);
}
void Antenna2(){
        digitalWrite(A, HIGH);
        digitalWrite(B, LOW);
        writeAntennaSet(2);
}
void Antenna3(){
        digitalWrite(A, LOW);
        digitalWrite(B, HIGH);
        writeAntennaSet(3);
}
void Antenna4(){
        digitalWrite(A, HIGH);
        digitalWrite(B, HIGH);
        writeAntennaSet(4);
}
```
These antenna functions control which antenna is selected using the two GPIO pins A and B and implementing the following truth table:

| A | B | Ant |
| --- | --- | --- |
| LOW | LOW | 1 |
| HIGH | LOW | 2 |
| LOW | HIGH | 3 |
| HIGH | HIGH | 4 |

```C
boolean setupNano(long baudRate){
        nano.begin(softSerial);
        softSerial.begin(baudRate);
        while (!softSerial);
        while (softSerial.available()) softSerial.read();

        nano.getVersion();
        if (nano.msg[0] == ERROR_WRONG_OPCODE_RESPONSE)
        {
                nano.stopReading();
                Serial.println(F("Module continuously reading. Asking it to stop..."));
                delay(1500);
        }
        else
        {
                softSerial.begin(115200);
                nano.setBaud(baudRate);
                softSerial.begin(baudRate);
                delay(250);
        }

        nano.getVersion();
        if (nano.msg[0] != ALL_GOOD) return (false);
        nano.setTagProtocol();
        nano.setAntennaPort();
        return (true);
}
```

THis code was largely copied from the example code provided by Sparkfun (manufacturers of the RFID breakout board. It starts communication with the device, reads the version to make sure it's communicating, and sets the tag and antenna protocols. 
