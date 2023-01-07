# Unofficial guide to the [WT32-ETH01](http://www.wireless-tag.com/portfolio/wt32-eth01/)

<img alt="WT32-ETH01 circuit board" src="http://www.wireless-tag.cn/wp-content/uploads/2020/08/12-300x263-1.jpg" width=200>

## What is this thing and why would I use it?

The WT32-ETH01 is effectively a small, cheap [ESP32](https://en.wikipedia.org/wiki/ESP32) development board with Ethernet, WiFi, and GPIO pins, made by [a company called "Wireless-Tag" (WT)](http://www.wireless-tag.com/). As of this writing, it's [around $15 at JacobsParts](https://www.jacobsparts.com/items/DEVBOARD-G) and [around $7 from AliExpress](https://www.aliexpress.us/item/3256804925683679.html).

There aren't a ton of ESP32 boards with Ethernet, and the WT32-ETH01 is by far the smallest, cheapest, and simplest. (The [OLIMEX ESP32-POE](https://www.olimex.com/Products/IoT/ESP32/ESP32-POE/open-source-hardware) and [wESP32](https://wesp32.com/) are the other notable options.) So, if you want a cheap-and-cheerful ESP32 board with the reliability of a wired network, the WT32-ETH01 may be a good choice.

It's marketed as a "serial port to Ethernet module" and comes loaded with firmware that lets you send [various "AT" commands](https://core-electronics.com.au/attachments/localcontent/ATCommandsofWT32-ETH01WiredModule_v1_1135593e704.pdf) over 3.3V serial to set up networking, open connections and exchange data. For most of us, it's more interesting to just flash our own programs on the unit and use standard ESP32 networking libraries.

Nobody knows much about [the WT company](http://www.wireless-tag.com/). Don't expect support, and don't lock yourself in too much.

## Device details

[The data sheet](http://www.wireless-tag.com/wp-content/uploads/2022/10/WT32-ETH01_datasheet_V1.3-en.pdf) gives a pretty good overview of the product, including a system block diagram:

<img alt="WT32-ESP01 system block diagram" src="https://user-images.githubusercontent.com/279819/211134688-df67c565-bd14-44cd-bdfb-e28279180e42.png" width=600>

- [WT32-S1](http://www.wireless-tag.com/portfolio/wt32-s1-2/) is WT's ESP32 module (metal box), an [ESP32-WROOM-32E](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wroom-32ue_datasheet_en.pdf) clone based on the same [ESP32-D0WD-V3](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf) chip
- [LAN8720A](https://www.microchip.com/en-us/product/LAN8720A) is the Ethernet physical layer controller (PHY)
- "Left Interface" and "Right Interface" are just the two pin headers

The data sheet also has a pin diagram, but [an older version](https://files.seeedstudio.com/products/102991455/WT32-ETH01_datasheet_V1.1-%20en.pdf) actually has better English labels:

<img alt="WT32-ESP01 pins" src="https://user-images.githubusercontent.com/279819/211134805-77965a29-976c-4b20-ae87-8b64f8f9d04e.png" width=600>

Note that `CFG`, `485_EN`, `RXD` and `TXD` describe functions specific to their "serial gateway" firmware-- if you run your own firmware, these are just GPIOs (`IO32`, `IO33`, `IO5` and `IO17` respectively).
