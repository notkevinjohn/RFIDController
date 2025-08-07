# RFID Software Guide #
The control software, written in python, communicates with the arduino via a low-level serial interface, and provides access to the firmware functions on that device. It leverages the common code base of SerialManager to handle serial communication with the Arduino, and the common PacketList class to manage the record of packets being sent to and from the device. 

## RFID Controller ##
```python3 
import time
import numpy as np
import re
from SerialManager import SerialManager

class RFIDController():
        holderPattern = '[0-9]{2,3}$'
        sensorPattern = '[0-9]{6,8}$'

```

This class imports time, numpy, and regular expressions as well as the local SerialManager class. The class establishes two patterns for identifitying the serial numbers of holders (used during overmolding) and the serial numbers of sensors. 

```python3
   def __init__(self, device):
                self.device = device
                self.serialManager = SerialManager(self.device)
                self.events = []
                self.blocked = False
```

The constructor for this class sets the device path as the one provided during instantiation (eg: /dev/arduino-uno) and creates the SerialManager class passing that value. It creates an empty events array for storing any events that may be read by the controlling application, and finally sets the blocked flag, which prevents multiple threads from tryintg to talk to the RFIDController simultaneously, to false. 

```python3
  def stop(self):
                self.serialManager.stop()

  def sendPacket(self, packet, priority=0):
          accepted, received, recognized = self.serialManager.sendPacket(packet, priority)
          return accepted, received, recognized

  def loop(self):
          accepted, received, recognized = self.sendPacket([255])
          if accepted & received & recognized:
                  response = self.serialManager.awaitResponse([255],2)
                  if response != None:
                          return True
                  return False

```
Here are some lower level utility functions. The stop function terminates the serial manager's communication with the device. The send packet function sends the provided packet to the device and returns flags indicating if it was accepted, received, and recognized by the Arduino firmware. The loop function sends a packet that does nothing, but tests that commuciation with the Arduino is working correctly. 

```python3 
  def read(self):
        accepted, received, recognized = self.sendPacket([1])
        if accepted & received & recognized:
                response = self.serialManager.awaitResponse([1],5)
                if not response == None:
                        success = response[1]
                        if success:
                                return self.IntFromBytes(response[2:6])
                        else:
                              return None
                else:
                      return None

def write(self, serial):
        accepted, received, recognized = self.sendPacket([2, serial])
        if accepted and received and recognized:
                return True
        else:
              return False
```
The read function reads a single value from the RFID tag, if one is present. If not, it returns None. The write function writes a single value to the RFID tag, if one is present. If not, it returns False. 

```python3
 def scan(self):
      returns = []
      accepted, received, recognized = self.sendPacket([3])
      if accepted & received & recognized:
              scanning = True
              while (scanning):
                      response = self.serialManager.awaitResponse([3], 3)
                      final = response[1]
                      if final == 1:
                              break;

                      rssi = -1*self.IntFromBytes(response[3:4])
                      serial = self.IntFromBytes(response[4:9])
                      returns.append([rssi, serial])
                      if response == None:
                              scanning =  False

      return returns
```

The scan function looks for multiple tags for a given period of time (defined in firmware as 500 milliseconds). It then returns a list of every return it had, and the RSSI value of that return. 

```python3
def parseScanResults(self):
      values = {}
      returns = self.scan()
      for ret in returns:
              rssi = ret[0]
              serial = ret[1]
              if not serial in values:
                      values[serial] = []
              values[serial].append(rssi)

      sensors = []
      holders = []
      for key in values:
              rssiList = values[key]
              rssiList = np.array(rssiList)
              rssiMax = int(np.amax(rssiList))
              if re.match(self.holderPattern, str(key)):
                      key = format(key, 'X')
                      holders.append([key, rssiMax])
              if re.match(self.sensorPattern, str(key)):
                      sensors.append([key, rssiMax])

      return sensors, holders

def getHolderAndSensor(self):
        sensor = None
        holder = None
        sensors, holders = self.parseScanResults()
        if len(sensors) > 0:
                sensors.sort(key=lambda x: x[1], reverse=True)
                sensor = sensors[0][0]
        if len(holders) > 0:
                holders.sort(key=lambda x: x[1], reverse=True)
                holder = holders[0][0]
        return holder, sensor

```

The getHolderAndSensor function leverages the lower level scan function, which it passes to the parseScanResults function, to return a list of unique serials and holder IDs; sorted by RSSI value to make sure the strongest signals are always listed first. 

```python3
 def setAnt(self, ant):
      accepted, received, recognized = self.sendPacket([4, ant])
      if accepted and received and recognized:
              response = self.serialManager.awaitResponse([4], 1)
              _ant = self.IntFromBytes(response[2:3])
              if ant == _ant:
                      return True
      return False
```

The setAnt function changes which of the 4 antenna ports the RFID values will be read using. 

```python3 
def getHeight(self):
          accepted, received, recognized = self.sendPacket([5])
          if accepted and received and recognized:
                  response = self.serialManager.awaitResponse([5], 2)
                  height = self.SignedIntFromBytes(response[1:3])/1000
                  return height
```
This function gets the height from the laser distance sensor. This value comes across as an integer that's been multiplied by 1000 for sufficient accuracy, so we divide by 1000 here. 
Note: If no laser is present, it will still return a value that is meaningless. 

```python3
def IntFromBytes(self, _bytes):
        byteArray = bytes(_bytes)
        return int.from_bytes(byteArray, "big")

def SignedIntFromBytes(self, _bytes):
        byteArray = bytes(_bytes)
        return int.from_bytes(byteArray, "big", signed=True)
```
These utility functions deserialize bytes as either an int of a signed int respectively. 

