---
layout: post
title:  "Foot Motion-Sensing Lighting System"
date:   2023-06-05 20:00:00 +0100
author: Jonathan Persson
categories: IoT
---

The "Foot Motion-Sensing Lighting System" is an innovative Internet of Things (IoT) project that aims to create a responsive desktop lighting solution. The project uses motion detection technology to activate a LED light strip based on the user's foot movement under the desk. By implementing this system, ease of use, energy efficiency and productivity in work or study environments are improved.

The system uses sensors to detect foot movements under the surface of the desk. When motion is detected, the IoT device communicates wirelessly with the LED strip and immediately illuminates the workspace. This real-time motion detection and lighting activation provides a hands-free and intuitive experience, enabling users to seamlessly control lighting without having to reach for switches or manually adjust brightness.

Estimated build time: 2 hours

![](https://hackmd.io/_uploads/HyhjDfAuh.jpg)

## Objective
The project aims to create a simple and practical solution to automatically activate the desk lighting based on movements from the user's feet under the desk. By eliminating the need for manual controls or switches, the system provides a smooth and comfortable user experience.

I chose the project because I thought it would give me important insights into IoT devices and how much energy lights use when no one is in the room.

## Material


| Hardware | Description | Price |
| -------- | -------- | -------- |
| Raspberry Pi Pico W     | RP2040 based microcontroller     | [electro:kit](https://www.electrokit.com/produkt/raspberry-pi-pico-w/) 98 kr |
| PIR sensor     | Movement sensor     | [electro:kit](https://www.electrokit.com/produkt/pir-rorelsedetektor-hc-sr501/) 49 kr|
| Breadboard     | Construction base | [electro:kit](https://www.electrokit.com/produkt/kopplingsdack-840-anslutningar/) 69 kr |
| Male-Male Jumper Wires     | Connect sensor to MCU | [electro:kit](https://www.electrokit.com/produkt/labbsladd-40-pin-30cm-hane-hane/) 49 kr|
| USB A - Micro B USB cable     | Power to MCU | [electro:kit](https://www.electrokit.com/produkt/usb-kabel-a-hane-micro-b-5p-hane-1-8m/) 39 kr |
|Total:|| 296 kr|

## Computer setup

* Microsoft VSCode
* Node JS
* Pymakr plugin
* Micropython plugin
* Micropython firmware for RP2040

Step 1: Download and install [Node js](https://nodejs.org/en).
Step 2: Download and install [VS Code](https://code.visualstudio.com/Download).
Step 3: Open VS Code and then the Extensions manager located on the left of the window.
Step 4: Look for Pymakr and Install it. 
Step 5: Create a new project by clicking "Create new project" inside of Pymakr.
Step 6: Download and install [Python3](https://www.python.org/downloads/).
Step 7: Download the [micropython firmware](https://micropython.org/download/rp2-pico-w/) and install it by holding down the BOOTSEL button and dragging the file to the microcontroller connected to your PC.
Step 8: In Pymaker your device should be visible. Click "Connect device" and "Start developer mode".
Step 9: Click "Create terminal"
Step 10: Start testing code in the REPL

You will also see two files have been created in your project, main.py and boot.py. Start writing the code in main.py and execute them on the device by saving your file (CTRL+S).

## Putting everything together

![Breadboard](https://hackmd.io/_uploads/HyNLyGCdh.png)
![Schematics](https://hackmd.io/_uploads/ryrc1GCdn.png)

The two pictures seen above describe how I've connected the components. The Raspberry Pi Pico W is connected to the Pir sensor via 3 separate cables, 5V, GND and Data. The voltage is provided to the sensor from the VSYS pin on the Pico, Ground from GND and lastly data from GPIO13.

![](https://hackmd.io/_uploads/Hy45MfC_2.jpg)

## Platform

I will be using a locally hosted MQTT broker alongside home assistant to directly control the LED light strip and log the activity of the light to an online-based broker at Adafruit. This is done since I'd like to see the state of the light when I'm away from home since I can't access home assistant outside my network.

I think in the future when I can access my home assistant from outside the network I'll completely remove Adafruit from this set-up.
## The code

### Wifi connectivity

To ultimately turn on my LED light strip we must be able to send messages across the internet. What better way to do this than with wifi? 

Enable wifi and connect by running the following code.
```python=
def wifi_connect():
    wlan = network.WLAN(network.STA_IF)

    if not wlan.isconnected():
        wlan.active(True)
        wlan.connect(secrets["ssid"], secrets["password"])
        
        while not wlan.isconnected() and wlan.status() >= 0:
            print('.')
            sleep(1)
```

### User movement sensing logic

Now how do we reliably determine if the user is sitting at the desk or not? The code handles the user activity status based on a PIR sensor's value. It then takes appropriate actions when the user is considered inactive for a certain duration and performs actions when activity is detected after a period of inactivity. The specific actions involve connecting to and sending messages to an MQTT broker and Adafruit MQTT broker to control devices or update their states.

```python=
 while True:
    state = sensor_pin.value()

    if(seconds_elapsed > user_inactive_timeout and not user_is_inactive):
        homeassistant_client.connect()
        send_message(homeassistant_client, "OFF")
        send_message(adafruit_client, "0")
        homeassistant_client.disconnect()
        seconds_elapsed = 0
        user_is_inactive = True

    if(state == 1 ):
        if(previous_sensor_value == 0 and user_is_inactive):
            homeassistant_client.connect()
            send_message(homeassistant_client, "ON")
            send_message(adafruit_client, "1")
            homeassistant_client.disconnect()

            time.sleep(10)
        user_is_inactive = False
        seconds_elapsed = 0
    time.sleep(1)
    seconds_elapsed += 1
    previous_sensor_value = state
```

The messages are sent with an MQTT library called "umqtt simple". This code is imported from an [open source library](https://github.com/micropython/micropython-lib). Since I didn't write the code I won't showcase it in this paper.
## Network architecture and data transmission

![](https://hackmd.io/_uploads/S1dqRxCO2.jpg)

For controlling my LED light strip, I opted to route the communication through Home Assistant, as the light strip is already connected to it. In this setup, the microcontroller sends MQTT messages to the locally running Home Assistant server on my network. The payload of these messages can be either "ON" or "OFF". Home Assistant acts as an intermediary, converting the MQTT messages into Zigbee messages that can be interpreted by the LED light strip and responded to accordingly. This solution allows for seamless communication between the microcontroller and the LED strip, leveraging the capabilities of Home Assistant for smart home automation.

The overall data-sending rate can vary based on the frequency of the user's foot movements. The system is designed to send data only when specific conditions are met, such as the user transitioning between active and inactive states or when the sensor detects a change in foot movement. However, to avoid unnecessary data transmission messages aren't sent if the state remains unchanged since the last was sent. When data is however received in home assistant, it is always saved to a database. In time I think this can become quite a huge amount of data which I'll have to find a solution to.


To design an effective and efficient system, considerations were made regarding data transmission and wireless protocols to address the device's range and battery consumption. The following design choices were implemented to optimize these aspects:

MQTT Protocol: The system utilizes the MQTT protocol for data transmission. MQTT is a lightweight, publish-subscribe messaging protocol that is known for its efficiency and low overhead.

Event-Based Transmission: Data is only transmitted when specific conditions are met, such as a change in the sensor state. This also reduces network stress.

Sleep Modes and Power Optimization: The system incorporates sleep modes and power optimization techniques to conserve battery power. When the device is inactive or not transmitting data, it can enter low-power sleep modes to reduce power consumption. 
## Presenting the data

The decision was made to log the data in the Adafruit MQTT broker due to its ease of use and the appealing user interface (UI) provided on the Adafruit website. The Adafruit MQTT broker was selected as the logging platform based on its simplicity and the positive experience with its UI design.

A dashboard was implemented to display information about the light's status and its states over specific time intervals. The dashboard presents real-time information indicating whether the light is currently enabled or disabled. Additionally, it provides insights into the light's state within the past hour and 24 hours, allowing for a comprehensive understanding of the light's operational patterns and behavior.

By using the Adafruit MQTT broker and the intuitive dashboard, the project ensures convenient data logging and offers a visually pleasing interface for monitoring and analyzing the light's performance characteristics. This logging approach facilitates data-driven decision-making and enables the user to gain valuable insights into the light's usage patterns and trends.

![](https://hackmd.io/_uploads/r1nZjb0Oh.png)


## Finalizing the design

I think this project was very fun to make and I am happy it turned out well. I didn't quite manage to come up with a mechanism for mounting the electronics under the desk, but I'll definitely come up with a solution sooner or later. 

In the future, I'd like to add more sensors to better determine if I am indeed sitting at my desk, perhaps Lidar or microwave sensing sensors? By adding more sensors I hope to further save on the electric bill and save money.

Here is the final result.

![](https://hackmd.io/_uploads/H1dcYM0Oh.jpg)
![](https://hackmd.io/_uploads/r1_ctMR_2.jpg)
![](https://hackmd.io/_uploads/rJd9KMAd2.jpg)

![](https://hackmd.io/_uploads/rJkhnMAu2.gif)
