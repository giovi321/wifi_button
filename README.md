This is a cheap, customizable, WiFi-enabled, battery-operated, ultra-low power consumption touch button with no moving parts that relies on ESPHome and is designed to be used with Home Assistant.
It is designed to be long-lasting both in terms of durability and battery life and easy to build & operate

# License
The content of this repository is licensed under the [WTFPL](http://www.wtfpl.net/).

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
- [Wemos D1 Mini](https://www.aliexpress.com/item/32651747570.html) [3.33€]
- [USB plug](https://www.aliexpress.com/item/1005005476223469.html) [1.77€]
- [USB type C charging circuit](https://www.aliexpress.com/item/1005005227218470.html) [1.87€]
- [Battery 2000mAh](https://www.aliexpress.com/item/1005002919536938.html) [5.44€]
- [Photocoupler TLP570](https://www.aliexpress.com/item/1005004627959174.html) [2.14€]
- [Capacitive touch sensor](https://www.aliexpress.com/item/32874585710.html) [0.60€]
- [180k ohm resistor](https://aliexpress.com/item/1005004562905294.html) [2.55€]
- [Autotapping screws M2.6 x 5mm](https://www.aliexpress.com/item/1005003330377202.html)  [1.30€]
- Optional: [Screw connector for PCB](https://www.aliexpress.com/item/4001283309384.html), if you want you can solder the battery directly to the PCB/breadboard [1.90€]
- Optional: [PCB pin header strip](https://www.aliexpress.com/item/1005003541838313.html), if you want you can solder the cables that connect the touch sensor directly to the PCB/breadboard [1.17€]
- Optional: [PCB mounted micro switch 90°](https://www.aliexpress.com/item/1005005338134247.html), if you don't require the switch, you can place a jumper in place of the switch in the board [3.15€]

- Options for PCB:
  - Breadbord (make your own design) [c. 3€]
  - PCB I have designed
    - Soon available for purchase [here](https://www.buymeacoffee.com/giovi321/e/147823) [15€ cad., 3€ for additional boards]
    - Print your own on [allpcb](https://www.allpcb.com/?Mb_InviteId=60399) [c. 30€ excl. import taxes], see the Gerber files in the repository
 - 3D printed case (see the .stl files in this repository)
   -  Print it yourself [c. 3€]
   -  Print it on xometry.eu [c. 36€]

# Resources
- Circuit schematic - see the files in the "PCB" folder of the repository
![Schematic](https://github.com/giovi321/wifi_button/assets/6443515/69339dc0-75fa-4347-9059-0b09e5a07b0d)

- 3D case - see the files in the "3D stl" folder of the repository
![Top 1](https://github.com/giovi321/wifi_button/assets/6443515/5813c38f-b581-444f-836f-050e6ff17dc5)
![Top 2](https://github.com/giovi321/wifi_button/assets/6443515/1286280e-becd-4afa-8b9c-e91157d0610c)
![Bottom 1](https://github.com/giovi321/wifi_button/assets/6443515/5aa3b782-1b2a-4216-b39c-ef8fe647e2d7)
![Bottom 2](https://github.com/giovi321/wifi_button/assets/6443515/9094013c-8715-4622-83c1-68e2a4e85942)

# How does it work
- The USB type C connector makes the device modern and easy to charge, but feel free to use any other type of connector
- The charger module keeps the battery charged and you safe from fire hazards, it is simply plug and play
- The device is based on the Wemos D1 Mini (I'm using a cheap clone, even though a branded one would be better for lower energy consumption) and ESPHome because they are very easy to use; certainly there are more energy efficient solutions
- We need a voltage sensor connected to one analog input of the Wemos D1 Mini in order to read the battery status and receive a notification on Home Assistant when it is about to die. To do this, we take advantage of the integrated voltage divider of the pin A0 of the Wemos D1 Mini, which can support up to 3.3 volts input. Since we are providing a 5 volts input, we need to add a 180kohm resistor
- In order to keep energy consumption low, we need to keep the Wemos D1 Mini in a deep-sleep state
	- however, in deep sleep state the Wemos D1 Mini won't be able to read an input offered by the touch sensor
	- therefore, I have opted to use the touch sensor to wake up the Wemos D1 Mini that will go in deep-sleep automatically every 10 seconds
	- the Wemos D1 Mini will be configured to send two mqtt messages whenever it wakes up:
		- one message with the battery voltage
		- one message with "hey somebody touched the touch sensor"
- 3.7v battery is the way to go to power the Wemos D1 Mini, and 2000 mAh fits the small design of the case; feel free to use a different amperage battery. It provides up to 1.5 years of uptime on a single charge (it can vary, see the dedicated section of this file)
   - I haven't tested it, but from my research online the power consumption of different versions of Wemos D1 Mini in deep-sleep mode powered from the 5v input (which is less efficient because there is a voltage regulator in the middle) can vary depending on the manufacturer (see table below)
   - Assuming we are unlucky and got the worse energy-performing Wemos D1 Mini, the average consumption is 69.47mA, giving it a life of almost 399 days (the calculation steps are available below). If we take into consideration the best case scenario, that is 78µA consumption, the battery would last almost 3x: 1,068 days
   - We are not taking into consideration the power consumption of the touch sensor, phototransistor and voltage sensor. The first two components have a very small consumption as it peaks mostly when you activate the touch sensor. For the voltage sensor I couldn't find anything online. In any case, even assuming an absurde value of 15% increase in power consumption, it's still a decent number of days.
   - I don't think I need to say that: i) these values are purely theorethical, the nominal rating of the battery does not equal its empirical performance; ii) the amount of times you will use the button will influence the duration of the battery
   - Pro tip: if you desolder the LED that is installed on the touch sensor, you save some precious µA. However I haven't tried it and I don't know if it would affect the touch sensor functionality
```
Wemos D1 Mini power consumption table
          | WEMOS D1 MINI CLONE | WEMOS D1 MINI LOLIN | WEMOS D1 MINI WAVGAT |
|:-------:|:-------------------:|:-------------------:|:--------------------:|
| Minimum |        144 µA       |         75µA        |        206 µA        |
| Maximum |        147 µA       |         80µA        |        214 µA        |
| Average |        156 µA       |         78µA        |        209 µA        |

Source: https://programarfacil.com/esp8266/esp8266-deep-sleep-nodemcu-wemos-d1-mini/
```
```
Battery life calculation steps
To calculate the approximate duration of a battery with a given load, you can use the formula:
	Battery Life (in hours) = Battery Capacity (in mAh) / (Load Current (in µA) / 1000)
The battery capacity is 2000 mAh and the load current is 209 µA. Plugging these values into the formula:
	Battery Life = 2000 mAh / (209 µA / 1000) ≈ 9569.38 hours
	Battery Life in days = 9569.38 hours / 24 ≈ 398.72 days
```
- As mentioned earlier, a touch sensor does not provide an analog output. To use it, you need to interface it with a microcontroller like the Wemos D1 Mini, or in our case, to minimize energy consumption, we can opt to use a phototransistor
- The phototransistor is a straightforward electronic component with two inputs and two outputs:
	- the two inputs are the anode (+) and cathode (-) of a LED integrated in the package of the chip
	- the two outputs are the Emitter and Collector of the transistor, to put it simple those two pins allow current to flow between the two pins when the phototransistor is "activated"
- When the touch sensor is activated with your finger, it provides the phototransistor with current, enough to light the internal LED of the phototransistor, therefore allowing current to pass between the emitter and the collector of the transistor
  - When the current flows between the Emitter and the Collector, we are simply shorting the RST and GND pins of the Wemos D1 Mini, therefore waking it up


# Building process
## Hardware
### Electronics
Just follow the schematics provided in the Resources chapter of the readme to build your own breadbord, otherwise simply solder the components on the PCB ordered either from me or another producer.

### Case
The assembly is extremly simple:
- Hot glue the touch sensor to the top flat part of the dome (inside the dome, obviously) - the sensor can feel touch also through a thin layer of non conductive material (the layer of the top of the dome is 4mm thick)
- Insert the USB type C connector from the bottom of the bottom plate and screw it in or glue it with cyanoacrylate glue
- Solder the connector to the battery charger module and plug the battery
- Hot-glue the main circuit board to the bottom plate that closes the case
- Close the dome and use four M2.6 screws to secure it

## Software
### Install ESPHome
There are three ways to install ESPHome:
- If you have hass.io (the official home-assistant OS), follow [this guide](https://www.home-assistant.io/integrations/esphome/)
- If you are running any other version of home-assistant such as a venv or docker you have the following two options:
  - Use docker, following [this guide](https://esphome.io/guides/installing_esphome.html)
  - Install it in a python venv following the simple steps below or follow [this guide](https://esphome.io/guides/installing_esphome.html)
```
python3 -m venv esphome
source esphome/bin/activate
pip3 install esphome
```
Check that ESPHome has been installed correctly:
```
esphome version
```
If you're not running hass.io, you can install ESPHome dashboard by following the steps below or [this guide](https://esphome.io/guides/getting_started_command_line.html?highlight=dashboard+install#bonus-esphome-dashboard)

```esphome dashboard config/
pip install tornado esptool
```
Test the installation:`esphome dashboard config/`
Create a new systemd service to run the dashboard automatically by issuing (as root) `nano /etc/systemd/system/esphome.service`.
Now put this in the file, close and save it.
```
[Unit]
Description=ESPHome Dashboard
After=network-online.target

[Service]
Type=simple
ExecStart=/root/esphome/bin/esphome config/ dashboard
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```
Issue the following:
```
systemctl --system daemon-reload
systemctl enable esphome.service
```
Then check if the service is running by issuing `service esphome status`.


### ESPHome code
ATTENTION: remember to switch off the main circuit board (or unplug the battery, if you didn't include the switch) before connecting the Wemos D1 Mini to the computer via USB!

```
mqtt:
  broker: 192.168.1.100
  username: yourmqttusername
  password: yourmqttpassword
  birth_message:
    topic: esphome/bedside_light_switch_1
    payload: "on"

deep_sleep:
  run_duration: 10s

sensor:
  - platform: adc
    pin: A0
    name: "Bedside light switch 1 battery"
    id: battery_voltage_12f
    filters:
      - multiply: 4.511904762
    accuracy_decimals: 3
```
ATTENTION: to calculate the 4.511904762 value (second line from the bottom of the code snippet for ESPHome) you need to:
- measure the battery voltage with a multimeter when the device is running (i.e., not in deep-sleep mode)
- set the value to 1 like this: `multiply: 1`
- check what value is reported on home assistant from the device
- do a very simple proportion:
```
a:b=c:x
b*c/a=x
a=the reported voltage from home assistant
b=the multiplier you set (1)
c=the voltage measured by your multimeter

0.840:1=3.79:x
1*3.79/0.840=4.511904762
```
So with those numbers (they might vary depending on the Wemos D1 Mini brand and the battery type) the multiplier we need to set is 4.511904762.

### Home Assistant code
Add the sensor for the battery to convert voltage to percentage (edit configuration.yaml)
Remember to change the name of the sensor `sensor.bedside_light_button_1_bedside_light_button_1_battery` to your sensor's name.
The values shown in this code snipped apply to 3.7v lithium batteries.
```
sensor:
  - platform: template
    sensors:
      bedside_light_button_1_battery_percentage:
        friendly_name: "Bedside light button 1 battery percentage"
        device_class: battery
        unit_of_measurement: "%"
        value_template: >
          {% set voltage = states('sensor.bedside_light_button_1_bedside_light_button_1_battery')|float %}
          {% if voltage < 2.64 %}
            0
          {% else %}
            {% set range_voltage = 3.85 - 2.64 %}
            {% set range_percentage = 100 %}
            {% set battery_percentage = ((voltage - 2.64) / range_voltage) * range_percentage %}
            {{ battery_percentage | round(0) }}
          {% endif %}
```

Add the automation to trigger the light when you press the button (edit automations.yaml):
```
- alias: bedside_light_switch_1
  description: 'Turn on bedside table lamp on button touch'
  trigger:
  - platform: mqtt
    topic: esphome/bedside_light_switch_1
    payload: 'on'
  mode: single
```

Add  a notification on low battery (edit automations.yaml):
```
- alias: Bedside light 1 low battery
  description: 'Send notification if bedside light button battery is below 10%'
  trigger:
  - platform: numeric_state
    entity_id: sensor.bedside_light_button_1_battery
    below: 10
  action:
  - service: notify.persistent_notification
    data:
      message: The battery of bedside light 1 is low. Please charge.
      title: Bedside light 1 low battery
  mode: single
```

# Buy me a coffee!
Did you like the wifi-button? Buy me a coffee!

[![Buy me a coffee](https://www.buymeacoffee.com/assets/img/custom_images/black_img.png)](https://www.buymeacoffee.com/giovi321)

# The outcome
The final result:
[Piucture TBU]

The bare metal device:
![image](https://github.com/giovi321/wifi_button/assets/6443515/64da228e-b42b-4f29-9183-e1def4d3971b)

The early prototype:
![image](https://github.com/giovi321/wifi_button/assets/6443515/9bc13564-ead0-46ce-8924-6469bd61dd8b)
