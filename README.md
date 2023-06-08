This is a cheap customizable WiFi-enabled battery-operated touch button with no moving parts that relies on ESPHome and is designed to be used with Home Assistant.

THIS IS A DRAFT

# License
The content of this repository is licensed under the WTFPL.

```
Copyright © 2023 giovi321
This work is free. You can redistribute it and/or modify it under the
terms of the Do What The Fuck You Want To Public License, Version 2,
as published by Sam Hocevar. See the LICENSE file for more details.
```

# Features
- Minimalist design (i.e., no moving parts, fit with the design of my house)
- No mechanical button (i.e., I just want to slap my hand on a big enough device while I'm laying on my bed)
- USB type C charging port (i.e., charge it with my laptop charger)
- At least one year of battery (i.e., charge it once a year and forget about it)
- WiFi enabled, I don't want to configure also BLE on my Home Assistant device
- Battery status (i.e., get a notification on Home Assistant when it needs to be charged)
- As cheap as possible (this makes z-wave not an option)

# Shopping list
The items in the following links are just proposals, you can obviously source the exact same products wherever you prefer.
Prices include shipping to Europe.
- Voltage sensor [1.74€] https://www.aliexpress.com/item/1005005106676499.html
- Wemos D1 Mini [3.33€] https://www.aliexpress.com/item/32651747570.html
- USB plug[1.77€] https://www.aliexpress.com/item/1005005476223469.html
- USB type C charging circuit [1.87€] https://www.aliexpress.com/item/1005005227218470.html
- Battery 2000mAh [5.44€] https://www.aliexpress.com/item/1005002919536938.html
- Photocoupler TLP570 [2.14] https://www.aliexpress.com/item/1005004627959174.html
- Capacitive touch sensor [0.60€] https://www.aliexpress.com/item/32874585710.html
- 2.54 mm 2 pin JST female connector for PCB [2.18€] https://www.aliexpress.com/item/32855763468.html
- Autotapping screws M2.6 x 5mm [1.30€] (https://www.aliexpress.com/item/1005003330377202.html)
- Breadboard [2.00€] https://whereveryouwanttogetit
- 3D printed case (see the files attached below) [3.00€ in-house, 36.00€ outsourced from xometry.eu]

# Resources
- Circuit sketch
![Circuit](https://github.com/giovi321/wifi_button/assets/6443515/a64e4adf-8eef-4c46-9b9f-9a0fcfdb87d9)
- 3D case - see the files in the dedicated folder of the repository
![Top 1](https://github.com/giovi321/wifi_button/assets/6443515/5813c38f-b581-444f-836f-050e6ff17dc5)
![Top 2](https://github.com/giovi321/wifi_button/assets/6443515/1286280e-becd-4afa-8b9c-e91157d0610c)
![Bottom 1](https://github.com/giovi321/wifi_button/assets/6443515/5aa3b782-1b2a-4216-b39c-ef8fe647e2d7)
![Bottom 2](https://github.com/giovi321/wifi_button/assets/6443515/9094013c-8715-4622-83c1-68e2a4e85942)

# How does it work
- The USB type C connector makes the device modern and easy to charge, but feel free to use any other type of connector
- The charger module keeps the battery charged and you safe from fire hazards, it is simply plug and play
- we need a voltage sensor connected to one analog input of the Wemos D1 Mini in order to read the battery status and receive a notification on Home Assistant when it is about to die
- The device is based on the Wemos D1 Mini (obviously a cheap clone, even though a branded one would be better for lower energy consumption) and ESPHome because they are very easy to use; obviously there are more energy efficient solutions
- In order to keep energy consumption low, we need to keep the Wemos D1 Mini in a deep-sleep state
	- however, in deep sleep state the Wemos D1 Mini won't be able to read an input offered by the touch sensor
	- therefore, I have opted to use the touch sensor to wake up the Wemos D1 Mini that will go in deep-sleep automatically every 60 seconds
	- the Wemos D1 Mini will be configured to send two mqtt messages whenever it wakes up:
		- one message with the battery voltage
		- one message with "hey somebody touched the touch sensor"
- 3.7v battery is the way to go to power the Wemos D1 Mini, and 2000 mAh fits the small design of the case; feel free to use a different amperage battery
   - I haven't tested it, but from my research online the power consumption of different versions of Wemos D1 Mini in deep-sleep mode powered from the 5v input (which is less efficient because there is a voltage regulator in the middle) can vary depending on the manufacturer (see table below)
   - Assuming we are unlucky and got the worse energy-performing Wemos D1 Mini, the average consumption is 69.47mA, giving it a life of almost 399 days (the calculation steps are available below). If we take into consideration the best case scenario, that is 78µA consumption, the battery would last almost 3x: 1,068 days
   - We are not taking into consideration the power consumption of the touch sensor, phototransistor and voltage sensor. The first two components have a very small consumption as it peaks mostly when you activate the touch sensor. For the voltage sensor I couldn't find anything online. In any case, even assuming an absurde value of 15% increase in power consumption, it's still a decent number of days.
   - I don't think I need to say that: i) these values are purely theorethical, the nominal rating of the battery does not equal its empirical performance; ii) the amount of times you will use the button will influence the duration of the battery
   - Pro tip: if you desolder the LED that is installed on the touch sensor, you save some precious µA. However I haven't tried it and I don't know if it would affect the touch sensor functionality
```
          | WEMOS D1 MINI CLONE | WEMOS D1 MINI LOLIN | WEMOS D1 MINI WAVGAT |
|:-------:|:-------------------:|:-------------------:|:--------------------:|
| Minimum |        144 µA       |         75µA        |        206 µA        |
| Maximum |        147 µA       |         80µA        |        214 µA        |
| Average |        156 µA       |         78µA        |        209 µA        |

Source: https://programarfacil.com/esp8266/esp8266-deep-sleep-nodemcu-wemos-d1-mini/
```
```
To calculate the approximate duration of a battery with a given load, you can use the formula:
Battery Life (in hours) = Battery Capacity (in mAh) / (Load Current (in µA) / 1000)
The battery capacity is 2000 mAh and the load current is 209 µA. Plugging these values into the formula:
Battery Life = 2000 mAh / (209 µA / 1000) ≈ 9569.38 hours
Battery Life in days = 9569.38 hours / 24 ≈ 398.72 days
```
- As said above, I'm using a touch sensor for a variety of reasons; touch sensors don't offer an analog output so you need a microcontroller (the Wemos D1 Mini) or, as I have chosen to do in order to reduce energy consumption of the device as I said above, use a phototransistor
- The phototransistor is a very simple electronic component that has 2 inputs and 2 outputs:
	- the two inputs are the anode (+) and cathode (-) of a LED encased in the package of the chip
	- the two outputs are the Emitter and Collector of a transistor, to put it simple those two pins allow current to flow between the two pins
- When the touch sensor is activated with your finger, it sends current to the phototransistor, activating the internal LED of the phototransistor and allowing current to pass between the emitter and the collector of the transistor
  - When the current flows between the Emitter and the Collector, we are simply shorting the RST and GND pins of the Wemos D1 Mini, therefore waking it up


# Building process
## Hardware
### Electronics
Just follow the schematics provided in the Resources chapter of the readme.
I have chosen to use a breadboard because I am not patient enough to learn how to design a circuit.
The other option, is to draw the circuit on a new PCB with a permanent marker and use acid to etch the circuit.

### Case
The assembly is extremly simple:
- hot glue the touch sensor to the top flat part of the dome (inside the dome, obviously) - the sensor can feel touch also through a thin layer of non conductive material (the layer of the top of the dome is 4mm thick)
- Insert the USB type C connector from the bottom of the bottom plate and screw it in or glue it with cyanoacrylate glue
- Solder the connector to the battery charger module and plug the battery
- Stuff the rest of the electronics inside the dome and hot-glue them to the walls of the dome
- Close the dome and use four M2.6 screws to secure it

## Software
- Install ESPHome on your Home Assistant server, if you haven't it yet.
  - Instructions for hass.io installation: https://www.home-assistant.io/integrations/esphome/
  - Instruction for manual installation: https://esphome.io/guides/installing_esphome.html
- Install ESPHome dashboard (if you're not running hass.io), at the end you'll need to add a systemd service to autostart the dashboard: https://esphome.io/guides/getting_started_command_line.html?highlight=dashboard+install#bonus-esphome-dashboard 

### ESPHome code
You can easily google how to connect the Wemos D1 Mini to your PC and program it with ESPHome.
```
mqtt:
  broker: 192.168.1.65
  username: yourmqttusername
  password: yourmqttpassword
  birth_message:
    topic: anythingyouwant
    payload: anything you want

sensor:
  - platform: adc
    pin: A0
    name: Battery
    id: battery_voltage_12f
    filters:
      - multiply: 4.79
    accuracy_decimals: 3
    update_interval: 6000s
```

### Home Assistant code
Add the sensor for the battery
```
  - platform: template
    sensors:
      esp12f_battery_fullness:
        friendly_name: ESP12f Battery Capacity
        device_class: battery
        unit_of_measurement: "%"
        value_template: >
            {{ ((states('sensor.esp12f_battery')|float-2.64)/0.0156)|round(2) }}
```

Add the sensor for the button
```
TBU
```

Add the automation for the button push
```
TBU
```

Add  a notification on low battery
```
TBU
```

# The outcome
TBU
