# RFID Hardware #

## Overview ##
![hardware](https://github.com/notkevinjohn/RFIDController/blob/master/images/Hardware_Inside.jpg)

The RFID Controller includes three major subsystems: RFID, Laser Height Measurement, and Cooling. 

### RFID ###
This includes the Arduino, which provides central control of all subsystems and communication back to the test computer. This requires a USB 3.0 source with ~1A of current, this will often mean a powered USB hub is required. It also includes the Sparkfun RFID board, which provides read/write communication with the RFID tags. It also includes the antenna selector which lets the communications be routed to as many as 4 different physical antennnas. Lastly, this includes all the SMA adapters to connect these devices. 

### Laser Height Measurement ###
This subsystem includes the 12V power supply, ADC, and the ciruclar connector to attach to the Panasonic laser height meaurement tool. The ADC connects to the Arduino with an i2C connection, and reads from the laser via a simple analog read pin using 5V logic. The 10-bit ADC on the Arduino was not sufficient to accurately from the device, so a 12-bit ADC was added. 

### Cooling ###
This system includes the two cooling fans on opposite sides of the enclosure, as well as the USB power adapter that delivers the 5V source. This system has dedicated USB connection (separate from the one used on the RFID system) that can be connected to any USB source provding ~250mA or better. 

## Wiring ##
![wiring guide](https://github.com/notkevinjohn/RFIDController/blob/master/images/wiringDiagram.png)



## Parts List ##
**Arduino Uno**  https://www.amazon.com/ELEGOO-Board-ATmega328P-ATMEGA16U2-Compliant/dp/B01EWOE0UU/ref=sr_1_1_pp?crid=22H06PQWC2CB&dib=eyJ2IjoiMSJ9.mMwW9AQ5cM5IcBhuz3WQYZgPI-1AHcBuuYjUkQvEAu3IZH7nceRn5T7-Q3m94430lRqjN5uFEn6j7cPxZ2bcCSxiX24Jis13Noc0UOXHarxPQloCBmZWTbqAlih4s4L3n6mxX25AmDiz-nSoJrM_LylMPEuR5QHl1lmmaiLVxHC3qmSDGv023gOVYyJ6AxGBgJ5hpJBhCTxcSpb7zHKb85d9-WuM7z-u9E8YZDA3psg.-tKRRuxua4SQo6SyaT1le2GeCLL8hW7NOrp0V6_rrys&dib_tag=se&keywords=arduino+uno&qid=1754589263&sprefix=arduno+uno%2Caps%2C167&sr=8-1 \
 
**RFID Reader**
https://www.digikey.com/en/products/detail/sparkfun-electronics/SEN-14066/6691388

**Antenna Selector**
https://www.digikey.com/en/products/detail/analog-devices-inc/EV1HMC241AQS16/5403560?gQT=1

**Enclosure**
https://www.amazon.com/dp/B0C27YKKC5?ref_=ppx_hzod_title_dt_b_fed_asin_title_1_3

**ADC**
https://www.amazon.com/dp/B0DP43DDZG?ref_=ppx_hzod_title_dt_b_fed_asin_title_1_1

**Regulator** 
https://www.amazon.com/dp/B01NALDSJ0?ref_=ppx_hzod_title_dt_b_fed_asin_title_0_1

**Screw Terminal Shield**
https://www.amazon.com/dp/B0D2K9QV7P?ref_=ppx_hzod_title_dt_b_fed_asin_title_3_0

**Fans**
https://www.amazon.com/Printer-80x80x10mm-Brushless-Cooling-Computer/dp/B0BHHYC97X?pd_rd_w=vbnq7&content-id=amzn1.sym.4cbcb1e0-172b-4571-a896-863b1ee67ab3&pf_rd_p=4cbcb1e0-172b-4571-a896-863b1ee67ab3&pf_rd_r=524VK12EW28VVG32MSS8&pd_rd_wg=qlcp9&pd_rd_r=717149cc-b0e6-4b2f-a9dc-f9e098fbfe0a&pd_rd_i=B0BHHYC97X&psc=1&ref_=pd_bap_d_grid_rp_0_1_ec_rp_c_d_sccl_1_4_t

**SMA Cables**
https://www.amazon.com/TUOLNK-Coaxial-Pigtail-Extension-Wireless/dp/B091TH6CCJ?pd_rd_w=vbnq7&content-id=amzn1.sym.4cbcb1e0-172b-4571-a896-863b1ee67ab3&pf_rd_p=4cbcb1e0-172b-4571-a896-863b1ee67ab3&pf_rd_r=524VK12EW28VVG32MSS8&pd_rd_wg=qlcp9&pd_rd_r=717149cc-b0e6-4b2f-a9dc-f9e098fbfe0a&pd_rd_i=B091TH6CCJ&psc=1&ref_=pd_bap_d_grid_rp_0_8_t

**SMA Adapter**
https://www.amazon.com/TUOLNK-Coaxial-Pigtail-Extension-Wireless/dp/B091TH6CCJ?pd_rd_w=vbnq7&content-id=amzn1.sym.4cbcb1e0-172b-4571-a896-863b1ee67ab3&pf_rd_p=4cbcb1e0-172b-4571-a896-863b1ee67ab3&pf_rd_r=524VK12EW28VVG32MSS8&pd_rd_wg=qlcp9&pd_rd_r=717149cc-b0e6-4b2f-a9dc-f9e098fbfe0a&pd_rd_i=B091TH6CCJ&psc=1&ref_=pd_bap_d_grid_rp_0_8_t

**Barrel Jack Adpater**
https://www.amazon.com/dp/B0DX1Z4QXL?ref_=ppx_hzod_title_dt_b_fed_asin_title_1_2&th=1

**USB Power Adapter**
https://www.amazon.com/dp/B08M5HXDSJ?ref=ppx_yo2ov_dt_b_fed_asin_title

**6-pin circular connector**
https://www.amazon.com/dp/B07V54YQ3R?ref=ppx_yo2ov_dt_b_fed_asin_title

**Terminal Block**
https://www.amazon.com/dp/B09891K2P8?ref=ppx_yo2ov_dt_b_fed_asin_title





