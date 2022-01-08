# eLife2021
Contains MATLAB 2018B analysis files used in Hamilos et al., _eLife_, 2021 (https://doi.org/10.7554/eLife.62583) with a sample dataset, instructions, all necessary custom dependencies and all original single session datasets available via Zenodo (https://doi.org/10.5281/zenodo.4062749).

Last Update on Jan 8, 2021. Please contact ahamilos{at}mit.edu if you have questions or something isn't working. 

  > For some example pre-processed datasets to use with the sample code, please see:
  > https://www.dropbox.com/sh/wdotym743hmo4jc/AABKfTVxkH2JVkZXJ-7UpLyCa?dl=0
  >
  > For all original datasets, see our Zenodo repository at:
  > https://doi.org/10.5281/zenodo.4062749
  >
  > (See section 0. Importing repo code and working with the Zenodo dataset repository)

-------------------------------------------------
INSTRUCTIONS 

Contents:
  
  0. Importing repo code and working with the Zenodo dataset repository
  1. Instructions for generating new analysis objects from raw single-session datasets
  2. Loading a sample session analysis object
  3. Code to reproduce figures from Hamilos et al., 2021
  
-------------------------------------------------
0. Importing repo code (designed for Windows 7 and MacOS 10.14)

    1. Clone or download the HSOManalysisPackages repository and add the files and subfolders to the path in MATLAB 2018B or higher
    2. Make sure the following toolboxes are enabled in MATLAB: 
    
          - Curve Fitting Toolbox v3.5.8+
          - Statistics and Machine Learning Toolbox v11.4+
          - Signal Processing Toolbox v8.1+
          - Optimization Toolbox v8.2+
          - Image Processing Toolbox v10.3+
          - DSP System Toolbox v9.7+
          - Control System Toolbox v10.5+
          
     3. Download the single session dataset repository from Zenodo. The zip file is ~57.6 GB. Extract the files. 
     4. Once the repository has been extracted, the data can be accessed in one of two ways:
     
     I. Each experiment is divided by directory, and following the directory path will terminate with single session directories marked with the mouse's name and session number (MOUSENAME_SIGNALNAME_SESSION#). These single session files may be imported directly into MATLAB for use as desired; each signal is its own timeseries variable with timestamps as well as voltage values for analog signals. All times are measured in seconds. This is the best option for someone desiring to write new analyses from scratch.
     
     II. For the user wishing to replicate analyses with the HSOM Analysis Suite, we have arranged datasets in the repository such that HSOM can automatically parse the files and create both single-session and composite-session analysis objects. To do so, one should open the "Dataset Map.txt" file from the base directory, which indicates which single session folders contain each type of data (e.g., SNc GCaMP6f photometry signals, DLS dLight1.1 photometry signals, etc). If storage space is not a barrier, we recommend copying the relevant single session directories from the repository folder into a HOST folder for each analysis, as follows:
     
     Suppose we wish to analyze GCaMP6f data recorded at SNc. We would first consult the "Dataset Map.txt" file and find all session folders containing SNc GCaMP6f signals that are nested under "\Single Session Data\Photometry\GCaMP6f and tdTomato Photometry and Behavior" (single sessions in this folder may have SNc, VTA and/or DLS data in the same file). Of course, there are many session folders fitting this description, but let's say for sake of demonstration we are interested in creating an analysis object to look at SNc GCaMP6f data from mouse H6 on days 5, 7, and 9. The Dataset Map will show the original folder names (H6_signalname_5, H6_signalname_7, H6_signalname_9). Copy these folders to a new directory named SNcHOST and change the "signalname" string in the foldername to "SNc" (the proper signal designation codes for all signals are found in the Dataset Map.txt file). Your analysis directory should be:
     
     ~/SNcHOST/, containing:
     
        ~/H6_SNc_5/
        ~/H6_SNc_7/
        ~/H6_SNc_9/
     
     You are now ready to run the automated HSOM analysis package on these files to conduct a wide variety of analyses from plotting to model fitting.
     
     NB: The single session folders each contain a text file with the word "exclusions" in the filename. This text file contains rare excluded trial numbers (if any). The user may decide to run analyses with or without exclusions. If no exclusions are desired, simply save an empty text file as "null_exclusions.txt" to the single session directory. Note that only one exclusions file may be present in each session's directory.
         
-------------------------------------------------

1. To generate new analysis objects from single session datasets (available on Zenodo):

    i. CED/Spike2 datasets have been saved to each single session directory (MOUSENAME_signalname_SESSION#) as .mat files. If you are saving a new file with Spike2 from your own experiment, ensure that the spike2 filename has the same name as the .mat file to avoid errors. Ensure waveforms and times are saved and use channel names.
  
    ii. The single session datasets on Zenodo are arranged into directories based on experiment for automatic collation into both single-session and composite analysis objects, as described in Section 0, above:
    
    
Put raw .mat files into directories in the following way:

    HOSTdirectory >
    
                MOUSENAME_SIGNAL_DAY#
                e.g.:
                - B5_SNc_13
                - B5_SNc_15

Each session folder must contain a CED .mat file and an exclusions text file (.txt) with a filename containing the word "exclusions." Exclusions (for trials) may be written as numbers delimited by any character EXCEPT a dash (-). A dash character denotes a range of trials (e.g., 4-6 = 4,5,6). Any non-existant trials will not affect the file, e.g., excluding trials 400-1000 on a session with 420 trials will be interpreted as 400-420.

For a standard photometry object, use:
    
        obj = CLASS_HSOM_photometry('v3x', MODE, NBINS, {'GFITMODE', GFITPARAM}, NSAMPLEBUFFER, [], [], 'off')
    MODE: either 'times' or 'trials' -- indicates whether signals should be binned by time in trial or for bins with even numbers of trials in each bin
  
    NBINS: a number. 
    
          For 'times' MODE: Using 17 for the standard task with 'times' results in 17 1-second timebins.   
                  
          For 'trials' MODE: enter the number of bins you wish. If number exceeds number of trials, object will not intialize.
                  
  GFITMODE, GFITPARAM: selects the signal dF/F method (or filtering for movement channels). In general:
  
        {'box', 200000} -- uses the moving average dF/F method with 200000 sample window (200 s at 1kHz sampling)
        {'EMG', []} -- applies default rectification of EMG signal
        {'X', []} -- applies default bandpass filtering followed by rectification for accelerometer signals
        {'CamO', 30} -- processes camera data sampled at 30 Hz
        
  NSAMPLEBUFFER: the number of samples buffered into the ITI. Standard is 30000 for photometry data, 100000 for movement control data


Standard init for photometry:
  
    obj = CLASS_HSOM_photometry('v3x', 'times', 17, {'box', 200000}, 30000, [], [], 'off')
  Standard init for EMG:
  
    obj = CLASS_HSOM_photometry('v3x', 'times', 17, {'EMG', []}, 100000, [], [], 'off')
  
NB: data can be rebinned anytime for single session objects. Composite objects (averaging across sessions) cannot be changed after the initialization step.
    
    
-------------------------------------------------
1. Loading single-session sample photometry object:

  - Using either your freshly-generated analysis objects OR using the sample analysis objects available at https://www.dropbox.com/sh/wdotym743hmo4jc/AABKfTVxkH2JVkZXJ-7UpLyCa?dl=0
  
    - Navigate to the sample datafolder for animal B5, SNc day 13 (B5_SNc_13)
  
    - Open the file. It will initialize to the workspace as sObj or obj, depending on the dataset
    
Loading composte session object:

  - Navigate to the sample Composite Datasets folder and open the file with the analysis of choice

-------------------------------------------------
2. Code to reproduce figures from Hamilos et al., 2021

NB that all original datasets from each figure are embedded within the paper on the eLife website. This code can be used to reproduce the same analyses, as well as to recapitulate the analyses on new datasets produced by the user.

  
  - Histograms
            
            % Open folder Figure 1 > (example output figure is in folder)
            load('CollatedStatAnalysisObj_cdf_20191107_18_10_runIDno4486.mat')
            obj.plot('hxg-xsesh')


  - CLTA (cue/lick triggered averages)
    
         NB: Example dataset has 2 mice recorded at SNc and shows basic pattern found more cleanly in the full average across all animals. The all-mice figures are saved to the Figure 2 folder
         
         To plot CLTAs of composite datasets with 250 ms timebins for averaging:
          
          obj = CLASS_HSOM_photometry('v3x', 'times', 68, {'box', 200000}, 30000, [], [], 'off')
          %   Select either SNc or tdt to make a composite SNc or tdt object. 
          %     Then select the corresponding HOST folder from the 
          %     Dropbox sample data.
          
          %     Plotting: NB: you can use the plot function on single-session 
          %       datasets (e.g., B5_SNc_13) and composite datasets
          obj.plot('CLTA', [6, 9, 13, 15], false, 10, 'last-to-first', 1) 
          %
          %     NB: [6,9,13,15] are the timebins -- you can plot any between 1-68
          %     NB2: Change tail in Matlab (click the warning) to 
          %       change limits on right-hand side of plot
          %       5000: makes plots in 1C, 1D
          %       -150:  makes plots in 1E, 1F
          %
          xlim([-1.5,5])
          %
          % Example figure files included in Dropbox folder
          %
          % See CLASS_HSOM_photometry > plot documentation for instructions 
          %   to parameterize the plot function, there are several 
          %   additional options and functionalities you can try!
          
          
  - Cue-aligned averages:
          
          %   Generate the object for the signal of choice:
          %
          obj = CLASS_HSOM_photometry('v3x', 'times', 68, {'box', 200000}, 30000, [], [], 'off')
          %   Select either SNc or tdt to make a composite SNc or tdt object. 
          %     Then select the corresponding HOST folder from the 
          %     Dropbox sample data.
          
          %     Plotting: NB: you can use the plot function on single-session 
          %       datasets (e.g., B5_SNc_13) and composite datasets
          obj.plot('CTA', 4:28, false, 100, 'last-to-first', 1) 
          %
          %     NB: 4:28 are the timebins -- you can plot any between 1-68
          %     NB2: 100 is the smoothing (in samples)
          %
          xlim([-4,10])
          
  - Baseline Cue-aligned and Lamp-Off aligned averages
      
          % Once again, we simply modify our object generating function:
          xlim([-4,10])
          obj = CLASS_HSOM_photometry('v3x', 'paired-nLicksBaseline', {2, [700,3330], [3334,7000], 5000}, {'box', 200000}, 30000, [], [], 'off')
          obj.plot('CTA', 'all', false, 100, 'last-to-first', 1)           
          obj.plotLOTA()

          
   - Lick-aligned averages (e.g., for movement control signals)
   
      These are run exactly the same way as above except using analysis objects containing movement signals: EMG, X (accelerometer), CamO, or one of the red channles (SNcred, VTAred, or DLSred). Generate the objects as before and run the same plotting function *except* call 'LTA2l' as the Mode argument rather than 'CTA' or 'CTLA'. You can also try plotting with 'LTA'. This can be a bit misleading because 'LTA' Mode will not truncate the plot at the earliest cue time; thus it will not be as clear what signal occurred before the trial started in the average. As such, we recommend plotting with 'LTA2l' to trim the signal at the time of the earliest cue.
          
     NB that we custom-binned the times in the movement control data plots to show both early and rewarded times that would have enough time before the lick to avoid truncating the entire signal. The 1 ms bin between 3333 and 3334 ms in the binning eliminates any ambiguity caused by rounding of the reward cut-off time.
          
          obj = CLASS_HSOM_photometry('v3x', 'custom', [0, 2000, 3333, 3334, 7000, 17000], {'box', 200000}, 60000, [], [], 'off')
          obj.plot('LTA2l', [2,4], false, 100, 'last-to-first', 1) 
     
          
  


   - Encoding models
   
      See the function we built to run the models and variations, which is documented in great detail in the CLASS_HSOM_photometry.m file. This function can run a variety of versions of the model on individual session data. There are a ton of additional QC and plotting features included in the model functions that go beyond what was shown in the paper, which may be of interest and use to the user. Please let me know if you have any questions or difficulty running the models by emailing ahamilos [at] mit [dot] edu, and I'll be happy to assist you. I also would like your feedback on how to make this part of the analysis more user-friendly, so any feedback is appreciated! See:
          
          % In CLASS_HSOM_photometry, see:
          
          function [th, X, a, yFit, x, xValidationStruct, CVmat] = nestedGLM(obj, cannedStyle, trimming, lam, th0_on, Events, x_style, basis, smoothing)
          
          % Run to automatically run encoding fits on multiple datasets:
          
          AutoGLMencodingModel.m
     

   - Single trial data
   
     WARNING: this won't work well for a composite object--the file will be too large! Use this for single session datasets. Load the object (original binning doesn't matter, the function will rewrite it):
          
          obj.getBinnedTimeseries(obj.GLM.gfit, 'singleTrial', [], 30000,[],[],false);
     
     Now you can use the plot or analysis function of your choice.

  - Decoding models
  
     To run single-session decoding models, see the documentation provided in the CLASS_HSOM_photometry.m file. Again, we are very interested in making these tools more intuitive and user-friendly. Please reach out to ahamilos [at] mit [dot] edu, and I'll be happy to assist you.
     
          % In CLASS_HSOM_photometry, see:
          
          function [b,dev,stats,X,y,yfit, LossImprovement] = GLM_predictLickTime(obj, Mode, Conditioning, Link, Debug, predictorSet,verbose,killMissing)

  - Julia Language models

    Please see script to run models and the function library in the Julia Language folder.
     
     
     


--------------------------------------

License Notes:

   1. Note that linspecer v1.4.0.0+ by Jonathan C. Lansey is available from MATLAB FileExchange and included in dependencies with its original license: 
    
        https://www.mathworks.com/matlabcentral/fileexchange/42673-beautiful-and-distinguishable-line-colors-colormap?focused=5372538&tab=function

