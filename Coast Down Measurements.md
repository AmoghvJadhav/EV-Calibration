# Coast Down Measurement (Estimation of F0,F1 and F2 Measurement)

## Overview
This document describes the procedure, setup, and data processing method used for **coast down measurements**.  
A coast down test is used to determine vehicle resistive forces (rolling resistance, aerodynamic drag, drivetrain losses) by measuring deceleration while the vehicle coasts freely.
This was small part of my Master Thesis which I had implemented on Python to estimate these coefficients and lets suppose if have any other variable like tyre from a different manufacturer then 
comparing the effect of resistances of these two tyres.

The results are typically used for:
- Vehicle resistance modeling
- Energy consumption simulation
- Correlation with dynamometer data
- Validation of vehicle performance models

---

## Test Objective
- Measure vehicle deceleration during free coast
- Calculate total road load forces
- Derive coefficients for resistance models (e.g. F0, F1 and F2 coefficients)

---

## Test Conditions
<img width="808" height="990" alt="Screenshot 2025-12-25 194730" src="https://github.com/user-attachments/assets/714e9717-a592-485a-9110-9d3da66fe3cf" />

Reference : Scientific Journal of Silesian University of Technology Series Transport 

Paper Name : VEHICLE COAST-DOWN METHOD AS A TOOL FOR CALCULATING TOTAL RESISTANCE FOR THE PURPOSES OF TYPE-APPROVAL FUEL CONSUMPTION


So including the test conditions in the above image more additional imformation regarding following parameters are needed.

### Vehicle Configuration
- Vehicle model
- Mass (including driver and instrumentation)
- Tire type and size
- Tire pressure
- Drivetrain configuration

## Instrumentation
- Speed measurement (GPS / wheel speed sensor)
- Sampling rate
---

## Test Procedure
1. Warm up the vehicle to normal operating temperature.
2. Verify tire pressure and vehicle mass.
3. Accelerate the vehicle to the target start speed.
4. Shift to neutral (or disengage drive) at the start point.
5. Ideally the regeneration in the EV must be set 0 or no regeneration.
6. Also some test engineers prefer to keep the Creep Torque on i.e the end speed for the test is not 0 kph but close to 8 - 9 kph which is dependent on individual manufacturer. 
7. Allow the vehicle to coast down without braking or steering input.
8. Record speed versus time until the minimum speed is reached.
9. Repeat the test multiple times in both directions (if applicable).

<img width="1536" height="963" alt="ChatGPT Image Dec 25, 2025, 09_35_02 PM" src="https://github.com/user-attachments/assets/7f5394e0-1074-4e68-97b2-fe5b9d6974ba" />

Reference : SAE J2452 
---

## Data Processing

### Raw Data
- Time [s]
- Vehicle speed [km/h]

### Data Filtering
- Remove invalid runs
- Cutting the measurement needed for analysis from the entire measurement file
- Filter noise : The most common filter is moving average but sometimes however the peaks and valleys are completed smoothened out. So I use Savitzky–Golay filter which takes into
account this features . The window size for this filter is generally odd number 11,21,31 etc and generally the polynomial of order 3 gives a good fit.

---
### Python Pseudo code

I don't have the exact python code that I used to implement the method mentioned in Scientific Journal of Silesian University of Technology Series Transport . I will put the code in another file 
, but here i have explained the logic building behind the code and one can build that with correct prompt on AI.

So I made a GUI for this where the user can upload the files that are cut from the measurement . Taking reference from the paper the Test Development Procedure was to do 3 Measurements
each from Side A and Side B each , so total of 6 measurements.

1st Step : Upload the 6 measurements , 3 from each side 
- Use tkinter library to create the GUI Background and use askopenfile with entry boxes for inserting the measurement file . Make a confirmation button once all the files are uploaded and it must
show error if any 1 is missing.
- Usually the measurement files are .dat or .MF4 format and the library used to read this files is asammdf . Every manufacturer has different labels for the signals so getting the correct signal name
is very important

Code for getting the data from the signal is as follow :

from asammdf import MDF
file = mdf.resample(raster=sampling rate)
Signal = file.get'Signal_name'
Signal_name is to be taken for Wheel Speed or Vehicle Speed
Time = numpy array  of signal timestamps 
Speed = numpy array of signal samples


2nd Step : Validation 
- In this step the measurement that are uploaded by the user in the entry boxes ; are analysed for the statistical accuracy. So I usually make 6 entry boxes A1 , A2 , A3 , B1 , B2 , B3 where A 
& B represents the side from where the measurement is done while the number is experiment run number .

<img width="1088" height="899" alt="image" src="https://github.com/user-attachments/assets/a8f539a3-b83b-4a53-95a4-3b59fff792a0" />

- Also one more trick I use to seperate the part of constant speed before the actual coast down begins which can cause error in the calculation . I introduce a term called as negative slope which
check whether the values of speed are in descending trend , so those constant speed values at the start are omitted from the from the dataframe.

3rd Step : Calculation of the Resistance Force across the Speed range
- Below image clears how the dataframe must be created for the entire speed range .
<img width="948" height="652" alt="image" src="https://github.com/user-attachments/assets/188ad23e-4ef5-436b-9a6d-c69e18d09440" />

- So lets say the time by vehicle to deacclerate from 110 kmh to 105 kmh is x secs and from 105 kmh to 100 kmh is y secs then the average of both these time is calculated . The average or mid interval
of speed that is in this case 105 kph is assigned the average computed time .

- I create dictionary in Python to store the results of dataframe for all the 6 measurements so that is it easy to retrieve and view . For further calculation the time for A1 , A2 and A3 at i.e 105 kmh
is averaged and similar with B1 , B2 and B3 . 

<img width="386" height="115" alt="image" src="https://github.com/user-attachments/assets/b2d37cac-fab1-407f-9b97-87a13fbc6e0e" />

- Resitantance Force A and Force B is then calculated from calculated data for the time from Direction A & B . We take average of Force A & Force B to form Force in the dataframe.  

4th Step : Curve fitting and approximating the values of F0 , F1 and F2 Coefficients

- Then we plot these values of Force against Speed where Force is on the Y axis and Speed on the X axis . Then we use the curve fit command to approximate the coefficients ; for this we make a
polynomial of 2nd order with three coefficients . That is F = F0 + F1 * Speed + F2 * (Speed) ^ 2
- Curve fit command fits the polynomial best to this set of points like a regression analysis which tries to estimate a function for a trend while reducing the deviation.

<img width="1536" height="1024" alt="ChatGPT Image Dec 26, 2025, 08_24_04 PM" src="https://github.com/user-attachments/assets/1ab36796-aa07-46ab-b6da-0e26eb710d92" />

So you would get a plot similar like this and the coefficient values for F0 , F1 and F2 .

---

### Physical meaning

Once this values are computed what does this value define about the area of improvement in the car

- F0 : Losses that are Independent of Speed i.e Bearing losses 
- F1 : Linear relation with Speed i.e Tyre Hystersis
- F2 : Aerodynamic losses

Also it important not compare these values with each other but with Competitor or Benchmark Vehicles of the same segment .



