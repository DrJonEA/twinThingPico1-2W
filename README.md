# twinThingPico1-2W

IoT Client framework using FreeRTOS coreMQTT-Agent for Pico W and Pico 2 W. Provide Pub and Sub messages, 
plus add a framework to maintain concept of state of a device which can  be then queried. For my own projects
I added a python based twins framework to provide a shadow copy of the devices state. 
[See TwinThingPyMQTT](https://github.com/jondurrant/TwinThingPyMQTT).

<img src="img/twinThing.jpeg" alt="isolated" width="200"/>

C/C++ Library which is dependent on the following libraries:
+ [Pico SDK](https://github.com/raspberrypi/pico-sdk) (tested with 2.2.0)
+ [LWIP](https://github.com/lwip-tcpip/lwip)
+ [FreeRTOS Kernel](https://github.com/FreeRTOS/FreeRTOS-Kernel) (tested with 11.2.0)
+ [FreeRTOS coreMQTT](https://github.com/FreeRTOS/coreMQTT) (tested with 2.3.1)
+ [FreeRTOS coreMQTT-Agent](https://github.com/FreeRTOS/coreMQTT-Agent) (tested with 1.3.1)
+ [json-maker](https://github.com/rafagafe/json-maker)
+ [tiny-json](https://github.com/rafagafe/tiny-json)

## Compatibility
The library should compile for Pico W or Pico 2 W. Should connect to any MQTT v4 broker. I commonly use
[Flexpi](https://flespi.com/) as an online broker service, [Mosquitto](https://mosquitto.org/) and [EMQX](https://www.emqx.com/en).

Note the library excludes any TLS transport. On my Udemy course I do add this by using [WolfSSL](https://www.wolfssl.com/). To include this within 
the library would move the Licence from MIT to GPLv3 which I wish to avoid doing. 

## Full Background to the Framework
I show how the framework is built from the ground up on my 
[Udemy course](https://www.udemy.com/course/iot-with-rpi-pico-w/?referralCode=331C13FDD6ABC00BFB81). 

This goes from WiFi connection, Sockets (TCPIP and TLS), connecting to MQTT, MQTT client lifecycle, 
publishing messages, subscribing, handling messages, security, and digital twin concepts to handle state.

## Compiling the Library
A CMake file is included for inclusion in your projects CMakeLists.txt file.  This define the library *twinThingPicoW*. This is also dependent on the following other libraries:
+ 	pico_stdlib 
+	pico_unique_id
+	hardware_adc
+	hardware_rtc
+	json_maker 
+	tiny_json
+	FreeRTOS-Kernel
+	coreMQTT 
+	coreMQTTAgent

An include file called MQTTConfig.h is expected to be available with configuration and definitions. An example is in the *default config* folder.

## Project concepts

### MQTT Agent
Active central service which maintains an MQTT connection with the broker. It handles publication
of messages and  subscriptions. Using a MQTTRouter class received messages are processed.

A device with the client ID "device1" sends a lifecycle online message to topic: *TNG/device1/LC/ON*

Message: "{'online':1}"

A Will is registed for the topic: *TNG/device1/LC/OFF* 

Message: "{'online':0}"

### MQTT Router
The MQTT Agent is provided with a single class confirming to MQTT Router interface.  This triggers all subscribtions through the "subscribe" member and handles any message through the "route" message.

There is an example router for ping messages MQTTRouterPing:

This expects  message on topic *TNG/device1/TPC/PING* with the message: "{"id": 821561}". The ID can be anything but it is echoed in the PONG response on topic: *TNG/device1/TPC/PONG*

Ping also works on groups to the topic *GRP/ALL/TPC/PING*.  Response is on the device specific
topic as before: *TNG/device1/TPC/PONG*


The MQTTRouterTwin examples extends the Router concept to handle digital twins notification of a devices state.

I commonly use inheritance to add additional subscriptions and message handlers.

### Twin
A Twin is an extension to the framework where the object contains a state. State must be conform to the State.h interface. An example StateTemp is provided. This is fairly simple and only allows for 16 state elements.

State is managed through the "TwinTask" which manage the Twin state as an active task in FreeRTOS Kernel.  the reason this is a task is so updates can get queued rather than running within the MQTT Agent Routers callback. The Router MQTTRouterTwin is provided to manage state.

A device publishes its states a delta or full state. Examples:

Topic: *TNG/device1/STATE/UPD*  Message: '{"delta":{"on":true}}'

Topic: *TNG/device1/STATE/UPD* Message: '{"state":{"trn":0,"temp":21.5208,"boot":643,"on":true}}'


## Example project 

*coming soon*






