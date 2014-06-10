EPR-clocked
===========

A somewhat more complicated but still simple enough simulation of a clocked EPR experiment violating the CHSH. This simulation was inspired by repeated claims in the literature that it is impossible to simulate QM correlations 
classically:  

For example:  
	"Considering a generalization of usual Bell scenarios where external quantum inputs are provided to the parties, we show that any entangled quantum state exhibits correlations that cannot be simulated using only shared randomness and classical communication, even when the amount and rounds of classical communication involved are unrestricted." -- Denis Rosset et al 2013 New J. Phys. 15 053025 doi:10.1088/1367-2630/15/5/053025
	
	"it is impossible to write a local realist computer simulation of a *clocked* experiment with no "non-detections", and which reliably reproduces the singlet correlations? (By reliably, I mean in the situation that the settings are not in your control but are delivered to you from outside; the number of runs is large; and that this computer program does this not just once in a blue moon, by luck, but most times it is run on different people's computers.)" -- Richard Gill
	

How it works:
------------
Based on my previous epr-simple simulation with some tweaks (more on this later):  

        - `p`, and `p'` are now instrument random variables using a very similar 
        ½ sin²t, t ∈ [0..π/6) distribution. Separately for each station
        - All particles emitted by the source are now detected. No zero outcomes.
        - A(a,λ) = sign(-1ⁿ cos n(a − e)), B(b,λ') = sign(-1ⁿ cos n(b − e'))

Running it:
-----------
There are 3 programs in the distribution.  
        
1) 'source.py':  

        Usage:
	         python source.py <spin> <duration in seconds>

Generates two files containing the emitted particles, one for each arm of the experiment.
Each file is a gzipped binary numpy array file containing 4 columns with the rows representing a the 
parameters of a single emitted particle. The files are named `SrcLeft.npy.gz` and `SrcRight.npy.gz`.
The columns are:  

        'emission-time'
        'hidden-variable 1'
        'hidden-variable 2'
        'hidden-variable 3'
        'hidden-variable 4'
        
The source program takes as input, 2 parameters:

        `spin` : (either 0.5 or 1.0) Whether to generate spin-½ or spin-1 particles
        `duration`: (in seconds), determines how long the source will run, and how many 
          particles pairs will be generated. The number of particles in each file is printed at the end.

NOTE, VERY IMPORTANT: 0.01% of particles are emitted as singles instead of pairs so that 
there will be no counterpart in the other file. Other 
than that, the particles are recorded sequentially in time. This means it won't 
work to simply assume that the nth particle in one file is a pair with the nth 
particle of the other file. This also means 99.9% of emitted particles are paired. 
Removing this *feature* will render this simulation meaningless. However, 99.999% should 
work just as well. Provided the simulation run for a long enough time.

NOTE: This only affects the source, the stations will still detect every emitted particle,
ie 100% detection efficiency. However, this *feature* makes it necessary to do matching during 
data analysis just as it is done in every real EPR-experiment, and takes away the need
to assume that the particles have already been pre-matched for us, which is a common mistake
in discusions surrounding Bell's theorem and experiments.


2) `station.py`:  
        
        Usage:
	         python station.py <ArmSrcFile> <StationName> [seting1,setting2,setting3,...]

Reads one of the source files generated by `source.py` and generates another file
containing the outcomes. The same program is run on both arms just with a different
source file. The input parameters are:  

        `ArmSrcFile`: one of `SrcLeft.npy.gz` or `SrcRight.npy.gz`
        `StationName`: The name of the station, e.g. "Alice" or "Bob"
        `settings sequence`: optional comma separated list of settings to randomly chose between
           for example 0,22.5,45,67.5 (no spaces between). if no sequence is provided, a sequence of
           33 evenly spaced angles between 0 and 2π will be generated and used.

The output file will be named based on the `StationName` provided (like `<StationName>.npy.gz`). The format is also 
a gzipped binary numpy array file containing 3 columns with the rows representing a detected 
outcome. The columns are:  
    
        `detection-time`: The time the detector fired
        `active-setting`: The setting that was active at the time
        `outcome`: +1 or -1

NOTE: There will be as as many rows in the output file as there were in the corresponding source file. 
Since all particles are detected. In other words, every particle which reaches the station is detected (100%).

3) `analyse.py`:  
        
        Usage: 
	         analyse.py <spin>


Although not required to run and test the simulation, this is an analysis program which does
the analysis in a manner similar to how it is done in typical EPR experiments. It borrows some 
matching algorithms from Jan-Åke Larsson's BellTiming code (http://people.isy.liu.se/jalar/belltiming/). 

The analysis program takes a single input parameter, which is the `spin` of the particles being simulated.
Tt expects to find two files named `Alice.npy.gz` and `Bob.npy.gz`. It prints some 
Statistics and the correlations, and also creates a plot for the measured angles.

Of course anyone is free to write their own analysis program.

To convert .npy.gz files to text, use the following commands:  
        
        python
        >>> import numpy
        >>> import gzip
        >>> a = numpy.load(gzip.open("Alice.npy.gz","rb"))
        >>> numpy.savetxt("Alice.txt", a)


Results:
--------

For a 120 second simulation of a spin-1 source (photons), I get the following results:  

        No. of detected particles, non-zero outcomes only
	        Alice:         2954402
	          Bob:         2954381


        Calculation of expectation values
          Settings       N_ab     Trials   <AB>_sim    <AB>_qm StdErr_sim
           0, 22.5       1837       2240      0.725      0.707      0.017
           0, 67.5       1806       2204     -0.734     -0.707      0.017
          45, 22.5       1780       2216      0.713      0.707      0.017
          45, 67.5       1748       2164      0.717      0.707      0.017

	        Same Angle <AB> = +1.00
	        Oppo Angle <AB> = -1.00
	        CHSH: <= 2.0, Sim: 2.890, QM: 2.828

        Statistics of residuals between exact QM curve and Simulation
              Skew:          0.6885
             Range: -0.01132 : 0.01624
            Length:              33
          Variance:       4.176e-05
          Kurtosis:          0.2354
              Mean:        0.000678


A 240s simulation for a spin-1/2 source (electrons), I get the following results:  

        No. of detected particles, non-zero outcomes only
	        Alice:         5432125
	          Bob:         5432142


        Calculation of expectation values
          Settings       N_ab     Trials   <AB>_sim    <AB>_qm StdErr_sim
           0, 22.5       3576       4228     -0.921     -0.924      0.015
           0, 67.5       3357       4163     -0.391     -0.383      0.007
          45, 22.5       3623       4319     -0.919     -0.924      0.015
          45, 67.5       3488       4153     -0.927     -0.924      0.016

	        Same Angle <AB> = -1.00
	        Oppo Angle <AB> = -0.00
	        CHSH: <= 2.0, Sim: 2.376, QM: 2.389

        Statistics of residuals between exact QM curve and Simulation
              Skew:          0.2406
             Range: -0.01207 : 0.01065
            Length:              33
          Variance:       2.667e-05
          Kurtosis:          0.2961
              Mean:      -0.0002568

The files `analysis-spin-1.png` and `analysis-spin-0.5.png` show the curves. 

Other comments:
---------------

This simulation is then equivalent to a networked version. The source files can be copied to different computers disconnected from each other at a long distance apart before running the station program to generate outputs.
