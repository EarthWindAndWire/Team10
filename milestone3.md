## Milestone 3: Maze Exploration
Team Members: Tejas Advait, Leandro Dorta Duque, Maria Bobbett, Joshua Diaz, Jennifer Fuhrer, Dean Wang

### Objective
The objective of this milestone was to design, test, and implement a maze exploration algorithm. This milestone is split into two parts - 1) development of an algorithm testing tool (simulation) and 2) implementation of the algorithm on the actual robot.

### Simulation
For simulation purposes, we decided to design and test our algorithm on MATLAB. We used the existing GUI implementation created by Team Alpha. We also looked at the code provided by Bruce Land for ideas. We add extra functionality for keeping track of orientation of the robot and walls.

GUI implementation in MATLAB is very straightforward. [Here’s](https://cei-lab.github.io/ECE3400-2017-teamAlpha/milestone3.html) the link to Team Alpha’s website for Milestone 3.

Firstly, we need to track the current position and orientation of the robot. We do this with the help of global variables cur_x and cur_y, which keep track of the x and y coordinates of the robot. For the tracking of orientation, we use another global variable “orientation.” These variables are modified depending on the command given to the robot. However, the current position of the robot is a function of the robot orientation and the command given to the robot. For eg, if the current orientation of the robot is North (N), and the robot is commanded to make a left turn, the orientation is changed to East (E). This produces no change in the position of the robot. Now, if the command is given to move forward (if there’s a node to go to), keeping in mind the orientation of the robot, the x coordinate decreases by 1 and y coordinate remains the same since the robot moved in the east direction.

[Here](https://docs.google.com/document/d/1FE8QCMpgpKX5vyR-UXhDWZ_2EFmZYrdXxQ5uZHTM7_8/edit?usp=sharing) are the truth tables that might help in understanding the change in orientation, coordinates and the direction of movement of the robot.

The algorithm we designed is a form of greedy algorithm combined with Dijkstra. The algorithm determines the next unvisited node the robot should explore. The algorithm first decides which adjacent node to visit from the current node position depending on the orientation of the robot. The adjacent node which the algorithm doesn’t decide to visit is stored in an array and is marked as partially explored. Once the robot reaches a dead-end, i.e, the robot is surrounded by already visited nodes, Dijkstra’s algorithm is run to determine the closest partially explored node from the current position by assessing path lengths to all the partially explored nodes. The robot goes to the closest partially explored node and repeats the previous steps.

The benefit of using this algorithm is that it performs, somewhat, intelligent back-tracking.

However, after thinking of numerous ways to implement this, we were not able to implement it before the deadline for this milestone. However, we are optimistic after attending the latest lab session that we would be able to implement this algorithm on MATLAB.

Therefore, we may resort to implement a simpler form of the above idea - utilizing depth first search with appropriate modifications.

The challenges faced to implement this algorithm were to first come up with a unique ID system for every node. Next, we had to figure out a way to store the edge connections information. We implement this using arrays in MATLAB.

Here’s a video of our simulation (console prints “Maze Explored” when the exploration is complete.) 

<iframe width="560" height="315" src="https://drive.google.com/open?id=1Bv0xKCXqSQcxPTuvB0KSftSBTIcPDFKA" frameborder="0" allowfullscreen></iframe>


### Simulation
We have been facing problems with our line sensors. Also, we didn’t have enough time to translate our MATLAB code for the Arduino. However, this work will be completed very soon, before the start of next week.

Also, we came up with a solution to account for turning cost in real life and aim to test its viability once we debug our robot and complete this part. This solution relies on performing a slight structural modification on how the information of the grid is stored. We wish to complete the basic maze exploration functionality as soon as possible, within the next two days. We will then look into our original algorithm and keeping track of turning cost.
