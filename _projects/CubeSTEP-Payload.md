---
layout: page
title: CubeSTEP Payload Processor
subtitle: Academic Research Project, 2023
description: ESP-32 Microcontroller sub-system component developed to manage the payload of the CubeSTEP satellite. This project included hardware, firmware, and driver development.
importance: 1
category: Academic
---
The CubeSat Technology Exploration Program (CubeSTEP) is a joint collaboration between Cal Poly Pomona (CPP) and NASA's Jet Propulsion Laboratory (JPL). The program recently received $900,000 in funding from NASA to carry out its mission[(link)](https://polycentric.cpp.edu/2023/08/nasa-awards-900k-to-cpp-space-defense-tech-and-security-news/). Our project was presented to the AIAA Science and Technology Forum and Exposition[(link)](https://arc.aiaa.org/doi/10.2514/6.2023-1879). I worked on the program during my '23-'24 school year at CPP.

#### Mission Statement

1. Develop a generic, yet fully-featured, CubeSat platform that future CPP projects can use as a foundation to advance their research.
2. Utilize additive manufacturing to allow for custom payloads with minimal modifications to the platform.
3. Develop and test an Oscillating Heat Pipe (OHP) integrated battery case which was provided by JPL.

#### Objective

The satellite had multiple computer systems that needed to be developed and integrated into the on-board computer (OBC) and the flight software.

* Attitude Determination and Control System (ACS)
* UHF Antenna and Transmitter
* Electrical Power System (EPS)
* Payload Processor

My primary assignment had two aspects: to develop the Payload processor (ESP32-Pico-D4) and model its required functionality in the flight software (JPL's F-Prime) to allow the OBC to interface with the payload.

This page will cover my experience developing the firmware for the ESP32 processor. See CubeSTEP Flight Software[(link)](/projects/CubeSTEP-Fprime) for information regarding the F-Prime development.

---

The firmware for the Payload Processor was developed using the PlatformIO extension for VS Code. This extension has all the functionality needed to build and upload code to the ESP32 microprocessor. The ESP32-Pico-D4 development kit was used to test and debug the firmware while the Payload component is being developed. The project's repository is available on GitHub[(link)](https://github.com/23navin/esp32-payload-development).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/daq.jpg" title="DAQ" class="img-fluid border border-dark rounded z-depth-1" %}
        <div class="caption">
            A prototype of the payload processor being used to control and monitor the payload during testing
        </div>
    </div>
</div>

The assignment of developing the payload processor came with some requirements. It needed to be able to read temperatures from the payload's temperature sensors, set a power level for a payload element, store and retrieve data on the processor's onboard memory, transmit and receive data via I2C, and receive operational commands via I2C. To satisfy these specific requirements, certain libraries had to be developed. I was able to achieve some of the requirements by adapting open-source libraries but others had to be built from scratch.

##### Core Libraries

Telemetry (struct)

* Stores the payload's temperatures and heater's output level at a point in time
* Has methods to set telemetry data and output the data as a string
* Designed to hold samples of system temperature for periodical system checks and during experiment
* Telemetry data will either eventually be transmitted over I2C or stored into a .csv log file

FileCore (class)

* Handles the internal flash storage over SPI
* Has methods to create, write to, and read from a .csv file

SensorCore (class)

* Sample temperature values from 16 thermistors using on-board ADC
* Can write data directly to a Telemetry struct via pointer.

HeaterCore (class)

* Creates a PWM output at a GPIO to control a payload element
* Uses two timers to create a Trailing-Edge PWM signal that has a period of 12 seconds
* The duty cycle and period can be set as needed by the experimen

I2cCore (class)

* Allows the OBC to communicate with the payload processor via I2C
* The ESP32 acts as the slave
* ESP32 expects telemetry data requests and commands to start and control the experiment

##### Bringing it all together

With the libraries developed to provide the core functionality, the next step is to tie it all together into a functioning system using FreeRTOS. Once the system is working as intended, a C++ driver will be developed for easy integration into the flight software on the OBC.
