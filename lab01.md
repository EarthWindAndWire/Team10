# Lab 1: Microcontroller 

## Objective
 
The goal of this introductory lab section was to get familiarized with the different functionalities of the Arduino Uno and the Arduino IDE. Our team had the opportunity to construct a simple functional Arduino program using multiple external components and the Arduino Uno. Once we had this basic understanding, we proceeded to assembly the robot and made it perform an autonomous task such as driving a square path.

## Materials

1 Arduino Uno
1 USB A/B cable
1 Continuous rotation servos
1 Pushbutton
1 LED (any color except IR!)
1 Potentiometer
Several resistors (kΩ range)
1 Solderless breadboard

## Part 1: Blinking LED

First, we ran the Blink sketch to verify that the Arduino board and internal LED were working. Then we modified the code to blink an external LED, and verified that all of the pins were functioning. 

Code:
void setup() {
  for (int i=0;i<14;i++)
  {
    pinMode(i,OUTPUT); //configure pins 0-14 as output
  }
}
// the loop function runs over and over again forever
void loop() {

    int i = 12; //changed this each run to check each pin
    digitalWrite(i, HIGH);   // turn the LED on (HIGH is the voltage level)
    delay(1000);                       // wait for a second
    digitalWrite(i, LOW);    // turn the LED off by making the voltage LOW
    delay(1000);                     // wait for a second
  
}

External LED circuit image:
![External LED](IMG_3192.JPG)



## Part 2: Analog Voltage and LED Output

For the first step of this part, we built a voltage divider circuit using a potentiometer. We then output the analog voltage values controlled by the potentiometer to the serial monitor.

Pic of serial monitor:


Serial output from potentiometer code:

const int analogInPin = A5;  // Analog input pin that the potentiometer is attached to
int voltageValue = 0;        // value read from the pot

void setup() {
  // initialize serial communications at 9600 bps:
  Serial.begin(9600);
}

void loop() {
  // read the analog in value:
  voltageValue = analogRead(analogInPin);

  // print the results to the Serial Monitor:
  Serial.print(voltageValue);
  Serial.print('\n');

  delay(15);
}

For the second half of this part, we modified the circuit and the code to do an analog to analog conversion so that the brightness of the LED varied linearly with the input voltage. 

Circuit image:
![Circuit](IMG_3195.JPG)



Code:
const int analogInPin = A5;  // Analog input pin that the potentiometer is attached to
const int analogOutPin = 11; // Analog output pin that the LED is attached to
int voltageValue = 0;        // value read from the pot

void setup() {
  // initialize serial communications at 9600 bps:
  Serial.begin(9600);
}

void loop() {
  // read the analog in value:
  voltageValue = analogRead(analogInPin);
  // map it to the range of the analog out:
  outputValue = map(voltageValue, 0, 1023, 0, 255);
  // write the value to the LED:
  analogWrite(analogOutPin, outputValue);

  delay(15);
}

The PWM frequency of the LED was 490.2 Hz.

![PWM frequency](PWNFreq.png)



Part 3: Servos

The purpose of this lab boils down to completing some autonomous task performed by a robot. Our robot will be moving in a simple pattern, requiring propulsion. In order to provide this, we will use Parallax Continuous Rotation Servos in tandem with the Arduino Servo.h library.

The servo was attached to the previous potentiometer in order to have the speed at which the servo operates become dependent on its setting. The following code was then uploaded to the arduino:


#include <Servo.h>

Servo myServo;
const int analogInPin = A5;  // Analog input pin that the potentiometer is attached to
const int analogOutPin = 11; // Analog output pin that the LED is attached to

int voltageValue = 0;        // value read from the pot
int outputValue = 0;        // value output to the PWM (analog out)

void setup() {

  Serial.begin(9600);     // initialize serial communications at 9600 bps:
  myServo.attach(11);
  
}

void loop() {
  // read the analog in value:
  voltageValue = analogRead(analogInPin);

  // map it to the range of the analog out:
  outputValue = map(voltageValue, 0, 1023, 0, 180);

  // write the value to the servo:
  myServo.write(outputValue);

  delay(15);
}

The code takes the voltage input to the arduino (controlled by the potentiometer) and translates it into a direction and speed for the servo to turn.

A video of the configuration in action can be found here [ https://youtu.be/RKeNJGQvyiw ]

<iframe width="560" height="315" src="https://www.youtube.com/embed/RKeNJGQvyiw?rel=0" frameborder="0" allowfullscreen></iframe>


## Part 4: Robot

Robot video -> https://www.youtube.com/watch?v=w1iMTuMnZG8&feature=youtu.be 
<iframe width="560" height="315" src="https://www.youtube.com/embed/w1iMTuMnZG8?rel=0" frameborder="0" allowfullscreen></iframe>
To wrap up this lab, we began assembling our robot. For this part, we used the following materials:
Chassis
Screws
9V battery with clip
Ball bearing
2 wheels 
Allen key 


Some of the materials we used in the assembly of the robot


![Materials](whenAMommyRobotAndADaddyRobotLoveEachOtherVeryMuch.png)



We started mounting the motors onto the motor brackets and then attaching them to the bottom part of the chassis. After this, we installed the wheels and placed the Arduino Uno and the 9V battery on the top side of the base plate. At first, we were using the breadboard for the circuit but we realized that it was better to connect the motors directly to the pins in the Arduino Uno. However, in the future we have to figure out a way to colocate the breadboard on the chassis since we will need it to connect other electrical components such as the sensors. 

To make our robot perform an autonomous task, we basically created two Arduino programs: the first one made the robot drive in a straight line and the second one offered it the capacity to trace a square pattern. Although these behaviors can be considered as autonomous, they do not make our robot an “intelligent physical system” yet. As we advance with the lab sections and we start adding some sensors to our robot, it will progressively gain the capacity to interact with the environment, what will make it more intelligent.

<iframe width="560" height="315" src="https://www.youtube.com/embed/PooEK1s3c94?rel=0" frameborder="0" allowfullscreen></iframe>

## Conclusions



![Robot](RobotCar.jpg)

## Website Sessions:

[1. Home](../index.md)  
[2. Milestones](../)
