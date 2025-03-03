# EyeHandPrimingMRI
Example fMRI pipeline (eye/hand priming)

This is a guide to running preprocessing of fMRI data. All scripts you will need are in the folder ProcessingScriptsMRI. These scripts use MATLAB, AFNI, SUMA, and Freesurfer. 

Set up AFNI & SUMA on Mac: https://gist.github.com/paulgribble/7291469.

Set up AFNI & SUMA on Ubuntu (Linux System): http://neuro.debian.net/pkgs/afni.html

Very nice video tutorials can also be found on Andrew Jahn's YouTube page: https://www.youtube.com/playlist?list=PLIQIswOrUH6-v5EWwFdMsTZttt4407KW9
He walks through processing a single subject and getting started with/installing AFNI. 
This tutorial has many differences because he processes data with afni_proc.py, however, I've modified some of the steps and have had cleaner and more robust results.

To download Freesurfer, visit http://freesurfer.net/fswiki/DownloadAndInstall.

PROLOGUE. A QUICK NOTE ON SHELL SCRIPTS FOR BEGINNERS

1. These scripts use the C shell derivative (tcsh). The following line must be the first line in any script: #!/bin/tcsh -xef

2. To execute a script, open a terminal, cd to the folder containing the script, then type 'tcsh', then the script name. E.g., tcsh DICOMto3d.sh. 
N.B. You must be in the folder containing the script to run it, or define the path in the terminal before this command.

3. Define variables according to the following convention: set $variableName = name

4. Note that once a variable is defined, it is referenced using the $ sign, then the variable name as above; however,
in some cases, brackets will also need to bound variables (e.g., ${variableName}) for proper separation. It's a bit tricky,
and if you don't want to figure out when it's OK, you can use brackets all the time.

5. The \ sign after a line of code tells the script to advance to the next line within a particular command. Without it the script will stop and fail!

6. When in doubt, Google! Information on all AFNI commands can be found on the help page, or type the command name in the terminal

7. Before starting, modify your bash profile in the terminal. To do so, type nano ~/.bash_profile into the terminal (nano ~/.bashrc for linux).
You want to add afni/freesurfer/python binaries to your path so they can be called in the scripts.
Mine looks as follows, but this may be different depending on your machine:

PATH="/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:$PATH"

export PATH=~/Documents/projects/DistractorSaliencyMRI/ProcessingScriptsMRI/Top$

export PATH=~/abin:$PATH

export PATH=/usr/lib/afni:$PATH
export PATH=/usr/lib/afni/bin:$PATH
export PATH=/usr/share/afni/atlases/:$PATH
export PATH=/home/jdmccart/Documents/suma_MNI_N27/:$PATH
export DYLD_FALLBACK_LIBRARY_PATH=/usr/lib/afni
export PYTHONPATH=/usr/local/lib/python2.7/site-packages

export FREESURFER_HOME=/usr/local/freesurfer
source $FREESURFER_HOME/SetUpFreeSurfer.sh

export SUBJECTS_DIR=/usr/local/freesurfer/subjects

8. To view data in AFNI, simply cd to the folder containing the files and type 'afni' in the terminal.
To  view AFNI & SUMA data, type 'afni -niml & suma -spec $SubjectSumaFolder/$subj_$hemi.spec -sv $SubjectSumaFolder/$subj_$surfVol'
For SUMA only, just type suma -spec $SubjectSumaFolder/$subj_$hemi.spec -sv $SubjectSumaFolder/$subj_$surfVol

========== PART I. PREPROCESSING ==========

STEP 1. CONVERT DICOMs TO NIFTI AND/OR BRIK

1. Move the DICOM folders to your subject directory ~/$subjectID/DICOM/

2. Open a Terminal window

3. cd to the subject DICOM directory containing the UNIX Executable Files (usually a couple folders beyond the DICOM folder, all in numbers; 
N.B. some subjects may have more than one folder of executables. Move all files to the same folder before continuing.

3. Run mcverter on all files (more info and download: https://lcni.uoregon.edu/downloads/mriconvert/mriconvert-and-mcverter).

N.B. Other programs do the same thing (e.g., dcm2nii), but I prefer mcverter because it outputs a text file 
with the scanning parameters (great when it comes time to write your methods section).

3a. To run on all subjects, modify and run the script “DICOMto3dStep1.sh”

4. This will output new .nii files. One for the MPRAGE, 3 for the Localizer, and the rest are the EPIs (should match the number of runs for that subject).

5. Rename the anatomical and functional files: e.g., $subjectID.MPRAGE.nii & $subjectID.EPI.runN. You can delete the Localizer files.

6. Make a new folder called ‘NIFTI’ in the subject directory (this has been created if you ran 'DICOMto3dStep1.sh') and move these files there: ~/$ExperimentID/$subjectID/NIFTI/

Optional (I tend to include this within the preprocessing script)

If you want AFNI formatted BRIKs instead of NIFTI files, use 3dcopy (e.g., 3dcopy $subjectID.EPI.Run1.nii subjectID.EPI.Run1+orig)

NOTE: From here on out, all the scripts should be ready to go except changing the subject names, run numbers, and number of run TRs 
(however you've decided to code them in your experiment script) as well as the directory structure. 
Any additional changes will be included in the instructions. 

STEP 2. MAKE THE INDIVIDUAL SURFACES (THIS TAKES AROUND 6 HRS PER SUBJECT; I RECOMMEND SETTING IT UP TO RUN OVER THE WEEKEND AND NOT DOING IT SERIALLY. I.E. DO ONE FOR EACH SUBJECT)

1. Run SurfaceReconAllStep2.sh

STEP 3. MAKE A SUMA FOLDER FOR THE SURFACES

1. Convert into something AFNI/SUMA can read. Run “MakeSpecFileStep3.sh” to convert and align the surfaces. This script will also move the SUMA folder into your experiment directory. 

2. Check that the alignment is excellent. Go to your subject’s SUMA folder and run: afni -niml & suma –spec $subjectID_lh.spec –sv $subjectID_SurfVol+orig &

2a. When SUMA and AFNI are open, press ’t’ in the SUMA window to make it talk to AFNI. This will bring up boundaries in AFNI corresponding to the surface. 
It should look like the gray matter has been outlined with boundaries along the pial and white matter edges. Do the same thing for the right hemisphere.

STEP 4. STRIP THE SKULL

1. If alignment looks good, run “SkullStripHiResStep4.sh” to create a skull stripped version of the surface volume that AFNI can read.

2. Check the skull stripping. It may look like some of the cortex is removed in comparison to the SUMA volume but that’s why we created surfaces! 
A surface-based analysis will include all the voxels on the suma surface that may be excluded by AFNI.

STEP 5. PREPROCESS THE FUNCTIONAL DATA

1. Run the script ‘preprocessFunctStep5.sh’. This de-obliques the functional data, optionally de-spikes the data to smooth out potential quick head movements, 
does time-slice correction (not necessary for the scanner at Brown), and performs motion correction (volume registration).

2. Check the motion parameters plot that comes up for each subject. There’s some discrepancy as to when a subject should be rejected, but I’ve seen 2x voxel-size as a standard (4 mm for the new scanner at Brown). If this results in losing a lot of data, however, other things can be done.

3. The outcount plots will show you time points that contain outliers (highly correlated with head motion). 
A breakdown of outlying timepoints in each run can be found in the $subj.allruns.outlierTRs.txt file in the newly created EPI folder (N.B. text index starts at 1, afni starts at 0, correct accordingly)
A warning file indicating whether there are outliers at the beginning of any run is shown in $subj.preSteadyStateOutlierWarning.txt. 
These can be used later to despike the data if necessary by censoring outlying time points (better than the global despike option above).

STEP 6. PREPROCESS THE ANATOMICAL AND ALIGN WITH THE FUNCTIONAL DATA

1. Run the script “preprocessAnat.sh”

2. cd to the subject's EPI folder and run AFNI to check the alignment between the EPI ($subj.$runN.tcat.warp.volreg+orig) and the anatomical ($subj.NoSkull.SurfVol.Alnd.Exp+orig) images

STEP 7. PREP THE DATA FOR THE GLM

1. Run the script ‘GLMprep’ to perform smoothing and normalization (so data can be expressed in terms of % signal change instead of beta weights)

1a. For smoothing in ‘3dmerge’ it is common to set this value to at least 2x your voxel size (many papers use 6-8 mm smoothing kernels).

STEP 8. MAKE STIM TIMING FILES

1. Before running the GLM, make stim files from our behavioral data to know when the different conditions were presented. 

2. This is done in MATLAB using the script 'GetStimTimesStep8'

3. This generates local stim times (according to the start of each run) to be used in the GLM based on sorting each condition in the behavioral data files.

========== PART II. SINGLE SUBJECT ANALYSIS ==========

STEP 1. RUN THE GLM

1. cd to the 'SingleSubjectAnalysisScriptsMRI' folder.

2. There are options for both volume- (volGLM) and surface-based (surfGLM) GLMs

3. You can run GLMs for all planned comparisons for your stim time files. 
You will need the motion parameters file created in the 'preprocessFunctStep5' script and the chosen stim time files.

4. Make sure to indicate which runs will be used for the GLM with the -input option. The '*' option flags to use all runs. 
Each run must be indicated separately if a bad run was removed due to motion or other factors.

5. Change the -CENSORTR option to eliminate the TRs at the beginning of the run you used as your baseline period. E.g., -CENSORTR '*:0-1' censors the 1st 2 TRs of all runs.

6. The first 6 stim files are for your motion parameters. How many you choose will be based on the number of conditions in your experiment. 
This script is currently set up for a 2x2 design (Factor 1: Eye/Hand, Factor 2: Color Repeat/Switch) with an event-related design, indicated by the 'GAM' option. 
For other types of designs (e.g., block) see the 3dDeconvolve help document.

7. Define the general linear tests you'd like to perform on each subject with the -gltsym option. 
N.B. if you want to compare a condition against all others, multiply that condition by the number of conditions being subtracted (if greater than 1).
E.g., -gltsym 'SYM: 2*EyeColorSwitch -HandColorRepeat -HandColorSwitch' 

8. You can change the number of cores used for this step with the '-jobs' option. This can make a big difference in the time it takes to run the script if you have a lot of processing power on your machine. 

========== PART III. GROUP WHOLE-BRAIN ANALYSIS ==========

STEP 1. RUN THE ANOVA FOR THE WHOLE-BRAIN GLM

1. Run the volGroupANOVA or surfGroupANOVA script in the 'GroupAnalysisScripts' folder. Read the script for an explanation of how to set up the ANOVA for your experiment. For more info, read the help documentation.
These will also give you the individual t-test comparisons.

N.B. For the whole brain analysis, use the blurred dataset (blur8).

STEP 2. RUN A MONTE CARLO SIMULATION TO DETERMINE YOUR SIGNIFICANT CLUSTER SIZE THRESHOLD

1. Run the '3dClusterSimulation' script to determine the voxel cluster size that would be unlikely to occur due to chance for the number of voxels in your ANOVA mask at your level of spatial smoothing (-fwhm option).

2. This will output a text file that shows the likelihood a cluster exceeding the given size would be unlikely to occur due to the given chance levels (top row, alpha) at a given p-level (left column).

3. Using this table, you can use the 'clusterize' option in AFNI when viewing the ANOVA output and set the appropriate cluster size for a given p-value.
After this correction, the whole-brain map will reflect 'real' significant clusters based on that significance level.

At this point, you have whole-brain group data!

========== PART IV. SINGLE SUBJECT ROI DEFINITION ==========

Regions of interest (ROIs) can be defined either functionally or anatomically. This tutorial only covers anatomical definition (functional and retinotopic methods soon to come).
Fortunately, recent advances have allowed for single-subject anatomical definition using Freesurfer parcellation methods. These methods allow for maps of visual topography (Wang et al. 2015),
ventral visual areas for face, place, and character-selective regions (Weiner et al. 2016), and a multi-modal map of 180 ROIs per hemisphere.

STEP 1. You will first need to install a few scripts and the atlases. A set of Python scripts created by my good friend Peter Kohler will make this process a breeze.
They can be downloaded here: https://github.com/pjkohler/MRI. Once you have these scripts, I recommend adding the MRI-master folder that was downloaded to your bash profile path so they are always accessible.
If not, you will need to indicate the folder containing the script before issuing a command. The atlases can be found elsewhere:

Wang et al (2016): http://www.princeton.edu/~napl/vtpm.htm

Glasser et al (2016): https://balsa.wustl.edu/study/show/RVVG

STEP 2. To generate the ROIs, run the script 'wangGlasserROIs.sh' in the 'SingleSubjectAnalysisMRI' folder. N.B. You may need to include more options for the mriWangRois.py command depending on your file structure. 
Type the command in the terminal window to see the help description. This takes ~45 mins per subject and generates wang_atlas and glass_atlas folders in the subject's Freesurfer folder.

To view an atlas in SUMA, go to the folder and type the following in the terminal: suma -spec $SUBJECTS_DIR/$subj/SUMA/$subj_$hemi.spec -sv $SUBJECTS_DIR/$subj/SUMA/$subj_NoSkull.SurfVol+orig

In SUMA, go to View > Object Controller to bring up the surface controller, then click on Load Dset and load the appropriate hemispheric atlas.

To separate out the ROIs, click on the Cmp color menu and change the colorbar to FreeSurfer. Now the ROIs should be easily distinguishable.

The labels corresponding to the Wang atlas can be found in the ROIfiles_Labeling.txt file in the ProbAtlas_v4 file.

The labels for the Glasser atlas can be found on pgs. 81-85 of the article's supplementary material.

STEP 3. After you have your ROIs, you can extract the beta value for each voxel within the ROI mask by running 'getBetaWeightsWangSurf'

N.B. When extracting beta weights, remember to use the unblurred GLM dataset (blur0)

STEP 4. Now you have text files for the beta values for each voxel of each ROI. Next, we want to take the mean of those voxels.

N.B. It is wise to threshold which voxels you include in the mean based on the critical F-value. 
This ensures that you are only analyzing significant voxels within a given ROI.
I recommend p < .05 at the very minimum, but you can be more strict and see how it affects your results.
It's often a trade-off because too high of a threshold will result in no significant voxels being extracted (none passed threshold).

STEP 5. Now you have the mean beta values for each ROI! Run the script 'plotBetaWeightROIs' in the GroupAnalysisScriptsMRI > ROI folder. 
This will generate bar graphs with standard error for your conditions and indicate significant effects. 

You now have results for your ROIs of interest!





