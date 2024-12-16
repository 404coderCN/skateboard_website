---
layout: post
category : docs
# tagline: "Example helium post in markdown and html classes"
tags : [jekyll, code, markdown]
# img : markdown-samples.jpg
# img2 : 
# img3 : 
# author : Luoyan & Jerry
# title2 : 
# title3 : 
# css: 
# js: 
# bgcolor: 
# keywords: html, css, markdown, jekyll, docs, jekyllthemes, theme
# canonical: https://fullit.github.io

---
{% include JB/setup %}

# Project details

## Why skateboard
Have you ever worried about being late to class or felt frustrated by the long, tiring walk from home to campus? As Cornell students, we’ve all been there. These moments of frustration sparked an idea: What if there were a way to get to our destination effortlessly, without the need for walking? We began exploring lightweight, practical transportation options commonly seen around campus and quickly found an intriguing candidate: the electric skateboard. Unlike bicycles, which are bulky and harder to store, an electric skateboard is compact, portable, and easy to carry around. It’s fast, efficient, and undeniably stylish—a transportation option that stands out and turns heads. With an electric skateboard, students can not only zip across campus to their classes with ease but also take in the breathtaking scenery of Ithaca, making the commute an experience in itself. This project aims to make that vision a reality, combining engineering creativity with practical solutions to enhance everyday student life.

## Let's start
With a project in mind, we began researching the components typically used in electric skateboards, but we quickly encountered our first challenge: budget. Most skateboard motors cost over $100, and since our project required all components—excluding the Raspberry Pi 4—to stay within a $100 budget, buying a new motor was simply not an option. After scouring the internet for alternatives, we landed on a practical solution: purchasing a second-hand skateboard and redesigning it. Fortunately, we found a seller on Facebook Marketplace located just two hours from Ithaca. With the skateboard in hand, we finally had the foundation we needed to bring our project to life.

The first thing we did after getting our hands on the skateboard was to test whether the motor was powerful enough. This is important because it needs to sustain enough weight and move at a reasonable speed to be practical. We conducted a full test run up the hill from Gates Hall to the Vet School and back, and tested with two people standing on the board. The skateboard's performance met our expectations, so the next step was to figure out how to control the motor as desired using a microcontroller. Upon inspecting the circuit, we observed that the motor connects via three thick wires to an onboard microcontroller and five thin wires to another port. The three thick wires correspond to the three phases of a brushless motor. A brushless motor works by sequentially energizing its phases to create a rotating magnetic field, which interacts with the permanent magnets in the rotor to produce motion. The other five thin wires appear to originate from the Hall effect motor encoder, which provides motor position and rotation direction data essential for proper motor commutation. Now, here comes the bad news: brushless motors are usually controlled by an Electronic Speed Control (ESC), which has three outputs corresponding to the three phases and takes in a PWM signal to determine the duty cycle at which to run the motor. The onboard circuit includes an embedded ESC, but it is completely sealed with glue, and there is no documentation available. This means we would need our own ESC to control the motor. 

We initially tested the motor using a small ESC designed for RC planes. To achieve greater precision, we opted for the hardware PWM library instead of the conventional software PWM provided by the RPi.GPIO library. After navigating through a series of tedious calibration steps, we finally managed to control the motor speed by sending 50 Hz PWM signals with duty cycles ranging between 1 ms and 2 ms. However, this ESC utilized an older digital protocol that required recalibration every time motor power was interrupted. Additionally, it couldn't handle the input voltage from the large 25.2V, 5.2Ah battery on the skateboard. Considering our budget constraints, we made a bold—albeit in hindsight, naive—decision: to build our own ESC.

![Brushless motor working mechanism](/skateboard_website/assets/images/blog/brushless_motor.gif) {width=300px}

## Building an ESC
Our confidence in building an ESC stems from the fact that most ESCs on the market operate in a similar manner. The circuit typically uses three gate drivers to control six MOSFETs, arranged in a three-phase bridge configuration. The gate drivers ensure precise switching of the MOSFETs at the correct time to energize the motor phases. Capacitors are added across the power supply lines to smooth out voltage fluctuations caused by rapid switching, protecting the components from voltage spikes and ensuring stable operation. Resistors are included to limit inrush current to the MOSFET gates, prevent ringing, and fine-tune the switching speed to minimize heat and energy loss. A microcontroller would be used to send out three PWM signals to control the siwtching of the hree gate drivers. A typical ESC circuit is show below.

<img src="{{ BASE_PATH }}/assets/images/blog/esc_circuit.png" alt="ESC circuit" width= "450" height="320">

We chose the Arduino Nano as the microcontroller for the ESC due to its compact size and ease of use. The gate driver and MOSFETs are soldered onto a protoboard following the schematic below. The three inputs to the gate drivers are connected to digital GPIO pins on the Nano, while the three outputs are linked to the motor's three-phase wires.

<img src="{{ BASE_PATH }}/assets/images/blog/gate_driver.png" alt="gate driver connection" width= "450" height="320">

Here are some pictures showing our soldered ESC.
<div style="text-align: center;">
  <img src="{{ BASE_PATH }}/assets/images/blog/esc_front.JPG" alt="ESC front" style="display: inline-block; width: 45%; margin-right: 5%;">
  <img src="{{ BASE_PATH }}/assets/images/blog/esc_back.JPG" alt="ESC back" style="display: inline-block; width: 45%;">
</div>

On the Nano, we defined nine variables representing the pulse start and end times for each phase, along with their active states. Inside a loop, we continuously check if the start time for each phase has been reached. When a true condition is met, we set the corresponding pin output to HIGH and update the next pulse start time. Similarly, when the end time is reached, we set the pin output to LOW and increment the pulse end time. Importantly, a pulse only starts if its current state is NOT active and only ends if it is active. We set the PWM period to 0.075 seconds and the duty cycle to one-third. A crucial factor in configuring the three PWM signals is the amount of overlap between them, which can vary depending on the brushless motor type. Common choices for overlap are 1/3, 2/3, and 1/2. We experimented with all three settings, but the motor only vibrated and appeared to get stuck as it received the signals. To better diagnose the problem, we used an oscilloscope to inspect the ESC’s output and verify that the desired signals were being sent. We also measured the current flowing through the motor's input wires to determine if stall current was a contributing factor. After these checks, we scoped the three-phase input directly on the motor side, where we noticed significant noise and the waveform became distorted from its original shape.

After several rounds of testing, we unfortunately burned one of the gate drivers. Without a replacement on hand, we had to reevaluate whether building our own ESC was a viable option. After some discussion, we decided it would be more practical to purchase an off-the-shelf ESC, given our time constraints. We selected a bulkier ESC with an input voltage rating of 25.5V, and we proceeded to restart our project from that point.

## Restarting

With the newly-arrived ESC, we were quickly able to start running our motor. The subsequent development of our project could be separated into three different parts based on functionality: (1)Bluetooth communication between controller and skatebaord, (2) Obstacle detcection, and (3) Smart power-on. A high-level overview of our system is presented below.

### Bluetooth communication
The first thing our skateboard must be able to do is to receive speed information from the controller and use it to adjust the PWM signal to the motor in order to run it at different speeds. As we have decided to use an ESP32 to control the remote controller and mount the Raspberry Pi on the skateboard, we need to find a wireless communication method in common between these two microcontrollers. After evaluating connection distance and network reuqirement, we landed on using Bluetooth. We started by setting the ESP32 as the master device and made use of the BluetoothSerial library. We start the bluetooth service on ESP32 by assigning it a recognizable name and have it continuously check the surrounding network for any available clients. On the Raspberry Pi side, we imported the bluetooth library and scan the surrounding to look for device with the name matching our ESP32. When a match is detected, the raspberry pi will connect with the device on RFCOMM channel 1. 

With the bluetooth connection established, we need to consider in what form should we send and interpret the data. Our remote contorller was designed to incorporate a joystick, allowing flexible speed control, and three buttons for three set speeds: brake, half speed, and full speed. Whenever a user moves the joystick up the y-axis, increasing its Y_out vlaue, we want the skateboard to go faster, menaing we have to increase the duty cycle. And we want the skateboard to slow down as the joystick returns to its original position. Additinally, pressing the buttons on the controller should set and maintain the skateboard at the corrsponding speed. Since the joystick requires continuous monitoring of the Y_out pin, and its analog value changes frquently as the joystick moves, we decided to check the input pin by polling and map it to some speed base on differet thresholds. As the buttons are less likely to get frequently pressed, we used interrupts to capture the state of each of the buttons, and assign each interrupt with their corresponding set speed. By connecting one side of each button to a GPIO pin and setting the pull-up resistors in the microcontroller, we can simply monitor falling edge to trigger the respective interrupt service. With this setup, the controller continuously send out these speed values as they get updated by either joystick movements or button press. 

On the Raspberry pi side, we take in 800 bytes of data from the Bluetooth channel at once and decode it in "utf-8" format. Because the speed setting on the ESP32 side weas configure to be from 5 to 10, we can directly set the received value as the new PWM duty cycle, as a 50Hz PWM signal should have duty cycle between 5 to 10 if we want the HIGH part of the signal to be within 1 ms to 2 ms. 