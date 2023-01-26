---
title: Water Softener Salt Tank Level Sensor
description: SwitcWater Softener Salt Tank Level Sensor Build Guide
image: /img/switchbot_bulb5.webp
keywords: [esphome, laser, tof, water softener, distance, vl53l0x]
---

# Device Name / Model Here


This project was originally implemented by @jrbwrjr of Discord - thanks Grandpa Jim!

Purchase VL53L0X on Amazon [Amazon](https://amzn.to/3WxlPr1)  
Purchase ESP32 DevBoard on Amazon [Amazon](https://amzn.to/3HqCzMl)


<details><summary>Settings</summary>     
<p>

| Setting | Description
|---------------|-------------
| setoption13 1 | Set On/Off switch to respond instantly
| setoption30 1 | Sets domain to a light
</p></details>

<details><summary>Rules</summary>     
<p>

Any rules

```
Rule1 on something do something endon
```
</p></details>

<details><summary>GPIO Layout</summary>     
<p>

| GPIO |    Component | Description |
|------ |-------------|-------------|         
|GPIO00	| None
|GPIO01	| None
|GPIO02	| None
|GPIO03	| None
|GPIO04	| None
|GPIO07	| VL530X #1 XSHUT
|GPIO08	| I2C SDA
|GPIO09	| I2C SCL
|GPIO10	| None
|GPIO12	| None
|GPIO13	| None
|GPIO14	| None
|GPIO15	| None
|GPIO16	| None
</p></details>

<details><summary>ESPHome YAML</summary>     
<p>

```yaml
esphome:
  name: water-softener

ota:
  password: !secret otapassword

preferences:
  flash_write_interval: 240min

esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: arduino

logger:

mqtt:
  broker: !secret mqtt_host
  username: !secret mqtt_user
  password: !secret mqtt_password

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    static_ip: !secret staticip_watersoftenerc3
    gateway: !secret staticip_gateway
    subnet: !secret staticip_subnet
    dns1: !secret staticip_dns1

  ap:
    ssid: "Water-Softener Fallback Hotspot"
    password: !secret fallback_password

captive_portal:

i2c:
  sda: GPIO8
  scl: GPIO9

sensor:
  - platform: vl53l0x
    name: "Softener Salt Level"
    id: tof_softener_salt_level
    address: 0x29
    # the enable_pin is for the XSHUT on the vl53l0x
    # allowing multiple on one i2c bus
    # Eliminate the line below if only using one.
    enable_pin: GPIO7
    update_interval: 12s
    filters:
      #- offset: -0.01
      - median
      # linear calibration – where .2m from the sensor is 100% fill level
      # and .79 meters is your 0%/empty level. Take your own measurements once
      # the sensor is in place and adjust.
      - calibrate_linear:
        - 0.790 -> 0
        - 0.537 -> 44 
        - 0.412 -> 64
        - 0.308 -> 82
        - 0.200 -> 100
    unit_of_measurement: "%"
    accuracy_decimals: 1
    on_value_range:
    - below: .25
      then:
        - switch.turn_on: red
    - above: .25
      below: .50
      then:
        - switch.turn_on: blue
    - above: .50
      then:
        - switch.turn_on: green

switch:
  - platform: gpio
    name: "Red"
    id: red
    interlock: &interlock_group [red, green, blue]
    pin: GPIO3
  - platform: gpio
    name: "Green"
    id: green
    interlock: *interlock_group
    pin: GPIO4
  - platform: gpio
    name: "Blue"
    id: blue
    interlock: *interlock_group
    pin: GPIO5
```
</p></details>


### Pics
![alt text](/img/devices/mj-s01_main.jpg "Martin Jerry MJ-S01")

![alt text](/img/devices/mj-s01_inside1.jpg "Martin Jerry MJ-S01 Insides")
