Investigating getting the Rovsun Minisplit 9000 BTU model #G46000946+G46000947 working with home assistant via esphome

Overview
* It looks like it's a variant of the TCL minisplit and kind of conforms to the specification used for the TCL air conditions. 
* I was easily able to get a ESP32 talking to it by making a custom USB pigtail and soldering it to an ESP32
* The packets I get from the unit have the same header as the TCL units but not the same checksum method, this repo ATM is only able to
  capture these packets and display them. It appears that the location of the packet length in the header is still valid
* They fail to parse with the remaining functions
* I've been unable to get mine to pair with Tuya Cloud to hook up a logic analyzer and get two-way communications


Here are some some of the packets I've captured:

Upon initial startup
BB000104020100BD643F27F5FB26A7A2334D6CF41D4BDA7330BAA236B71906655F92CD094801EF3376E8F7E59C22B3F8048A3CE0BF856612D2AC6E3132

Heating at 63F 
BB000104020100BD0000000000000000000100000000008041010300000000F841010000000000803F010000000000803F000000000000000000310E40

Left sitting overnight heating at 63F
BB000104020100BD00000000000000000001F3BA6E0000804101AE95700000F84101D2A96A0000803F011896910000803F00227367F5623981003F693C

Changing settings on the remote does not cause the packets to change, I think I'm missing the next step which is sending a command to indicate the
wifi module is active and available. 




#### Original Readme is below ####

An external component for TCL air conditioners and similar models for Home Assistant using ESPHome.

Supported are air conditioners of the TAC-07CHSA type and similar. Unfortunately, it is impossible to predict with certainty whether a particular unit can be connected because there is a huge variation in configurations: even the same model, letter-for-letter, may differ — for example, the native WiFi module may not have a USB cable, or the control board may not have a UART header soldered in. However, in general — with or without soldering — the following air conditioners have been tested:

- Axioma ASX09H1/ASB09H1
- Ballu Discovery DC BSVI-07HN8
- Ballu Discovery DC BSVI-09HN8
- Ballu Discovery DC BSVI-12HN8
- Daichi AIR20AVQ1/AIR20FV1
- Daichi AIR25AVQS1R-1/AIR25FVS1R-1
- Daichi AIR35AVQS1R-1/AIR35FVS1R-1
- Daichi DA35EVQ1-1/DF35EV1-1
- Dantex RK-12SATI/RK-12SATIE
- Ecostar Radium KVS-RAD09CH
- Royal Clima Gloria Inverter
- TCL ELI ONF 12
- TCL Liferise ONF 09
- TCL TAC-CT09INV/R
- TCL One Inverter TACM-09HRID/E1 (possibly different pin order)
- TCL TAC-07CHSA/TPG-W
- TCL TAC-09CHSA/TPG
- TCL TAC-09CHSA/DSEI-W
- TCL TAC-09HRID/E1
- TCL TAC-12CHSA/TPG
- TCL TAC-12CHSA/TPGI
- TCL TAC-XAL24I
- TCL TPG31IHB

This component requires Home Assistant and ESPHome versions not older than 2023.3.0!

____

This is intended to work EXCLUSIVELY with Home Assistant and ESPHome. If you are interested in other options or connecting in some other way to other systems, I can offer an alternative:
[MQTT connection option](https://github.com/pavel211/TCL-TAC-07-WiFi)

____

An article about the project is available on my Dzen channel:
https://dzen.ru/a/ZmdoyUNswXWnulhg

Everything works — even stably. I have fixed the bugs I encountered and implemented requested features. Of course, not everything is perfect, but using the component now should not risk your peace of mind; however, unexpected glitches can still happen — if they do, please report them to me on Dzen and I will take measures. Detailed documentation will gradually appear on my Dzen channel; I will post the most important information here as I can.

____

Sample ESPHome configuration files are provided: TCL-Conditioner.yaml (full example) and Sample_conf.yaml (simplified example). Download them and use in ESPHome, or simply copy the entire configuration and replace your own — but don't forget to fill in all fields. The files contain hints for each field.

Two questions may arise: platform (chip or module) configuration and remote (included) files. I will try to explain.

## Platform configuration
The platform is configured in the same way as usual in ESPHome. For example, here's a snippet for ESP8266:
```yaml
esp8266:
  board: esp01_1m
```

And here's a snippet for the Hommyn HDN/WFN-02-01 module from the first article about the air conditioner:
```yaml
esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: arduino
```

You can also configure the platform via the main config. For example, as suggested by an alpha tester:
```yaml
esphome:
  platform: ESP32
  board: nodemcu-32s
```

And this is an example for Wemos D1 Mini / NodeMCU esp12f:
```yaml
esphome:
  platform: ESP8266
  board: esp12e
```

In general — it's the same as usual; the right option for your platform can be easily found on the internet.

Important: do not forget to comment out or remove the lines for other platforms!

## Remote (included) files configuration
To add or remove certain parts of the configuration I decided to use remote-included files — they are loaded if the Home Assistant server has internet access. This approach allows editing and updating not the entire configuration at once, but only parts of it, without touching what already works.

Another advantage — you don't need to comment or uncomment huge blocks of code, and you don't need to worry about indentation and other formatting quirks. Everything is done by adding or removing links to files. So, the block looks like this:
```yaml
packages:
  remote_package:
    url: https://github.com/I-am-nightingale/tclac.git
    ref: master
    files:
    # v - align the option lines at this position, otherwise it will cause issues
      - packages/core.yaml # Core module
      # - packages/leds.yaml
    refresh: 30s
```

All included files are specified in the files: section. At minimum, you must have:
```yaml
- packages/core.yaml # Core module
```

All other modules are optional (they are described in that same file a little above). Important: make sure that all file lines are aligned exactly as shown — they must line up with the improvised marker I indicated — otherwise ESPHome will raise many questions. For example, the correct layout:
```yaml
packages:
  remote_package:
    url: https://github.com/I-am-nightingale/tclac.git
    ref: master
    files:
    # v - align the option lines at this position, otherwise it will cause issues
      - packages/core.yaml # Core module
      - packages/leds.yaml
    refresh: 30s
```

And this is incorrect:
```yaml
packages:
  remote_package:
    url: https://github.com/I-am-nightingale/tclac.git
    ref: master
    files:
    # v - align the option lines at this position, otherwise it will cause issues
      - packages/core.yaml # Core module
        - packages/leds.yaml
    refresh: 30s
```
