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



## Getting started
After downloading the bundle of programs from ASTRA, the programs Generator and Astra are the most important.

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




