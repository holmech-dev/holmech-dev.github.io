---
layout: post
---
This post covers creating a simple proportional controller in c++ to manage the water level in a water tank. 

## The Theory

The target is to manage the volume of the water tank by controlling the mass flow rate ($\dot m$) of the water. The diagram below outlines the closed loop control system used for the program.
<br>

![Test Image](/system.png)

The output of the proportional controller will be $\dot m$, which will be the input to our plant (water tank). The output of the plant will be the volume of water in the tank ($Vol$). To mathematically model the water tank, the below equations can be used. The discrete form will be required for the computer program. 

Continous Form
```math
Vol(t) = Vol_i + {1 \over \rho} \int_{0}^{t} \dot m * dt
```

Discrete Form
$$
Vol[t_i] = Vol[t_i-1] + ({ {1 \over \rho} \ \dot m \ Vol[t_i-1] + {1 \over \rho} \ \dot m \ Vol[t_i] \over 2}) \ \Delta t
$$

The volume output will be compared with the reference value, then the proportional controller will use this to generate a $\dot m$ output value. The controller uses a gain denoted as $K_p$ with the error term to determine the new $\dot m$ value.

## The Program

When executed, the program will request the user to input a reference value, this tells the program what the desired final tank volume should be and gives the reference value to the controller.

The target reference value and start volume value are then printed for confirmation. The start volume is always zero. Two integer variables are created to store these values respectively. ```tankVolumeReference```,```tankVolumeReference```.

```
std::cout << "Please input a reference value: ";
std::cin >>  tankVolumeReference;
std::cout << "\n";
std::cout << "Target Volume Reference at Start -> " << tankVolumeReference << std::endl;
std::cout << "Tank Volume Value at Start -> " << tankVolumeInitial << std::endl;
std::cout << "\n";
```

Before the simulation loop commences, the time length and sample time interval of the program is set. The total length is calculated by dividing the end time by the time interval. For example, for a time lenght of 10 secs with a sample time of 0.04 secs, we will receive 250 samples.

```
double dt = 0.04; // Simulation time interval
int t_end = 10; // Simulation End Time
double tLength = t_end / dt; // Vector length of time points based on interval
```

A for loop is used for the simulation loop using ```tLength```. First, the reference tank volume is retrieved, this is then compared with the actual tank volume to find the error. This error is then multiplied by the controller gain $K_p$ to give the resultant $\dot m$ for the loop iteration $i$.

Then using the discrete water tank model from above, the new tank volume can be calculated. **Note. the program includes water density in the denominator, this allows for simulation of liquids other than water where $\rho$ $\ != 1$ 

Previous $\dot m$ and error values are then updated ready for the next loop iteration.

```
for (int i = 0; i < tLength; i++){

    // Determine reference value vector for the tank
    tankVolumeRefVec[i] = tankVolumeReference;

    // Calculate the error between reference and actual
    errorPrev = tankVolumeRefVec[i] - tankVolumeActualPrev;

    // Calculate the control input, mass flow rate
    mDot[i] = Kp * errorPrev;
    
    // Calculate the true tank volume in the next time step using numerical integration
    tankVolumeActualVec[i] = tankVolumeActualPrev + (mDotPrev + mDot[i]) / (2*waterDensity)*(dt);

    tankVolumeActualPrev = tankVolumeActualVec[i];
    mDotPrev = mDot[i];
}
```

To have data available for analysis, a simple file writer is shown. A file called ```sim_out``` is created with a header ready for population of the simulation data. 

```
std::ofstream sim_out_file;
sim_out_file.open("sim_out.csv");
sim_out_file << "t,reference,error,actual,dt,mdot\n";
```

At the end of each simulation loop, a new row of data is populated inside the file. 

```
sim_out_file << i << "," << tankVolumeRefVec[i] << "," << errorPrev << "," << tankVolumeActualVec[i] << "," << dt << "," << mDotPrev << "\n";
```

## The Analysis

This section is in Python, I use the Pandas library for importing and handling the data. the Bokeh package is used for plotting. 

```
import pandas as pd
from bokeh.io import push_notebook, show, output_notebook
from bokeh.resources import INLINE
from bokeh.models import Label, LabelSet
from bokeh.models.tools import HoverTool
from bokeh.plotting import figure
output_notebook(INLINE)
```

The data file is then imported into a dataframe and the first 11 entries are displayed. 

```
sim_data = pd.read_csv("sim_out.csv", sep=',', skiprows=0)
sim_data.columns = ['index', 'reference', 'error', 'actual', 'dt', 'mdot']
sim_data.head(11)
```

```
index	reference	error	actual	    dt	    mdot
0	80	        80.0000	1.60000	    0.04    80000.0
1	80	        78.4000	4.76800	    0.04    78400.0
2	80	        75.2320	7.84064	    0.04    75232.0
3	80	        72.1594	10.78850    0.04    72159.4
4	80	        69.2115	13.61590    0.04    69211.5
5	80	        66.3841	16.32780    0.04    66384.1
6	80	        63.6722	18.92890    0.04    63672.2
7	80	        61.0711	21.42380    0.04    61071.1
8	80	        58.5762	23.81670    0.04    58576.2
9	80	        56.1833	26.11190    0.04    56183.3
10	80	        53.8881	28.31340    0.04    53888.1
```

Then finally, we can see graph the reference and actual volume values. We can see that the blue reference maintains its value, the user must have input a value of 80. The red line shows the new volume being updated by our controller. When the error reaches zero and the reference value is matched, the $\dot m$ then tends to zero as we would expect. 

![Test Image](/analysis.png)


