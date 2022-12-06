---
title: "Microcontroller"
date: 2022-12-06T22:01:14+01:00
draft: false
---

## First Day

On the first day we learned some basics about the Arduino microcontroller 
and how to connect it so we can program it. We got a starterkit and 
started trying out different things.
![](img/microcontroller/img1.jpg "The starterkit")

First I started to make the example we got shown of an LED blinking. I 
connected the LED and a resistor on the breadboard and connected them 
to the Arduino. I then modified the example code of an internal LED blinking 
to set it on the pin I connected to the LED.
![](img/microcontroller/img2.jpg "The LED connected to the Arduino")

The next thing I wanted to try is using a servomotor (as it is also useful 
for my project). I connected the VCC and GND pins to the corresponding pins 
on the Arduino and connected the third pin to a digital output pin on the 
Arduino. I looked up a library that automatically converts a degree input in 
the code to a Pulse Wide Modulation (PWM) signal and used it to let the 
motor spin to 180° and back to 0°.
![](img/microcontroller/img3.jpg "The servomotor connected to the Arduino")

After I saw a joystick in the starterkit I wanted to control the servomotor 
with it. For that I needed to connect the joystick to two analog inputs. One 
input was for the x-axis and one was for the y-axis. I converted the signal 
of the x-axis to have a range between 0° and 180° and send that value to the 
servomotor. I still hat the y-axis doing nothing though so I decided to 
use it to control a stepper motor. For the steppermotor I needed a 
motorcontroller chip which I could just connect to the arduino and with the 
help of yet another library I could spin the stepper motor in both directions.
<iframe src="https://drive.google.com/file/d/1hBBp36CflfbsCCt0l7eV-iws1JWu88fw/preview" width="640" height="480" allow="autoplay"></iframe>

![](img/microcontroller/img4.jpg "The joystick controlling both the servomotor and the stepper motor")

I still wanted to try a sensor so I decided to use an ultrasonic sensor 
to detect if something was closer than 10cm and if that was the case to 
set the servomotor to 180°. The sensor is connected to the Arduino with two 
pins: One serving as a trigger and one serving as an echo. To calculate the 
distance of an object in front of it I send a signal to the trigger pin for 
10ms and waited for a signal to come in through the echo pin. With the speed 
of sound the duration it took for the echo pin to get a signal I could 
calculate the distance and then just checked if it was less than 10cm. 
![](img/microcontroller/img5.jpg "The ultrasonic sensor connected to the Arduino")
![](img/microcontroller/img6.jpg "Closeup of the ultrasonic sensor")

## Second Day

The Second Day working with the Arduino I decided to try out the Bluetooth 
module. For that I used the SoftwareSerial library and set it to the two 
pins I connected the RX and TX of the module. Now I could use that serial 
to recieve and transmit messages to and from the module. To connect to it I 
downloaded a free app designed for Arduino Bluetooth testing purposes.
![](img/microcontroller/img7.jpg "The Bluetooth module connected to the Arduino")
![](img/microcontroller/img8.jpg "Another angle of the Bluetooth module")

I then programmed it to control the motors like on the first day so I could control 
them with my phone. I also switched the stepper motor with a universal brushed 
motor because that's what I am going to need for the project. The motorcontroller 
of that motor didn't work right away though so the rest of the day was spent 
troubleshooting.
![](img/microcontroller/img9.jpg "The universal brushed motor connected to the Arduino")

## Third Day

To better control the motors exactly how I want to I made an Android app to connect 
to the Bluetooth module. I used a slider to control the steering and two buttons 
to control driving forwards/backwards.
![](img/microcontroller/img10.jpg "The Bluetooth Controller app")

Unfortunately it had a few bugs still and couldn't connect to the Bluetooth module 
properly so I decided to look into interrupts. I took my code from the first day 
where the servomotor spins to 180° and back to 0° and assigned a button as an 
interrupt so I could stop the loop with a press of the button, which to be fair 
could be implemented without an interrupt the way I did it but it was only to try 
it out.
![](img/microcontroller/img11.jpg "A button connected to the Arduino")

All in all I would say I learned a lot about microcontrollers and programming the 
Arduino and I probably got all the knowledge I need for my project.

## Interrupts

Here I'm going to explain interrupts in more detail. In general the code of a microcontroller 
is run inside a loop and an interrupt is a way to, as the name suggests, interrupt the loop 
while some other predefined code gets executed. As an example I show the code I 
used for the test mentioned above:
```c
#include <Servo.h>

Servo servo;

#define button 3;

bool stop = false;

void setup() {
  Serial.begin(9600);
  servo.attach(2);
  pinMode(button, INPUT);
  attachInterrupt(1, interruptTest, FALLING);
}

void loop() {
  if (!stop)
  {
    servo.write(180);
    delay(1000);
    servo.write(0);
    delay(1000);
  }
}

void interruptTest() {
  Serial.println("interrupted");
  stop = !stop;
}
```

The interrupt can be attached to two pins: pin 2 and pin 3, which 
correspond to 0 and 1 in the attachInterrupt() method. I 
connected the button to pin 2 so I put in 1. Then I assigned the 
method interruptTest() to that interrupt and defined that it 
only gets triggered with a falling impulse.

Another use of the interrupt is to wake the Arduino while it is 
in sleep mode. To put the Arduino in sleep mode we can use the 
following code:

```c
#include <avr/sleep.h>

void isrAwake(void) {
  detachInterrupt(0);
}

void enterSleepMode(void) {
  attachInterrupt(0, isrAwake, FALLING);
  set_sleep_mode(SLEEP_MODE_PWR_DOWN);
  sleep_enable();
  sleep_mode();
  sleep_disable();
}

void setup() {
  
}

void loop() {
  if (!stop)
  {
    servo.write(180);
    delay(1000);
    servo.write(0);
    delay(1000);
  }
}

void interruptTest() {
  Serial.println("interrupted");
  stop = !stop;
}
```

This code wakes the Arduino when pin 2 get a signal.