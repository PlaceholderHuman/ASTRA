# ASTRA
[ASTRA](https://www.desy.de/~mpyflo/) is a beamline simulation that takes an input distribution and a defined beamline to run it through. 
## Manual
The [manual](https://www.desy.de/~mpyflo/Astra_manual/Astra-Manual_V3.2.pdf) lists everything about it, so any answers will typically be listed in there (there are not many online help resources). The most important parts of the manual are going to be:
* Definition of particle distribution (page 7)
This is for checking formatting of distribution data
* Generator program description (page 9)
This is a guide on initial distributions
* Astra program description (page 11)
This is for solving issues with Astra and how specifically things are defined in the program 
* Input namelists for Astra (page 54)
This is for adding or editing elements of your beamline
* Input namelists for Generator (page 98)
This is for editing the initial distribution
* Types of distributions (page 103)
The different distributions are listed here; use the bold part of the text in the keyword to define inside of Generator. Example Dist_z = '**u**niform' so use Dist_z ='u'


Notes:
* Always remember to leave a blank line or several on the Astra file otherwise it will crash. 
* Cathode = True crashes on Mac.
* The programs use a language called Fortran; use ! to start comment lines


## Getting started
After downloading the bundle of programs from ASTRA, the programs Generator and Astra are the most important. 
Download cavity.txt, distribution.in, and alphacode.in from this github.
Place everything into one folder and check 


## Definitions
### Generator
Generator is a program that takes in a .in file and generates a .ini file. To create a .in file, I just create a blank text file and then change the file end to .in. You give it some specifications and then it will create a distribution of particles based off of those. For example, this is a .in file to generate 1000 particles:
```
&INPUT
 FNAME = 'test.ini'
 Add=FALSE, N_add=0,
 IPart=1000, Species='electrons'
 Probe=True, Noise_reduc=T
 Cathode=F
 Q_total=60e-3
 Ref_zpos=0.0E0, Ref_Ekin=0.7

 Dist_z='u', sig_z=0.6, C_sig_z=1,
 sig_clock=1.5E-3, C_sig_clock=5
 Dist_pz='u', sig_Ekin=50, cor_Ekin=0.0E0,

 Dist_x='u', sig_x=0.578, C_sig_x=3,
 Dist_px='u', Nemit_x=1.0E0, cor_px=0.0E0
 Dist_y='u', sig_y=0.289, C_sig_y=3,
 Dist_py='u', Nemit_y=1.0E0, cor_py=0.0E0
/
```
The file it will generate filled with the data for the distribution is test.ini.   
The adding = FALSE means there will be no additional defined distributions to merge together. If wanted to add multiple distributions together, check add.in file in this github. The N_add will define how many files to add together, and the distributions can be defined in the same .in file.   
The IPart controls how many particles will be generated, and the type (species) are electrons.  
The Probe particles generate spaced out particles which do not really affect distribution unless you are doing very small distributed tests (to my knowledge). Noise reduction I am not sure but I kept it on and it was fine.  
Cathode should be turned on for further tests after getting comfortable as the UH FEL utilizes a thermionic cathode. There will be time-dependancy and specifications will need to be looked at. The Mac version crashes when using cathode, so my distributions had it turned off.
Q_total is the total charge of the electron bunch. Ref_zposition should be at 0. Ref_Ekin is the kinetic energy of the electron in MeV, so it should be around 0.4-1.1 for the alpha magnet tests (I believe 0.7-0.8 would be centered).   
After all of those definitions, the parameters are defined according to the "types of distributions" section in the manual. Each dimension (x,y,z) and its momenta (px,py,pz) have distributions like "u" for uniform or "g" for gaussian.   
Control the spread of each dimension with defined elements for each distribution type (gaussian would have sigma or cut off sigma while plateau would have rise time or length). This would be the C_sig_y=3 or Nemit_x=1.0E0 or sig_x=0.578 everything after the distribution types.  

After finishing the code, open generator ("./generator file.in" for Mac) and run the file. It will create a .ini file that you will plug into Astra.  


### Astra
The program Astra takes a .in file (and a .ini for distribution and .txt for cavity) and will return many different files (according to how you set it up). I have the cavity file and the distribution file in the same folder as the .in and Astra program.
An Astra.in file would look like this: 
```
&NEWRUN
    Run = 1

    Auto_Phase = .T.

    Distribution = 'test.ini'
    !astrabroken.0060.443
    !test.ini
    !Insert relative to parent folder distribution file as string
    !H_max = 1.000000e-03
    
    Head = 'UH FEL'

    Lprompt = .F.
    
    PHASE_SCAN = .T.

    Track_All = .T.

    check_ref_part = .F.
    !Condition to quit program if reference particle lost
    Max_step = 100000
    !H_min = 0.001
    !H_max = 0.01
    Z_min = -1
    !Condition to quit program if reference touches -1 meters
/

&OUTPUT
    !Prints an output file every step_max/step_width
    PhaseS = .F.
    RefS = .T.
    T_PhaseS = .T.
    Step_width = 5
    !How many times to divide Step_max into
    ZSTART = 0
    ZSTOP = 1
    Zemit = 100
    Zphase = 1
    TcheckS = .F.
    TrackS = .F.
    EmitS = .T.
    LandFS = .T.
    Step_max = 1000
    !How many generations before quitting
    Lmagnetized = .F.
    Lsub_cor = .F.
/

&SCAN
    !This is for setting up screens in a beamline
    FOM(10) = 'ver offset'
    FOM(1) = 'charge'
    FOM(2) = 'mean energy'
    FOM(3) = 'rms energy'
    FOM(4) = 'length'
    FOM(5) = 'hor emit'
    FOM(6) = 'hor spot'
    FOM(7) = 'hor offset'
    FOM(8) = 'ver emit'
    FOM(9) = 'ver spot'
    LScan = .F.
    S_max = 0.15
    S_min = 0.05
    S_numb = 61
    Scan_para = 'D_strength(1)'
/

&CHARGE
    Cell_var = 2
    LSPCH = .F.
    !space charge off
    Lmirror = .T.
    Max_scale = 1.000000e-01
    Nlong_in = 20
    Nrad = 20
    min_grid = 4.000000e-07
/


&APERTURE
    LApert = .F.
    !aperture on is true
    File_Aperture(1) = 'Col_X'
    Ap_Z1(1) = -0.001
    Ap_Z2(1) = 0.001
    Ap_R(1) = 8
    A_xrot(1) = -2.28131986526
    A_pos(1) =  0.0874606
    A_xoff(1) = -0.06665015
/



&CAVITY
    C_Smooth(1) = 0
    C_pos(1) = 0
    LEfield = .F.
    MaxE(1) = 33
    Nue(1) = 2.856
    Phi(1) = 0
    File_Efield(1) = 'cavity.txt'
/

&QUADRUPOLE
    !quadrupole off
    Lquad = .F.
    Q_type(1) = 'do'
    Q_xrot(1) = 1.57079632679
    Q_K(1) = 1500
    Q_pos(1) = 0
    Q_length(1) = 0.026
    Q_dist(1) = 0.06985
    Q_xoff(1) = 0.1
    
    !Q_type(2) = 'do'
    !Q_K(2) = -150
    !Q_pos(2) = 0.28
    !Q_length(2) = 0.026
    !Q_dist(2) = 0.06985

/

&DIPOLE
    LDipole = .T.
    D_Type(1) = 'hor'
    D1(1) = (0.000424660,0.01036540)
    D2(1) = (0.038219540,0.04288560)
    D3(1) = (-0.007248640,0.019283290)
    D4(1) = (0.030546230,0.051803490)
    D_strength(1) = -0.007323532325619287
    D_Gap(1,1) = 0.00
    D_Gap(2,1) = 0.00
    D_Type(2) = 'hor'
    D1(2) = (-0.00725230,0.019287530)
    D2(2) = (0.030542580,0.051807730)
    D3(2) = (-0.014921950,0.028201180)
    D4(2) = (0.022872930,0.060721380)
    D_strength(2) = -0.021970596976857864
    D_Gap(1,2) = 0.00
    D_Gap(2,2) = 0.00
    D_Type(3) = 'hor'
    D1(3) = (-0.01492560,0.028205430)
    D2(3) = (0.022869270,0.060725630)
    D3(3) = (-0.022595260,0.037119070)
    D4(3) = (0.015199620,0.069639270)
    D_strength(3) = -0.03661766162809644
    D_Gap(1,3) = 0.00
    D_Gap(2,3) = 0.00
    D_Type(4) = 'hor'
    D1(4) = (-0.022598910,0.037123320)
    D2(4) = (0.015195970,0.069643520)
    D3(4) = (-0.030268560,0.046036960)
    D4(4) = (0.007526310,0.078557160)
    D_strength(4) = -0.05126472627933501
    D_Gap(1,4) = 0.00
    D_Gap(2,4) = 0.00
    D_Type(5) = 'hor'
    D1(5) = (-0.030272210,0.046041210)
    D2(5) = (0.007522660,0.078561410)
    D3(5) = (-0.037941870,0.054954860)
    D4(5) = (-0.000146990,0.087475050)
    D_strength(5) = -0.06591179093057359
    D_Gap(1,5) = 0.00
    D_Gap(2,5) = 0.00
    D_Type(6) = 'hor'
    D1(6) = (-0.037945520,0.05495910)
    D2(6) = (-0.000150640,0.08747930)
    D3(6) = (-0.045615170,0.063872750)
    D4(6) = (-0.00782030,0.096392950)
    D_strength(6) = -0.08055885558181217
    D_Gap(1,6) = 0.00
    D_Gap(2,6) = 0.00
    D_Type(7) = 'hor'
    D1(7) = (-0.045618830,0.063876990)
    D2(7) = (-0.007823950,0.096397190)
    D3(7) = (-0.053288480,0.072790640)
    D4(7) = (-0.01549360,0.105310840)
    D_strength(7) = -0.09520592023305074
    D_Gap(1,7) = 0.00
    D_Gap(2,7) = 0.00
    D_Type(8) = 'hor'
    D1(8) = (-0.053292130,0.072794890)
    D2(8) = (-0.015497260,0.105315090)
    D3(8) = (-0.060961780,0.081708530)
    D4(8) = (-0.023166910,0.114228730)
    D_strength(8) = -0.10985298488428934
    D_Gap(1,8) = 0.00
    D_Gap(2,8) = 0.00
    D_Type(9) = 'hor'
    D1(9) = (-0.061386440,0.081343140)
    D2(9) = (-0.031660140,0.106920820)
    D3(9) = (-0.071800220,0.093445990)
    D4(9) = (-0.042073910,0.119023670)
    D_strength(9) = -0.11717651720990861
    D_Gap(1,9) = 0.00
    D_Gap(2,9) = 0.00
    D_Type(10) = 'hor'
    D3(10) = (-0.000424660,0.00963460)
    D4(10) = (-0.038219540,-0.02288560)
    D1(10) = (-0.008097970,0.01855250)
    D2(10) = (-0.045892840,-0.01396770)
    D_strength(10) = -0.007323532325619287
    D_Gap(1,10) = 0.00
    D_Gap(2,10) = 0.00
    D_Type(11) = 'hor'
    D3(11) = (-0.008101620,0.018556740)
    D4(11) = (-0.04589650,-0.013963460)
    D1(11) = (-0.015771270,0.027470390)
    D2(11) = (-0.053566150,-0.005049810)
    D_strength(11) = -0.021970596976857864
    D_Gap(1,11) = 0.00
    D_Gap(2,11) = 0.00
    D_Type(12) = 'hor'
    D3(12) = (-0.015774930,0.027474630)
    D4(12) = (-0.05356980,-0.005045560)
    D1(12) = (-0.023444580,0.036388280)
    D2(12) = (-0.061239450,0.003868080)
    D_strength(12) = -0.03661766162809644
    D_Gap(1,12) = 0.00
    D_Gap(2,12) = 0.00
    D_Type(13) = 'hor'
    D3(13) = (-0.023448230,0.036392530)
    D4(13) = (-0.061243110,0.003872330)
    D1(13) = (-0.031117880,0.045306170)
    D2(13) = (-0.068912760,0.012785970)
    D_strength(13) = -0.05126472627933501
    D_Gap(1,13) = 0.00
    D_Gap(2,13) = 0.00
    D_Type(14) = 'hor'
    D3(14) = (-0.031121540,0.045310420)
    D4(14) = (-0.068916410,0.012790220)
    D1(14) = (-0.038791190,0.054224060)
    D2(14) = (-0.076586060,0.021703860)
    D_strength(14) = -0.06591179093057359
    D_Gap(1,14) = 0.00
    D_Gap(2,14) = 0.00
    D_Type(15) = 'hor'
    D3(15) = (-0.038794840,0.054228310)
    D4(15) = (-0.076589720,0.021708110)
    D1(15) = (-0.046464490,0.063141960)
    D2(15) = (-0.084259370,0.030621760)
    D_strength(15) = -0.08055885558181217
    D_Gap(1,15) = 0.00
    D_Gap(2,15) = 0.00
    D_Type(16) = 'hor'
    D3(16) = (-0.046468150,0.06314620)
    D4(16) = (-0.084263020,0.0306260)
    D1(16) = (-0.05413780,0.072059850)
    D2(16) = (-0.091932680,0.039539650)
    D_strength(16) = -0.09520592023305074
    D_Gap(1,16) = 0.00
    D_Gap(2,16) = 0.00
    D_Type(17) = 'hor'
    D3(17) = (-0.054141450,0.072064090)
    D4(17) = (-0.091936330,0.039543890)
    D1(17) = (-0.061811110,0.080977740)
    D2(17) = (-0.099605980,0.048457540)
    D_strength(17) = -0.10985298488428934
    D_Gap(1,17) = 0.00
    D_Gap(2,17) = 0.00
    D_Type(18) = 'hor'
    D3(18) = (-0.061386440,0.081343140)
    D4(18) = (-0.091112750,0.055765450)
    D1(18) = (-0.071800220,0.093445990)
    D2(18) = (-0.101526520,0.06786830)
    D_strength(18) = -0.11717651720990861
    D_Gap(1,18) = 0.00
    D_Gap(2,18) = 0.00
/


```
The file is split up into many different namelists (the sections with ALL CAPS text). The manual will define each of these. I recommend starting at page 54 and reading some of the basics of the first namelist NEWRUN. Each namelist controls a specific group of elements or parameters in the beamline. For example, if you wanted to turn off the alpha magnet you would set LDipole = .F. or just delete the entire dipole section.  
NEWRUN controls the basics of setting up what should happen; OUTPUT controls what files to generate and the time-based tracking; SCAN is if you want to do a linear slice in the beam and get any type of data from a certain z (z is the length going through the beamline); CHARGE is mostly for space charge effects which is off for more basic simulations; APERTURE is basically a defined cutoff for a beam (like a wall with a pipe going through); CAVITY is how the beam accelerates (I set this up with Harsh); QUADRUPOLE is for defining focusing magnets; DIPOLE is to bend the beam.  
A key aspect of this code is that it hase time-based tracking rather than the typical linear tracking that you would define with ZSTART and ZSTOP (this is because the alpha magnet curves the beam completely around and linear tracking fails). In OUTPUT, Step_width = 5 and Step_max = 1000 are crucial for obtaining data. Step_width is how many times each step would be split into while Step_max would control how many steps in a run. For example, if I wanted very precise data I would increase the step width for precision and then also increase the step max (because the step width will consume more steps making the tracking end earlier than previous runs).   
The most important namelists will be NEWRUN, OUTPUT, and DIPOLE if making changes to the alpha magnet.  

Running Astra:   
1. Ensure the RUN number (under NEWRUN) is different for each trial
2. Change Step_width Step_max values to suit the run (under OUTPUT)
3. Add elements (under DIPOLE) to define the alphamagnet (already predefined for 8 splittings 12amp)
4. Save file and ensure there are new lines underneath the code (otherwise it will crash)

The output files generated:  
The OUTPUT section (page 57) lists parameters that will control what kind of output files will be generated. Sometimes in the defintion it will say "according to tables 3 and 4..." these tables (page 30-32) show what the files will contain. For example, having parameter LandFS = TRUE will generate a file named project.LandF.run number that contains the particles that were lost and or found in the simulation. The data in this file will have z, number of active particles, charge, number of lost particles, deposited energy, and total energy exchange.  
Along with this, the time-based tracking will generate a file for every step (Step_max/Step_width). In this example file, there will be 200 files (1000/5) along with a log, reference, lost and found, emittance in 3 dimensions, and a cavity scan generated for every run.

### Alpha Magnet
The alpha magnet is a bending magnet that filters out particles with low/high energy above a certain cutoff. Since ASTRA does not have an alpha magnet namelist, a series of dipoles will be used to simulate an alpha magnet. Here are some [notes](https://docs.google.com/document/d/1pKsXQAKB06jLlrmyTY50P4nL7XAqHx2SgLA-vxI9VvE/edit?usp=sharing) on it. The key paper used to formulate the alpha magnet is [here](https://proceedings.jacow.org/FEL2014/papers/thp023.pdf). The jupyter notebook code to generate the alphamagnets is attached (as rotated dipoles). 

