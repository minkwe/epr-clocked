EPR-clocked
===========

A somewhat more complicated but still simple enough simulation of a clocked EPR experiment violating the CHSH.

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
        
The source program takes as input, 2 parameters:

        `spin` : (either 0.5 or 1.0) Whether to generate spin-½ or spin-1 particles
        `duration`: (in seconds), determines how long the source will run, and how many 
          particles pairs will be generated. The number of particles in each file is printed at the end.

NOTE, VERY IMPORTANT: 0.01% of particles emitted are randomly discarded so that 
for 0.01% of particles, there will be no counterpart in the other file. Other 
than that, the particles are recorded sequentially in time. This means it won't 
work to simply assume that the nth particle in one file is a pair with the nth 
particle of the other file. This also means 99.9% of emitted particles are paired.


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
Although not required to run and test the simulation, this is an analysis program which does
the analysis in a manner similar to how it is done in typical EPR experiments. It borrows some matching algorithms from Jan-Åke Larsson's BellTiming code (Jan-Åke Larsson, http://people.isy.liu.se/jalar/belltiming/). At some point, I might rewrite the matching algorithm but this suffices for now.

The analysis program does not take any inputs. Instead it expects to find two files named `Alice.npy.gz` and `Bob.npy.gz`. It prints some 
Statistics.

Of course anyone is free to write their own analysis program. **Note however, that you can not avoid matching since, a few of the particles do not have counterparts and there is no way to know which ones without a matching algorithm.** To convert .npy.gz files to text, use the following commands:  
        
        python
        >>> import numpy
        >>> import gzip
        >>> a = numpy.load(gzip.open("Alice.npy.gz","rb"))
        >>> numpy.savetxt("Alice.txt", a)


Results:
--------

For a 120second simulation of the source, I get the following results:  

        No. of detected particles, non-zero outcomes only
	        Alice:         2654386
	          Bob:         2654381


        Calculation of expectation values
          Settings       N_ab     Trials   <AB>_sim    <AB>_qm
           0, 22.5       1646       2034      0.738      0.707
           0, 67.5       1599       1963     -0.739     -0.707
          45, 22.5       1632       2012      0.714      0.707
          45, 67.5       1615       2024      0.714      0.707

	        Same Angle <AB> = +1.00
	        Oppo Angle <AB> = -1.00
	        CHSH: <= 2.0, Sim: 2.905, QM: 2.828

        Statistics of residuals between exact QM curve and Simulation
              Skew:         -0.4193
             Range: -0.01045 : 0.007545
            Length:              32
          Variance:        1.76e-05
          Kurtosis:        -0.08083
              Mean:        0.000445


The file `analysis.png` shows the curves. You might not even notice that there are two curves because the fit is good.