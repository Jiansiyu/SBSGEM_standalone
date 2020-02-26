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

## Dev Plan
- [x] add the display code used for display the clusters for each GEMs
- [x] add the code to show number of clusters found
- [ ] add the function used for loop on the event and compare with the standard PRex analysis code
- [ ] diagest code used for generate the plot for the cut 
- [ ] add the function used display how many tracks found 
- [ ] add the diagnose code used to automaticly generate suitable cut



