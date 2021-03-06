# TinyLoRa-BME280 v1.1
Arduino library for a Atmospheric Sensor BME280 LoRaWan Node.
The LoRa Node measures barometric pressure, humidity, and temperature every 5 min. 
After the measurements, the ATtiny85 goes into sleep mode and is awakened by the watchdog timer.
The RFM module sends the values to the TTN backend with Activation by Personalization (ABP) 
a fixed spreading factor and one of the four random channels.
The example sketch fits in the 8K Bytes Flash Memory of the ATtiny85

## Version 1.1
This version does not make use of the hardware interrupt DIO0 pin from the RFM module.
Two pins from the ATtiny85 are required for SS.
* PB3 ---> CS BME280 (Slave Select)
* PB4 ---> NSS RFM95 (Slave Select)

And the Universal Serial Interface pins
* SCK
* DO
* DI

## Installing
Install the zip file in your Arduino IDE.
Click on the toolbar menu: Sketch -> Include Library -> Add .ZIP Library -> "downloaded zip file"
Once done click OK to save and restart the Arduino IDE.

This library uses the tinySPI library http://github.com/JChristensen/tinySPI
for communication with the BME280 and the Hope RFM95 module.
Install this library too.

## Adding Third Party Boards to the Arduino IDE
The Boards Manager makes it easy to add support for other hardware.
Click on the toolbar menu: File -> Preferences -> Additional Boards Manager URLs: "Paste the following URL into the field"
https://raw.githubusercontent.com/damellis/attiny/ide-1.6.x-boards-manager/package_damellis_attiny_index.json

## Configuration
* Change the NwkSkey,AppSkey,DevAddr acording your own TTN keys.
```
// use your keys from TTN
uint8_t NwkSkey[16] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
uint8_t AppSkey[16] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
uint8_t DevAddr[4] = { 0x00, 0x00, 0x00, 0x00 };
```
### optional	
* Change the interval of measurements and radio transmission with the SLEEP_TOTAL var
```
#define SLEEP_TOTAL 37 // 37*8s = 296s ~5min
```
* Change Spreading factor as you like.
```
  //SF7 BW 125 kHz
  RFM_Write(0x1E,0x74); //SF7 CRC On
  RFM_Write(0x1D,0x72); //125 kHz 4/5 coding rate explicit header mode
  RFM_Write(0x26,0x04); //Low datarate optimization off AGC auto on
  
  //SF8 BW 125 kHz
  RFM_Write(0x1E,0x84); //SF8 CRC On
  RFM_Write(0x1D,0x72); //125 kHz 4/5 coding rate explicit header mode
  RFM_Write(0x26,0x04); //Low datarate optimization off AGC auto on
  
  //SF9 BW 125 kHz
  RFM_Write(0x1E,0x94); //SF9 CRC On
  RFM_Write(0x1D,0x72); //125 kHz 4/5 coding rate explicit header mode
  RFM_Write(0x26,0x04); //Low datarate optimization off AGC auto on 

  //SF10 BW 125 kHz
  RFM_Write(0x1E,0xA4); //SF10 CRC On
  RFM_Write(0x1D,0x72); //125 kHz 4/5 coding rate explicit header mode
  RFM_Write(0x26,0x04); //Low datarate optimization off AGC auto on
```
## Features
supported
* Sending packets uplink
* Encryption and message integrity checking.
* Custom frequencies and datarate settings.
* (ABP) Activation by personalization
* Custom sleep time in power down mode 
* Humidity in %, Pressure in hPa, Temperature negative and positive celsius degree.

not supported

* receiving packets downlink
* Over the Air Activation (OTAA)

## Needed Parts
* 1x ATtiny85V 10PU Microcontroller
* 1x BME280 Breakout Board
* 1x RFM95 Transceiver Module
* 1x Breadboard
* 2x Li-Ion Rechargeable Batteries 1.2 V
* 1x Arduino UNO (for programming)

## Wiring Everything Up
* ATtiny85VCC ---> Breadboard VCC
* ATtiny85 GND ---> Breadboard GND
* ATtiny85 PB3 ---> CS BME280 (Slave Select)
* ATtiny85 PB4 ---> NSS RFM95 (Slave Select)
* ATtiny85 PB2 ---> SCK RFM59  ---> SCK BME280
* ATtiny85 PB1(DO) ---> MOSI RFM95 ---> SDI BME280 (Data going into the BME280)
* ATtiny85 PB0(DI) ---> MISO RFM95 ---> SDO BME280 (Data coming out of the BME280) TTN Decoder function

## TTN Decoder function
```
function Decoder(bytes) {

  //for negative and positive celsius degree
  var t = (bytes[0] & 0x80 ? 0xFFFF<<16 : 0) | bytes[0]<<8 | bytes[1];
  var h = bytes[2];
  var p = bytes[3] << 8 | bytes[4];

  return {
    temperature: t / 100,
    humidity: h,
    pressure : p / 10
  }
}
```

