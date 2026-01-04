# Serial Protocol

Serial Protocol is a lightweight and versatile communication protocol library designed for (serial) communication between devices. It provides simple interfaces and a basic framework for serial data transmission. Due to the modular design, it can be easily extended to support various communication protocols. The included examples demonstrate how to use the library for different applications.

This library is the basis for the communication used by [MineDuino](https://github.com/JayPi4c/MineDuino). A Simple application to test the client side of the library is given with the [SerialController](https://github.com/JayPi4c/SerialController) project.

## Installation

This library is currently not available through the Arduino Library Manager. To use it in your project, follow these steps:
1. Download this repository as ZIP
2. In the Arduino IDE, go to `Sketch` -> `Include Library` -> `Add .ZIP Library...`
3. Select the downloaded ZIP file
4. Include the library in your sketch with `#include <SerialProtocol.h>`
5. Explore the examples provided in `File` -> `Examples` -> `Serial Protocol`

Alternatively, you can clone the repository directly into your Arduino libraries folder. To update the library, simply pull the latest changes from the repository.

## Quick Start

The library includes a predfined protocol called `TLVProtocol` (Type-Length-Value Protocol) which can be used to get started quickly. Below is a simple example demonstrating how to set up a TLV Protocol communication between two devices. (Roughly based on the Basci example provided with the library)

```cpp
#include "SerialProtocol.h"
#include "TLVProtocol.h"


void onMessage(uint8_t type, int len, uint8_t* msg);
bool state = false;

TLVIncomingProtocol incoming(64, onMessage);
TLVOutgoingProtocol outgoing;

SerialProtocol serial(&incoming, &outgoing);

// define message handler
void onMessage(uint8_t type, int len, uint8_t* msg) {
  switch (type) {
    case 0x01: // Echo command
      {
        String text = "";
        for (int i = 0; i < len; i++) text += (char)msg[i];
        String echo = "ECHO: " + text;
        serial.send(0x00, echo.c_str());
        break;
      }
    case 0x02: // Toggle built-in LED
      {
        state = !state;
        digitalWrite(LED_BUILTIN, state ? HIGH : LOW);
        serial.send(0x00, "OK");
        break;
      }
    default:
      {
        serial.send(0x00, "ERROR: Unknown command");
        break;
      }
  }
}

void setup() {
  // enable built-in LED to indicate "waiting" status
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LED_BUILTIN, HIGH);

  serial.begin();

  // turn off built-in LED to indicate "ready" status
  digitalWrite(LED_BUILTIN, LOW);
}

void loop() {
}


void serialEvent() {
  serial.update();
}
```


## Tested Platforms

- Arduino Uno
- Arduino Mega
- Arduino Nano
- Arduino Leonardo
- ESP32
- rp2040 (Raspberry Pi Pico)
