**Spatial registration**
</br>
The goal of this lab is to learn the process of co-registration between two image modalities such as function to structure, and spatial normalization of lower-resolution images (such as funcitonal MRIs). We will continue to use skills of checking alignment of image registration with fsleyes and contour maps.
</br>

**By the end of this practical you should be able to:** <br/>
* [ ] use FSL's [flirt](http://web.mit.edu/fsl_v5.0.10/fsl/doc/wiki/FLIRT(2f)UserGuide.html) tool co-register your T2* bold image to your high-resolution T1 image <br/>
* [ ] store and run your code in a bash script file 
* [ ] use FSL's tool [epi_reg](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FLIRT/UserGuide#epi_reg) tool to run boundary-based registration [bbr](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FLIRT_BBR) to compare with `flirt` alone
* [ ] concatenate transformation matrices to perform spatial normalization on the T2* bold image <br/> 
* [ ] use FSLeyes and contour overlays to check the alignment of co-registration and normalization results <br/> 

**Access FastX** through the remote login: <br>
https://fastx.divms.uiowa.edu:3443/  <br/>
<br/>

**Download lab assignment questions**: <br>
[Lab 5 assignment doc](https://www.dropbox.com/s/d1fjzopcrm0imxc/Lab-05_questions.docx?dl=0) <br>
<br/>

**Download bold data mixtape**: <br/>
*  Open the terminal
*  Change directories from your home directory to `fmriLab` with the command: `cd ~/fmriLab`
*  To download data, copy/paste to the terminal `wget -O bold_image-mixtape.tar.gz https://www.dropbox.com/s/uwyb3uyqfnl31zn/bold_image-mixtape.tar.gz?dl=0`
*  To unpack your download copy/paste `tar -xvf bold_image-mixtape.tar.gz`
*  You should now have a `bold_image-mixtape` directory with bold images from the same participants as your `T1w_image-mixtape_proc-defaced` directory
<br/>


**Step 1: Check prerequisite anat derivatives** <br>
* Prerequisites for co-registration of a bold image to T1 and MNI template space include:
    * a skull-stripped T1 image from the last lab. For example, for sub-2801 this should be a valid path: `~/fmriLab/T1w_image-mixtape_proc-defaced/sub-2801_reg/sub-2801_brain.nii.gz`
    * an affine transformation for aligning the T1 to MNI. For example, for sub-2801 this should be a valid path: `~/fmriLab/T1w_image-mixtape_proc-defaced/sub-2801_reg/sub-2801_brain_MNIaff.mat`
    * non-linear warp transformation for aligning T1 to MNI. For example, for sub-2801 this should be a valid path: `~/fmriLab/T1w_image-mixtape_proc-defaced/sub-2801_reg/sub-2801_MNI_warp.nii.gz`
</br>


**Step 2: Prepare a new derivatives directory for the functional bold data**
* Move yourself to the `~/fmriLab/bold_image-mixtape/` directory
* At the terminal, type: `mkdir sub-2801_reg`
    * co-registration aligns two 3D volumes, so we need to "splice" out a bold volume that is representative of the series - we'll compare two common options which are the first and middle volumes
        * let's look at sub-2801's bold image to see the 4D image we're working with
        * fsl's command-line tool to "splice" volumes from a 4D image: `fslroi` 
        * example usage to splice out the first volume: `fslroi sub-2801_task-rest_bold.nii.gz sub-2801_reg/example_func_vol-first.nii.gz 0 1`
            * Reads: "Reads: take input image, give back image after extracting volumes starting at vol 0 with length 1"
        * use this example to create an image named `example_func_vol-mid.nii.gz` that is the middle volume of the series
            * hint, what command line tool can you use to figure out what the middle volume would be indexed as?
    * use `bet` to strip the skull from the functional images
        * example for first volume: `bet sub-2801_reg/example_func_vol-first.nii.gz sub-2801_reg/example_func_vol-first_brain.nii.gz`
* By the end of this step, within the registration directory, you should have these two output images prepared for the next step:
    * `example_func_vol-first_brain.nii.gz`
    * `example_func_vol-mid_brain.nii.gz`
</br>

**Step 3: Co-registration of the bold image to the same subject's T1 image**

We will use flirt with the following options, and let's talk about why

```
flirt -in ~/fmriLab/bold_image-mixtape/sub-2801_reg/example_func_vol-first_brain.nii.gz \
-ref ~/fmriLab/T1w_image-mixtape_proc-defaced/sub-2801_reg/sub-2801_brain.nii.gz \
-out ~/fmriLab/bold_image-mixtape/sub-2801_reg/example_func_vol-first_toT1_6dof \
-omat ~/fmriLab/bold_image-mixtape/sub-2801_reg/example_func_vol-first_toT1_6dof.mat \
-cost corratio \
-dof 6 \
-searchrx -90 90 -searchry -90 90 -searchrz -90 90 \
-interp trilinear
```

Put this command in a script with the following steps:
* in the terminal type: `gedit run-reg_first-vol.sh`
* with this file open, copy in the text below, and hit `Save`
    * notice what's going on with the # to help document your code
    * what does the `echo` command do and how is that helpful?
    * why is declaring the subject variable only once helpful?
    * what addditional variable would be helpful if we're registering the middle volume instead of the first?

```
#!/bin/bash

sub='2801'

# Co-register first bold volume with T1 using FLIRT rigid body alignment
echo "running flirt rigid body co-registration"

flirt -in ~/fmriLab/bold_image-mixtape/sub-${sub}_reg/example_func_vol-first_brain.nii.gz \
-ref ~/fmriLab/T1w_image-mixtape_proc-defaced/sub-${sub}_reg/sub-${sub}_brain.nii.gz \
-out ~/fmriLab/bold_image-mixtape/sub-${sub}_reg/example_func_vol-first_toT1_6dof \
-omat ~/fmriLab/bold_image-mixtape/sub-${sub}_reg/example_func_vol-first_toT1_6dof.mat \
-cost corratio \
-dof 6 \
-searchrx -90 90 -searchry -90 90 -searchrz -90 90 \
-interp trilinear
```

* run the script in the terminal by typing: `bash run_registration.sh`
* **look at** registration result by comparing your T1 reference to your bold input image:
    * check alignment with fsleyes: `fsleyes ~/fmriLab/T1w_image-mixtape_proc-defaced/sub-2801_reg/sub-2801_brain.nii.gz ~/fmriLab/bold_image-mixtape/sub-2801_reg/example_func_vol-first_toT1_6dof.nii.gz`
</br>

For comparison, run flirt result with flirt+bbr using FSL's `epi_reg` tool from the sub-2801_reg directory. 
```
epi_reg --epi=example_func_vol-first_brain.nii.gz \
--t1=../../T1w_image-mixtape_proc-defaced/sub-2801.nii.gz \
--t1brain=../../T1w_image-mixtape_proc-defaced/sub-2801_reg/sub-2801_brain.nii.gz \
--out=example_func_vol-first_toT1_bbr.nii.gz
```

* compare results in fsleyes:
`fsleyes ~/fmriLab/T1w_image-mixtape_proc-defaced/sub-2801_reg/sub-2801_brain.nii.gz example_func_vol-first_toT1_6dof.nii.gz example_func_vol-first_toT1_bbr.nii.gz example_func_vol-first_toT1_bbr_fast_wmedge.nii.gz -cm red`

* compare results with paired contour overlay images: 
`slicesdir -o example_func_vol-first_toT1_6dof.nii.gz example_func_vol-first_toT1_bbr_fast_wmedge.nii.gz  example_func_vol-first_toT1_bbr.nii.gz example_func_vol-first_toT1_bbr_fast_wmedge.nii.gz`

</br>


**Step 4: Spatial normalization of the bold image to the MNI template**

We now have the transformation "recipe" for bold->T1w->MNI. However, we don't want to apply these sequentially on the images themselves because once the bold image is transformed we lose some precision in the image compared to the original. So we'll combine these transforms mathematically via concatenation with FSL's `convertwarp` tool.</br>
</br>

**Run concatenation step on both transform recipes:**
* Flirt corratio with 6 dof
```
# flirt corratio with 6 dof
convertwarp --ref=$FSLDIR/data/standard/MNI152_T1_1mm_brain.nii.gz \
--premat=example_func_vol-first_toT1_6dof.mat \
--warp1=../../T1w_image-mixtape_proc-defaced/sub-2801_reg/sub-2801_MNI_warp.nii.gz \
--out=example_func_vol-first_toMNI_warp.nii.gz
```


* Flirt corratio + bbr
```
# flirt corratio + bbr
convertwarp --ref=$FSLDIR/data/standard/MNI152_T1_1mm_brain.nii.gz \
--premat=example_func_vol-first_toT1_bbr.mat \
--warp1=../../T1w_image-mixtape_proc-defaced/sub-2801_reg/sub-2801_MNI_warp.nii.gz \
--out=example_func_vol-first_toMNI-bbr_warp.nii.gz
```

**Apply concatenated transforms for both recipes:**
* Flirt corratio with 6 dof
```
# flirt corratio
applywarp --ref=$FSLDIR/data/standard/MNI152_T1_1mm_brain.nii.gz \
--in=example_func_vol-first_brain.nii.gz \
--out=example_func_vol-first_toMNI.nii.gz \
--warp=example_func_vol-first_toMNI_warp.nii.gz
```

* Flirt corratio + bbr
```
# flirt corratio + bbr
applywarp --ref=$FSLDIR/data/standard/MNI152_T1_1mm_brain.nii.gz \
--in=example_func_vol-first_brain.nii.gz \
--out=example_func_vol-first_toMNI-bbr.nii.gz \
--warp=example_func_vol-first_toMNI-bbr_warp.nii.gz
```

* Make a contour overlay of our result for a static image:
```
slicesdir -o \
example_func_vol-first_toT1_6dof.nii.gz example_func_vol-first_toT1_bbr_fast_wmedge.nii.gz \
example_func_vol-first_toT1_bbr.nii.gz example_func_vol-first_toT1_bbr_fast_wmedge.nii.gz \
example_func_vol-first_toMNI.nii.gz $FSLDIR/data/standard/MNI152_T1_1mm_brain.nii.gz \
example_func_vol-first_toMNI-bbr.nii.gz $FSLDIR/data/standard/MNI152_T1_1mm_brain.nii.gz
```


**Lab Homework** 

* Repeat steps above for sub-2801 with the middle volume as your input functional image. 
* Compare results to pick the best registration options from those we tried for sub-2801.
* Use at least two screenshots from fsleyes and/or your contour overlays to support your decision. 






