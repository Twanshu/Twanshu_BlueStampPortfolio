# Ball Tracking Robot
The Ball Tracking Robot is an autonomous computer vision powered rover engineered to lock onto and dynamically follow a moving ball in real time. There are many different components needed such as: Raspberry Pi, ultrasonic sensors, motors, and more. All of these components work together simultaneously to help it function.


| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Twanshu K. | Irvington HS | Electrical Engineering | Incoming Senior


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




# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/EMb4H3xo51I?si=cZg5NEOur1T-7WnN" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

# Summary
This milestone was a crucial part of my project, since I got a significant portion of my software done for this milestone. The code has to different shades of red noted in RGB, and any red that the Pi camera sees within that range of colors, it checks for the largest area of red pixels. Then after it identifies and makes a square around the object with the biggest area of red pixels (ball), it also makes a blue point in the middle of the object and gives the coordinates. I also permenently mounted my ultrasonic sensors in the front of my car chassis. 

# Challenges
In milestone 2, along with this massive progress cam a lot of setbacks. My goal for this milestone was to make the Pi camera be able to detect red color, and also a circle. But, everytime I tried to identify the circularity it don't detect the ball. I kept testing with different lengths for circuarity, and it never worked. Later, I resovled this problem by making the camera detect the largest area of red pixels instead of making it also detect a circle.

# Code
This is the code for detecting a the largest area  of red pixels 

```
import cv2
import numpy as np
from picamera2 import Picamera2

def main():
    # Initialize and configure Picamera2
    picam2 = Picamera2()
    config = picam2.create_video_configuration(main={'format': 'RGB888', 'size': (1280, 720)})
    picam2.configure(config)
    picam2.start()

    print("Starting object tracking. Press 'q' in the video window to quit.")

    try:
        while True:
            # 1. Capture frame
            frame = picam2.capture_array()

            # 2. Convert to HSV for tracking
            hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
         
            # 3. Define range of red color in HSV and threshold
            lower_red = np.array([155, 80, 80])
            upper_red = np.array([179, 255, 255])
            mask = cv2.inRange(hsv, lower_red, upper_red)
         
            # 4. Find contours on the mask
            contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
            
            # 5. Process ONLY the largest contour
            if contours:
                # Find the single largest contour in the list by area
                largest_contour = max(contours, key=cv2.contourArea)
                
                # Check if it meets the minimum size threshold
                if cv2.contourArea(largest_contour) > 500: 
                    
                    # Draw the bounding box for ONLY this largest contour
                    x, y, w, h = cv2.boundingRect(largest_contour)
                    cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 3)
                    
                    # Calculate the exact center using Image Moments
                    M = cv2.moments(largest_contour)
                    if M["m00"] != 0: # Prevent division by zero
                        x_cord = int(M["m10"] / M["m00"])
                        y_cord = int(M["m01"] / M["m00"])
                        
                        # Draw a small blue circle at the center of the tracked object
                        cv2.circle(frame, (x_cord, y_cord), 5, (255, 0, 0), -1)
                        print(f"Tracking coordinates: X={x_cord}, Y={y_cord}")

            # 6. Display the processed frame locally
            cv2.imshow('Red Object Tracking', frame)
            
            # 7. Break the loop if the 'q' key is pressed
            if cv2.waitKey(1) & 0xFF == ord('q'):
                break

    finally:
        # Clean up resources properly when exiting
        picam2.stop()
        cv2.destroyAllWindows()
        print("Stream stopped and resources released.")

if __name__ == '__main__':
    main()

```


# First Milestone


<iframe width="560" height="315" src="https://www.youtube.com/embed/KfYuc8oXssA" title="Twanshu K. Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

# Summary
The ultimate objective for this robot is to be able to find the largest area of red pixels seen by the PiCamera with Python code using the OpenCV library. Until the first milestone, I built the basic structure and placement of the components of my robot, set up my Raspberry Pi minicomputer, wired my two motors to the L298N motor driver board and to a power source, and finally wrote some simple lines of code to test the functionality of my DC motors.

# Challenges
The biggest challange I faced was setting up my Raspberry Pi, I resolved it by downloading imager and replcing the current raspberry os with the one installed by imager. Another major challange I faced was the wire management I connected all the wires without putting the breadboard and Raspberry Pi on the car, so I remedied this issue by placing my breadboard on the batteries, Raspberry Pi in the middle and having the ultrasonic sensors in the front.

# Schematics 

<img width="800" height="500" alt="Screenshot 2026-07-13 at 2 14 42 PM" src="https://github.com/user-attachments/assets/dfd737b5-622f-4bc2-9c34-32fbb09ff4ed" />



# Code
Basic code to have the motors running

```
# Basic Python Motor Code
import RPi.GPIO as GPIO
import time

# Use Broadcom (BCM) pin numbering layout (uses GPIO number instead of physical pin number)
GPIO.setmode(GPIO.BCM) 


# PIN CONFIGURATION
# The following are the names of the raspberry-pi pins that control each motor

# Left Motor direction pins
MOTOR1B = 23
MOTOR1E = 24

# Right Motor direction pins
MOTOR2B = 16
MOTOR2E = 26

# Speed Control Pins (Enable pins on the motor driver)
ena = 25  # Enable A - controls Left Motor speed
enb = 12  # Enable B - controls Right Motor speed


# --- GPIO SETUP ---
# Configure all the motor control pins as outputs so we can send signals to them
GPIO.setup(MOTOR1B, GPIO.OUT)
GPIO.setup(MOTOR1E, GPIO.OUT)
GPIO.setup(ena, GPIO.OUT)
GPIO.setup(MOTOR2B, GPIO.OUT)
GPIO.setup(MOTOR2E, GPIO.OUT)
GPIO.setup(enb, GPIO.OUT)


# --- PWM / SPEED SETUP ---
# Set up Pulse Width Modulation (PWM) on the Enable pins at a frequency of 100Hz
pwmA = GPIO.PWM(ena, 100)
pwmB = GPIO.PWM(enb, 100)

# Start the PWM signals at a 60% duty cycle (motors will run at roughly 60% max speed)
pwmA.start(60)
pwmB.start(60)


# --- MOTOR MOVEMENT ---
# Set direction pins to move the motors.
# Note: The original comment said 'forward', but setting 1B/2E HIGH and 1E/2B LOW typical moves them.
# Change these combinations if your specific robot drives backward instead of forward.
GPIO.output(MOTOR1B, GPIO.HIGH)  # Spin Left Motor in direction 1
GPIO.output(MOTOR1E, GPIO.LOW)   

GPIO.output(MOTOR2E, GPIO.HIGH)  # Spin Right Motor in direction 1
GPIO.output(MOTOR2B, GPIO

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





