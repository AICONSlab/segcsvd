# **segcsvd**

A CNN-based tool for segmentation of white matter hyperintensities (WMH) on FLAIR MRI and enlarged perivascular spaces (PVS) on T1 MRI 


## **UPDATE**

* 20250731: PVS segmentation now available

## **i) Overview**
* **Purpose and Development**: segcsvd was developed to improve segmentation accuracy for large-scale, heterogenous imaging datasets with varying degrees of cerebrovascular disease (CVSD) burden
* **Technical Foundation**: segcsvd was developed using patient MRI and leverages FreeSurfer's SynthSeg tool (https://surfer.nmr.mgh.harvard.edu/fswiki/SynthSeg)
* **Availability**: segcsvd is distributed as a docker/singularity/apptainer image
* **Citation Request**: Users of segcsvd in their research are kindly requested to cite the following in their work:

    * **SynthSeg:** B Billot, DN Greve, O Puonti, A Thielscher, K Van Leemput, B Fischl, AV Dalca, JE Iglesias.Segmentation of brain MRI scans of any contrast and resolution without retraining. Medical Image Analysis, 83, 102789 (2023). 
    * **segcsvd<sub>WMH</sub>:** Gibson E, Ramirez J, Woods LA, Ottoy J, Berberian S, Scott CJM, Yhap V, Gao F, Coello RD, Valdes Hernandez M, Lang AE, Tartaglia CM, Kumar S, Binns MA, Bartha R, Symons S, Swartz RH, Masellis M, Singh N, Moody A, MacIntosh BJ, Wardlaw JM, Black SE, Lim ASP, Goubran M; ONDRI Investigators, ADNI, CAIN Investigators, colleagues from the Foundation Leducq Transatlantic Network of Excellence. segcsvdWMH: A Convolutional Neural Network-Based Tool for Quantifying White Matter Hyperintensities in Heterogeneous Patient Cohorts. Hum Brain Mapp. 2024 Dec 15;45(18):e70104. doi: 10.1002/hbm.70104. PMID: 39723488; PMCID: PMC11669893.
    * **segcsvd<sub>PVS</sub>:** Gibson E, Ramirez H, Woods LA, Berberian S, Ottoy H, Scott CJM, Yhap V, Gao F, Coello RD, Valdes-Hernandez M, Lang AE, Tartaglia CM, Kumar S, Binns MA, Bartha R, Symons S, Swartz RH, Masellis M, Singh N, MacIntosh BJ, Wardlaw JM, Black SE, ONDRI Investigators, ADNI, CAIN Investigators, colleagues from the Foundation Leducq Transatlantic Network of Excellence, Lim ASP, Goubran M. segcsvd<sub>PVS</sub>: A convolutional neural network-based tool for quantification of enlarged perivascular spaces (PVS) on T1-weighted images. medRxiv 2025.07.29.25332360; doi: https://doi.org/10.1101/2025.07.29.25332360

 <br>

![Example](synthsegcsvd_example.png)

<br>
 
## **ii) Installation**

>**Singularity Users**
>* download segcsvd_rc03.sif from: 
> https://huggingface.co/datasets/AICONSlab/segcsvd/resolve/main/singularity/segcsvd_rc03.sif
>
>**Docker Users**
>* download segcsvd_rc03.tar.gz from:
>  https://huggingface.co/datasets/AICONSlab/segcsvd/resolve/main/docker/segcsvd_rc03.tar.gz
>* install: `sudo docker load -i segcsvd_rc03.tar.gz`

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
>* **Masking and bias correction**: the "skip_mask_and_bias" option should be set to false if FLAIR/T1 images are not already skull-stripped and bias corrected
>
 

## **iv) Running segcsvd**
> **Overview**:
>* define variables -- described in the variable setup sections below
>* execute run command -- should be copy/pasted without modification once variables are defined
>
>**Typical workflow:**
>  * coregister FLAIR and T1 images 
>  * generate synthseg image (i.e. run: `mri_synthseg --i <FLAIR/T1.nii.gz> --o <synthseg.nii.gz>`)
>  * run WMH segmentation (section 1)
>  * run PVS segmentation (section 2 - coming soon)



<br>
 

# **1. segcsvd<sub>WMH</sub>**
> ## **1.1 Variable Setup**
```bash
in_dir=$(pwd)
out_dir=${in_dir}
flair_fn=FLAIR.nii.gz
synth_fn=synthseg.nii.gz
sif=${HOME}/segcsvd_rc03.sif
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
> **skip_mask_and_bias** : true | false (true if FLAIR has been skull-stripped and bias corrected, otherwise false)
>
> **cleanup** : true | false (true to remove temporary files, otherwise false)

<br>

> ## **1.2. Run Command (copy/paste after variable setup)**
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
  segcsvd_rc03 \
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
sif=${HOME}/segcsvd_rc03.sif
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


> ## **2.2. Run Command (copy/paste after variable setup)**

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
  "1.0,1.4" 0 \
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
  segcsvd_rc03 \
  segment_pvs \
  /indir/${t1_fn} \
  /indir/${synth_fn} \
  /indir/${seg_wmh_fn} \
  /outdir/${out_fn} \
  "1.0,1.4" 0 \
  ${skip_mask_and_bias} \
  ${cleanup} \
  ${seg_pvs_thr}
```
> ## **2.3. Output**
>
> * pvs_seg.nii.gz : unthresholded PVS segmentation [0,1]
> * thr_pvs_seg.nii.gz : thresholded binarized PVS segmentation {0,1}


