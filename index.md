# B.O.L.T (Ball - Oriented Localisation Tech)
The Ball Tracking Robot is an autonomous computer vision powered rover engineered to lock onto and dynamically follow a moving ball in real time. I am interested in this to learn about Raspberry Pi and use python to command the different components to work how I desire. I chose to do this project to learn about raspberry pi and python.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Twanshu K. | Irvington HS | Electrical Engineering | Incoming Senior


![Headstone Image](myHeadshot.svg)
  

<p align="center">
  <img width="1053" height="340" alt="Screenshot 2026-07-27 at 4 20 53 PM" src="https://github.com/user-attachments/assets/5f8f40dd-2632-4fb8-91ca-a53f6a85be1c" />




# Third Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/l4Mx1eJBpDw?si=AWDyxiqmuoOV6oTd" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Summary
For my third milestone I have fully coded the robot, and having it working with every component. Since my last milestone, I have had the the motors go forward toward the red ball. I have set the motors to move forward, right, or left depending on where the ball is in the camera frame. I have also made the robot spin to left in one spot in order to find the red ball. The robot is also using the ultrasonic sensors to detect obstacles and manuver them. Whenever there is an obstacle detected it moves backward, and then right.

## Challanges
This milestone came with my biggest challange yet. My raspberry pi got completely reset. I had to redownload the os and I had all of my codes were erased from it. Fortunetly, I had documented my code, so I just had to download the cv2 (camera software) and flask (camera streaming software). After getting to my second milestone, in my full code the ultrasonic sensors didn't let the code run. In order to resolve this issue I rewired all the ultrasonic sensors, and I tested them individually. Then I ran my complete code and it was working.

## Code
This is the full code for the robot with flask stream. 

```python
import cv2
import numpy as np
import threading
import time
from flask import Flask, render_template_string, Response
from picamera2 import Picamera2
import RPi.GPIO as GPIO
from gpiozero import DistanceSensor, Motor

app = Flask(__name__)

# 1. HARDWARE SETUP

GPIO.setmode(GPIO.BOARD)

# Distance Sensors
# Note: gpiozero measures distance in meters. The HC-SR04 max range is ~4m.
ultrasonic_right = DistanceSensor(echo=17, trigger=4, max_distance=10, threshold_distance=0.2)
ultrasonic_front = DistanceSensor(echo=9, trigger=10, max_distance=10, threshold_distance=0.2)
ultrasonic_left = DistanceSensor(echo=22, trigger=27, max_distance=10, threshold_distance=0.2)

# Motors
motor_left = Motor(forward=23, backward=24)
motor_right = Motor(forward=26, backward=16)

# Initialize Picamera2
picam2 = Picamera2()
# Using BGR888 directly so OpenCV can process it natively without color-space mismatches
config = picam2.create_video_configuration(main={'format': 'RGB888', 'size': (1280, 720)})
picam2.configure(config)
picam2.start()

# 2. GLOBAL VARIABLES FOR THREADING

global_frame = None
frame_lock = threading.Lock()

# Minimum contour area to count as a real detection (not noise)
MIN_AREA = 500
MAX_AREA = 250000

# Obstacle avoidance: distance (meters) below which something is "too close"
OBSTACLE_DISTANCE = 0.1

# 3. MOTOR CONTROL FUNCTIONS

def move_forward():
    motor_left.forward(0.7)
    motor_right.forward(0.7)

def stop_move():
    motor_left.stop()
    motor_right.stop()

def move_left():
    motor_left.backward(0.2)
    motor_right.forward(0.4)

def move_right():
    motor_left.forward(0.4)
    motor_right.backward(0.2)

def move_backward():
    motor_left.backward(0.75)
    motor_right.backward(0.75)

def spin_search():
    """Rotate in place (no forward/backward drift) to scan for the ball."""
    motor_left.forward(0.4)
    motor_right.backward(0.4)

def avoid_obstacle():
    """Stop, back up a bit, then turn right, out of the way of an obstacle.

    This is a short blocking sequence (~1 second total). The camera feed
    will pause briefly during this maneuver, which is an acceptable
    trade-off for keeping the avoidance logic simple and reliable.
    """
    stop_move()
    time.sleep(0.1)

    move_backward()
    time.sleep(0.5)
    stop_move()
    time.sleep(0.1)

    move_right()
    time.sleep(0.4)
    stop_move()


# 4. BACKGROUND TASK: CAMERA & ROBOT LOGIC

def control_loop():
    global global_frame

    # Define range of red color in HSV
    lower_red = np.array([155, 80, 80])
    upper_red = np.array([179, 255, 255])

    while True:
        # Capture frame
        frame = picam2.capture_array()

        # Convert to HSV for tracking
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
        mask = cv2.inRange(hsv, lower_red, upper_red)

        # Find contours on the mask
        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        x_cord = 0
        area = 0

        # Process ONLY the largest contour
        if contours:
            largest_contour = max(contours, key=cv2.contourArea)
            area = cv2.contourArea(largest_contour)

            # Minimum size threshold to avoid tracking tiny specs of noise
            if area > MIN_AREA:
                # Draw the bounding box
                x, y, w, h = cv2.boundingRect(largest_contour)
                cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 3)

                # Calculate exact center
                M = cv2.moments(largest_contour)
                if M["m00"] != 0:
                    x_cord = int(M["m10"] / M["m00"])
                    y_cord = int(M["m01"] / M["m00"])

                    # Draw a small blue circle at the center
                    cv2.circle(frame, (x_cord, y_cord), 5, (255, 0, 0), -1)

        # Safely update the global frame so Flask can read it
        with frame_lock:
            global_frame = frame.copy()

        # Obstacle check (highest priority, overrides ball tracking)
        front_dist = ultrasonic_front.distance
        left_dist = ultrasonic_left.distance
        right_dist = ultrasonic_right.distance

        obstacle_detected = (
            front_dist < OBSTACLE_DISTANCE
            or left_dist < OBSTACLE_DISTANCE
            or right_dist < OBSTACLE_DISTANCE
        )

        # --- Robot Movement Logic ---
        # These three branches are mutually exclusive, so exactly one motor
        # command is issued per loop iteration (no command gets silently
        # overwritten by a later one in the same cycle).
        if obstacle_detected:
            # Something is too close: back off and reorient, then resume
            # searching next cycle.
            avoid_obstacle()
        elif MIN_AREA < area < MAX_AREA:
            # Ball locked on: steer/drive toward it.
            if x_cord > 900 or x_cord < 400:
                if x_cord > 840:
                    move_left()
                else:
                    move_right()
            else:
                move_forward()
        else:
            # No ball in view: spin in place. Because this runs every
            # iteration of the loop (every ~20ms) whenever the ball isn't
            # locked on, the robot effectively spins continuously until
            # red is spotted again.
            spin_search()

        # Tiny sleep to prevent this loop from maxing out the Raspberry Pi's CPU
        time.sleep(0.02)

# ==========================================
# 5. FLASK WEB SERVER ROUTES
# ==========================================
def generate_frames():
    global global_frame
    while True:
        # Safely grab the latest frame from the background thread
        with frame_lock:
            if global_frame is None:
                time.sleep(0.05)
                continue
            current_frame = global_frame.copy()

        # Encode the frame as JPEG
        success, buffer = cv2.imencode('.jpg', current_frame)
        if not success:
            continue

        # Yield frame in MJPEG format
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + buffer.tobytes() + b'\r\n')

        # Limit the stream framerate to save Wi-Fi bandwidth
        time.sleep(0.05)

@app.route('/')
def index():
    return render_template_string('''
        <html>
          <head>
            <title>Raspberry Pi Live Stream</title>
          </head>
          <body>
            <h1>Raspberry Pi Camera Live Feed</h1>
            <img src="{{ url_for('video_feed') }}" width="640" height="480">
          </body>
        </html>
    ''')

@app.route('/video_feed')
def video_feed():
    return Response(generate_frames(), mimetype='multipart/x-mixed-replace; boundary=frame')


# 6. MAIN EXECUTION

if __name__ == '__main__':
    try:
        # Start the camera/motor control loop as a background thread
        logic_thread = threading.Thread(target=control_loop, daemon=True)
        logic_thread.start()

        # Start the Flask server on the main thread
        app.run(host='0.0.0.0', port=5000, threaded=True)
    finally:
        stop_move()
        picam2.stop()
        print("\nHardware modules cleanly disconnected.")

```




# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/EMb4H3xo51I?si=cZg5NEOur1T-7WnN" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


<img width="703" height="609" alt="Screenshot 2026-07-21 at 4 07 31 PM" src="https://github.com/user-attachments/assets/99b554c9-f1a0-4357-a311-09622dd3513d" />


## Summary
This milestone was a crucial part of my project, since I got a significant portion of my software done for this milestone. The code has to different shades of red noted in RGB, and any red that the Pi camera sees within that range of colors, it checks for the largest area of red pixels. Then after it identifies and makes a square around the object with the biggest area of red pixels (ball), it also makes a blue point in the middle of the object and gives the coordinates. I also permenently mounted my ultrasonic sensors in the front of my car chassis. 

## Challenges
In milestone 2, along with this massive progress cam a lot of setbacks. My goal for this milestone was to make the Pi camera be able to detect red color, and also a circle. But, everytime I tried to identify the circularity it don't detect the ball. I kept testing with different lengths for circuarity, and it never worked. Later, I resovled this problem by making the camera detect the largest area of red pixels instead of making it also detect a circle.

## Code
This is the code for detecting a the largest area  of red pixels 

```python
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

## Summary
The ultimate objective for this robot is to be able to find the largest area of red pixels seen by the PiCamera with Python code using the OpenCV library. Until the first milestone, I built the basic structure and placement of the components of my robot, set up my Raspberry Pi minicomputer, wired my two motors to the L298N motor driver board and to a power source, and finally wrote some simple lines of code to test the functionality of my DC motors.

## Challenges
The biggest challange I faced was setting up my Raspberry Pi, I resolved it by downloading imager and replcing the current raspberry os with the one installed by imager. Another major challange I faced was the wire management I connected all the wires without putting the breadboard and Raspberry Pi on the car, so I remedied this issue by placing my breadboard on the batteries, Raspberry Pi in the middle and having the ultrasonic sensors in the front.

# Schematics 

<img width="850" height="520" alt="Screenshot 2026-07-13 at 2 14 42 PM" src="https://github.com/user-attachments/assets/dfd737b5-622f-4bc2-9c34-32fbb09ff4ed" />



<img width="850" height="520" alt="Screenshot 2026-07-23 at 2 17 40 PM" src="https://github.com/user-attachments/assets/0e069bcb-699d-47f9-b793-e1196dc6f1b8" />



## Code
Basic code to have the motors running

```python
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
| CanaKit Raspberry Pi 4 4GB Starter PRO Kit - 4GB RAM | Minicomputer used to write code and control the robot and cables for connection | $149.98 | <a href="https://www.amazon.com/CanaKit-Raspberry-4GB-Starter-Kit/dp/B07V5JTMV9/ref=sr_1_3?crid=31PI9EIW8CCWH&dib=eyJ2IjoiMSJ9.6RZammJY5JsyJpwezt2mn8Hqd28JgVwq_KHhbgkkY4IrbciDs0ZQeIg3QSP8QcDBHnvlqopIjgdMxGnKPns4ApM3z8QreEye5MSZwkqWPhyYIWVlqKFS8hN0jO1ENYcKoHigR8EkPlid5EiUbCD1x4QUFi1G1lRUrjjEBnqw7NxIJQcE_0M3wLJgwqbLpvTNRZTRonrOJx-AfVz0vv-nvJ3TdrD29g6t3w7ibXHCygc.dK6Odzl1K8g7DL1F4lAd5GQmWTymPgk7itiraMfSn-8&dib_tag=se&keywords=raspberry%2Bpi%2B4&qid=1784751448&sprefix=rasp%2Caps%2C236&sr=8-3&th=1"> Link </a> |
| Raspberry Pi Camera Module | The camera used to detect object | $9.99 | <a href="https://www.amazon.com/Arducam-Raspberry-Camera-Module-1080P/dp/B07RWCGX5K/ref=sr_1_9?crid=39T3Q8B81QHID&dib=eyJ2IjoiMSJ9.yC5LE0RHI618cjkfC-IY2dZ4Fgg2dUoaMR5VLELavDTcishCoSHxaZ4-vVROpfmfR2W-Dz6Y9F1UyXqxug0vqe-d3n1lUEM44RB3F6NQrFmutvYqTjXxZ2oaFITfBRMjZR0N7CfCLXnPOuVfWHsT6p3A_I6WagLFXwrQjJfbbT3FjNtaiSGyrrAO_Ufrbg4ma2-b4K5Wp2A1i_XaBFqta_XLiqJzDhvOri5QBTfgM0Q.o7acKcYFSNJNM2G1A34lXMzTokkp1z-Imsl2GOh6Jqo&dib_tag=se&keywords=raspberry%2Bpi%2Bcamera%2Bmodule&qid=1784751654&sprefix=raspberry%2Bpi%2Bcamera%2Caps%2C481&sr=8-9&th=1)"> Link </a> |
| L298N Driver Board | Motor driver board to roll the wheels | $6.99 | <a href="https://www.amazon.com/Qunqi-Controller-Module-Stepper-Arduino/dp/B014KMHSW6/ref=sr_1_6?crid=2PQSAZ6Z83CB9&dib=eyJ2IjoiMSJ9.hK2FjV8Ukp8CCyVTI1seMk4n3aguoO_lNXX3xoiH-O2FX3IB9FmL1nLFhsJ12bhRz20gFS2Ql4DDjn9H5YtvX3EodunP9RQFN1c-4MqDdR02UIkYJTS26phq3PKrK_ISWcZ2gp_yehbcEDB96EI0Ke6a7sS9UMqVDtLIrjQCWkBmgEno20G9y_-Y3X9DOCcgga8fSQg9mvQm0PdQYFtqHsqmP94tDOXIpt_koZgwaMU.jb12o1Av-wYupfatbl8mnaDtCn1hV9o0Ze_fhPnbSeE&dib_tag=se&keywords=l298n+motor+driver&qid=1784751959&sprefix=l298n%2Caps%2C296&sr=8-6"> Link </a> |
| Motors and Board Kit | Basic harware pieces for the robot structure | $13.99 | <a href="https://www.amazon.com/Smart-Chassis-Motors-Encoder-Battery/dp/B01LXY7CM3/ref=sr_1_5?crid=2WAPWRMPHBP0G&dib=eyJ2IjoiMSJ9.Dvn17bIjmxTMwWIxV7i6ch3TNGBoJYNfCC-Wf3CU4zd25swKfo9Pfj7dqPCOoEEsB0jloOspJi_DJGglDL8W8hG7GDJ0UGFobzh98H6OqkAJahWrV8j6EmWH4vSYvxJm5NnXx4WBsaG42KofgIlbiKwp8ulZIHzuMp6mEQ-H9qffjmApb2Wjg75-x7bVQezCwuQjSF91Q03hXHqhTz9S5uMX1htCWhjtXWkIYaYCr9LPFUa6FSTo6RWO05aPcyOKM39tsIFcO1VkAiIwO6caM8kDE0W7PSEKr_YwrMQHlcw.ljQEIsniDveORJTK5Zhlk-Zg-bFo82qe7MT2-ylIFgs&dib_tag=se&keywords=chassis+car+kit&qid=1784752024&sprefix=chasis+car+%2Caps%2C306&sr=8-5"> Link </a> |
| Powerbank (10,000 mAh) | Compact and portable power device with USB-C for Raspberry Pi | $15.99 | <a href="https://www.amazon.com/10000mAh-Portable-Essentials-Powerbank-Compatible/dp/B0FVXM2W9P/ref=sr_1_1?crid=1QY23LJPDY5FB&dib=eyJ2IjoiMSJ9.z9C6PukW8_a12iRHaJofx7bd3ys2IEx__Ra1jpMot0CbcQ0LpfJJCqbpnBc8MCZ6lSNURlvfwDO0dk2rdWVZ3_bPn7-y2WUI9z404TEYbfXz7yMYQq7iWm1aI6lwBLzygwcwpa2jqTttj-RejJkIkxRatSh_LNGRzXjxTsD-xHCM8DPnHGNjm_szRv_2Prj7W3oKi_ikJm3EtiSWGSTUnqyhz8hM9JTX1LS7k_ajXOw.jvOLee4yJMM5H1CrE6JF3Q4JKtbDzxYj_avWTkciwGE&dib_tag=se&keywords=powerbank%2Bn001%2Bblack&qid=1784752151&sprefix=powerbank%2Bn001%2Bbla%2Caps%2C184&sr=8-1&th=1"> Link </a> |
| HC-SR04 sensors (3 pcs) | Used to calculate distance of obstacles | $6.99 | <a href="https://www.amazon.com/Ferwooh-Ultrasonic-Distance-Measuring-Mounting/dp/B0D1MDP9V3/ref=sr_1_10?crid=2LUYG19N0HEOW&dib=eyJ2IjoiMSJ9.w-v74CMMP9eRh1BFF5BJ6-T10O8UBzaKAmKgKchgfBBC6a19YGwGvPn8D6_w7bG_yobe42uOSHG1CmSAOY5XF3SmJ9RZ5lab8hYE5ESM_X3LvC2Liz1fPTAZGR1a_Li0tKHhxzqotY0WzKE0GZ_uI_tZ1O5_mCJTAmoxZp7A_DCqdF2wb3WB4N6xHPQ7L0XyvgdoS4YrvtFcpjkMjRQT0e5u_x1liUIoQdyr5dlX35c.zyuUIojQnrknntgdcJDXqe6eBLFBkSBdFVUZqPRgRQ0&dib_tag=se&keywords=hc+sr04+ultrasonic+sensor&qid=1784752229&sprefix=hc+sr%2Caps%2C221&sr=8-10"> Link </a> |
| Basic connections components kit | All the necessary parts for connections such as: breadboard, jumper wires, resistors, and LEDs | $9.99 | <a href="https://www.amazon.com/EL-CK-002-Electronic-Breadboard-Capacitor-Potentiometer/dp/B01ERP6WL4/ref=sr_1_6?crid=FNH6BWJTZZG2&dib=eyJ2IjoiMSJ9.VNel8a9YKAdQOttMORwrLFKm7BcwrGQdUiF0yQPfuOCjJwbjvjMdRlLcqX5kskxTajWkOkv3H7NJXhGZRNPG_CSjhwnoD59MxD8ZKwTD9N4ljo4WEWOLb_4g2gqdnHIkq0gDwx_vY7-R_RDcPHFiPLnkTXyFaKOV16b9eVLB1VhzN3tUWiGltmreM2RjNwBrMdGdZ54s8vr2jG238TplxGjokHValC1trzTBjk-XG14.DZAb52FIdiuFwaOO362EXPxRrSxewTInrP3cXTwaUKc&dib_tag=se&keywords=jumper+wires%2C+breadboard%2C+resistors+kit+without+raspberry+pi&nsdOptOutParam=true&qid=1784752384&sprefix=jumper+wires%2C+breadboard%2C+resistors+kit+without+raspberry+p%2Caps%2C182&sr=8-6"> Link </a> |
| Wireless Mouse and Keyboard | A separate mouse and keyboard is needed to use Raspberry Pi | $19.99 | <a href="https://www.amazon.com/Wireless-Keyboard-Ergonomic-Lag-Free-Cordless/dp/B0DLBD36HL/ref=sr_1_21?crid=3QRKZZRB2AU5R&dib=eyJ2IjoiMSJ9.mEAzOpyD6BFiOKTUz5iqTb9gS2cWqELO2nBZdm7WUIxEJwz9rc3UxHFI7YDqIue68_zLU9dMHG_3uY7QSwVgf2KrVSDQNSxlceqkCeCOB91GhzCV2rXuHJHdM2wo159vSqdxPAZH_trGVy63qO6ew-O8ACOsMyweuPxQ0qZMtjTmzM6R6Ip9ZPvZyhgN-PwnRE5nmQ6uZ3GBiYZ0T3z8w1FUPZNoiG2kC9bgzZIWdvE.BdtBxOz0anSv6wiaJEBLiBQeIk1ECxoGQWa7PHGgovs&dib_tag=se&keywords=wireless%2Bkeyboard%2Band%2Bmouse&qid=1784752469&sprefix=wireless%2Caps%2C244&sr=8-21&th=1"> Link </a> |



# Starter

<iframe width="560" height="315" src="https://www.youtube.com/embed/-c2njXLZAdY?si=5l4_m6QyWvNGTKp1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


The handheld game console has many different games; these games include: Tetris, snake, slot machine, and more. I was able to learn about soldering and how it is used to make conductive joints for the eletricity to pass through. The game console was made up of a circuit board that is connected to a battery pack through wires, with buttons and screen soldered on it. A challanged that I faced is that I accidentally soldered a button diagonally instead of it being straight. Later, I fixed this issue by keeping the screws of the cover loose which would allow me to be able to press the button properly. This was an amazing project to work on and build up the basic skills needed for my Ball Traking Robot project.


## Other Resources/Examples
https://samvit-kini.github.io/Samvit_BSE_Portfolio/
https://deringur.github.io/BSE_Derin_Portfolio/#schematics



