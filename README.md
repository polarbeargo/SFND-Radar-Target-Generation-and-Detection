# SFND-Radar-Target-Generation-and-Detection
[//]: # (Image References)

[image1]: ./images/fig2.png
[image2]: ./images/fig3.png
[image3]: ./images/Range_from_first_FFT.png
[image4]: ./images/project_layout.png 
[image5]: ./images/image8.png 

### Project Layout  
  
- Configure the FMCW waveform based on the system requirements.  
- Define the range and velocity of target and simulate its displacement.  
- For the same simulation loop process the transmit and receive signal to determine the beat signal  
- Perform Range FFT on the received signal to determine the Range  
- Towards the end, perform the CFAR processing on the output of 2nd FFT to display the target.   
![][image4]  

### Radar System Requirements   

|Parameter|Value|
|---------|-----|
|Frequency|77GHz   |
|Range Resolution   |1m   |
|Max Range   |200m   |
|Max Velocity   |70m/s   |
|Velocity Resolution   |3m/s   |  

### FMCW Waveform Design  
Using the given system requirements, design a FMCW waveform. Find its Bandwidth (B), chirp time (Tchirp) and slope of the chirp.  
```
B = c/(2*rng_res)
Tchirp = 5.5*(range_max*2/c)
slope = B/Tchirp
```
Result: slop = `2.0455e+13` 
### Simulation Loop  
Simulate Target movement and calculate the beat or mixed signal for every timestamp.  
```
B = c/(2*rng_res)
Tchirp = 5.5*(range_max*2/c)
slope = B/Tchirp
```

### Range FFT  
Implement the Range FFT on the Beat or Mixed Signal and plot the result   
![][image3]  

Range Doppler Map   

![][image1]  
### 2D CFAR  
![][image5] 
 
Implement the 2D CFAR process on the output of 2D FFT operation, i.e the Range Doppler Map  

- Determine the number of Training cells for each dimension. Similarly, pick the number of guard cells.
- Slide the cell under test across the complete matrix. Make sure the CUT has margin for Training and Guard cells from the edges.
- For every iteration sum the signal level within all the training cells. To sum convert the value from logarithmic to linear using db2pow function.
- Average the summed values for all of the training cells used. After averaging convert it back to logarithmic using pow2db.
- Further add the offset to it to determine the threshold.
- Next, compare the signal under CUT against this threshold.
- If the CUT level > threshold assign it a value of 1, else equate it to 0.

```
%Select the number of Training Cells in both the dimensions.
Tr = 10;
Td = 4;

%Select the number of Guard Cells in both dimensions around the Cell under 
%test (CUT) for accurate estimation
Gr = 5;
Gd = 2;

% offset the threshold by SNR value in dB
offset = 1.2;
```


```
for i=1:Nr/2-2*(Td+Gd)
     for j=1:Nd-2*(Tr+Gr)

    
           % Use RDM[x,y] as the matrix from the output of 2D FFT for implementing
           % CFAR

          train_noise_sum = db2pow(RDM(i:i+2*(Td+Gd),j:j+2*(Gr+Tr)));
          noise_level(i,j) = pow2db(sum(sum(train_noise_sum))/training_cells);
          threshold = noise_level(i,j)* offset;
          signal = RDM(i+Td+Gd,j+Td+Gr);

          if (signal > threshold)
            signal = 1;
          else
            signal = 0;    
          end
          CFAR(i+Td+Gd,j+Td+Gr) = signal;
    end
end
```
![][image2] 