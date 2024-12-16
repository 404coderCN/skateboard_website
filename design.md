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

The first thing we did after getting our hands on the skateboard was to test whether the motor was powerful enough. This is important because it needs to sustain enough weight and move at a reasonable speed to be practical. We conducted a full test run up the hill from Gates Hall to the Vet School and back, and tested with two people standing on the board. The skateboard's performance met our expectations, so the next step was to figure out how to control the motor as desired using a microcontroller. Upon inspecting the circuit, we observed that the motor connects via three thick wires to an onboard microcontroller and five thin wires to another port. The three thick wires correspond to the three phases of a brushless motor. A brushless motor works by sequentially energizing its phases to create a rotating magnetic field, which interacts with the permanent magnets in the rotor to produce motion. The other five thin wires appear to originate from the Hall effect motor encoder, which provides motor position and rotation direction data essential for proper motor commutation. Now, here comes the bad news: brushless motors are usaully controlled by an Electronic Speed Control (ESC), which has three outputs corresponding to the three phases, and takes in a PWM signal to detremine the duty cycle at which to run the motor. The onboard circuit embeds an ESC, but it's completely sealed by glue and we don't have any documentation for it. This means we would need our own ESC to control the motor. Considering the budget limit, we made a bold yet naive, judging from the present, decision: to build our own ESC.

![Brushless motor working mechanism](/skateboard_website/assets/images/blog/brushless_motor.gif)

# Building an ESC
Our confidence in building an ESC stems from the fatc that most ESC on the market works in a similar manner: there are three gate drivers to control the six MOSFET

