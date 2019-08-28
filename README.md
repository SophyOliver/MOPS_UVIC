# MOPS_UVIC
Code for Muhammad Kasim

This repository contains all (except Petsc) files necessary to run the MOPS ocean biogeochemical model with the UVIC ocean circulation TMM (transport) files.

1) Install PETSc (http://www.mcs.anl.gov/petsc/). The TMM driver code is compatible 
with PETSc version 3.6.x.

2) Download all files in this git hub repository

3) Edit runscript (in Model_Run_Example) to include necessary job submission text

4) Edit ruscript to export PETSC_DIR=LOCATION_TO_YOUR_PETSC_FOLDER/petsc-3.6.4

5) Edit the runscript (line just below "export PETSC_DIR ...") to use mpirun or aprun (whatever your cluster uses), and specify the number of cores you have available.

6) Edit the runscript (at 2 locations) according to how much memory you have:

  6.1) -write_steps 73000 \
  
        One year of the model equates to 730 timesteps. This line above makes the model write outputs for every 100 years of the 3000 year model run (each time it writes out it appends the 18 files like po4.petsc and po4avg.petsc with another column). This is very useful for us (me and my supervisor), but not so much for what you are doing, as you will only be calculating the misfit/cost fuction at the final (3000) year. Every time you run the model, you will write out this amount of data:
        
        -write_steps 73000 (desirable)          = every 100 yrs   = 21MB per output file    = 378MB for all 18 output files
        -write_steps 365000 (acceptable)        = every 500 yrs   = 4.2MB per output file   = 75.6MB for all 18 output files
        -write_steps 730000 (just ok)           = every 1000 yrs  = 2.1MB per output file   = 37.8MB for all 18 output files
        -write_steps 2190000 (preferably not!)  = every 3000 yrs  = 0.7MB per output file   = 12.6MB for all 18 output files
        
So depending on how many samples you will run of the MOPS model, you will need to choose which option's data output you can handle/store, then edit -write_steps accordingly.

  6.2) -time_avg -avg_start_time_step 72271 -avg_time_steps 730 -avg_start_time_step_reset_freq 73000 \
  
        This controls the annual-averaged outputs like po4avg.petsc (which will be used to compare to observations), and needs to be edited depending on which -write_steps option you choose:
        
        -write_steps 73000 \
        -time_avg -avg_start_time_step 72271 -avg_time_steps 730 -avg_start_time_step_reset_freq 73000 \
        
        -write_steps 365000
        -time_avg -avg_start_time_step 364271 -avg_time_steps 730 -avg_start_time_step_reset_freq 365000 \
        
        -write_steps 730000
        -time_avg -avg_start_time_step 729271 -avg_time_steps 730 -avg_start_time_step_reset_freq 730000 \
        
        -write_steps 2190000
        -time_avg -avg_start_time_step 2189271 -avg_time_steps 730 -avg_start_time_step_reset_freq 2190000 \
        
6) In MASTER_CODE at the terminal, type "make clean", then "make mops" to recompile the mops executable.

7) The 6 parameters to be optimised are specified within the input file 'parameters_input.txt' (in Model_Run_Example). The parameter names are specified in MASTER_FILES/parameters_names.txt (don't change), and number of parameters are specified in MASTER_FILES/num_bgc_params.txt (don't change). The paramater bounds for these 6 parameters are as follows:

Parameter name: Lower Bound:  Upper Bound
ro2ut           150           200
ACik            4.0           48
ACkpo4          0.0001        0.5
ACmuzoo         0.1           4.0
AComniz         0             10
detmartin       0.4           1.8

8) Run the MOPS model by submitting 'runscript' to the cluster. 'runscript_commented' is well commented for further information. All messages are logged in the 'log' file.

9) The 3 output data files to be used in the misfit/cost function are:

      'po4avg.petsc' - this is modelled phosphate
      'oxyavg.petsc' - this is modelled oxygen
      'no3avg.petsc' - this is modelled nitrate. 
      
The final columns of these 3 files are to be used in the misfit calculation. The final columns can be read in MATLAB like this:

      data = readPetscBinVec('po4avg.petsc',1,-1);

10) The 3 observational files of real ocean data, to be used in the misfit/cost function are:

      'woa18_0_0_p_UVic.petsc' - this is observed phosphate
      'woa18_0_0_o_UVic.petsc' - this is observed oxygen
      'woa18_0_0_n_UVic.petsc' - this is observed nitrate
      
These are single vectors, to be compared to the final column vectors of the 'po4/oxy/no3avg.petsc' files using your choice of root mean square error.

11) Good luck :)

