# SPLAT-CIWS
Simple Projectile Locating and Aiming Tower (SPLAT) - a Close in Weapons System (CIWS) for Short Range Missile Defense by Vikram Bala and Andy Liu. ESE350 Spring 2022 Final Project.

Overview: Our project is intended to be a mock example of the close in missile defense systems used in the real world, such as the PHALANX CIWS. We took a computer vision based approach to projectile tracking, and created utilities such as an LCD radar display and remote website command to monitor information collected about the projectile and control the turret's defense weapon (a laser in this case). Our goals included minimizing latency in tracking the object, writing comprehensive libraries and peripherials that could be used with various parameters and use cases, and ultimately creating a working demo within the time constraints of just a few weeks.

# How to Run Code



# Media

**S.P.L.A.T CIWS Full Demo Video**
https://youtu.be/s5mFUPCWUQY

**S.P.L.A.T CIWS Turret In Action** *(Short Video)*
https://youtu.be/M7niDRb0H9w

*The Turret System, Cameras, and Radar Operator LCD*
<img width="1427" alt="The Turret System with Cameras and Radar LCD" src="https://user-images.githubusercontent.com/56012430/165188330-ddabab07-831e-414f-8a18-d6097b6f1c5d.png">

*The Remote Command Website used to control Turrert Laser and Monitor Real Time Position of Turret*
<img width="1439" alt="The Remote Command Website for the Turret" src="https://user-images.githubusercontent.com/56012430/165188180-9a2c7414-455c-43a6-b066-fe21175eeee3.png">

# Components
**Servo Turret System (Relevant Files: src/Custom_Servo.c, include/Custom_Servo.h, src/main.c)**

We wrote a servo motor control library for the SG90 servo motor; the datasheet can be accesed [here](http://www.ee.ic.ac.uk/pcheung/teaching/DE1_EE/stores/sg90_datasheet.pdf). Note that this library was limited to the use of one 8-bit and one 16-bit timer on the Atmega328P microcontroller (see header file for the specific output PWM pins for the servos). This meant that both servos were not able to operate with the same level of precision in movements. Using this library, in main.c the system clock was prescaled by 1/2 to bring the frequency to 8MHz; however the library can operate at any CPU frequency so long CPU_FREQ and PRESCALE macros in it's header are changed. 
  

**UART Serial Rx/Tx Library (src/uart.c, incluce/uart.h)**

We wrote a UART serial communication library for the Atmega328P to both receive and transmit data packets in between other microcontrollers, like the ESP8266, as well as the computer vision system running on a laptop. The baud rate was set to 9600, alhtough changing the BAUD macro and CPU_FREQ macro in the uart.c file allows for this baud rate to be chaged. In addition, we configured out protocol to use 8-N-1 frame formats (8 data bis, 1 stop bit, no parity bit). The library supplies the ability to enable an interrupt service routine (ISR) on the Atmega328P that is triggered when new serial data is received. We used this ISR capability to ensure low latency between the object's coordinates being transmited and the servo motors receiving a pwm signal command from the microcontroller. 


**TFT-LCD Radar Display (Relevant Files: include/LCD_GFX.h, src/LCD_GFX.c, src/LCD_main.c)**

We wrote a LCD graphics library for the ST7735R. The datasheet can be accessed [here](https://www.crystalfontz.com/controllers/Sitronix/ST7735R/). This library uses a standard Serial Peripheral Interface (SPI) communication protocol to write real time positional data about the tracked object to our LCD, thus making it akin to a "radar display" of objects around the turret. The red dot displayed in the demo video at the bottom of the LCD is to depict the position of the turret, while the blue dot shown is the position of the object being tracked relative to the turret, as if someone was facing in the direction that the cameras point at. We used a *separate* Atmega328p to control the LCD, due to the latency that writing to the LCD would cause if done on the same Atmega328p being used to control the servo motor turret. The LCD controlling Atmga328p received the positional data from the computer vision system using a UART based serial communication protocol (see Serial Library Component). 


**Stereo Vision System (Relevant Directory: /StereoVision)**

When deciding what type fo computer vision system to use, we immidiately realized that distance measurement would be one of our most important metrics. Without depth perception, we would not be able to accurately calculate angles to aim the turret at a projectile. Thus, we chose a stereo vision system, which uses two cameras to allow for depth perception to be simulated. 


**ESP8266 Web Server (Relevant Files: esp8266/esp8266/esp8266.ino)**

We used an ESP8266 microcontroller to host a webserver that supplied information to the user about the current aiming position of the turret (see media above), as well as the ability to turn on and off the laser on the turret. The website was written using HTML/CSS/JS along with some basic api requests in the website back to the ESP8266, such that as new data was received over UART by the ESP8266, the website would asynchronously request and receive this new data to be displayed. The website code is inside of the .ino file in a C-string. There may have been better ways to host the code besides this, but given the simplicty of our website, and the time constraints of the project we didn't focus on this. This device is also wired to an input capture pin on the Atmega328p to send information about controlling the laser. In addition, we used a voltage divider to step down the logicl level from 5V to 3.3V when sending serial data from the Atmega328p to the ESP8266.
