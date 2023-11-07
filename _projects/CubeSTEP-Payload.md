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

The firmware for the Payload Processor was developed using the PlatformIO extension for VS Code. This extension has all the functionality needed to build and upload code to the ESP32 microprocessor. The ESP32-Pico-D4 development kit was used to test and debug the firmware while the Payload component is being developed. The project's repository is available on GitHub[(link)](https://github.com/23navin/esp32-payload-development). The Payload Processor will often be referred to simply as 'the device.'

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/daq.jpg" title="DAQ" class="img-fluid border border-dark rounded z-depth-1" %}
        <div class="caption">
            A prototype of the payload processor being used to control and monitor the payload during testing
        </div>
    </div>
</div>

The assignment of developing the payload processor came with some requirements. It needed to be able to read temperatures from the payload's temperature sensors, set a power level for a payload element, store and retrieve data on the processor's onboard memory, transmit and receive data via I2C, and receive operational commands via I2C. To satisfy these specific requirements, certain libraries had to be developed. I was able to achieve some of the requirements by adapting open-source libraries but others had to be built from scratch.

##### Libraries that I made

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

With the libraries developed to provide the core functionality, the next step is to tie it all together into a functioning system using FreeRTOS. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/firmware_code.png" title="app_main" class="img-fluid border border-dark rounded z-depth-1" %}
        <div class="caption">
            Firmware's app_main start up sequence. Intiatializes hardware drivers, sets up i2c calls, and starts 'i2c_scan' task.
        </div>
    </div>
</div>

The 'install_handler' funtion that is provided by the i2c slave class allows me to link a function to an opcode. For example, when the 'i2c_scan' task receives a message with the opcode '0x21', it will call the function 'i2c_restart_device' and pass through the message's data contents. The call function can receive the message's contents as a parameter and perform some actions accordingly. Most i2c functions are expected to return a one byte status message over i2c after they have completed their functionality.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/firmware_callfunction.png" title="I2C Call Function" class="img-fluid border border-dark rounded z-depth-1" %}
        <div class="caption">
            An example i2c call function that can start, stop, and return the status of a task.
        </div>
    </div>
</div>

When the above function is called by 'i2c_scan', it checks the parameter to see whether the master wants to start the 'plogger' task, stop the task, or receive whether the task is running or not. If the parameter does not correspond to any of those actions, it returns a one-byte 'invalid' message over i2c to indicate to the master that it did not perform any actiond despite receiving an operation request. If the parameter is '0x01', the function will first check that the task has not been started, then start the task and write a 'valid' byte over i2c indicating that the operation that was requested had been completed. If the task had already been started, the function will wrtie an 'invalid' byte over i2c to indicate that the operation could not be completed. A parameter of '0x02' uses the same dataflow to end the 'plogger' task upon request. Finally, a parameter of '0x03' will simply return a 'valid' or 'invalid' byte depending on if the task is running or not.

Most i2c call functions behave similarly. The parameter value can be used to choose an action to be performed (like the above example), hold a 32-bit data value (such as a time value), or both (using one byte to define an action and the remaining three bytes to hold a value). The functions always return some value back over i2c, usually a confirmation/acknowledge byte or a 32-bit value.

The entire system is built on top of the 'i2c_scan' task and the i2c call functions. The task is configured at 0 priority (idle) and is the only task that runs on core 0 aside from the call functions. 'i2c_scan' is only suspended when a call function is running. All other tasks, including the experiment and logging tasks, run on core 1 to minimize the time between an operation request being sent by the master device over i2c and the confirmation of completion being sent back over i2c to the master.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/firmware_task.png" title="FreeRTOS Task" class="img-fluid border border-dark rounded z-depth-1" %}
        <div class="caption">
            The FreeRTOS task that handles recording system telemetry to the SPI Flash storage,
        </div>
    </div>
</div>

The above function shows one of three FreeRTOS tasks that the system uses. 'exp_log' runs on an interval that is passed through as a parameter when the task is created. On each interval, the task records system telemetry (system time, temperature data derived from thermistors on the payload, and the duty cycle and period of the PWM battery heater signal). Two instances of this task are normally used. One is started on a longer interval to record ambient temperatures whenever the device is not in sleep mode. The other is started with a short interval to record detail temperature telemetry during a payload experiment. The other two tasks are 'exp_run', which performs an experiment sequence by increasing the PWM duty cycle over time, and 'i2c_scan' which checks the i2c receive buffers for operation requests from the master device.

##### How I tested functionality

I used a Raspberry Pi 3 to simulate the OBC as the i2c master. I created a few C++ scripts using the wiringPi library to model all the i2c behavior the the slave should expect from the OBC. Fuctionally, the scripts can package an opcode and parameter, send it over i2c to the device, and receive messages from the device per the i2c communications protocol I developed.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/firmware_getlog.png" title="C++ Driver getlog" class="img-fluid border border-dark rounded z-depth-1" %}
        <div class="caption">
            The FreeRTOS task that handles recording system telemetry to the SPI Flash storage,
        </div>
    </div>
</div>

The above code segment is part of a script that offloads the telemetry log from the device. Not shown in the image, is an i2c sequence to establish a connection with the device which happens prior. In a loop, the script requests the device to send it one line of the experiment log file. Upon receiving the line, it saves it to a local log file. It also checks for erroneous bytes that were often read from the devices' i2c transmit buffers. The device will send a line from the log file in sequential order until the last line. Once the file has been fully read, it will send an 'End of File' message so that the master can stop asking for more lines. The entire process can take nearly 30 seconds to transmit a file of 5000 lines.

##### Next Steps

The project is functionally complete but there is still plenty to do. What I mean is that all the technical requirements that I was given have been satisfied and that the firmware is fully featured and operational. However, it is not yet in a state where it can be uploaded onto a processor and launched into space. The firmware itself needs more system and error checks and with error resolution. Additionally, the Payload Processor's hardware needs to be developed from an ESP32 devkit on a breadboard to a bespoke PCB designed for the PC104 hardware platform.

Personally, I am very satisfied with how I left the project. The entire task of making the ESP32 act as an in-between for the payload's temperature sensors and the main computer was a hard problem to be thrown into. Building each piece of functionality that was required to make it work was a new challenge that I only overcame after many sleepless nights of studying and debugging. I believe that the remaining work can be transferred to next years' team without too much difficulty. Furthermore, I will likely continue to work on the firmware and look into the hardware for my own personal edification.
