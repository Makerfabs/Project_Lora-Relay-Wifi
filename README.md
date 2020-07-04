---
typora-root-url: md_pic
---


# Lora Relay Wifi Project

```c++
/*
Version:		V1.1
Author:			Vincent
Create Date:	2020/6/16
Note:
	2020/6/29	v1.1: Change doc in INFO and pic num.  By Vincent
*/
```

[youtube:Lora relay](https://youtu.be/7O8pRCf0avk)
![1](https://github.com/Makerfabs/Project_Lora-Relay-Wifi/blob/master/md_pic/1.png)

[TOC]



# OVERVIEW

## Intruduce

[Makerfabs home page](https://www.makerfabs.com/)

[Makerfabs Wiki](https://makerfabs.com/wiki/index.php?title=Main_Page)

![2](https://github.com/Makerfabs/Project_Lora-Relay-Wifi/blob/master/md_pic/2.png)

Makerfabs provides two Lora communication solutions, MakePython Lora and Maduino Lora. This experiment introduces how to use SX127X series module for Lora transceiver communication across platforms.And realize a soil moisture detection project:

> With Lora chip SX127X series as the core, connect Makepython ESP32 Lora gateway, Lora Relay.The on/off state of Lora Relay is controlled with ESP32 via the Lora protocol.

```sequence

wifi-> gate : Timed query (request)
gate -> relay :Command signal (broadcast)
gate -> gate : Wait response
relay-> gate : Callback (broadcast)
gate->wifi : Humidity value (response)

```

## Equipment list

- [MakePython ESP32](https://www.makerfabs.com/wiki/index.php?title=MakePython_ESP32)
- [MakaPython Lora](https://www.makerfabs.com/wiki/index.php?title=MakaPython_Lora)
-  [Lora Relay](https://www.makerfabs.com/wiki/index.php?title=Lora_Relay)

# STEPS

## 1 Lora Relay

- Arduino IDE to open \\LoraSensor\LoraSensor.ino

-  Connect the Lora Relay to PC through the download line and press the Moisture Sensor on the serial port welding plate with your hand on the other end.

![3](https://github.com/Makerfabs/Project_Lora-Relay-Wifi/blob/master/md_pic/3.png)



- After saving, select "Tools", select "Development Board" Arduino Pro or Pro min, select processor ATmega328p 3.3V 8MHz, and select corresponding serial port.

![4](https://github.com/Makerfabs/Project_Lora-Relay-Wifi/blob/master/md_pic/4.png)

- Open the serial port monitor and click "Upload" from the IDE interface.

- If all goes well, the serial port monitor will display startup module Settings, send messages and other information.

  ```c++
  18:27:44.956 -> LoRa radio init OK!
  18:27:44.956 -> Set Freq to: 433.00
  18:27:44.956 -> Set setPreambleLength to: 8
  ```
  
- Connect Relay with 12V DC。Since the current of Pro Mini itself is not enough to drive the relay module, it is unnecessary to add 12V DC when connecting the power supply.

![5](https://github.com/Makerfabs/Project_Lora-Relay-Wifi/blob/master/md_pic/5.png)

## 2 MicroPython Lora

- Plug the ESP32 and Lora extension boards together.

![6](https://github.com/Makerfabs/Project_Lora-Relay-Wifi/blob/master/md_pic/6.png)

![7](https://github.com/Makerfabs/Project_Lora-Relay-Wifi/blob/master/md_pic/7.png)

- Connect MakePython ESP32 to your PC, open uPyCraft, and select connect to the serial port.
- Firmware will be prompted if it has not been burned before or for other reasons.Board selects ESP32, BURN_addr selects 0x1000, Erase_Flash selects Yes, com selects the port number.Firmware Choose Users, click Choose to Choose ESP32-IDF3-20190125-v1.10.bin in the folder.

![8](https://github.com/Makerfabs/Project_Lora-Relay-Wifi/blob/master/md_pic/8.png)

- Download all python programs ending in.py from the \LoraS2G\workSpace to ESP32.

![9](https://github.com/Makerfabs/Project_Lora-Relay-Wifi/blob/master/md_pic/9.png)

- Press the RST button on ESP32 to reset the development board.The 
