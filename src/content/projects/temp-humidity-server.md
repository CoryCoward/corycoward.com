---
title: Temperature and Humidity Sensor Server
description: A small .NET background service that receives CBOR sensor packets over UDP and forwards them as JSON to a configurable REST endpoint.
sortOrder: 2
year: 2024
technologies:
  - C#
  - .NET 9
  - CBOR
  - UDP
  - HTTP
  - Raspberry Pi
links:
  repository: https://github.com/CoryCoward/TempHumidityDBBackend
---

![Close-up of an electronics development board](https://images.unsplash.com/photo-1518770660439-4636190af475?auto=format&fit=crop&w=1400&q=85)

## Context

The sensor firmware handles one side of this project: it samples the environment and sends compact UDP messages. This service is the other side. It receives those messages on the local network, turns the binary payload into useful application data, and forwards it to the endpoint responsible for storing and presenting it.

It runs as a long-lived .NET background service on a [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/) in my home network. The application keeps the device-specific transport details away from the REST API, leaving each part of the system with a smaller job to do.

## Build log

### Receiving and decoding messages

The service listens on a configurable UDP port for datagrams from the ESP32. Each packet contains a [CBOR](https://cbor.io/) payload with temperature and relative-humidity values. The service deserializes that payload into a structured C# model before doing anything else with it.

### Forwarding data to the application layer

After decoding a message, the service sends the same fields as JSON in an HTTP `POST` request to a configurable REST endpoint. Using dependency injection and `HttpClient` keeps this bridge small and testable while avoiding a direct dependency between the sensor firmware and the web-facing API.

### Running it where it belongs

The service is designed to run continuously rather than as a manual command. On the Raspberry Pi, it is hosted under `systemd`, which gives it a predictable start-up path and lets it restart after a failure without someone needing to notice first.

## Future Plans

CBOR is a good fit for the current sensor, but the service could become more flexible by detecting and decoding more than one incoming format. That would make it easier to bring new devices into the same local pipeline without forcing them all into the same wire format.

I would also like to make data handling pluggable. A received reading could be written to a local database, sent to a REST endpoint, or passed to another destination without changing the UDP receiver itself. The current service is intentionally narrow; that separation would be a sensible way to grow it without losing the simplicity that makes it useful now.
