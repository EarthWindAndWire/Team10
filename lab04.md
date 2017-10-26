## Lab 4: Radio Communication and Map Drawing

### Objective
For this lab assignment, we split into two groups: one group works on the radio component and another on the FPGA component. For the last portion of the lab, both pieces are combined. The radio team uses the Nordic nRF24L01+ transceivers and the corresponding Arduino R24 library in order to achieve wireless communication between the Arduino at the base station and the Arduino on the robot. At the end of the lab, the team will be able to send messages from one Arduino to the other (simulating actual maze information) and have the FPGA display the received data on the monitor. 

### The Radio Team
Maria Bobbett, Leandro Dorta Duque, Tejas Advait

#### Material
 - 2 Nordic nRF24L01+ transceivers
 - 2 Arduino Unos (one must be shared with the other sub-team)
 - 2 USB A/B cables
 - 2 radio breakout boards with headers
 
 #### Getting Started 
 
![Arduino Image](whatanicearduino.PNG "Arduino and transceiver connection")
 
First we downloaded and installed the RF24 Arduino library required to work with the transceiver. The transceivers were soldered apart from the 3.3V supply wire. We soldered the wire and connected the transceiver as shown in figure 1. We also used two different laptops to test and program the sender and the receiver Arduino board.
Next we downloaded the getting started sketch from ECE 3400 lab repository and modified the identifier number for two pipes according to our team number and lab day. 
2*(3D+N) + X ---> 2*(3*2 + 10) + X [D = 2 for Wed Night, N = 10 for team number 10 and X is 0 for one Arduino and 1 for the other]. We get 32 (20 in hex) and 33 (21 in hex)
// Radio pipe addresses for the 2 nodes to communicate.
const uint64_t pipes[2] = { 0x0000000020LL, 0x0000000021LL };
 
![Serial Monitor Image](whataniceserialmonitor.jpg "Serial monitor for initial test")

#### Sending the entire maze wirelessly 
For this part of the lab, we have to be able to send the entire maze wirelessly from one Arduino to the other. In order to achieve this functionality, we first need to define what the information would be for our maze. We use a 4x5 matrix (two dimensional array) to represent the maze where the state of each box is represented by an integer (in this case, the integers are considered unsigned characters, so they count 1 byte each). We have to make sure that our maze (2-dimensional array) can be sent in a single payload. Since the default size of a payload is 32 bytes and our array contains 20 bytes (20 unsigned characters), we are able to send the entire maze in one payload. 
The following is an example of the code we use to represent the maze (the number for the states are randomly picked, their real values depend on the states we decide to use for each box in the maze). 

![Maze Array](whatanicemazearray.PNG "It's no labyrinth but it does the job")

We include the following code in order to send the maze from the transceiver Arduino to the receiver one and to receive the acknowledgement from the receiver to the transceiver to indicate the information was successfully received.

![Reception](whatanicereception.PNG "Received")

In this part, our Arduino is sending a payload of 25 bytes, which is fine because the payload supports up to 32 bytes. However, this is not recommendable because it is a huge size of information which can be easily fragmented due to interference and even though our system can correct any incomplete information through the Auto-ACK feature, this could make us lose time. 
The following represents the code of the receiver Arduino which basically processes the information received, displays it and sends the acknowledgement to the transceiver Arduino to indicate the information was correctly received. 

![Fetch it](whatanicefetchcode.PNG "Acknowledgement")

Serial Monitor Sender:
![Serial Monitor Sender](whatanicemonitorAGAIN.PNG "Serial Monitor Sender")

Serial Monitor Receiver:
![Serial Monitor Receiver](ohhelloagainserialmonitor.PNG "Serial Monitor Receiver")

#### Sending New Information Only
Three pieces of information is essential to update the maze - x coordinate, y coordinate and the box value.
The maze is 4x5 and we follow the system where x coordinate varies from 0 to 3 and the y coordinate varies from 0 to 4. Thus, we can encapsulate new maze data as follows - 2 bits for the x coordinate, 3 bits for the y coordinate and 3 bits for the box value [depending on whether the box has been explored, contains treasure etc]. In this lab, we do not stress on the values for the treasure, but since we’re using three bits for the box value, we have enough bits to associate with various states of the box.
In total, we have 8 bits or 1 byte of data which needs to be relayed between the Arduinos. Since the default payload size is 32 bytes, we change this accordingly to establish an efficient communication system.
We pack the byte using the following scheme: 
x-coordinate (2bits) | y-coordinate (3bits) | box value (3bits)

Sender Side Code:
![Sender side code](lab4radioarduioncode.PNG "Sender Side Code")

Receiver Side Code:
![Receiver Side Code](Receiversidecode.PNG "Receiver Side Code")


### The FPGA Team
Yixuan Wang  Joshua Diaz  Jennifer Fuhrer

#### Material
 - FPGA
 - 1 Arduino Uno
 - 1 VGA cable
 - 1 VGA connector
 - 1 VGA switch
 - Various resistors
 
 #### Overview
Communication between Arduino and FPGA

Before we could implement wireless communication, we needed a reliable scheme of wired communication between the Arduino and FPGA. We chose to implement parallel communication. Despite the fact this form of communication uses several of the pins between the two boards, it simplified the code we needed to write for the two boards and in theory would limit debugging time.

The radio team decided we would need to transmit 8-bit packages between the boards, 3 bits for the x coordinates, 2 bits for the y coordinates, and 3 for the state of the robot and treasure detection. This meant we had to use 8 pins on both boards. We used 0-7 on the Arduino and GPIO_0_D 10-17 for the FPGA. In addition, we needed to create voltage dividers for each pin because the Arduino outputs 5V, but the FPGA inputs up to 3.3V. Using two resistors, one 328 ohms and the other about 177 ohms, we pulled the voltage down outputted by the Arduino to 1.76 volts (when measured on the oscilloscope). 

![Wiring](ParallelCommunicationwithVoltageDivider.PNG "Who knew the FPGA could rock dreads")

![Oscilliscope Image](lab4VGA.PNG "Come back!")

CH1 is a reading from pin 7 on the Arduino when the program is finished (for the MSB of the x coordinate which is reading 4 or 100)

Once all the inputs and outputs were safely wired up on a breadboard, we wrote code for the Arduino to simulate a robot moving across all the spaces on the grid in a zigzagging pattern. The code started at x=0 and incremented up the y values (from 0 to 3), then once it reached y=3, the x state would increment up to 1, then decrement the y values to 0. This pattern continued until x=4 and y=3 and would stay at that position.

To transmit this information, the Arduino used pins 3 and 4 for the y values (3 LSB, 4 MSB) and 5-7 for the x values (5 LSB, 7 MSB). These were connected to the FPGA GPIO_0_D[14:13] and GPIO_0_D[12:10] respectively. 

![Arduino Image](Lab4VGAArduinoCode.PNG "Good Code")

Once the information was inputted into these pins, the FPGA reads the numbers as xcoord and ycoord. These values are combined into a single five bit address to be evaluated in a case statement to check whether the robot is currently on this coordinate. Once the robot reaches this coordinate, and subsequent address, the corresponding grid turns red and changes a state bit to 1, meaning the robot has already visited this location. The state variable is a 4x5 matrix, with one bit for each grid square. A 0 indicates the robot hasn’t visited that location and a 1 indicates it has. As previously described, once a robot goes to a location, it flips the state bit to 1. Once this state bit is changed, several if statements evaluate this state bit before the case statement. If the state bit is 0, the grid square is white, if it is 1 then the grid square is green.

![Verilog Code](Lab4VGAVerilogCode.PNG "Not as good code")

With this logic, a red square should travel through the grid in a zigzag pattern, turning all the originally white squares to green as it goes along. However, we had several issues implementing the Verilog code which resulted in undesired outputs. Initially, all our entire grid was initialized green despite us only inputting a single positional value of x=0 and y=0. In addition, if we removed the if statements that evaluated the state, the entire grid would turn red. At this point, we realised this issue is probably with the implementation of address.

Upon advisement, we added a default case to our case statement. However, our single positional value of x=0 and y=0 would still not trigger the desired outcome. The entire grid was white instead. At the present moment, we are still are having problems displaying the correct inputs so we were unable to get past this incorrectly colored maze. Unfortunately we are still in a state of debugging because we are unsure of the exact problem plaguing our code.

![Our First Issue](Lab4VGAFirstIssue.PNG "Wah wah wah")
![Is this a solution](Lab4VGAInitialSolution.PNG "We're still workin on it")

