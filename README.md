# branch informations 

WorkBranch_PlotCheck 

The track efficiency seems like pretty low. In this branch, add the event by event display functions used to check each indivadual event.


# SBSGEM_standalone
Standalone ROOT macros for analysis of GEM data.

Requirements: Working ROOT installation; decoded GEM data in standard ROOT Tree format. 

GEM_reconstruct.C: main clustering/hit reconstruction/tracking code; developed for analysis of GEM data from INFN cosmic ray test stand. Takes decoded GEM data as input (strips fired by module and six ADC samples from APV25), reconstructs 2D hits, finds straight-line tracks, produces diagnostic histograms and ROOT Tree. 

GEM_reconstruct_HallAtest.C: same as GEM_reconstruct.C, but specialized for analysis of Hall A GEM test data from 2016. Uses additional information from the calorimeter that was part of the test; computes simple sum of calorimeter ADC values.

GEM_align.C: Alignment code; takes the output of GEM_reconstruct.C as input, determines best set of translational (x0, y0, z0) and rotational (alpha_x, alpha_y, alpha_z) "yaw, pitch, roll" offests to minimize chi-squared of straight line tracks.

plot_GEM

Usage example:

~~~~
root [0] .L GEM_reconstruct.C+
root [1] GEM_reconstruct("../HitData/GEMfixNov18/Hit_run3805_*.root","configINFN.txt","temp.root");
~~~~

General usage: GEM_reconstruct( infilename, configfilename, outfilename );

Here infilename gives the absolute or relative path name from the working directory to the decoded GEM data file in ROOT Tree format.

configfilename gives the name of the text configuration file, which defines the geometry and all relevant parameters for the clustering, hit reconstruction, and tracking.

outfilename is the desired name of the output ROOT file.

Similarly, for the alignment code:

~~~~
root [0] .L GEM_align.C+
root [1] GEM_align(inputfilename, configfilename, outputfilename);
~~~~

where inputfilename is the name of a root file that is the output of GEM_reconstruct containing the relevant reconstructed hit and track information, configfilename is the text configuration file, and outputfilename is the name of a text output file containing the new alignment parameters for each module. The output of GEM_align can be directly copied and pasted into the configuration file for GEM_reconstruct to repeat the tracking with the new alignment parameters.

Example configuration files are:

configINFN.txt: Example configuration file for INFN GEM four layer cosmic test data from Nov. 2018
configHallAtest.txt: Example configuration file for five layer UVA GEM Hall A beam test data from 2016.
configalign.txt: Example alignment configuration file for INFN four-layer cosmic test data.
configalignHallAtest.txt: Example alignment configuration file for five-layer UVA GEM Hall A beam test data 2016.

I have also added two utility macros:

~~~~
plot_GEMrun_summary.C
plot_GEMrun_summary_HallA.C
~~~~

which can be used to generate a PDF file with a nice group of summary plots of the analysis. 

Detailed documentation of configuration parameters is forthcoming. In the mean time, read the source code.

## Issues and Solutions 
* When run
  ~~~~
  root [0] .L GEM_reconstruct.C+
  root [1] GEM_reconstruct("../HitData/GEMfixNov18/Hit_run3805_*.root","configINFN.txt","temp.root");
  ~~~~

  It will crash. It seems like doing the following step solve the issue, Why?
  ~~~~
  root [0] .L GEM_reconstruct.C
  root [1] GEM_reconstruct("../HitData/GEMfixNov18/Hit_run3805_*.root","configINFN.txt","temp.root");
  ~~~~
* many adjacent cluster was split into two clusters?

## Config Instruction 

The configure file is decoded in function 'GEM_reconstruct'. 

```shell_script
#! make sure all the configure should match the nmodules number (larger should be fine)
#! for mod_xxx configure, each row is the configuration for the module 
nlayers 6      #! How many layer of GEM, on each layer, maybe there are Server GEM modules
nmodules 6     # How many GEM modules in total 
#NMAX 200000
#position: ModuleID start from 0, and the connected LayerID
mod_layer  0   1   2   3   4    5        # Layer ID 
# for prex experiment, all the GEMs are aligned to VDC, probabaly the current initial position is not good enough
# X, Y, and Z offsets are given in mm:
# Original offsets:
# need to check where is the reference point
# here take the detector center to be the reference point
# all the modules in same layer need to take same reference point

#! The Initial Position for Each GEMs, 
#   
#  correction   Module-0     Module-1,       Module-2,      Module-3,      Module-4,        Module-5,   etc  
#  x-translation
#  y-translation
#  z-translation 
#  x rotation
#  y rotation
#  z rotation

mod_x0         -9.35341        -5.15471        -3.29799        -140.55        -140.071        -140.981
mod_y0          3.65935         8.20683         10.4202        47.2596        47.3891         54.17
mod_z0            900.0          1585.8          1789.0         2341.35         2443.05          2540.0
mod_ax        	  0.000        	  0.000        	  0.000        	  0.000        	  0.000        	  0.000        
mod_ay        	  0.000        	  0.000        	  0.000        	  0.000        	  0.000        	  0.000
mod_az        	  0.000        	  0.000        	  0.000        	  0.000        	  0.000        	  0.000

# going forward, more of the module geometry is configurable: "u" = X and "v" = Y
mod_nstripsu  512 512 512 1536 1536 1536       # N of APV channels 
mod_nstripsv  256 256 256 1280 1280 1280 
# for the "u = X, v = Y" convention, the planeID = 1, 0, for u, v
mod_uplaneID 0 0 0 0 0 0                       # X or Y dimension as labled 
mod_vplaneID 1 1 1 1 1 1 
mod_ustrip_pitch 0.4 0.4 0.4 0.4 0.4 0.4       # strip pitch 
mod_vstrip_pitch 0.4 0.4 0.4 0.4 0.4 0.4
mod_Lx 204.8 204.8 204.8 614.4 614.4 614.4     # GEM size of each module in mm
mod_Ly 102.4 102.4 102.4 512.0 512.0 512.0
# u strip angle relative to X axis: under u = X, v = Y convention, uangle = 0 and vangle=90
mod_uangle 0.0 0.0 0.0 0.0 0.0 0.0
# Setting the "V" strip angle to -90 degrees amounts to reversing the direction of the Y axis (I hope):
mod_vangle -90.0 -90.0 -90.0 -90.0 -90.0 -90.0 
eventdisplay 0
clustersigma 0.4
clustertau 54.0
varyclustersigma 0
varyclustertau 0
maxstripsperclusterX 11
maxstripsperclusterY 9
# thresholds for "pruning" strips and clusters (not necessarily discarding):
maxcor_threshold  -1.1
stripcor_threshold -1.1
clustcor_threshold -1.1
clust2D_ADCasymcut 1.1
clust2D_dTcut 1000.0
threshold_maxsample 0.0
threshold_stripsum 100.0
threshold_clustersum 0.0
#threshold_maxsample 75.0
#threshold_stripsum  300.0
#threshold_clustersum 300.0
trackchi2cut 1000.0
trackmaxradius 30.0 # mm, max radial separation between hit and projection of track from previous layers to add hit to track.
trackmaxslopeX 1.0
trackmaxslopeY 0.5
maxADCXYthreshold 5000.0
tstripcut_nsigma 5.0
endconfig

```



## Dev Plan
- [x] add the display code used for display the clusters for each GEMs
- [x] add the code to show number of clusters found
- [x] add the function used for loop on the event and compare with the standard PRex analysis code
- [ ] diagest code used for generate the plot for the cut 
- [ ] add the function used display how many tracks found 
- [ ] add the diagnose code used to automaticly generate suitable cut



