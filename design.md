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

![Brushless motor working mechanism](/skateboard_website/assets/images/blog/brushless_motor.gif)

## Building an ESC
Our confidence in building an ESC stems from the fact that most ESCs on the market operate in a similar manner. The circuit typically uses three gate drivers to control six MOSFETs, arranged in a three-phase bridge configuration. The gate drivers ensure precise switching of the MOSFETs at the correct time to energize the motor phases. Capacitors are added across the power supply lines to smooth out voltage fluctuations caused by rapid switching, protecting the components from voltage spikes and ensuring stable operation. Resistors are included to limit inrush current to the MOSFET gates, prevent ringing, and fine-tune the switching speed to minimize heat and energy loss. A microcontroller would be used to send out three PWM signals to control the siwtching of the hree gate drivers. A typical ESC circuit is show below.

<img src="{{ BASE_PATH }}/assets/images/blog/esc_circuit.png" alt="ESC circuit" width= "450" height="380">

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

With the newly-arrived ESC, we were quickly able to start running our motor. The subsequent development of our project could be separated into three different parts based on functionality: (1)Bluetooth communication between controller and skatebaord, (2) Obstacle detcection, and (3) Smart power-on. 

### Bluetooth communication
The first objective for our skateboard was to receive speed information from the controller and adjust the PWM signal to the motor to control its speed. Since we decided to use an ESP32 as the remote controller and a Raspberry Pi mounted on the skateboard, we needed a reliable wireless communication method compatible with both microcontrollers. After evaluating connection distance and network requirements, we chose Bluetooth.

We began by configuring the ESP32 as the master device, utilizing the BluetoothSerial library. The ESP32 starts the Bluetooth service by assigning it a recognizable name and continuously scans for available clients in the vicinity. On the Raspberry Pi side, we imported the Bluetooth library and scanned for devices matching the ESP32’s name. Once a match was found, the Raspberry Pi connected to the ESP32 over RFCOMM channel 1.

With the Bluetooth connection established, we needed to decide how to send and interpret data. Our remote controller includes a joystick for continuous speed control and three buttons for preset speeds: brake, half speed, and full speed. When the joystick moves upward along the Y-axis, increasing the Y_out value, the skateboard should accelerate, meaning we must increase the PWM duty cycle. Conversely, when the joystick returns to its neutral position, the skateboard should decelerate. The buttons are used to set and maintain specific speeds.

Since the joystick provides continuous analog readings that change frequently, we decided to poll the input pin and map its value to speeds based on predefined thresholds. For the buttons, we employed interrupts, as button presses are less frequent. Each button was connected to a GPIO pin with pull-up resistors, and we monitored falling edges to trigger the corresponding interrupt service routines. With this setup, the controller continuously sends updated speed values based on joystick movements and button presses.

On the Raspberry Pi side, we received 800 bytes of data from the Bluetooth channel at once and decoded it in UTF-8 format. Since the speed setting on the ESP32 ranges from 5 to 10, we directly mapped the received value to the PWM duty cycle. For a 50Hz PWM signal, a duty cycle range of 5% to 10% ensures a HIGH duration between 1 ms to 2 ms.

We tested the setup by running Bluetooth scripts on both the ESP32 and Raspberry Pi simultaneously. The results were successful: we successfully controlled the motor, demonstrating a working Bluetooth communication system between the two microcontrollers.

<!-- <div style="display: flex; justify-content: space-between;">
  <video width="480" height="360" controls>
    <source src="{{ BASE_PATH }}/assets/images/blog/bluetooth_skateboard.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>

  <video width="480" height="360" controls>
    <source src="{{ BASE_PATH }}/assets/images/blog/bluetooth_controller.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div> -->

### Obstacle detcection

One of the standout features of our skateboard is its ability to detect obstacles and provide warnings. This is particularly useful for outdoor rides, as unexpected objects can appear in the path, and early warnings enhance user safety. To achieve this, we mounted an HC-SR04 ultrasonic sensor at the front of the skateboard. The sensor continuously emits ultrasonic pulses to detect obstacles ahead. The receiver waits for the echo to bounce back, and by measuring the time it takes for the pulse to travel to and from the obstacle, the sensor calculates the distance.

We implemented two warning mechanisms: a visual alert on the remote controller and an audible alarm via a buzzer mounted on the skateboard.

To handle communication and ensure smooth operation, we utilized two threads on the Raspberry Pi. A communication thread handles receiving speed data from the controller and transmitting distance information back to it. To improve accuracy, we implemented a function to clean the distance data by filtering outliers and averaging over 10 samples. A distance calculation thread continuously collects samples, cleans the data, and updates the processed results. This design ensures that data processing does not interfere with real-time communication, preventing any delays. The remote controller displays a "DANGER" message on its LCD screen if an obstacle is detected within 80 cm of the skateboard.

Additionally, we added a buzzer to provide audible warnings for nearby obstacles. One side of the buzzer is connected to ground, while the other is connected to a GPIO pin. The distance calculation thread triggers the buzzer if the average distance of obstacles over 10 samples is less than 1 meter. A PWM signal is sent to the buzzer, with the frequency inversely proportional to the distance. During testing, we observed that closer obstacles produced higher-frequency buzzing, while the buzzing slowed as the distance increased. To make the feature optional, we added a button on the controller to enable or disable the buzzer, allowing users to avoid constant noise in public spaces.



  <!-- <video width="480" height="360" controls>
    <source src="{{ BASE_PATH }}/assets/images/blog/buzzer.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video> -->

### Smart power on

To make our skateboard a bit fancier, we added an automatic power-on feature that activates when someone stands on it. This is achieved using two custom tactile sensors made from Adafruit’s tactile sensing sheets—one for each foot. We connected these in series to detect the combined weight from both feet. The principle is simple: when pressure is applied, the resistance of the sensing sheet changes. We used copper tape as electrodes and integrated the sensing sheets into a basic voltage divider circuit. By measuring the circuit’s output voltage, we can track resistance changes and know when someone is on the skateboard. An Arduino Nano is used to read the output voltage and, if the value crosses a preset threshold, it sends a signal to a relay that powers up the motor. We also thought it’d be smart (actually amusing) to add a weight indicator system for safety.  Two LEDs are mounted on the skateboard: one red and one green. When you step on the skateboard, the power turns on through the relay. If your weight is within the weight range, the green LED lights up, indicating you are safe to go. But if you’re over the limit, the red LED lights up—a gentle reminder that the skateboard has its limits. While demonstrating the function, someone joked that even our skateboard is judging us. But that’s not true! We intentionally set the “overweight” threshold so low that almost everybody will trigger the red LED. This turns the feature into more of a playful gag than a serious warning. Despite its dramatic flair, the skateboard can actually handle up to 70 kg just fine, as long as you give it a little push to get started. Additionally, we also implemented a physical button on the skateboard so that no matter if there is someone standing on the skateboard, you can always turn on it manually.

  <!-- <video width="480" height="360" controls>
    <source src="{{ BASE_PATH }}/assets/images/blog/tactile.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video> -->

## Putting everything together

With all desired features implemented, we began testing the complete system and assembled everything in place. To make the skateboard fully untethered, we mounted a power bank beneath the Raspberry Pi to keep it powered. We then wrote a bash script to enable Bluetooth, run the necessary communication scripts, and set it to execute automatically at startup using cron. During testing, we discovered that the Bluetooth connection wouldn't establish at startup because the Bluetooth module required extra time to initialize. To address this, we added a delay by putting the processor to sleep for a short period before running the script. However, the connection remained somewhat unstable, occasionally requiring us to manually intervene by SSHing into the Pi and restarting the script.

Although not perfect, our skateboard was finally operational, and we conducted extensive testing ourselves. The remote control worked smoothly, and the speed was quite impressive. However, we noticed a significant drawback: the braking was too abrupt, which could throw the user off balance. Unfortunately, due to the limited speed options in our design, there was little we could do to fine-tune the braking performance.

Now, sit back, relax, and watch as our skateboard roars down the hallway!

  <!-- <video width="480" height="360" controls>
    <source src="{{ BASE_PATH }}/assets/images/blog/ride.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video> -->
