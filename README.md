# Passive BLE Monitor integration

### BLE Monitor for Xiaomi MiBeacon, Xiaomi Scale, Qingping, ATC, Govee, Kegtron, Thermoplus, Brifit, Ruuvitag and iNode sensors and device tracking

### Table of content
<!-- TOC -->

- [INTRODUCTION](#introduction)
- [SUPPORTED SENSORS](#supported-sensors)
- [DEVICE TRACKER](#device-tracker)
- [HOW TO INSTALL](#how-to-install)
  - [1. Grant permissions for Python to have rootless access to the HCI interface](#1-grant-permissions-for-python-to-have-rootless-access-to-the-hci-interface)
  - [2. Install the custom integration](#2-install-the-custom-integration)
  - [3. Add your sensors to the MiHome app if you haven’t already](#3-add-your-sensors-to-the-mihome-app-if-you-havent-already)
  - [4. Configure the integration](#4-configure-the-integration)
    - [4a. Configuration in the User Interface](#4a-configuration-in-the-user-interface)
    - [4b. Configuration in YAML](#4b-configuration-in-yaml)
- [CONFIGURATION PARAMETERS](#configuration-parameters)
  - [Configuration parameters at component level](#configuration-parameters-at-component-level)
  - [Configuration parameters at device level](#configuration-parameters-at-device-level)
  - [Configuration in the User Interface](#configuration-in-the-user-interface)
  - [Configuraton in YAML](#configuraton-in-yaml)
  - [Deleting devices and sensors](#deleting-devices-and-sensors)
- [FREQUENTLY ASKED QUESTIONS](#frequently-asked-questions)
- [CREDITS](#credits)
- [FORUM](#forum)

<!-- /TOC -->

## INTRODUCTION

This custom component is an alternative for the standard build in [mitemp_bt](https://www.home-assistant.io/integrations/mitemp_bt/) integration, the [Bluetooth Tracker](https://www.home-assistant.io/integrations/bluetooth_tracker/) integration and the [Bluetooth LE Tracker](https://www.home-assistant.io/integrations/bluetooth_le_tracker/) integration that are available in Home Assistant. BLE monitor supports much more sensors than the build in integration. Unlike the original `mitemp_bt` integration, which is getting its data by polling the device with a default five-minute interval, this custom component is parsing the Bluetooth Low Energy packets payload that is constantly emitted by the sensor. The packets payload may contain temperature/humidity/battery and other data. Advantage of this integration is that it doesn't affect the battery as much as the built-in integration. It also solves connection issues some people have with the standard integration (due to passivity and the ability to collect data from multiple bt-interfaces simultaneously). Read more in the  [FAQ](https://github.com/custom-components/ble_monitor/blob/master/faq.md#why-is-this-component-called-passive-and-what-does-it-mean). BLE monitor also has the possibility to track BLE devices based on its (static) MAC address. It will listen to incoming BLE advertisements for the devices that you have chosen to track.

## SUPPORTED SENSORS

This integration supports **Xiaomi MiBeacon, Qingping, ATC, Xiaomi Scale, Kegtron, Thermoplus, Brifit, Ruuvitag and iNode** sensors at the moment. Support for additional sensors can be requested by opening an [issue](https://github.com/custom-components/ble_monitor/issues). Check the [Frequently Asked Questions (FAQ) page](https://github.com/custom-components/ble_monitor/blob/kegtron-v2/faq.md#my-sensor-from-the-xiaomi-ecosystem-is-not-in-the-list-of-supported-ones-how-to-request-implementation) on how to provide usefull information for adding new sensors.

|Name|Model|Description|Broadcasted properties|Broadcast Rate|Requires [active_scan](#active_scan)|[encryption_key](#encryption_key)|Custom Firmware|Notes|
|---|---|---|---|:---:|:---:|:---:|:---:|---|
|**Xiaomi Hygro thermometer**|**LYWSDCGQ**|![LYWSDCGQ](/pictures/LYWSDCGQ.jpg)<br/>Round body, segment LCD|<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>|~20/min.|No|No|No|
|**Qingping Hygro thermometer**|**CGG1**|![CGG1](/pictures/CGG1.png) ![CGG1](/pictures/CGG1-back.png)<br/>Round body, E-Ink.No logo (pictured on right)|<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>|~20/min.|No| No|No|<ul><li>about 20 readings per minute, although expection have been reported with 1 reading per 10 minutes.</li></ul>|
|**Qingping/MiHome Hygro thermometer** (stock firmware)|**CGG1-M**|![CGG1](/pictures/CGG1.png)![CGG1](/pictures/CGG1-back.png)<br>Round body, E-Ink. `qingping` logo at the back (pictured on left) |<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>|~20/min.|No|[Yes](#encryption_key)|[pvvx](https://github.com/pvvx/ATC_MiThermometer)|<ul><li>about 20 readings per minute, although expection have been reported with 1 reading per 10 minutes.</li></ul>|
|**Qingping/MiHome Hygro thermometer** (custom firmware)|**CGG1-M**|![CGG1](/pictures/CGG1.png)![CGG1](/pictures/CGG1-back.png)<br>Round body, E-Ink. `qingping` logo at the back (pictured on left) |<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li><li>`battery voltage`</li></ul>||No|[Optional](#encryption_key)|N/A||
|**Qingping Temp & RH Monitor Lite**|**CGDK2**|![CGDK2](/pictures/CGDK2.png)<br/>Round body, E-Ink|<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>|~1/10mins.|No|[Yes](#encryption_key)|No||
|**Xiaomi Temperature and Humidity sensor**|**LYWSD02**|![LYWSD02](/pictures/LYWSD02.jpeg)<br/>Rectangular body, E-Ink|<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`*</li></ul>|~20/min.|No|No|No|battery level is available for firmware version 1.1.2_00085 and later|
|**Xiaomi Hygro thermometer** (stock firmware)|**LYWSD03MMC**|![LYWSD03MMC](/pictures/LYWSD03MMC.jpg)<br/>Small square body, segment LCD|<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>|~1/10mins <br/>(battery ~1/hour)|No|[Yes](#encryption_key)|[pvvx](https://github.com/pvvx/ATC_MiThermometer) or [atc1441](https://github.com/atc1441/ATC_MiThermometer)||
|**Xiaomi Hygro thermometer** (custom firmware)|**LYWSD03MMC**|![LYWSD03MMC](/pictures/LYWSD03MMC.jpg)<br/>Small square body, segment LCD|<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li><li>`battery voltage`</li></ul>||No|[Optional](#encryption_key)|N/A||
|**Qingping bluetooth alarm clock**|**CGC1**|![CGC1](/pictures/CGC1.jpg)| <ul><li>`temperature`</li><li>`humidity`</li><li>(WIP) `battery level`*</li></ul>||No|[Yes (Xiaomi MiBeacon advertisement)](#encryption_key)|No|<ul><li>For battery level, we do not have accurate periodicity information yet.</li><li>The sensor sends BLE advertisements in Xiaomi MiBeacon format and Qingping format, but only MiBeacon format is supported currently.</li><li>*Note* - If you have information about update frequency, encryption key requirement, and/or a log with `report_unknown: "qingping"`, we can improve the documentation and implement qingping format support without encryption. Please open an issue with this information.</li></ul>|
|**Qingping Cleargrass CGD1 alarm clock**|**CGD1**|![CGD1](/pictures/CGD1.jpg)<br/>Segment LCD|<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`*</li></ul>|~1/10min.|No|[Yes (Xiaomi MiBeacon advertisement)](#encryption_key)|No|<ul><li>For battery level, we do not have accurate periodicity information yet.</li><li>The sensor sends BLE advertisements in Xiaomi MiBeacon format and Qingping format, but only MiBeacon format is supported currently.</li><li>*Note* - If you have information about update frequency, encryption key requirement, and/or a log with `report_unknown: "qingping"`, we can improve the documentation and implement qingping format support without encryption. Please open an issue with this information.</li></ul>|
|**Qingping Cleargrass indoor weather station with Atmospheric pressure measurement**|**CGP1W**|![CGP1W](/pictures/CGP1W.jpg)|<ul><li>`temperature`</li><li>`humidity`</li><li>`air pressure`</li><li>`battery level`*</li></ul>||No|No|No|<ul><li>For battery level, we do not have accurate periodicity information yet.</li>|
|**MHO-C303 Alarm clock**|**MHO-C303**|![MHO-C303](/pictures/MHO-C303.png)<br/>Rectangular body, E-Ink| <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>|~20/min.|No|No|No||
|**MHO-C401 Alarm clock** (stock firmware)|**MHO-C401**|![MHO-C401](/pictures/MHO-C401.jpg)<br/>Small square body, E-Ink display|<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>|~1/10mins <br/>(battery ~1/hour)|No|[Yes](#encryption_key)|[pvvx](https://github.com/pvvx/ATC_MiThermometer)|
|**MHO-C401 Alarm clock** (custom firmware)|**MHO-C401**|![MHO-C401](/pictures/MHO-C401.jpg)<br/>Small square body, E-Ink display|<ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li><li>`battery voltage`</li></ul>||No|[Yes](#encryption_key)|N/A|
|**Xiaomi Honeywell Formaldehyde Sensor**|**JQJCY01YM**|![JQJCY01YM](/pictures/JQJCY01YM.jpg)<br>OLED display|<ul><li>`temperature`</li><li>`humidity`</li><li>`formaldehyde` (mg/m³)</li><li>`battery level`</li></ul>|~50/min.|No|No|No||
|**Xiaomi Honeywell Smoke Detector (Bluetooth)**|**JTYJGD03MI**|![JTYJGD03MI](/pictures/JTYJGD03MI.png)<br>Smoke detector| <ul><li>`smoke detected`</li><li>`button press`</li><li>`battery level`</li>||No|No|No|<ul><li>Only the Bluetooth model is supported</li></ul>|
|**MiFlora plant sensor**|**HHCCJCY01**|![HHCCJCY01](/pictures/HHCCJCY01.jpg)|<ul><li>`temperature`</li><li>`moisture`</li><li>`illuminance`</li><li>`conductivity`</li></ul>| 1/min.|No|No|No|<ul><li>no battery info with firmware v3.2.1.</li></ul>|
|**VegTrug Grow Care Garden**|**GCLS002**|![GCLS002](/pictures/GCLS002.png)|<ul><li>`temperature`</li><li>`moisture`</li><li>`illuminance`</li><li>`conductivity`</li></ul>||No|No|No|<ul><li>Similar to MiFlora HHCCJCY01.</li></ul>|
|**FlowerPot, RoPot**|**HHCCPOT002**|![HHCCPOT002](/pictures/HHCCPOT002.jpg)|<ul><li>`moisture`</li><li>`conductivity`</li></ul>| 2/min.|No|No|No|<ul><li>No battery info with firmware v1.2.6</li></ul>|
|**Xiaomi Mija Mosquito Repellent (Smart version)**|**WX08ZM**|![WX08ZM](/pictures/WX08ZM.jpg)| <ul><li>`switch state`</li><li>`tablet resource`</li><li>`battery level`</li></ul>|~50/min.|No|No|No||
|**Xiaomi Mijia Window/Door Sensor 2**|**MCCGQ02HL**|![MCCGQ02HL](/pictures/MCCGQ02HL.png)|<ul><li>`open state`</li><li>`light state`</li><li>`battery level`</li></ul>|(Battery level 1/day.)|No|[Yes](#encryption_key)|No|
|**Qingping Window Door/Sensor**|**CGH1**|![CGH1](/pictures/CGH1.png)|<ul><li>`open state`</li><li>`battery level`</li></ul>||No|[Yes](#encryption_key)|No|<ul><li>Battery level is broadcasted, but interval is currently not known.</li></ul>
|**Xiaomi Mijia Smart kettle**|**YM-K1501**|![YM-K1501](/pictures/YM-K1501.png)|<ul><li>`temperature`</li><li>`ext_state` *</li></ul>||No|No|No|<ul><li>`ext_state` values<ul><li>`0` - kettle is idle,</li><li>`1` - kettle is heating water</li><li> `2` - warming function is active with boiling,</li><li>`3` - warming function is active without boiling.</li></ul></li></ul>|
|**Viomi Smart Kettle**|**V-SK152**|![V-SK152](/pictures/V-SK152.png)|<ul><li>`temperature`</li><li>`ext_state` *</li></ul>|~2/min.|No|No|No|<ul><li>`ext_state` values<ul><li>`0` - kettle is idle,</li><li>`1` - kettle is heating water</li><li> `2` - warming function is active with boiling,</li><li>`3` - warming function is active without boiling.</li></ul></li></ul>|
|**Xiaomi Smart Water Leak Sensor**|**SJWS01LM**|![SJWS01LM](/pictures/SJWS01LM.png)|<ul><li>`moisture state` (wet/dry)</li></ul>||No |[Yes](#encryption_key)|No|
|**Xiaomi Motion Activated Night Light**|**MJYD02YL**|![MJYD02YL](/pictures/MJYD02YL.jpg)|<ul><li>`light state` (light detected/ no light)</li><li>`motion` (motion detected/ clear)</li><li>`battery level`</li></ul>|*See Notes*<br/>(Battery level ~12/hour).|No |[Yes](#encryption_key)|No|<ul><li>Light state is broadcasted once every 5 minutes when no motion is detected, when motion is detected the sensor also broadcasts the light state.</li><li>Motion state is broadcasted when motion is detected, but is also broadcasted once per 5 minutes. If this message is within 30 seconds after motion, it's broadcasting `motion detected`, if it's after 30 seconds, it's broadcasting `motion clear`. Additonally, `motion clear` messages are broadcasted at 2, 5, 10, 20 and 30 minutes after the last motion. </li><li>You can use the [reset_timer](#reset_timer) option if you want to use a different time to set the sensor to `motion clear`.</li></ul>|
|**Xiaomi Philips Bluetooth Night Light**|**MUE4094RT**|![MUE4094RT](/pictures/MUE4094RT.jpg)|<ul><li>`motion detected`</li></ul>||No|No|No|<ul><li>The sensor does not broadcast `motion clear` advertisements. It is therefore required to use the [reset_timer](#reset_timer) option with a value that is not 0.</li></ul>
|**Xiaomi Mi Motion Sensor 2**|**RTCGQ02LM**|![RTCGQ02LM](/pictures/RTCGQ02LM.png)|<ul><li>`light state` (light detected/ no light)</li><li>`motion detected`</li><li>`button press`</li><li>`battery level`</li></ul>||No|[Yes](#encryption_key)|No|<ul><li>Light state is broadcasted upon a change in light in the room and is also broadcasted at the same time as motion is detected</li><li>The sensor does not broadcast `motion clear` advertisements. It is therefore required to use the [reset_timer](#reset_timer) option with a value that is not 0)</li><li>The sensor also broadcasts `single press` if you press the button. After each button press, the sensor state shortly shows `single press` and will return to `no press` after 1 second. </li><li>The sensor has an attribute which shows the `last button press`. You can use the state change event to trigger an automation in Home Assistant</li><li>Battery is broadcasted once every few hours</li></ul>|
|**Qingping Motion and ambient light sensor**|**CGPR1**|![CGPR1](/pictures/CGPR1.png)|<ul><li>`illuminance` (in lux)</li><li>`motion` (motion detected/ clear)</li><li>`battery level`</li></ul>|*See Notes*|No|[Yes](#encryption_key)|No|<ul><li>Illumination is broadcasted upon every 10 minutes and when motion is detected.</li><li>Motion state is broadcasted when motion is detected. Additonally, `motion clear` messages are broadcasted at 1, 2, 5, 10, 20 and 30 minutes after the last motion. You can use the [reset_timer](#reset_timer) option if you want to use a different time to set the sensor to `motion clear`.</li><li>Battery level is broadcasted, but interval is currently not known.</li></ul>|
|**Xiaomi Miaomiaoce Digital Baby Thermometer**|**MMC-T201-1**|![MMC-T201-1](/pictures/MMC-T201-1.jpg)|<ul><li>`temperature`</li><li>'calculated body temperature' (please note the disclaimer under *notes*)</li><li>`battery level`</li></ul>|~15-20/min.|No|No|No|**DISCLAIMER**<br />The sensor sends two temperatures in the BLE advertisements, that are converted to a body temperature with a certain algorithm in the original app. We tried to reverse engineering this relation, but we were only able to approximate the relation in the range of 36.5°C - 37.9°C at this moment. It has not been calibrated at elevated body temperature (e.g. if someone has a fever), so measurements displayed in Home Assistant might be different (wrong) compared to those reported in the app. It is therefore advised NOT to rely on the measurements in BLE monitor if you want to monitor your or other peoples body temperature / health). If you have additional measurements, especially outside the investigated range, please report them in this [issue](https://github.com/custom-components/ble_monitor/issues/264).)|
|**Xiaomi Mi Electric Toothbrush T500**|**M1S-T500**|![M1S-T500](/pictures/M1S-T500.jpg)|<ul><li>`toothbrush mode`</li><li>`battery level`</li></ul>||No|No|No|At the moment, we are looking into the meaning of the different states. If you have more info which state corresponds to what, please post a message in [this topic](https://github.com/custom-components/ble_monitor/issues/319)|
|**Yeelight Smart Wireless Switch**|**YLAI003**|![YLAI003](/pictures/YLAI003.jpg)|<ul><li>`single press`</li><li>`double press`</li><li>`long press`</li></ul>|*See Notes*|No|[Yes](#encryption_key)|No|<ul><li>After each button press, the sensor state shows the type of press. It will return to `no press` after the time set with the [reset_timer](#reset_timer) option. It is advised to change the reset time to 1 second (default = 35 seconds).</li><li>The sensor has an attribute which shows the `last button press`. You can use the state change event to trigger an automation in Home Assistant.</li></ul>|
|**Yeelight Remote Control**|**YLYK01YL**|![YLYK01YL](/pictures/YLYK01YL.jpg)|<ul><li>`button pressed` (`on`, `off`, `color temperature`, `+`, `M`, `-`)</li><li>Type of press(`single press` or `long press`)</li></ul>||No|[Partially](#encryption_key)|No| <ul><li>The state of the remote sensor shows the combination of both, the attributes shows the button being used and the type of press individually.</li><li>It will return to `no press` after the time set with the [reset_timer](#reset_timer) option. It is advised to change the reset time to 1 second (default = 35 seconds). </li><li>Additinally, two binary sensors are generated (one for `short press`, one for `long press`), which is `True` when pressing `on`, `+` or `-` and `False` when pressing `off`.</li></ul>|
|**Yeelight Fan Remote Control**|**YLYK01YL-FANCL**|![YLYK01YL-FAN](/pictures/YLYK01YL-FAN.jpg)|<ul><li>`button pressed` (`fan toggle`, `light toggle`, `standard wind speed`, `color temperature`, `natural wind speed`, `brightness`)</li><li>Type of press(`single press` or `long press`)</li></ul>||No|[Partially](#encryption_key)|No|<ul><li>The state of the remote sensor shows the combination of both, the attributes shows the button being used and the type of press individually.</li><li>It will return to `no press` after the time set with the [reset_timer](#reset_timer) option. It is advised to change the reset time to 1 second (default = 35 seconds). </li></ul>|
|**Yeelight Ventilator Fan Remote Control**|**YLYK01YL-VENFAN**|![YLYK01YL-VENFAN](/pictures/YLYK01YL-VENFAN.jpg)| <ul><li>`button pressed` (`swing`, `power toggle`, `timer 30 minutes`, `timer 60 seconds`, `strong wind speed`, `low wind speed`)</li><li>Type of press(`single press` or `long press`)</li></ul>||No|[Partially](#encryption_key)|No|<ul><li>The state of the remote sensor shows the combination of both, the attributes shows the button being used and the type of press individually.</li><li>It will return to `no press` after the time set with the [reset_timer](#reset_timer) option. It is advised to change the reset time to 1 second (default = 35 seconds).</ul>|
|**Yeelight Bathroom Heater Remote Control**|**YLYB01YL-BHFRC**|![YLYB01YL-BHFRC](/pictures/YLYB01YL-BHFRC.jpg)|<ul><li>`button pressed` (`heat`, `air exchange`, `dry`, `fan`, `swing`, `speed -`, `speed +`, `stop` or `light`)</li><li>Type of press(`single press` or `long press`)</li><li>Type of press(`single press` or `long press`)</li></ul>||No|[Partially](#encryption_key)|No|<ul><li>The state of the remote sensor shows the combination of both, the attributes shows the button being used and the type of press individually.</li><li>It will return to `no press` after the time set with the [reset_timer](#reset_timer) option. It is advised to change the reset time to 1 second (default = 35 seconds).</li></ul>|
|**Yeelight Rotating Dimmer**|**YLKG07YL, YLKG08YL**|![YLKG07YL_YLKG08YL](/pictures/YLKG07YL_YLKG08YL.png)|<ul><li>press type (`rotate`, `rotate (presses)`</li><li>Type of press(`single press` or `long press`)</li></ul>|No|[Yes](#encryption_key)|No|<ul><li>Broadcasts the , `short press`, `long press`). For rotation, it reports the rotation direction (`left`, `right`) and how far you rotate (number of `steps`). For `short press` it reports how many times you pressed the dimmer, for `long press` it reports the time (in seconds) you pressed the dimmer. The dimmer sensor state will return to `no press` after the time set with the [reset_timer](#reset_timer) option. It is advised to change the reset time to 1 second (default = 35 seconds).</li></ul>|
|**Linptech Switch**|**K9B**|Broadcasts the press type (`short press`, `double press`, `long press`) for each button. There are three different versions of this switch, with one, two or three buttons. The switch sensor state will return to `no press` after the time set with the [reset_timer](#reset_timer) option. It is advised to change the reset time to 1 second (default = 35 seconds). Advertisements are probably encrypted (not confirmed yet), you need to set the encryption key in your configuration, see for instructions the [encryption_key](#encryption_key) option.|![YLKG07YL_YLKG08YL](/pictures/Linptech_K9B.png)||
|**Mi Smart Scale 1 / Mi Smart Scale 2**|**XMTZC01HM, XMTZC04HM**|Broadcasts `weight`, `non-stabilized weight` and `weight removed`. The `weight` is only reported after the scale is stabilized, while the `non-stabilized weight` is reporting all weight measurements. For additional data like BMI, viscaral fat, etc. you can use e.g. the [bodymiscale](https://github.com/dckiller51/bodymiscale) custom integration. If you want to split your measurements into different persons, you can use [this template sensor](https://community.home-assistant.io/t/integrating-xiaomi-mi-scale/9972/533) https://community.home-assistant.io/t/integrating-xiaomi-mi-scale/9972/533?u=ernst |![XMTZC05HM](/pictures/XMTZC04HM.png)||
|**Mi Body Composition Scale 2 / Mi Body Fat Scale**|**XMTZC02HM, XMTZC05HM, NUN4049CN**|Broadcasts `weight`, `non-stabilized weight`, `impedance` and `weight removed`. The `weight` is only reported after the scale is stabilized, while the `non-stabilized weight` is reporting all weight measurements. For additional data like BMI, viscaral fat, muscle mass etc. you can use e.g. the [bodymiscale](https://github.com/dckiller51/bodymiscale) custom integration. If you want to split your measurements into different persons, you can use [this template sensor](https://community.home-assistant.io/t/integrating-xiaomi-mi-scale/9972/533)|![XMTZC05HM](/pictures/XMTZC05HM.png)||
|**Kegtron KT-100, KT-200**|**KT-100 / KT-200**|Broadcasts `volume dispensed` for each port and port attributes (`keg size`, `start volume`, `state`, `index` and `port name`.|![Kegtron](/pictures/kegtron.jpg)|&#10004;|
|**Smart Hygrometer**|**Thermoplus**|Rounded square body, LCD screen, e.g. Brifit, Oria. <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![smart Hygrometer](/pictures/Thermoplus_smart_hygrometer.jpg)||
|**Lanyard Hygrometer**|**Thermoplus**|Square body, no screen, e.g. Brifit, Oria. <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![lanyard Hygrometer](/pictures/Thermoplus_lanyard_hygrometer.jpg)||
|**Mini Hygrometer**|**Thermoplus**|Round body, no screen, is also sold under different brands, e.g. Brifit, Oria. <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![mini hygrometer](/pictures/Thermoplus_mini_hygrometer.jpg)||
|**Brifit Thermometer Hygrometer**|**T201**|Square body, no screen, is also sold under different brands, e.g. Oria. <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>, about 80 readings per minute.|![T201](/pictures/T201.jpg)||
|**Ruuvitag**|**Ruuvitag**|Round body. Broadcasts temperature, humidity, air pressure, battery voltage, battery level, motion and acceleration. If some of these sensors are not updating, make sure you use the latest firmware (v5). `motion detected` is reported in HA when the motion counter is increased between two advertisements. You can use the [reset_timer](#reset_timer) option to set the time after which the motion sensor will return to `motion clear`, but it might be overruled by the advertisements from the sensor.|![ruuvitag](/pictures/ruuvitag.jpg)||
|**iNode Energy Meter**|**iNode Energy Meter**|Energy meter based on pulse measuring. Broadcasts energy, power, battery and voltage. Energy and power are calculated based on the formula's as given in the [documentation](https://docs.google.com/document/d/1hcBpZ1RSgHRL6wu4SlTq2bvtKSL5_sFjXMu_HRyWZiQ/edit#heading=h.l38j4be9ejx7). The `constant` factor that is used for these calculations as well as the light level are given in the energy sensor attributes. Advertisements are broadcasted every 1 a 2 seconds, but the measurement data is only changed once a minute.|![iNode_Energy_Meter](/pictures/iNode_Energy_Meter.png)||
|**Govee H5051 Thermometer Hygrometer (BLE only)**| **H5051** |Oval body, LCD screen.  <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![H5051](/pictures/Govee_H5051.jpg)|&#10004;|
|**Govee H5075 Thermometer Hygrometer**| **H5072** |Oval body, LCD screen.  <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![H5072](/pictures/Govee_H5072.jpg)|&#10004;|
|**Govee H5074 Thermometer Hygrometer**| **H5074** |Square body, no screen.  <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![H5074](/pictures/Govee_H5074.jpg)|&#10004;|
|**Govee H5075 Thermometer Hygrometer**| **H5075** |Rounded square body, LCD screen.  <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![H5075](/pictures/Govee_H5075.jpg)|&#10004;|
**Govee H5101/Govee H5102 Thermometer Hygrometer**| **H5101, H5102** |Rounded square body, LCD screen.  <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![H5101/H5102](/pictures/Govee_H5101_H5102.jpg)|&#10004;|
|**Govee H5177 Thermometer Hygrometer**| **H5177** |Rounded square body, Backlight LCD Touchscreen.  <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![H5177](/pictures/Govee_H5177.jpg)|&#10004;|
|**Govee H5179 Thermometer Hygrometer (BLE only)**| **H5179** |Square body, no screen.  <ul><li>`temperature`</li><li>`humidity`</li><li>`battery level`</li></ul>.|![H5179](/pictures/Govee_H5179.png)|&#10004;|


## NOT SUPPORTED
 * Qingping Hygro thermometer `CGG1-H` (Homekit version)

*The amount of actually received data is highly dependent on the reception conditions (like distance and electromagnetic ambiance), readings numbers are indicated for good RSSI (Received Signal Strength Indicator) of about -75 till -70dBm.*

**Do you want to request support for a new sensor? In the [FAQ](https://github.com/custom-components/ble_monitor/blob/master/faq.md#my-sensor-from-the-xiaomi-ecosystem-is-not-in-the-list-of-supported-ones-how-to-request-implementation) you can read instructions how to request support for other sensors.**

## DEVICE TRACKER

This integration is also capable of tracking Bluetooth devices, as long as it is using a static MAC address (public or random static (lifetime) address). To track a device, add the [mac](#mac) address of the device to track under the [devices](#devices) option and enable the option [track_device](#track_device). The tracker will listen to every advertisement that is send by the device. As this can be quite often, an [tracker_scan_interval](#tracker_scan_interval) can be set to reduce the number of state updates in Home Assistant (default 20 seconds). When no advertisments are received anymore, the device tracker entity state will turn to `Away` after the set [consider_home](#consider_home) interval (default 180 seconds).

## HOW TO INSTALL

### 1. Grant permissions for Python to have rootless access to the HCI interface

This is usually only needed for alternative installations of Home Assistant that only install Home Assistant core.

- to grant access:

     ```shell
     sudo setcap 'cap_net_raw,cap_net_admin+eip' `readlink -f \`which python3\``
     ```

- to verify:

     ```shell
     sudo getcap `readlink -f \`which python3\``
     ```

*In case you get a PermissionError, check the [Frequently Asked Questions (FAQ) page](faq.md).

### 2. Install the custom integration

The easiest way to install the BLE Monitor integration is with [HACS](https://hacs.xyz/). First install [HACS](https://hacs.xyz/) if you don't have it yet. After installation you can find this integration in the HACS store under integrations.

Alternatively, you can install it manually. Just copy paste the content of the `ble_monitor/custom_components` folder in your `config/custom_components` directory. As example, you will get the `sensor.py` file in the following path: `/config/custom_components/ble_monitor/sensor.py`. The disadvantage of a manual installation is that you won't be notified about updates.

### 3. Add your sensors to the MiHome app if you haven’t already

Many Xiaomi ecosystem sensors do not broadcast BLE advertisements containing useful data until they have gone through the "pairing" process in the MiHome app. The encryption key is also (re)set when adding the sensor to the MiHome app, so do this first. Some sensors also support alternative ATC firmware, which doesn't need to be paired to MiHome.

### 4. Configure the integration

There are two ways to configure the integration and your devices (sensors), in the User Interface (UI) or in your YAML configuration file. Choose one method, you can't use both ways at the same time. You are able to switch from one to the other, at any time.

#### 4a. Configuration in the User Interface

Make sure you restart Home Assistant after the installation in HACS. After the restart, go to **Configuration** in the side menu in Home Assistant and select **Integrations**. Click on **Add Integrations** in the bottom right corner and search for **Passive BLE Monitor** to install. This will open the configuration menu with the default settings. The options are explained in the [configuration parameters](#configuration-parameters) section below and can also be changed later in the options menu. Depending on the sensor, the sensors should be added to your Home Assistant automatically within a few seconds till 10 minutes.

  ![Integration setup](https://raw.githubusercontent.com/custom-components/ble_monitor/master/pictures/configuration_screen.png)

#### 4b. Configuration in YAML

Alternatively, you can add the configuration in `configuration.yaml` as explained below. The options are the same as in the UI and are explained in the [configuration parameters](#configuration-parameters) section below. After adding your initial configuration to your YAML file, or applying a configuration change in YAML, a restart is required to load the new configuration. Depending on the sensor, the sensors should be added to your Home Assistant automatically within a few seconds till 10 minutes.

An example of `configuration.yaml` with the minimum configuration is:

```yaml
ble_monitor:
```

An example of `configuration.yaml` with all optional parameters is:

```yaml
ble_monitor:
  bt_interface: '04:B1:38:2C:84:2B'
  discovery: True
  active_scan: False
  report_unknown: False
  decimals: 1
  period: 60
  log_spikes: False
  use_median: False
  restore_state: False
  devices:
    # sensor
    - mac: 'A4:C1:38:2F:86:6C'
      name: 'Livingroom'
      encryption_key: '217C568CF5D22808DA20181502D84C1B'
      temperature_unit: C
      decimals: 2
      use_median: False
      restore_state: default
    - mac: 'C4:3C:4D:6B:4F:F3'
      name: 'Bedroom'
      temperature_unit: F
    - mac: 'B4:7C:8D:6D:4C:D3'
      reset_timer: 35
    # device tracker
    - mac: 'D4:3C:2D:4A:3C:D5'
      track_device: True
      tracker_scan_interval: 20
      consider_home: 180
```

Note: The encryption_key parameter is only needed for sensors, for which it is [pointed](#supported-sensors) that their messages are encrypted.

## CONFIGURATION PARAMETERS

### Configuration parameters at component level


#### bt_interface

   (MAC address or list of multiple MAC addresses)(Optional) This parameter is used to select the Bluetooth-interface of your Home Assistant host. When using YAML, a list of available Bluetooth-interfaces available on your system is given in the Home Assistant log during startup of the Integration, when you enable the Home Assistant [logger at info-level](https://www.home-assistant.io/integrations/logger/). If you don't specify a MAC address, by default the first interface of the list will be used. If you want to use multiple interfaces, you can use the following configuration:

```yaml
ble_monitor:
  bt_interface:
    - '04:B1:38:2C:84:2B'
    - '34:DE:36:4F:23:2C'
```

   Default value: First available MAC address

#### hci_interface

   (positive integer or list of positive integers)(Optional) Like the previous option `bt_interface`, this parameter is also used to select the bt-interface of your Home Assistant host. It is however strongly advised to use the `bt_interface` option and not this `hci_interface` option, as the hci number can change, e.g. when plugging in a dongle. However, due to backwards compatibility, this option is still available. Use 0 for hci0, 1 for hci1 and so on. On most systems, the interface is hci0. In addition, if you need to collect data from several interfaces, you can specify a list of interfaces:

```yaml
ble_monitor:
  hci_interface:
    - 0
    - 1
```

   Default value: No default value, `bt_interface` is used as default.

#### discovery

   (boolean)(Optional) By default, the component creates entities for all discovered, supported sensors. However, situations may arise where you need to limit the list of sensors. For example, when you receive data from neighboring sensors, or when data from part of your sensors are received using other equipment, and you don't want to see entities you do not need. To resolve this issue, simply add an entry of each MAC-address of the sensors you need under `devices`, by using the `mac` option, and set the `discovery` option to False:

```yaml
ble_monitor:
  discovery: False
  devices:
    - mac: '58:C1:38:2F:86:6C'
    - mac: 'C4:FA:64:D1:61:7D'
```

Data from sensors with other addresses will be ignored. Default value: True

#### active_scan

   (boolean)(Optional) In active mode scan requests will be sent, which is most often not required, but slightly increases the sensor battery consumption. 'Passive mode' means that you are not sending any request to the sensor but you are just receiving the advertisements sent by the BLE devices. This parameter is a subject for experiment. Default value: False

#### batt_entities [DEPRECATED]

   (boolean)(Optional) This option is deprecated, please remove from your configuration.

#### decimals

   (positive integer)(Optional) Number of decimal places to round. This setting can be overruled with for specific devices with settings [at device level](#configuration-parameters-at-device-level). Default value: 1

#### period

   (positive integer)(Optional) The period in seconds during which the sensor readings are collected and transmitted to Home Assistant after averaging. Default value: 60.

   *To clarify the difference between the sensor broadcast interval and the component measurement period: The LYWSDCGQ transmits 20-25 valuable BT LE messages (RSSI -75..-70 dBm). During the period = 60 (seconds), the component accumulates all these 20-25 messages, and after the 60 seconds expires, averages them and updates the sensor status in Home Assistant. The period does not affect the consumption of the sensor. It only affects the Home Assistant sensor update rate and the number of averaged values. We cannot change the frequency with which sensor sends data.*

#### log_spikes

   (boolean)(Optional) Puts information about each erroneous spike in the Home Assistant log. Default value: False

   *There are reports (pretty rare) that some sensors tend to sometimes produce erroneous values that differ markedly from the actual ones. Therefore, if you see inexplicable sharp peaks or dips on the temperature or humidity graph, I recommend that you enable this option so that you can see in the log which values were qualified as erroneous. The component discards values that exceeds the sensor’s measurement capabilities. These discarded values are given in the log records when this option is enabled. If erroneous values are within the measurement capabilities (-40..60°C and 0..100%H), there are no messages in the log. If your sensor is showing this, there is no other choice but to calculate the average as the median (next option).*

#### use_median

   (boolean)(Optional) Use median as sensor output instead of mean (helps with "spiky" sensors). Please note that both the median and the mean values in any case are present as the sensor state attributes. This setting can be overruled with for specific devices with settings [at device level](#configuration-parameters-at-device-level). Default value: False

   *The difference between the mean and the median is that the median is **selected** from the sensor readings, and not calculated as the average. That is, the median resolution is equal to the resolution of the sensor (one tenth of a degree or percent), while the mean allows you to slightly increase the resolution (the longer the measurement period, the larger the number of values will be averaged, and the higher the resolution can be achieved, if necessary with disabled rounding).*

#### restore_state

   (boolean)(Optional) This option will, when set to `True`, restore the state of the sensors. If your [devices](#devices) are configured with a [mac](#mac) address, they will restore immediately after a restart of Home Assistant to the state right before the restart. If you didn't configure your [devices](#devices), the state will be restored upon the first BLE advertisement being received.
   with `restore_state` set to `False`, the integration needs some time (see [period](#period) option) after a restart before it shows the actual data in Home Assistant. During this time, the integration receives data from your sensors and calculates the mean or median values of these measurements. During this period, the entity will have a state "unknown" or "unavailable" when `restore_state` is set to `False`. Setting it to `True` will prevent this, as it restores the old state, but could result in sensors having the wrong state, e.g. if the state has changed during the restart. By default, this option is disabled, as especially the binary sensors would rely on the correct state. For measuring sensors like temperature sensors, this option can be safely set to `True`. It is also possible to overrule this setting for specific devices with settings [at device level](#configuration-parameters-at-device-level). Default value: False

#### report_unknown

   (`Xiaomi`, `Qingping`, `ATC`, `Mi Scale`, `Kegtron`, `Thermoplus`, `Brifit`, `Govee`, `Ruuvitag`, `Other` or `False`)(Optional) This option is needed primarily for those who want to request an implementation of device support that is not in the list of [supported sensors](#supported-sensors). If you set this parameter to `Xiaomi`, `Qingping`, `ATC`, `Mi Scale`, `Kegtron`, `Thermoplus`, `Brifit`, `Govee` or `Ruuvitag`, then the component will log all messages from unknown devices of the specified type to the Home Assitant log (`logger` component must be enabled at info level). When set to `Other`, all BLE advertisements will be logged. **Attention!** Enabling this option can lead to huge output to the Home Assistant log, especially when set to `Other`, do not enable it if you do not need it! Details in the [FAQ](https://github.com/custom-components/ble_monitor/blob/master/faq.md#my-sensor-from-the-xiaomi-ecosystem-is-not-in-the-list-of-supported-ones-how-to-request-implementation). Default value: False


### Configuration parameters at device level

#### devices

   (Optional) The devices option is used for setting options at the level of the device and/or if you want to whitelist certain sensors with the `discovery` option. For tracking devices, it is mandatory to specify your devices to be tracked. Note that if you use the `devices` option, the `mac` option is also required.

### Configuration in the User Interface

   To add a device, open the options menu of the integration and look for the `mac` of your device in the devices drop down menu. Most sensors are automatically added to the drop down menu. If it isn't shown or if you want to add a device to be tracked, select **Add Device** in the device drop down menu and click on Submit. You can modify existing configured devices in a similar way, by selecting your device in the same drop down menu and clicking on Submit. Both will show the following form.

  ![device setup](/pictures/device_screen.png)

### Configuraton in YAML

   To add a device, add the following to your `configuration.yaml`

```yaml
ble_monitor:
  devices:
    # sensors
    - mac: 'A4:C1:38:2F:86:6C'
      name: 'Livingroom'
      encryption_key: '217C568CF5D22808DA20181502D84C1B'
      temperature_unit: C
      decimals: 2
      use_median: False
      restore_state: default
    - mac: 'C4:3C:4D:6B:4F:F3'
      reset_timer: 35
    # device trackers
    - mac: 'D4:3C:2D:4A:3C:D5'
      track_device: True
      tracker_scan_interval: 20
      consider_home: 180
```

#### mac

   (string)(Required) The `mac` option (`MAC address` in the UI) is used to identify your device based on its mac-address. This allows you to define other additional options for this specific device, to track it and/or to whitelist it with the `discovery` option. You can find the MAC address in the attributes of your sensor (`Developers Tools` --> `States`). For deleting devices see the instructions [below](#deleting-devices-and-sensors).

#### name

   When using configuration in the User Interface, you can modify the device name by opening your device, via configuration, integrations and clicking on devices on the BLE monitor tile. Select the device you want to change the name of and click on the cogwheel in the topright corner, where you can change the name. You will get a question wether you want to rename the individual entities of this device as well (normally, it is advised to do this).

   (string)(Optional) When using YAML, you can use the `name` option to link a device name and sensor name to the mac-address of the device. Using this option (or changing a name) will create new sensor/tracker entities. The old data won't be transfered to the new sensor/tracker. The old sensor/tracker entities can be safely deleted afterwards, but this has to be done manually at the moment, see the instructions [below](#deleting-devices-and-sensors). The sensors/trackers are named with the following convention: `sensor.ble_sensortype_device_name` (e.g. `sensor.ble_temperature_livingroom`) in stead of the default `ble_sensortype_mac` (e.g. `sensor.ble_temperature_A4C1382F86C`). You will have to update your lovelace cards, automation and scripts after each change. Note that you can still override the entity_id from the UI. Default value: Empty

```yaml
ble_monitor:
  devices:
    - mac: 'A4:C1:38:2F:86:6C'
      name: 'Livingroom'
```

#### encryption_key

   (string, 24 or 32 characters)(Optional) This option is used for sensors broadcasting encrypted advertisements. The encryption key should be 32 characters (= 16 bytes) for most devices (LYWSD03MMC, CGD1, MCCGQ02HL, and MHO-C401 (original firmware only). Only Yeelight YLYK01YL (all types), YLYB01YL-BHFRC, YLKG07YL and YLKG08YL require a 24 character (= 12 bytes) long key. The case of the characters does not matter. The keys below are an example, you need your own key(s)! Information on how to get your key(s) can be found [here](https://github.com/custom-components/ble_monitor/blob/master/faq.md#my-sensors-ble-advertisements-are-encrypted-how-can-i-get-the-key). Default value: Empty

```yaml
ble_monitor:
  devices:
    - mac: 'A4:C1:38:2F:86:6C'
      encryption_key: '217C568CF5D22808DA20181502D84C1B'
```

#### temperature_unit

   (C or F)(Optional) Most sensors are sending BLE advertisements with temperature data in Celsius (C), even when set to Fahrenheit (F) in the MiHome app. However, sensors with custom ATC firmware will start sending temperature data in Fahrenheit (F) after changing the display from Celsius to Fahrenheit. This means that you will have to tell `ble_monitor` that it should expect Fahrenheit data for these specific sensors, by setting this option to Fahrenheit (F). Note that Home Assistant is always converting measurements to C or F based on your Unit System setting in Configuration - General. Default value: C

```yaml
ble_monitor:
  devices:
    - mac: 'A4:C1:38:2F:86:6C'
      temperature_unit: F
```

#### decimals (device level)

   (positive integer or `default`)(Optional) Number of decimal places to round. Overrules the setting at integration level. Default value: default (which means: use setting at integration level)

```yaml
ble_monitor:
  devices:
    - mac: 'A4:C1:38:2F:86:6C'
      decimals: 2
    - mac: 'A4:C1:38:2F:86:6B'
      decimals: default
```

#### use_median (device level)

   (boolean or `default`)(Optional) Use median as sensor output instead of mean (helps with "spiky" sensors). Overrules the setting at integration level. Please note that both the median and the mean values in any case are present as the sensor state attributes. Default value: default (which means: use setting at integration level)

```yaml
ble_monitor:
  devices:
    - mac: 'A4:C1:38:2F:86:6C'
      use_median: True
    - mac: 'A4:C1:38:2F:86:6B'
      use_median: default
```

#### restore_state (device level)

   (boolean or `default`)(Optional) This option will, when set to `True`, restore the state of the sensors immediately after a restart of Home Assistant to the state right before the restart. Overrules the setting at integration level. See for a more detailed explanation the setting at integration level. Default value: default (which means: use setting at integration level)

```yaml
ble_monitor:
  devices:
    - mac: 'A4:C1:38:2F:86:6C'
      restore_state: True
    - mac: 'A4:C1:38:2F:86:6B'
      restore_state: default
```

#### reset_timer

   (possitive integer)(Optional) This option sets the time (in seconds) after which a sensor is reset to `motion clear` (motion sensors) or `no press` (button and dimmer sensors). After each `motion detected` advertisement or `button/dimmer press`, the timer starts counting down again. Setting this option to 0 seconds will turn this resetting behavior off.

   Note that motion sensors also sends advertisements themselves that can overrule this setting. To our current knowledge, advertisements after 30 seconds of no motion send by the sensor are `motion clear` messages, advertisements within 30 seconds are `motion detected` messages. For button and dimmer sensors, it is advised to set the `reset_timer` to 1 second. Default value: 35

```yaml
ble_monitor:
  devices:
    - mac: 'A4:C1:38:2F:86:6C'
      reset_timer: 35
```

#### track_device

   (boolean)(Optional) Enabling this option will create a device tracker in Home Assistant. The device tracker will be `Home` as long as it receives data and will move to `Away` after no data is received anymore for more that the set period with [consider_home](#consider_home). Note that your device should have a fixed MAC address to be able to track. Default value: False


```yaml
ble_monitor:
  devices:
    - mac: 'A4:C1:38:2F:86:6C'
      track_device: True
      tracker_scan_interval: 20
      consider_home: 180
```

#### tracker_scan_interval

   (positive integer)(Optional) To reduce the state updates in Home Assistant and not spam your Home Assistant, it is advised to set a scan interval. After a BLE advertisement is received and the state has been updated, Home Assistant Scan will not update the state during the set interval to safe resources. The setting is in seconds. Default value: 20

#### consider_home

   (positive integer)(Optional) This option sets the period with no data after which the device tracker is considered to be away. The setting is in seconds. Default value: 180


### Deleting devices and sensors

Removing devices can be done by removing the corresponding lines in your `configuration.yaml`. In the UI, you can delete devices by typing `-` in the `MAC address` field. Note that if the [discovery](#discovery) option is set to `True` sensors will be discovered automatically again.

Unfortunately, old devices and entities are not entirely deleted by this, they will still be visible, but will be `unavailable` after a restart. The same applies for changing a name of an existing device in YAML, the entities with the old name will still remain visible, but with an `unavailable` state after a restart. To completely remove these left overs, follow the following steps.

#### 1. Remove old entities

First, delete the old entities, by going to **configuration**, **integrations** and selecting **devices** in the BLE monitor tile. Select the device with old entities and select each unavailable entity, to delete it manually. If the delete button isn't visible, you will have to restart Home Assistant to unload the entities. Make sure all old entities are deleted before going to the next step.

#### 2. Remove old devices

If the device doesn't have any entities anymore, you can delete the device as well. Unfortunately, Home Assistant doesn't have an delete option to remove the old device. To overcome this problem, we have created a `service` to help you solve this. Go to **developer tools**, **services** and select the `ble_monitor.cleanup_entries` service. Click on **Call service** and the device should be gone. If not, you probably haven't deleted all entities (go to step 1).

## FREQUENTLY ASKED QUESTIONS

Still having questions or issues? Please first have a look on our [Frequently Asked Questions (FAQ) page](faq.md) to see if your question is already answered. There are some useful tips also.
If your question or issue isn't answered in the FAQ, please open an [issue](https://github.com/custom-components/ble_monitor/issues).

## CREDITS

Credits and big thanks should be given to:

- [@Magalex](https://community.home-assistant.io/u/Magalex) and [@Ernst](https://community.home-assistant.io/u/Ernst) for the component creation, development, and support.
- [@koying](https://github.com/koying) for implementing the configuration in the user interface.
- [@Thrilleratplay](https://github.com/Thrilleratplay) for the Govee sensor support
- [@tsymbaliuk](https://community.home-assistant.io/u/tsymbaliuk) for the idea and the first code.

## FORUM

You can more freely discuss the operation of the component, ask for support, leave feedback and share your experience in [our topic](https://community.home-assistant.io/t/passive-ble-monitor-integration/303583) on the Home Assistant forum.
