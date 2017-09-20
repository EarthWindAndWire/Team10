## Lab 2: Analog Circuitry and FFTs

### Objective
Now that we’ve got our robot moving, we want it to be able to sense the surrounding environment. In this lab, we were working with a 
microphone to detect the 660 Hz start signal and an IR sensor to identify treasures of different frequencies.

#### Materials
- Arduino Uno
- Electret microphone
- IR receivers
- Treasure board
- Resistors
- Capacitors
- Op amps

### Part 1: FFTs

In this lab, we needed to be able to distinguish between signals of different frequencies. We used the Fast Fourier Transform (FFT) 
algorithm in Open Music Lab’s FFT library to accomplish this. A Fourier transform takes a signal in the time domain and breaks it down 
into its component frequencies. An FFT works by splitting an N point time domain signal into N individual time domain signals, calculating 
the frequency spectra of each, and then merging them into a single frequency spectrum. This is a sped-up version of a Discrete Fourier 
Transform (DFT).

For this lab, we used Open Music Labs’ FFT library to analyze the signals from our sensor. After downloading and installing the library, we
opened up the fft_adc_serial.ino sample code to get a better understanding of how the library worked. This program takes 256 samples from 
the ADC and places them in even bins (odd bins are used for the complex signal components - since we are only dealing with real signals, 
odd bins are excluded). The program then processes the data in the fft and sends it out over the serial monitor, in the form of a histogram.
Each bin represents a frequency range, determined by the ADC clock frequency.

To verify that the library was working as expected, we hooked our adc up to a function generator and tested it using a 10 kHz signal.
