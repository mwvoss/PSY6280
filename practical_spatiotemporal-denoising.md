**Spatiotemporal denoising with independent component analysis**
</br>
The purpose of this lab is to get hands-on experience interpreting the output of spatial independent component analysis as run in FSL's MELODIC program, with emphasis on labeling components to denoise fMRI data. The experience of manual labeling will help help you become familiar with the various sources of noise in fMRI data even after standard preprocessing, and will help you appreciate when and how to apply ICA approaches that are a blend of manual and automatic component classification (e.g., FSL's FIX) or fully automated with tuning to specific types of noise (e.g., ICA-AROMA for motion). 

</br>


**By the end of this practical you should be able to:** <br/>
* [ ] interpret motion timeseries plots
* [ ] interpret the output of FSL's [MELODIC](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/MELODIC) tool for a single-subject ICA analysis
* [ ] open MELODIC-estimated components within [fsleyes's melodic scene](https://users.fmrib.ox.ac.uk/~paulmc/fsleyes/userdoc/latest/ic_classification.html#loading-a-melodic-analysis)
* [ ] understand the process of manually labeling components as signal or noise using a conservative "innocent until proven guilty" criteria such as outlined in [Griffanti et al., 2017](https://github.com/mwvoss/MRI-lab-classes/tree/master/PSY6280-2020-FA2020/pdfs/Griffanti-2017-ICA.pdf)
<br/>

**Access FastX** through the remote login: <br>
https://fastx.divms.uiowa.edu:3443/  <br/>
<br/>


**Lab data** <br>
We will continue working with the data from our mixtape directories.
</br>

**Step 1: Prepare a new derivatives directory for melodic output**
* Move yourself to the `~/fmriLab/bold_image-mixtape/` directory
* At the terminal, type: `mkdir sub-2801_melodic`

**Step 2: Run preprocessing and ICA with Melodic** 
* Open the fsl GUI menu by typing `fsl` in the terminal
* Click on the `MELODIC` button
* Data tab
    * Click `Select 4D data` and select sub-2801's bold image (that has not been registered)
    * Select the `sub-2801_melodic` directory as your Output directory
    * Delete the first 4 volumes (to remove the impact of dummy volumes)
    * Maintain the High Pass filter as 100s
![ ](images/denoising_melodic-input.png)

* Pre-Stats tab
    * Maintain MCFLIRT with the default reference image of the middle volume
    * Maintain "BET" brain extraction as "on"
    * Change spatial smoothing to 6
    * Maintain the Highpass filter as set "on"
* Registration tab
    * Select a `Main structural image` and select your skull-stripped T1w image from your T1w mixtape directory
    * For the purposes of time we'll run linear registration
        * Main structural Image cost function: Set `Normal search` to `12 DOF` 
            * Why are we avoiding BBR?
            * What would we need to do to enable BBR to work better?
        * Standard space cost function: Set `Normal search` to `12 DOF` 
            * Notice we are resampling to 4mm resolution
            * This combined with spatial should help reduce impact of less than perfect alignment from affine registration 
![ ](images/denoising_melodic-registration.png)
* Stats and Post-Stats tabs
    * leave defaults on
* Click `Go`
</br>

**Step 3: Open the melodic web report for overview of pre-stats (motion) and melodic results** 
* Within the file browser, navigate to the `melodic.ica` directory for sub-2801 and open the `report.html` file with a web browser
* **Break-out questions**
    * `Pre-Stats` page: What kind of motion profile do you see? Broadly, would we characterize this participant as "low-motion" or "high-motion"?
    * `Registration` page: You should now know what you're looking at! Any concerns?
    * `Log` page: here's where everything that was run is documented, starting with `Initialisation` you can see the initial input was the raw nifti bold image without preprocessing. What file is the input to melodic processing?
    * Finally, the main results are shown in the `ICA` page: We will talk through what you're seeing.
</br>


 **Step 3: Open melodic-derived components in fsleyes** 
* Now let's see how to view the component output interactively in fsleyes to support manual labeling of components as signal or noise
* In the terminal, move yourself to the `melodic.ica` output directory
* Open fsleyes with the melodic scene setting: `fsleyes --scene melodic -ad filtered_func_data.ica filtered_func_data.ica/melodic_IC.nii.gz`
* You should see something like below: <br>
![melodicview](images/denoising_melodicView.png)
* **Break-out questions to prep for manual labeling**
    * The `Nyquist frequency` is half the sampling rate. The Nyquist frequency is also called the Nyquist limit or folding limit, because it is the highest frequency that can be faithfully recovered at a given sampling rate in order to be able to fully reconstruct the signal. When a source signal is faster than this limit, then it may get aliased or "folded" into the reconstructed signals as a slower frequency signal.
        * The sampling rate for fMRI data is the Time of Repetition (TR), which is the rate of measurement for each 3D bold image in our 4D series. Reminder the `TR` for this data is 2 seconds.
        * Convert this period to frequency as `1/TR = 1/2 = .5 Hz`
        * Our Nyquist limit is then half the sampling frequency: `.5/2 = .25 Hz`
        * This means the highest frequency signal that could be truly recovered in this bold data is less than or equal to .25 Hz.
        * Compare this limit to the power spectrum shown for each component in fsleyes. 
            * What does this mean for physiological noise that would occur at higher frequencies? How might the timeseries look for components with these sources?
    * To label a component, double click within the open text box next to the component in the `Melodic IC classification` viewer.
        * Think about:
            * What are the similarities and differences of respiratory and cardiac related noise?
            * What is the first component that looks like signal of interest? How does do the spatial map, timeseries, and power spectrum look different than the noise components?
        * Note that for denoising purposes, only the distinction of signal vs. noise is used later
            * Additional details about the source of noise could be useful if you would like to track the number of components from different sources in your data, and you could use that information for testing selective denoising tuned to one type of noise source.
    * You do not need to label all the components for this lab
        * After you've looked at about half and/or seen at least two components that reflect signal, run melodic with another participant for whom you have a skull stripped brain for. 
</br>
