## Final Design of MJ

### Introduction

### Mechanical Design

### Treasure Sensor Circuit Design
In the final competition of design, we made additional changes to our previous treasure detection circuit. In our previous design, we made two separate circuits the  for two phototransistors. This creates problem for the robot’s mechanical design as it is would require additional mechanical work to stabilize the circuit and balance the robot if we have separate circuits on the side of the robot. It would also require two pins on the arduino uno. Therefore to minimize pins used and circuit used, we used a single filter and amplifier for the final circuit. Two inputs from the phototransistors are connected in parallel and analyzed.

In addition, although the previous design in Milestone4 had dramatically increased the range of detection (about 15 cm), it made the phototransistor possible to detect treasures that are not on the current grid. Therefore we decided to decrease the amplification and increase the threshold for the FFT results. The final set of resistors used on the robot are 100Ω and 75kΩ, which could reliably detect treasures within distance of 10cm, and ignore treasures from further distance.

The picture below is the treasure circuit with one phototransistor plugged in:

![TreasureCircuit](TreasureCircuit.png "TreasureCircuit")

Here’s a schematic of our final treasure circuit (the circuit model software does not have symbols for phototransistors, so I used the symbol of photodiodes instead)

![TreasureSchematic](TreasureSchematic.png "TreasureSchematic")


### Maze-Mapping Algorithm


### Base Station
![BaseStation](BaseStation.png "BaseStation")

We were able to assemble a fully-functioning base station. We decided to use parallel communication between the Arduino and FPGA (described in lab 4), as data is sent faster and it is easier to program.

#### Arduino
We ended up using 17 bits to send all of the maze information (x and y current, x and y last, treasures, walls, and the done signal). Since we also needed to attach the radio, the Arduino Uno’s 20 analog/digital pins weren’t going to cut it. We decided to upgrade to the Mega board. At first, we set the radio on the Mega up the same way it was configured on the Uno in lab 4, but it wasn’t able to receive the transmission from the Uno. After looking at the pin configuration, we realized that the SPI pins were in a different location on the Mega. After rewiring the board and updating the code with the new pins, we were able to send and receive maze information over radio. The robot sent the maze information in a 32-bit packet, of which only the 17 least significant bits were used.

![RadioSetup](RadioSetup.png "RadioSetup")

Once the base station received the packet from the robot, it sent it to the FPGA by writing to the digital pins in a for loop over each bit of the message. A valid bit (pin 32) was used to tell the FPGA to sample the signals after they had all been sent, so they weren’t sampled while they were still in flux, which could cause errors in the display.

Code: 
``` 
    void write_new_data(unsigned long new_data){
/*Writes new_data to digital pins 53-23 w/ the following mapping
New_data: b16 | b15 | b14 | b13 | b12 | b11 | b10 | b9 | b8 | b7 | b6 | b5 | b4 | b3 | b2 | b1 | b0
d_pins: 33 |  34 |  35 |  36 |  37 |  38 | 39  | 40 | 41 | 42 | 43 | 44 | 45 | 46 | 47 | 48 | 49
data: x1 |  x0 |  y2 |  y1 |  y0 |  T1 | T0  | NW | EW | SW | WW |xL1 | xL0| yL2| yL1| yL0| Dn
 */
  unsigned long mask = 0b00000000000000010000000000000000;
  for (int i = 33; i<=49 ; i++){
      if(new_data & mask){
        digitalWrite(i,HIGH); 
      }
      else{
        digitalWrite(i,LOW);   
      }
      mask >>= 1;
    }
    printf("\n");
    delay(10);
    digitalWrite(32, HIGH);
    delay(100);
    digitalWrite(32,LOW);
    delay(1000);
  }

```
#### FPGA
We were able to expand the Verilog code from milestone 4 to display the entire maze, using the same scheme described in milestone 4 (a 9x11 matrix to represent the 4x5 maze).

We used assign statements to read the information sent from the arduino:
```
assign north = GPIO_1_D[8];
assign east = GPIO_1_D[9];
assign south = GPIO_1_D[10];
assign west = GPIO_1_D[11];
assign treasure_in[1] = GPIO_1_D[6];
assign treasure_in[0] = GPIO_1_D[7];
//	
assign xcoord[1] = GPIO_1_D[1];
assign xcoord[0] = GPIO_1_D[2];
assign ycoord[2] = GPIO_1_D[3];
assign ycoord[1] = GPIO_1_D[4];
assign ycoord[0] = GPIO_1_D[5];
assign x_last[1] = GPIO_1_D[12];
assign x_last[0] = GPIO_1_D[13];
assign y_last[2] = GPIO_1_D[14];
assign y_last[1] = GPIO_1_D[15];
assign y_last[0] = GPIO_1_D[16];
```

As we expanded the maze from the original 2x2 grid, the display would occasionally mis-assign treasures, or show multiple current positions, or throw up rogue walls. The weirdest part of this bug was that it would only happen ~20% of the time, and the other 80% of the time it worked fine. We were finally able to solve this issue by using a valid bit to make sure we were only updating the maze information after all of the signals from the arduino had been set.
```
if (reset == 1'b1)begin
	//initialize maze to unvisited, show exterior walls
end
else if (validbit == 1'b1) begin
	//update the maze matrix with new position information

	//update maze x and y coordinates
	maze_x = 2*xcoord + 1;
	maze_y = 2*ycoord + 1;
			
	maze_xlast = 2*x_last + 1;
	maze_ylast = 2*y_last + 1;
		
	//record treasure information in the treasure matrix
	treasure[xcoord][ycoord] = treasure_in;
	
//any variable ending with c is an 8-bit RGB color corresponding to that description
//assign walls	
	if(north) begin
maze[maze_x][maze_y-1] = wallc; end
	else begin
		maze[maze_x][maze_y-1] = unvisitedc; end
			
	if(south)begin
		maze[maze_x][maze_y+1] = wallc; end
	else begin
		maze[maze_x][maze_y+1] = unvisitedc; end
		
	if(east) begin
		maze[maze_x+1][maze_y] = wallc; end
	else begin
		maze[maze_x+1][maze_y] = unvisitedc; end
		
	if(west) begin
		maze[maze_x-1][maze_y] = wallc; end
	else begin
		maze[maze_x-1][maze_y] = unvisitedc; end
	
//set the color of the center square depending on the state		
	if(treasure[x_last][y_last]==2'b00) begin //no treasure
		maze[maze_xlast][maze_ylast] = visitedc; end //set as visited
	else if(treasure[x_last][y_last]==2'b01) begin //treasure 1
		maze[maze_xlast][maze_ylast] = treasure1c; end //set as treasure 1
	else if(treasure[x_last][y_last]==2'b10) begin //treasure 2
		maze[maze_xlast][maze_ylast] = treasure2c; end //set as treasure 2
	else if(treasure[x_last][y_last]==2'b11) begin //treasure 3
		maze[maze_xlast][maze_ylast] = treasure3c; end //set as treasure 3

end
else if (validbit == 1'b0) begin
	//don’t change anything
maze_x = maze_x;
maze_y = maze_y;
			
maze_xlast = maze_xlast;
maze_ylast = maze_ylast;
		
treasure[xcoord][ycoord] = treasure[xcoord][ycoord];
end
```

Drawing the maze for coordinate 0,0:
```
//coordinate 0,0
if(PIXEL_COORD_X <= 2 && PIXEL_COORD_Y <= 75) begin //west wall
		pixel_color <= maze[0][1]; //set pixel color to the color in the maze matrix
end
else if(PIXEL_COORD_X >=3 && PIXEL_COORD_X <= 72 && PIXEL_COORD_Y <= 2) begin //north
		pixel_color <= maze[1][0]; 
end
else if(PIXEL_COORD_X >= 73 && PIXEL_COORD_X <= 77 && PIXEL_COORD_Y <= 75) begin //east
		pixel_color <= maze[2][1]; 
end
else if(PIXEL_COORD_X >= 3 && PIXEL_COORD_X <= 72 && PIXEL_COORD_Y >= 73 && PIXEL_COORD_Y <= 77) begin //south
		pixel_color <= maze[1][2];
end		
else if(PIXEL_COORD_X>=3 && PIXEL_COORD_X<= 72 && PIXEL_COORD_Y >=3 && PIXEL_COORD_Y <= 72) begin //inner square
		pixel_color <= maze[1][1];
end
```

#### Integration & Testing
We integrated the audio code from Lab 3 for the done signal, and wrote code for an arduino to simulate the robot sending information over radio. We also hooked up the microphone circuit to the same arduino to test the start signal.

Arduino simulation code:
```
//Collect data to be sent (Simulation)
    x_coord = get_x_coord(i); //these hold hard-coded values
    y_coord = get_y_coord(i); 
    pos_data = get_pos_data(i); //pos data holds treasure and wall info
    
    if(i == 25){
      DONE = 1;
    }
    if(i<26){
      i = i+1;
    } 
    
    // Pack the bits into a single 32 bit message 
    unsigned long new_data = packdata(x_coord, y_coord, x_coord_last, y_coord_last, pos_data, DONE);

    // Save the previous coordinates for the next iteration
    x_coord_last = x_coord;
    y_coord_last = y_coord;
    
    // Take the time, and send it.  This will block until complete
    printf("Now sending new map data\n");
    bool ok = radio.write( &new_data, sizeof(unsigned long) );
    if (ok)
      printf("ok...");
    else
      printf("failed.\n\r");
```

Here's a video of our completed simulation on the base station:

<iframe width="560" height="315" src="https://youtu.be/3dvZtSK8KO0" frameborder="0" allowfullscreen></iframe>






### Robot Cost

### Conclusion/Lessons Learned

