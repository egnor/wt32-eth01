# Unofficial guide to the [WT32-ETH01](http://en.wireless-tag.com/product-item-2.html)

<img alt="WT32-ETH01 circuit board" src="wt32-eth01.png" width=200>

## What is this thing and why would I use it?

The WT32-ETH01 is effectively a small, cheap [ESP32](https://en.wikipedia.org/wiki/ESP32) development board with Ethernet, WiFi, and GPIO pins, made by [a company called "Wireless-Tag" (WT)](http://en.wireless-tag.com/). As of this writing, it's [around $15 at JacobsParts](https://www.jacobsparts.com/items/DEVBOARD-G) and [around $7 from AliExpress](https://www.aliexpress.com/w/wholesale-wt32%2525252deth01.html).

There aren't a ton of ESP32 boards with Ethernet, and the WT32-ETH01 is by far the smallest, cheapest, and simplest. (The [OLIMEX ESP32-POE](https://www.olimex.com/Products/IoT/ESP32/ESP32-POE/open-source-hardware) and [wESP32](https://wesp32.com/) are the other notable options.) So, if you want a cheap-and-cheerful ESP32 board with the reliability of a wired network, the WT32-ETH01 may be a good choice.

It's marketed as a "serial port to Ethernet module" and comes loaded with firmware that lets you send [various "AT" commands](https://core-electronics.com.au/attachments/localcontent/ATCommandsofWT32-ETH01WiredModule_v1_1135593e704.pdf) over 3.3V serial to set up networking, open connections and exchange data. For most of us, it's more interesting to just flash our own programs on the unit and use standard ESP32 networking libraries.

Nobody knows much about [the WT company](http://en.wireless-tag.com/). Don't expect support, and don't lock yourself in too much.

See also [Luberth Dijkman's notes on the part](https://github.com/ldijkman/WT32-ETH01-LAN-8720-RJ45-).

## Pins (and gotchas!)

[The data sheet](WT32-ETH01_datasheet_V1.3-en.pdf) has a pin diagram, though
[an older version](WT32-ETH01_manual.pdf) actually has better English labels:

<img alt="WT32-ESP01 pins" src="https://user-images.githubusercontent.com/279819/211134805-77965a29-976c-4b20-ae87-8b64f8f9d04e.png" width=600>

Note that `CFG`, `485_EN`, `RXD` and `TXD` describe functions specific to their "serial gateway" firmware-- if you run your own firmware, these are just GPIOs (`IO32`, `IO33`, `IO5` and `IO17` respectively).

### Beware!

There are limitations on several of the pins; see "Strapping Pins"
(section 3.3) in the
[processor data sheet](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wroom-32ue_datasheet_en.pdf).

**IO0:** At boot, must be pulled low to program, must float or be pulled high
to boot normally. After booting, used to receive the Ethernet 50Mhz clock
(enabled by IO16). Best to avoid any other use.

**IO1:** ESP32 serial output. Used when programming and active by default
when running. Best to avoid any other use.

**IO2:** At boot, must float or be pulled low to program. You can use this pin
(especially for output) but make sure nothing pulls it high while booting.

**IO3:** ESP32 serial input. Used when programming and active by default
when running. Best to avoid any other use.

**IO5, IO15 (MTDO):** At boot, IO5 controls whether ESP32 libraries will print
debug messages to the serial port (IO1). Also at boot, IO5 and IO15 together
control timings of the module that lets the ESP32 act as an SD card.
You are probably not using that module, so you can use these pins (especially
for output) but note the effect on debug chatter if pulled while booting.

**IO12 (MTDI):** At boot, must float or be pulled low or the chip won't work
(wrong voltage). You can use this pin after booting, but make sure nothing
pulls it high while booting.

**IO35, IO36, IO39:** These pins are input only, but are otherwise free to use.

## Power

You may supply 3.3V power on the `3V3` pins, or 5V power on the `5V` pins, but not both at once!

The schematic lists an "LM1117F-1.8V" voltage regulator, which is clearly wrong-- the output is 3.3V, not 1.8V, and also the actual part on the board appears to be an [AMS1117-3.3](http://www.advanced-monolithic.com/pdf/ds1117.pdf). This LDO has a max dropout of 1.3V and a max input voltage of 15V, so in theory the "5V" input should take anything from 4.6V-15V, though the LDO may get hot with higher voltages if you draw much current. It's thermally protected so it shouldn't burn up but may shut off.

The WT32-ETH01 does not support Power over Ethernet (PoE), you'll need an external "splitter" if you want that.

### Power-on reset

The WT32-S1 module (silver box) includes an R-C circuit on EN with 10KΩ × 0.1µF = 1msec time constant. The WT32-ETH01 board includes a separate R-C circuit for the LAN8720A Ethernet controller, also with 1msec time constant. These are quite fast, so if input voltage ramps slowly, the chip may not start reliably, [as discussed in this thread](https://wled.discourse.group/t/wt32-eth01-hangs-on-boot-after-psu-power-up/2937/5).

This can be fixed if necessary by adding another capacitor (e.g. 10µF for 100msec of reset delay), or a proper reset supervisor.

## Programming

Astute readers will have noticed this part has no USB port, so you need an adapter of some kind. As with most ESP32 boards, using a regular USB-serial adapter is possible but annoying (make sure to use 3.3V output if you try!). You'll need to ground IO0 when booting.

Instead, I recommend [M5Stack's ESP32 Downloader](https://shop.m5stack.com/products/esp32-downloader-kit) (I use this) or [wESP32-Prog](https://wesp32.com/wesp32-prog/) (ought to work, but untried).

Both these products have 6 pins (in a different order, sadly) that you connect to the 6 pins (3 on either side) of the top (WiFi antenna end) of the WT32-ETH01, at which point `esptool` will work out ot the box. When I design carrier boards for the WT32-ETH01, I route those 6 pins to a 6-pin header laid out for the programmer. If you're working with a raw board, you can use [jumper wires like these](https://www.adafruit.com/product/1950).

When wiring up a programmer, make sure RX from the programmer goes to TX on the board and vice versa! Also mind the power connections. The M5Stack programmer has a 3.3V output, the wESP32-Prog has a 5V output, make sure you use the right power pin if you want the programmer to power the board. (The 5V input isn't actually in the top 6 "programming pins", but it is labeled.)

[PlatformIO has board support for the WT32-ETH01](https://docs.platformio.org/en/latest/boards/espressif32/wt32-eth01.html) which works for me "out of the box".

You may also be able to use the [Arduino IDE ESP32 add-on](https://espressif-docs.readthedocs-hosted.com/projects/arduino-esp32/en/latest/installing.html) if you pick `ESP32 Dev Board`, but I haven't tried this.

## Using Ethernet

WiFi is internal to the ESP32 and works "out of the box", but wired Ethernet takes a bit of configuration.

[The ESP32-Arduino "Ethernet" library](https://espressif-docs.readthedocs-hosted.com/projects/arduino-esp32/en/latest/api/ethernet.html) should work, though I haven't tried it. You will need to call something like this (untested!!):

```
ETH.begin(1, 16, 23, 18, ETH_PHY_LAN8720);
```

I personally use code like this based on ESP-IDF interfaces:

```
#include <esp_netif.h>
#include <esp_netif_types.h>

...

// https://esphome.io/components/ethernet.html
ESP_LOGD(TAG, "Initializing Ethernet MAC for WirelessTag WT32-ETH01...");
eth_mac_config_t mac_config = ETH_MAC_DEFAULT_CONFIG();
mac_config.clock_config.rmii.clock_mode = EMAC_CLK_EXT_IN;
mac_config.clock_config.rmii.clock_gpio = EMAC_CLK_IN_GPIO;
mac_config.smi_mdc_gpio_num = 23;
mac_config.smi_mdio_gpio_num = 18;
mac_config.sw_reset_timeout_ms = 1000;  // from ETH.cpp
mac = esp_eth_mac_new_esp32(&mac_config);

ESP_LOGD(TAG, "Initializing Ethernet PHY (LAN8720A) for WT32-ETH01...");
eth_phy_config_t phy_config = ETH_PHY_DEFAULT_CONFIG();
phy_config.phy_addr = 1;
phy_config.reset_gpio_num = -1;
phy = esp_eth_phy_new_lan87xx(&phy_config);

// Enable external oscillator (pulled down at boot to allow IO0 strapping)
ESP_ERROR_CHECK(gpio_set_direction(GPIO_NUM_16, GPIO_MODE_OUTPUT));
ESP_ERROR_CHECK(gpio_set_level(GPIO_NUM_16, 1));
ESP_LOGD(TAG, "Starting Ethernet interface...");

// Install and start Ethernet driver
esp_eth_config_t eth_config = ETH_DEFAULT_CONFIG(mac, phy);
esp_eth_handle_t eth_handle = nullptr;
ESP_ERROR_CHECK(esp_eth_driver_install(&eth_config, &eth_handle));

esp_netif_config_t const netif_config = ESP_NETIF_DEFAULT_ETH();
global_netif = esp_netif_new(&netif_config);
auto const eth_netif_glue = esp_eth_new_netif_glue(eth_handle);
ESP_ERROR_CHECK(esp_netif_attach(global_netif, eth_netif_glue));
ESP_ERROR_CHECK(esp_eth_start(eth_handle));
```

## Internal details

[The data sheet](WT32-ETH01_datasheet_V1.3-en.pdf) gives a pretty good overview of the product, including a system block diagram:

<img alt="WT32-ESP01 system block diagram" src="https://user-images.githubusercontent.com/279819/211134688-df67c565-bd14-44cd-bdfb-e28279180e42.png" width=600>

- [LAN8720A](https://www.microchip.com/en-us/product/LAN8720A) is the Ethernet physical layer controller (PHY)
- [WT32-S1](https://www.lcsc.com/product-detail/WIFI-Modules_Wireless-tag-WT32-S1_C477832.html) is WT's ESP32 module (metal box), a (discontinued?) [ESP32-WROOM-32E](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wroom-32ue_datasheet_en.pdf) clone based on the same [ESP32-D0WD-V3](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf) chip
- "Left Interface" and "Right Interface" are just the two pin headers

There are multiple revisions of the device -- I've seen "V1.2" and "V1.4" on the silkscreen, and the trace routing does differ -- but the differences are unknown. You can get it with or without pin headers pre-soldered on.

Finally, there's a [schematic floating around](WT32_ETH01_V2.schematic.pdf). Its provenance and accuracy are unknown, but it's been helpful to me.

## TODO

- Post KiCad files for footprints, breakout boards, and programming connectors
- Discussion of issues reported elsewhere?
