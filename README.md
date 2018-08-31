# local-variability-network-integration-garrett-2018-neuroimage


* This folder contains the scripts used for the different analysis steps reported in:

	* [Garrett, Epp, Perry, & Lindenberger (NeuroImage 2018). Local temporal variability reflects functional integration in the human brain](https://www.sciencedirect.com/science/article/pii/S1053811918307171)

* The raw data is publicly available [**here**](http://fcon_1000.projects.nitrc.org/indi/enhanced/download.html)

* Raw data files for each participant should be stored in the subfolder 'RAW' in A_preproc.

* All names of folders and scripts are prefixed with letters and sometimes numbers. To replicate the results of the paper, scripts should be run in alphabetical and numerical order. All scripts can be found in the scripts folder of the respective analysis step. Results are stored separately in different subfolders. Before running the scripts, the user must specify the path of the root directory (NKI_enhanced_rest) and the Subject ID List in the config file of the respective scripts folders. Results reported in the paper are based on the default ID list specified in these config files.

* If the scripts should be run on different data than the NKI data set, please make sure to alter/adapt the names and paths in all of the scripts provided. 

* All standard images and masks should be stored in the folder G_standards_masks. These include:

    | Image/Source | Details |
    | ------ | ------ |
    | [T1 gray matter mask](https://fsl.fmrib.ox.ac.uk/fsldownloads_registration) | See note below |
    | [Harvard-Oxford cortical atlas](https://neurovault.org/collections/262/) | Use: HarvardOxford-cort-maxprob-thr25-2mm.nii |
    | [Shirer 14 networks masks](https://findlab.stanford.edu/functional_ROIs.html) | Download all 90 fROIs folder |
    | [Thalamic Connectivity atlas](http://www.lead-dbs.org/?page_id=45) | [Horn, 2016](https://www.sciencedirect.com/science/article/pii/S1053811915007673) |
	| [Craddock parcellation](https://www.nitrc.org/projects/cluster_roi/) | [Craddock et al., 2011](https://onlinelibrary.wiley.com/doi/abs/10.1002/hbm.21333) |
    | Gray matter common coordinates mask | generated by the script B_GMcommonCoords.m, located in the B_PLS folder |

**NOTE:** avg152_T1_gray_mask_90_2mm.nii was created using the following inputs and parameters, using the base image from the FSL standard/tissuepriors folder:

>> fslmaths avg152T1_gray.img -thrp 45 -bin avg152_T1_gray_mask_90.nii

* All required toolboxes should be stored in the folder E_toolboxes. Toolbox versions of the original analysis include:

    * [Nifti toolbox](https://de.mathworks.com/matlabcentral/fileexchange/8797-tools-for-nifti-and-analyze-image) (01/22/2014)
    * [PLS toolbox](https://www.rotman-baycrest.on.ca/index.php?section=84) (version 6.1306040)
    * [REST toolbox](http://restfmri.net/forum/index.php) (v1.8)
    * [spm 8](http://www.fil.ion.ucl.ac.uk/spm/)
	* preprocessing tools - custom script
---
**The following sections contain further information regarding scripts for specific analysis steps.**

## Preprocessing

0. **preproc_bash_config.sh & preproc_mat_config.m**

 Set up base path for project folder and Subject List in these files. ProjectPath variable must be set by the user to location of the NKI_enhanced_rest folder. Both the mat file and the bash file must be altered. Subject list corresponds to 100 subjects used in study. 

1. **A_BET_fsl.sh**

 This script will perform brain extraction of anatomical images. Individual parameters are provided in separate file contained in scripts folder.
 
 Input: /NKI_enhanced_rest/A_preproc/RAW/<ID>/anat/mprage.nii.gz
 Output: /NKI_enhanced_rest/A_preproc/data/<ID>/anat/mprage_brain.nii.gz

2. **B_0_design_NKI.fsf & B_1_feat_fsl.sh**
 
 B_0_design_NKI.fsf is a template design file used for FEAT. The B_1_feat_fsl.sh script performs desired preprocessing procedures, such as smoothing and motion correction.
 
 Subject Design File: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/design_<ID>.fsf
 Input: /NKI_enhanced_rest/A_preproc/RAW/<ID>/session_1/RfMRI_mx_645/rest.nii.gz
 Output: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>.feat

3. **C_BPfilt_restfmri.m**
 
 This script uses the rest_fmri toolbox's rest_bandpass function to perform a 10th order bandpass filter on the data.
 
 Input: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>.feat/filtered_func_data.nii.gz
 Output: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt.nii.gz
 
4. **D_ICA_fsl.sh**

 This script performs ICA decomposition. 

 Input: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt.nii.gz
 Output: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt.ica
 
5. **E_0_rejcomps & E_1_denoise_fsl.sh**

 ICA components are screened **manually** and a list of noise components is made for later removal. Noise components must be stored in a subject specific text file. The E_1_denoise_fsl.sh script then removes the desired components.
 
 **NOTE**: As ICA calculation is probabilistic, the components may change slightly after each run of FSL MELODIC. Therefore, we provided our input images (<ID>_rest_feat_BPfilt.nii.gz) together with the original ICA component output under /NKI_enhanced_rest/A_preproc/D_ICA_results/ together with our denoising decicions: /NKI_enhanced_rest/A_preproc/E_0_rejcomps. 

 Text file of components: /NKI_enhanced_rest/A_preproc/scripts/E_0_rejcomps/<ID>_rejcomps.txt
 Input:  /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt.nii.gz
 Output: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised.nii.gz
  
6. **F_FLIRT_fsl.sh**

 This script performs a three step registration procedure. First a functional-to-anatomical registration matrix is created, then an anatomical-to-2mm_MNI matrix. These are then concatenated and the resulting registration matrix is applied to the funtional data.

 Input: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised.nii.gz
 Output: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt.nii.gz

7. **G_detrend_spm8.m**

 This script performs detrending to the third order (cubic trend) using spm_detrend.

 Input: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt.nii.gz
 Output: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt_denoised.nii.gz
 
## PLS

1. **A_Sessiondatamat_file_modification.m**

 This script creates session data mat-files for PLS from txt template (NKI_PLS_NKIrest_template_batch_file.txt). It creates the structure of the sessiondata.mat files, needed for PLS. This file structure can then be altered to our needs in later steps and filled with different values of interest.

 Template path: /NKI_enhanced_rest/B_PLS/scripts/
 Output: /NKI_enhanced_rest/B_PLS/SD_NKIrest/MeanBOLD_files/mean_<ID>_NKIrest_BfMRIsessiondata.mat 

2. **B_GMcommonCoords.m**

 This script creates a mask of coordinates including only gray matter (GM) coordinates and commonly activated coordinates for all subjects in the sample called final coordinates. Outputs _GMcommoncoords.mat_.

 Data path (preprocessed nifti): /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt_denoised.nii.gz
 Output: /NKI_enhanced_rest/G_standards_masks/GM_mask/GMcommoncoords.mat

3. **C_1_BfMRIsessiondata_creation_power_SD_wholebrain.m**

 This script performs the 2nd pre-step for PLS: Replace mean activity values by values of interest (SD and sqrtPower). SD values are calculated by taking the standard deviation of each voxel's time series and sqrtPower values are the square roots of the summed power values of the pWelch function (power/frequency decomposition).  

 Path of Mean .mat files: NKI_enhanced_rest/B_PLS/SD_NKIrest/MeanBOLD_files/
 Input (preprocessed nifti): /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt_denoised.nii.gz
 Output: NKI_enhanced_rest/B_PLS/SD_NKIrest/SD_<ID>_NKIrest_SPMdetrend_BfMRIsessiondata.mat

4. **C_2_BfMRIsessiondata_creation_power_14networks.m**

 This script creates masks for each network and applies it to subjects' SD_sessiondata.mat matrices. It creates one sessiondata.mat per subject and network, containing only network coordinates and network SD/sqrtPowert values. 

 Input: NKI_enhanced_rest/B_PLS/SD_NKIrest/SD_<ID>_NKIrest_SPMdetrend_BfMRIsessiondata.mat
 Output: NKI_enhanced_rest/B_PLS/SD_NKIrest/14networks_PLS/SD_<ID>_<network>_NKIrest_SPMdetrend_BfMRIsessiondata.mat
	 
5. **C_3_BfMRIsessiondata_creation_power_Craddock.m**

 This script applies Craddock's parcellation scheme (500 parcels and 950 parcels) to the preprocessed niftis. It first calculates median time series per parcel to then subject these median time series to pwelch (for calculating power values) and standard deviation calculation (SD values). It creates one sessiondata.mat per subject, containing the SD/sqrtPowert values of both Craddock parcellations (500 and 950 parcels). 

 Path of sessiondatamat.files used as a template: NKI_enhanced_rest/B_PLS/SD_NKIrest/SD_<ID>_NKIrest_SPMdetrend_BfMRIsessiondata.mat
 Input: preprocessed niftis (/NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt_denoised.nii), Craddock parcellation (/NKI_enhanced_rest/G_standards_masks/craddock_2011_parcellations/tcorr05_2level_all_MNI2mm_NN.nii.gz)
 Output: NKI_enhanced_rest/B_PLS/SD_NKIrest/Craddock_parcellation/SD_<ID>_NKIrest_SPMdetrend_Craddock_BfMRIsessiondata.mat
	 
## Dimensionality

0. **preproc_mat_config.m**

 Set up base path for project folder and Subject List in these files. ProjectPath variable must be set by the user to location of the NKI_enhanced_rest folder. Subject list corresponds to 100 subjects used in study. 

1. **A_PCA_factor_calculation_90perc.m**
 
 This script calculates whole brain PCA (principal component analysis) and determines how many components/dimensions are needed to account for 90% of variability.
 
 Input: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt_denoised.nii.gz
 Output: /NKI_enhanced_rest/C_Dimensionality/PCAdimSPAT_wholebrain/NKI_<ID>_spatialPCAcorr_90variance.mat
 
2. **B_PCA_factor_calculation_90perc_Craddock.m**

 This script subjects Craddock's brain parcels to PCA (principal component analysis) and determines how many components/dimensions are needed to account for 90% of variability. Therefore,  median time series are calculated as a first step per parcel (for both 500 and 950 parcellations).
 
 Input: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt_denoised.nii.gz
 Output: /NKI_enhanced_rest/C_Dimensionality/PCAdimSPAT_wholebrain_craddock/NKI_<ID>_spatialPCAcorr_90variance_craddock500.mat and NKI_<ID>_spatialPCAcorr_90variance_craddock950.mat

3. **C_PCA_factor_calculation_14networks.m**

 This script subjects each of Shirer's networks to PCA and determines how many dimensions are needed to account for at least 90% of variability (within each network).

 Input: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt_denoised.nii.gz
 Output: /NKI_enhanced_rest/C_Dimensionality/PCAdimSPAT_14networks/NKI_<ID>_spatialPCAcorr_<NetworkName>_>90variance.mat

3. **D_PCA_factor_calculation_14networks_corticalROIs.m**
 
 This script subjects the cortical parts of each of Shirer's networks to PCA, and determines how many dimensions are needed to account for at least 90% of variability. Therefore, as a first step, the Harvard-Oxford cortical atlas is loaded and intersected with each network to determine only cortical network ROIs. 

 Input: preprocessed nifti: /NKI_enhanced_rest/A_preproc/data/<ID>/rest/<ID>_rest_feat_BPfilt_denoised_MNI2mm_flirt_denoised.nii.gz; Harvard-Oxford cortical atlas: /NKI_enhanced_rest/G_standards_masks/Harvard-Oxford_atlases/HarvardOxford-cort-maxprob-thr25-2mm.nii
 Output: /NKI_enhanced_rest/C_Dimensionality/PCAdimSPAT_14networks_cortical/NKI_<ID>_spatialPCAcorr_<NetworkName>_corticalROIs_>90variance.mat
 
4. **X_readout_PCAdim_results.m**

 This script loads all single subject's mat files (containing the number of PCA dimensions) and store them in one file, for each of the previous steps separately.

 Input: /NKI_enhanced_rest/C_Dimensionality/*90variance.mat
 Output: /NKI_enhanced_rest/C_Dimensionality/PCAdimSPAT_results/NKI_N100_*.mat
 
 
 
## Thalamocortical Upregulation

1. **A_test_thalamocortical_upregulation_for_ShirerNetworks.m**

 This script test thalamocortical upregulation for each of the 14 Shirer networks. Outputs _thalamocortical_upregulation_for_ShirerNetworks.mat.


 SD PLS path: NKI_enhanced_rest/B_PLS/SD_NKIrest/
 Output path: NKI_enhanced_rest/D_thalamocortical_upregulation/results/

2. **B_ControlAnalysis_hub_regions_power.m**

 This script extracts power in hub regions of each Shirer network. Outputs _hub_regions_power.mat_.

 SD PLS path: NKI_enhanced_rest/B_PLS/SD_NKIrest/
 Output path: NKI_enhanced_rest/D_thalamocortical_upregulation/results/
