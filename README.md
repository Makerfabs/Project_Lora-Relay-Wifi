---
typora-root-url: md_pic
---


# Lora Relay Wifi Project

> Version:	V1.0
> Author：	Vincent
> Create Date：	2020年6月16日
> Note：

![image-20200628100309002](/../image-20200628100309002.png)

[TOC]



# OVERVIEW

## Intruduce

[Makerfabs home page](https://www.makerfabs.com/)

[Makerfabs Wiki](https://makerfabs.com/wiki/index.php?title=Main_Page)

![image-20200628100727830](/../image-20200628100727830.png)

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

![image-20200628094837130](/../image-20200628094837130.png)



- After saving, select "Tools", select "Development Board" Arduino Pro or Pro min, select processor ATmega328p 3.3V 8MHz, and select corresponding serial port.

![image-20200624111248264](/image-20200624111248264.png)

- Open the serial port monitor and click "Upload" from the IDE interface.

- If all goes well, the serial port monitor will display startup module Settings, send messages and other information.

  ```c++
  18:27:44.956 -> LoRa radio init OK!
  18:27:44.956 -> Set Freq to: 433.00
  18:27:44.956 -> Set setPreambleLength to: 8
  ```
  
- Connect Relay with 12V DC。Since the current of Pro Mini itself is not enough to drive the relay module, it is unnecessary to add 12V DC when connecting the power supply.

![image-20200628094909132](/../image-20200628094909132.png)

## 2 MicroPython Lora

- Plug the ESP32 and Lora extension boards together.

![image-20200619170934064](/image-20200619170934064.png)

![image-20200619170946542](/image-20200619170946542.png)

- Connect MakePython ESP32 to your PC, open uPyCraft, and select connect to the serial port.
- Firmware will be prompted if it has not been burned before or for other reasons.Board selects ESP32, BURN_addr selects 0x1000, Erase_Flash selects Yes, com selects the port number.Firmware Choose Users, click Choose to Choose ESP32-IDF3-20190125-v1.10.bin in the folder.

![image-20200619152601330](/image-20200619152601330.png)

- Download all python programs ending in.py from the \LoraS2G\workSpace to ESP32.

![image-20200624111415256](/image-20200624111415256.png)

- Press the RST button on ESP32 to reset the development board.The serial monitor displays the boot self - check information.
- If all goes well, the serial port will have a log of the wifi connection and display on the ESP32'S LCD screen.

![image-20200624152537711](/image-20200624152537711.png)

- The first IP is the IP address that the node connects to WiFi.

- Open the browser (Chrome is recommended) and enter the first IP in the address bar, such as 192.168.1.123.

- Then we can access the relay control page.(The IP in the figure below is different from the above because of the change of network environment, please enter the address on the screen of ESP32)


![image-20200628095604978](/../image-20200628095604978.png)

- You can also try using your phone to connect to WiFi and type in the same url so you can check the Lora Relay anytime, anywhere.

![image-20200628095649404](/../image-20200628095649404.png)

- We can then click ON the RELAY ON and within three seconds the RELAY ON Lora's RELAY will be turned ON.The blue light will light up and the sound of the relay closing will be heard.

![image-20200628100221870](/../image-20200628100221870.png)

- The Lora Relay serial port will show the received message and send a response signal.

```c++
18:49:38.758 -> Got reply: RELAYON
18:49:38.758 -> RSSI: -27
18:49:38.758 -> 
18:49:38.758 -> Already ON
18:49:39.765 -> Sending...
18:49:39.765 -> Sending...
18:49:39.832 -> Sending...
```

- The serial port on ESP32 also receives relay status information back.

```c++
Sending message:
RELAYON
wait lora callback
[Memory - free: 72800   allocated: 38368]
On gate receive
Received message:
Already ON
with RSSI -44
```



## 3 Arduino Explain

### 3.1 Definition

```c++
#include <SPI.h>
#include "RH_RF95.h"
```
- The Lora module is based on THE SPI protocol and Arduino communication, and USES the hardware SPI, MOSI, MISO, and SCK pins using the hardware default configuration pins.
- Rh_rf95.h, Lora chip model, compatible with SX127X series.


```C++
#define RFM95_CS 10
#define RFM95_RST 4
#define RFM95_INT 2
#define RF95_FREQ 433.0
#define RF95_PREMABLE_LENGTH 8
```
- RFM95_CS,RST,INT for Lora module segment selection, reset, interrupt.
- RF95_FREQ is the main communication frequency of Lora module at 433MHz.
- RF95_PREMABLE_LENGTH is the leading code length of Lora message.

### 3.2 Lora init

```c++
RH_RF95 rf95(RFM95_CS, RFM95_INT);
```

- Create a Lora chip object

```c++
pinMode(RFM95_RST, OUTPUT);
digitalWrite(RFM95_RST, HIGH);
delay(10);
digitalWrite(RFM95_RST, LOW);
delay(10);
digitalWrite(RFM95_RST, HIGH);
delay(10);
```

- Restart the Lora module

```c++
while (!rf95.init())
{
  Serial.println("LoRa radio init failed");
  while (1)
    ;
}
Serial.println("LoRa radio init OK!");
```

- Initialize the Lora module object

```c++
  if (!rf95.setFrequency(RF95_FREQ))
  {
    Serial.println("setFrequency failed");
    while (1)
      ;
  }
  Serial.print("Set Freq to: ");
  Serial.println(RF95_FREQ);
```

- Set the Lora communication frequency

```c++
rf95.setTxPower(23, false);
rf95.setPreambleLength(RF95_PREMABLE_LENGTH);

//rf95.printRegisters();
```

- setTxPower:  Set transmitting power
- setPreambleLength:  Sets the length of the leading code
- printRegisters:  Prints all register contents



### 3.3 Relay Control

```c++
set_relay();
```

- Turn on/off relay fuction.

```c++
  uint8_t buf[RH_RF95_MAX_MESSAGE_LEN];
  uint8_t len = sizeof(buf);

  //Serial.println("Waiting for reply...");
  delay(1000);
  if (rf95.available())
  {
    if (rf95.recv(buf, &len))
    {
      Serial.print("Got reply: ");
      String msg = String((char *)buf);
      Serial.println(msg);
    }
  }
```

- Recevie lora message and print to serial port.

```c++
if (msg.indexOf("RELAY") == 0)
{
        String message = "COMMAND WRONG";
        if(msg.indexOf("ON") != -1)
        {
          relay_state = true;
          message = "Already ON";
        }
          
        else if(msg.indexOf("OFF") != -1)
        {
          relay_state = false;
          message = "Already OFF";
        }
}
```

- Check for relay switch control instructions.

```c++
delay(1000);
int i;
for (i = 0; i < 3; i++)
{
  // Send a message to rf95_server
  uint8_t radioPacket[message.length() + 1];
  message.toCharArray(radioPacket, message.length() + 1);
  radioPacket[message.length() + 1] = '\0';
  Serial.println("Sending...");
  delay(10);
  rf95.send((uint8_t *)radioPacket, message.length() + 1);
}
delay(3000);
```

- Send three response signals to ESP32 to return relay status.


## 4 Python Explain

### 4.1 controller.py


- The Controller class is the parent class that abstracts the basic pin content of the Lora module.
- Add_transceiver () adds an SX127X chip object as the send chip, because the MakePython Lora can carry two SX127X chips.
- The remaining functions are some of the early test functions, or hardware pin preparation functions, of no concern to this experiment.

### 4.2 controller_esp.py

- Machine module of MicroPyton is referenced to prepare SPI and GPIO initialization.
- The Controller_esp.Controller class inherits from controller.Controller.
- The model and SPI pins of the hardware for ESP were determined.
- The rest of the overloaded member functions are initialized by hardware pins such as SPI.

### 4.3  controller_esp_lora_oled.py

- controller_esp_lora_oled.Controller类继承于controller_esp.Controller。
- The Controller_ESP_lora_OLd.Controller class inherits from the Controller_esp.Controller.

### 4.4 ssd1306.py

- Ssd1306 led screen driver.

### 4.5 display_ssd1306_i2c.py

- Ssd1306 LED screen based on I2C basic display library.
- Text display function.
- Image display function.

### 4.6 sx127x.py

- Lora module driver function.
- Includes basic Lora module Settings such as frequency, lead code, calibration, spread spectrum factor, etc.

### 4.7 config_lora.py

- To accommodate some configuration code for multiple platforms.

### 4.8 lora_node.py

- Lora Gateway object.
- Encapsulated send method and receive interrupt and other functional functions.



# INFO

## 1 Lora parameter description

### 1.1 Default parameters

|属性|attribute|value|
| ---- | ---- | ---- |
|频率| frequency|433MHz|
| 发射功率 |tx_power_level| 23 |
|信号带宽 |signal_bandwidth | 125KHz |
|扩频因子| spreading_factor | 7|
| 编码率 |coding_rate|8|
| 前导码长度 |preamble_length|8|
| 隐式报头模式 |implicitHeader|True|
| 同步字 |sync_word| 0x12|
| 循环冗余码校验 |enable_CRC| True|

### 1.2 SpreadingFactor 扩频因子

LoRaTM扩频调制技术采用多个信息码片来代表有效负载信息的每个位。扩频信息的发送速
度称为符号速率（Rs），而码片速率与标称符号速率之间的比值即为扩频因子，其表示每
个信息位发送的符号数量。LoRaTM调制解调器中扩频因子的取值范围见下表。

|RegModulationCfg|Chips / symbol|
| -- | -- |
| 6 | 64 |
| 7 | 128 |
| 8 | 256 |
| 9 | 512 |
| 10 | 1024 |
| 11 | 2048 |
| 12 | 4096 |

### 1.3 Coding Rate 编码率

为进一步提高链路的鲁棒性，LoRaTM调制解调器采用循环纠错编码进行前向错误检测与纠
错。使用这样的纠错编码之后，会产生传输开销。每次传输产生的数据开销见下表。

| 编码率 | 循环编码率 | 开销比率 |
| ------ | ---------- | -------- |
| 1      | 4/5        | 1.25     |
| 2      | 4/6        | 1.5      |
| 3      | 4/7        | 1.75     |
| 4      | 4/8        | 2        |

### 1.4 Signal Bandwidth 信号带宽

单位为**kHz**，常用值为**125kHz**

### 1.5 Preamble 前导码

前导码用于保持接收机与输入的数据流同步。默认情况下，数据包含有12个符号长度的前
导码。前导长度是一个可以通过编程来设置的变量，所以前导码的长度可以扩展。

### 1.6 Payload 有效负载

数据包有效负载是一个长度不固定的字段，而实际长度和纠错编码率CR则由显式模式下的
报头指定或者由隐式模式下在寄存器的设置来决定。



## 2 Q&A

### 2.1 Arduino Lora differs from Python Lora byte stream

Byte streams emitted by ArduinoLora:

```c++
b'\xff\xff\x00\x00{"msg":"test","code":0,"sys":{"name":"xbw","num":"001","type":0},"data":{"index":160,"ADC":123}}\x00'
```

PythonLora emits a byte stream:

```c++
b'{"msg":"test","code":0,"sys":{"name":"xbw","num":"001","type":0},"data":{"index":160,"ADC":123}}'
```

Because the Arduino and Python use different Lora libraries, the messages sent are different.The Arduino message has an extra 4 bytes in the front and one byte in the tail, so it cannot be decoded directly by Python decod() function, so it needs to be truncated in advance.It must also be added manually when it is sent to the Arduino side, otherwise the Arduino side cannot receive it normally.