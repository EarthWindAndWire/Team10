## Milestone 2: Treasure Detection and Wall Detection
### The Goal

For Milestone 1, we have two primary objectives:
- Detect frequencies at 7kHz, 12kHz and 17kHz.
- Autonomously detect walls

### Treasure Detection
The purpose of this part of Milestone 2 is an to extend the range of frequencies that can be detected and classified. Since our Lab 2 optical circuit worked well, the circuit design of detecting different treasures is the same as the one we used in lab2.
Based on our code and the testing frequency, we can figure out right bins to check for in the FFT results, which are also the bins where the highest peaks occur. 
For the 7kHz testing frequency, the highest peaks occurs at bin 46-48. 

![7kHz](7kHz 7kHz.png)

For the 12kHz testing frequency, the highest peaks occurs at bin 79-81.

![12kHz](12kHz 12kHz.png)

For the 17kHz testing frequency, the highest peaks occurs at bin 116-117.

![17kHz](17kHz 17kHz.png)

To confirm our result, we also used the function generator to generator 12kHz and 17kHz sine frequencies like we did for the 7kHz frequency in lab2, and these are the result we had. 

![12kFG](12kFG 12kFG.png)

![17kFG](17kFG 17kFG.png)

From the above results, we wrote a simple algorithm to classify the FFT results. We used an else-if statement in case that small peaks might lead confusion to the result.

![TreasureCode](TreasureCode TreasureCode.png)

As we can see from the above graphs, it might be helpful to add an analog filter to the circuit in the future. This would be helpful to filter low frequency noises, and enable the FFT to focus on the signal from the IR sensors. 
