# Ball Tracking Robot
The Ball Tracking Robot is an autonomous computer vision powered rover engineered to lock onto and dynamically follow a moving ball in real time. There are many different components needed such as: Raspberry Pi, ultrasonic sensors, motors, and more. All of these components work together simultaneously to help it function.


| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Twanshu K. | Irvington HS | Electrical Engineering | Incoming Senior

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  


<!---
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE
-->



<!---
# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 
-->


# First Milestone


<iframe width="560" height="315" src="https://www.youtube.com/embed/KfYuc8oXssA" title="Twanshu K. Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

# Summary
The ultimate objective for this robot is to be able to find the largest area of red pixels seen by the PiCamera with Python code using the OpenCV library. Until the first milestone, I built the basic structure and placement of the components of my robot, set up my Raspberry Pi minicomputer, wired my two motors to the L298N motor driver board and to a power source, and finally wrote some simple lines of code to test the functionality of my DC motors.

# Challenges
The biggest challange I faced was setting up my Raspberry Pi, I resolved it by downloading imager and replcing the current raspberry os with the one installed by imager. Another major challange I faced was the wire management I connected all the wires without putting the breadboard and Raspberry Pi on the car, so I remedied this issue by placing my breadboard on the batteries, Raspberry Pi in the middle and having the ultrasonic sensors in the front.

# Schematics 

<img width="650" height="400" alt="Screenshot 2026-07-06 at 2 28 00 PM" src="https://github.com/user-attachments/assets/db230257-2a9e-4421-b89c-73d26980a624" />

# Code
Basic code to have the motors running

```
#Basic Python Motor Code

import  RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM) 


# The following are the names of the raspberry-pi pins that control each of them
#Left Motor
MOTOR1B = 23
MOTOR1E = 24

#Right Motor
MOTOR2B = 16
MOTOR2E = 26
ena = 25
enb = 12

# Moves the wheels forward
GPIO.setup(MOTOR1B, GPIO.OUT)
GPIO.setup(MOTOR1E, GPIO.OUT)
GPIO.setup(ena, GPIO.OUT)
GPIO.setup(MOTOR2B, GPIO.OUT)
GPIO.setup(MOTOR2E, GPIO.OUT)
GPIO.setup(enb, GPIO.OUT)

pwmA = GPIO.PWM(ena, 100)
pwmB = GPIO.PWM(enb, 100)
pwmA.start(60)
pwmB.start(60)

# Moves the wheels Backwards
GPIO.output(MOTOR1B,GPIO.HIGH)
GPIO.output(MOTOR1E, GPIO.LOW)
GPIO.output(MOTOR2E, GPIO.HIGH)
GPIO.output(MOTOR2B, GPIO.LOW)

time.sleep(3)


GPIO.output(MOTOR1B, GPIO.LOW)
GPIO.output(MOTOR1E, GPIO.LOW)

GPIO.output(MOTOR2B, GPIO.LOW)
GPIO.output(MOTOR2E, GPIO.LOW)

```

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Raspberry Pi 4 Model B | Minicomputer used to write code and control the robot | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Raspberry Pi Camera Module | The camera used to detect object | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| L298N Driver Board | Motor driver board to roll the wheels | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Motors and Board Kit | Basic harware pieces for the robot structure | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Powerbank | Compact and portable power device with USB-C for Raspberry Pi | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| HC-SR04 sensors (3 pcs) | Used to calculate distance of obstacles | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| HDMI to micro HDMI Cable | Used to display Raspberry Pi on monitor | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Basic connections components kit | All the necessary parts for connections such as: breadboard, jumper wires, resistors, and LEDs | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Video Capture card | This card is necessary to display onto laptops only  | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Wireless Mouse and Keyboard | A separate mouse and keyboard is needed to use Raspberry Pi | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Soldering Kit | For motor connections | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |

<!--
# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
-->



# Starter

<iframe width="560" height="315" src="https://www.youtube.com/embed/-c2njXLZAdY?si=5l4_m6QyWvNGTKp1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


The handheld game console has many different games; these games include: Tetris, snake, slot machine, and more. I was able to learn about soldering and how it is used to make conductive joints for the eletricity to pass through. The game console was made up of a circuit board that is connected to a battery pack through wires, with buttons and screen soldered on it. A challanged that I faced is that I accidentally soldered a button diagonally instead of it being straight. Later, I fixed this issue by keeping the screws of the cover loose which would allow me to be able to press the button properly. This was an amazing project to work on and build up the basic skills needed for my Ball Traking Robot project.





