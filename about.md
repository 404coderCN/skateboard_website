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
keywords: html, css, markdown, jekyll, docs, jekyllthemes, theme
# canonical: https://fullit.github.io

---
{% include JB/setup %}

<!--more-->

# About

This is the final project for the Embedded OS class at Cornell University. Our project involves building a fully functional electric skateboard capable of speed control, obstacle detection, and smart power on mechanism through tactile sensors. The skateboard is driven by a powerful brushless motor controlled by an ESC. An onboard Raspberry Pi receives speed control messages via Bluetooth from a remote controller to adjust the supply PWM signal to the ESC. A separate circuit on the board monitors the change in voltage as a result of external pressure on the tactile sensor. As the voltage passes a certain threshold, an Arduino nano switches on a relay to power on the brushless motor. Additionally, an ultrasonic sensor is mounted at the front to detect and give out obstacle warnings. 

<div style="text-align: center;">
  <img src="{{ BASE_PATH }}/assets/images/blog/skateboard_top.JPG" alt="Image 1" style="display: inline-block; width: 45%; margin-right: 5%;">
  <img src="{{ BASE_PATH }}/assets/images/blog/zoom_in.JPG" alt="Image 2" style="display: inline-block; width: 45%;">
</div>


Our remote controller has a simple yet elegant design. A small LCD screen shows the current speed of the skateboard from 0 to 5, where 0 is stop and 5 is full speed. We also included a warm message to remind the users to "Stay Slim". A joystick is included to allow flexible speed control, and an additional three buttons on the left allow for setting cruise speed. We also included a button to turn on buzzer warning for obstacle detection within 1m of the skateboard. 



<div style="text-align: center;">
  <img src="{{ BASE_PATH }}/assets/images/blog/top_view.JPG" alt="Image 1" style="display: inline-block; width: 30%; margin-right: 3%;">
  <img src="{{ BASE_PATH }}/assets/images/blog/front_view.JPG" alt="Image 2" style="display: inline-block; width: 30%; margin-right: 3%;">
  <img src="{{ BASE_PATH }}/assets/images/blog/backview.JPG" alt="Image 2" style="display: inline-block; width: 30%;">
</div>


Just hop on and start your ride!

