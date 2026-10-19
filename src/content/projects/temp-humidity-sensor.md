---
title: Temperature and Humidity Sensor
description: A compact ESP32 environmental sensor that reads temperature and humidity, then sends lightweight CBOR messages to a local service over Wi-Fi.
sortOrder: 1
year: 2024
technologies:
  - C
  - ESP32
  - ESP-IDF
  - FreeRTOS
  - I2C
  - UDP
links:
  repository: https://github.com/CoryCoward/TempHumiditySensorProject
---

![Close-up of an electronics development board](https://images.unsplash.com/photo-1518770660439-4636190af475?auto=format&fit=crop&w=1400&q=85)

## Context

Reading from a sensor is a straightforward embedded exercise. The more interesting part for me was treating it as one end of a small, practical IoT system: sample the physical world, move the data across a network, and give it a useful destination.

The device is an [ESP32-S3 DevKitC-1](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/hw-reference/esp32s3/user-guide-devkitc-1.html) paired with an [AHT20 I2C temperature and humidity sensor](https://www.adafruit.com/product/5183). It uses the [ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/index.html) framework and its FreeRTOS-based runtime to handle periodic sampling and network delivery.

## Build log

### Separating sampling from delivery

The firmware uses two tasks with a queue between them. One task reads the AHT20 at regular intervals and places the result in the queue. The second task retrieves readings, prepares the outgoing message, and handles delivery. Keeping those responsibilities separate makes the sampling path simple and prevents network timing from becoming part of the sensor-reading logic.

### A lightweight message path

I considered sending JSON directly to a REST endpoint, but the device does not need that much ceremony. The sender encodes readings with [CBOR](https://cbor.io/) and transmits them through a UDP socket instead. The receiver acknowledges each message; if an acknowledgement does not arrive, the device retries up to three times. It is a small protocol, but it adds useful protection against a dropped packet without turning the project into a larger networking exercise.

### Making it practical to run

The project keeps environment-specific settings in a local configuration file rather than the source itself. Once connected to Wi-Fi, the device sends readings at the configured interval, which makes it easy to move between a bench setup and a longer-running test without changing the core application.

## Future Plans

The current firmware is centered on one sensor. The next useful step would be supporting several sensors and a broader set of environmental measurements, such as air pressure and air quality.

I would also like to add SD-card storage for readings collected while Wi-Fi is unavailable, then flush the backlog after the connection returns. A small display would make the device more useful on its own by showing the current conditions at a glance.

On the software side, I am interested in trying [FreeRTOS + TCP](https://www.freertos.org/FreeRTOS-Plus/FreeRTOS_Plus_TCP/index.html) for the UDP layer in place of the base socket API.
