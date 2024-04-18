# **segcsvd**

A CNN-based tool for segmentation of white matter hyperintensities (WMH) on FLAIR MRI and enlarged perivascular spaces (PVS) on T1 MRI 


## **UPDATE**

* 20240518: updated version now available that fixes WMH segmentation issue affecting non-isotropic data processed with skip_mask_and_bias=false 

## **i) Overview**
* **Purpose and Development**: segcsvd was developed to improve segmentation accuracy for large-scale, heterogenous imaging datasets with varying degrees of cerebrovascular disease (CVSD) burden
* **Technical Foundation**: segcsvd was developed using patient MRI and leverages FreeSurfer's SynthSeg tool (https://surfer.nmr.mgh.harvard.edu/fswiki/SynthSeg)
* **Availability**: segcsvd is distributed as a docker/singularity/apptainer image
* **Citation Request**: Users of segcsvd in their research are kindly requested to cite the following in their work:

    * **SynthSeg:** B Billot, DN Greve, O Puonti, A Thielscher, K Van Leemput, B Fischl, AV Dalca, JE Iglesias.Segmentation of brain MRI scans of any contrast and resolution without retraining. Medical Image Analysis, 83, 102789 (2023). 
    * **segcsvd:** E Gibson, J Ramirez, LA Woods, R Sommers, N M Ghahjaverestan, CJM Scott, F Gao, AE Lang, C Marras, DP Breen, MC Tartaglia, MA Binns, R B, S Symons, RH Swartz, M Masellis, SE Black, A Moody, ONDRI Investigators, CAIN Investigators, Colleagues from the Foundation Leducq Transatlantic Network of Excellence, Andrew SP Lim, M Goubran. Examining perivascular spaces (PVS) in cerebral small vessel disease (csvd) using a novel T1-based automated PVS segmentation tool. The International Society of Vascular Behavioural and Cognitive Disorder (VASCOG) Conference, 2023, Gothenburg, Sweden.
    * **segcsvd<sub>WMH</sub>:** [TODO] Coming soon
    * **segcsvd<sub>PVS</sub>:** [TODO] Coming soon

 <br>

![Example](synthsegcsvd_example.png)

<br>
 
## **ii) Installation**

>**Singularity Users**
>* download segcsvd_rc02.sif from: 
> https://s3.us-east.cloud-object-storage.appdomain.cloud//cloud-synthsegcsvdrc05/segcsvd_rc02.sif
>
>**Docker Users**
>* download segcsvd_rc02.tar.gz from:
>  https://s3.us-east.cloud-object-storage.appdomain.cloud//cloud-synthsegcsvdrc05/segcsvd_rc02.tar.gz
>* install: `sudo docker load -i segcsvd_rc02.tar.gz`

<br>
 
## **iii) Inputs** ##
>**Images required for WMH Segmentation:**
>* FLAIR image
>* FreeSurfer's SynthSeg output (version 2.0 with CSF)
>
>**Images required for PVS Segmentation:**
>* T1 image
>* FreeSurfer's SynthSeg output (version 2.0 with CSF)
>* WMH segmentation (can be an empty image for populations without WMHs)
>
> **General requirements for the FLAIR/T1 input images:**
>* **Intensity values**: should begin at 0 (background) -- negative voxels will be ignored
>* **Masking and bias correction**: the "skip_mask_and_bias" option should be set to false if FLAIR/T1 images are not already masked and bias corrected
>
> **Additional notes for PVS segmentation:**
> * Other pathology (e.g. lacunes), if present, may segment as PVS (issue to be addressed in future versions)
> * PVS within WMH are not segmented by the current model (issue to be addressed in future versions)
> * relatively clean data is expected to avoid segmentation of noise and MR artifacts as PVS
> * an isotropic voxel size of ~0.9/1.0 mm (resampling beforehand if necessary) is strongly recommended
 

## **iv) Running segcsvd**
> **Overview**:
>* define variables -- described in the variable setup sections below
>* execute run command -- can be copy/pasted without modification once variables are defined
>
>**Typical workflow:**
>  * coregister FLAIR and T1 images 
>  * generate synthseg image (i.e. run: `mri_synthseg --i <FLAIR/T1.nii.gz> --o <synthseg.nii.gz>`)
>  * run WMH segmentation (section 1)
>  * run PVS segmentation (section 2)



<br>
 

# **1. segcsvd<sub>WMH</sub>**
> ## **1.1 Variable Setup**
```bash
in_dir=$(pwd)
out_dir=${in_dir}
flair_fn=FLAIR.nii.gz
synth_fn=synthseg.nii.gz
sif=${HOME}/segcsvd_rc02.sif
out_fn=seg_wmh.nii.gz
seg_wmh_thr=0.35
skip_mask_and_bias=true
cleanup=true
```

where:

> **in_dir** : full path to input data, or $(pwd) for current directory
>
> **out_dir** : full path to the output data
>
> **flair_fn** : FLAIR input filename 
>
> **synth_fn** : FreeSurfer synthseg input filename (output from "mri_synthseg")
>
> **sif** : full path + filename of singulairty file downloaded above
>
> **out_fn** : wmh segmentation output filename (with file extension)
> 
> **seg_wmh_thr** : threshold for binarizing WMH segmentation output 
>
> **skip_mask_and_bias** : true | false (true if FLAIR has been masked and bias corrected, otherwise false)
>
> **cleanup** : true | false (true to remove temporary files, otherwise false)

<br>

> ## **1.2. Run Command (copy/paste)**
FOR SINGULARITY/APPTAINER USERS:
```bash
singularity run \
  --bind ${in_dir}:/indir,${out_dir}:/outdir  --pwd /  ${sif}  segment_wmh  \
  /indir/${flair_fn} \
  /indir/${synth_fn}  \
  /outdir/${out_fn}  \
  1  \
  "96,128"  \
  ${seg_wmh_thr} \
  1 \
  ${skip_mask_and_bias} \
  ${cleanup} 
```
FOR DOCKER USERS:
```bash
sudo docker run \
  -v ${in_dir}:/indir \
  -v ${out_dir}:/outdir \
  -w / \
  rc02 \
  segment_wmh \
  /indir/${flair_fn} \
  /indir/${synth_fn} \
  /outdir/${out_fn} \
  1 \
  "96,128" \
  ${seg_wmh_thr} \
  1 \
  ${skip_mask_and_bias} \
  ${cleanup}
```
<br>

> ## **1.3. Output**

> * seg_wmh.nii.gz : unthresholded WMH segmentation [0,1]
> * thr_seg_wmh.nii.gz : thresholded binarized WMH segmentation {0,1}

# **2. Generate PVS Segmentation**
> ## **2.1. Variable Setup**
```bash
in_dir=$(pwd)
out_dir=${in_dir}
t1_fn=T1.nii.gz
sif=${HOME}/segcsvd_rc02.sif
seg_wmh_fn=thr_seg_wmh.nii.gz
synth_fn=synthseg.nii.gz
out_fn=seg_pvs.nii.gz
skip_mask_and_bias=true
cleanup=true
seg_pvs_thr=0.35
```


**WHERE:**

> **in_dir** : full path to input data, or $(pwd) to use current directory
>
> **out_dir** : full path to output data
>
> **t1_fn** : T1 filename 
> 
> **sif** : full path + filename of singulairty file downloaded above
>
> **seg_wmh_fn** : WMH segmentation filename (or empty image to skip)
> 
> **out_fn** : output filename (with extension)
> 
> **skip_mask_and_bias** : true | false (true if FLAIR has been masked and bias corrected, otherwise false)
>
> **cleanup** : true | false (true to remove temporary files, otherwise false)
>
> **seg_pvs_thr** : threshold for binarizing PVS segmentation output 


> ## **2.2. Run Command (copy/paste)**

FOR SINGULARITY/APPTAINER USERS:
```bash
singularity run --bind ${in_dir}:/indir,${out_dir}:/outdir \
  --pwd / \
  ${sif} \
  segment_pvs \
  /indir/${t1_fn} \
  /indir/${synth_fn} \
  /indir/${seg_wmh_fn} \
  /outdir/${out_fn} \
  1 0 \
  ${skip_mask_and_bias} \
  ${cleanup} \
  ${seg_pvs_thr}
```
FOR DOCKER USERS:
```bash
sudo docker run \
  -v ${in_dir}:/indir \
  -v ${out_dir}:/outdir \
  -w / \
  rc02 \
  segment_pvs \
  /indir/${t1_fn} \
  /indir/${synth_fn} \
  /indir/${seg_wmh_fn} \
  /outdir/${out_fn} \
  1 0 \
  ${skip_mask_and_bias} \
  ${cleanup} \
  ${seg_pvs_thr}
```

> ## **2.3. Output**
>
> * pvs_seg.nii.gz : unthresholded PVS segmentation [0,1]
> * thr_pvs_seg.nii.gz : thresholded binarized PVS segmentation {0,1}
