# Final Analysis Homework

This homework will provide you with the first of many 'real-world' opportunities to exercise your fMRI analysis skills from start to finish. Choose from one of the two options below. Each involves reproducing a published analysis to some extent. In theory, the corresponding paper will provide all of the necessary information to fully replicate the published analysis. In practice, some methodological details may be missing or vague, in which case you should make an educated guess and carry on. For both options, there will also be some prescribed deviations from the published analysis in the interests of making the assignment a little easier.

**Warning**
The datasets provide raw (BIDS format) real-world data. Data for some subjects may have unexpected problems that may or may not be described in the documentation. It is acceptable to exlcude subjects from the analysis to get around these, but these exclusions should be documented in your methods.

## Directory Structure and Submission Requirements

For either option, you must structure your directories in the repository as:

- `scripts/` to contain your processing script(s)
- `analysis/` to contain the AFNI analyses. This folder should **not** be committed to your repository.
- `results/` to contain the requested tables, figures, and text

### Scripts

Your analysis must be fully automated, with the possible exception of combining result images and tables into their print-ready forms. This means you must provide a single script that reproduces your entire analysis when run. This script can, and probably should, call on other scripts to execute specific processing stages. For example, you may consider breaking your processing into several different scripts that are called from the main script. A non-exhaustive list of these steps might include

- Some per subject scripts that
	- Convert event timing files into AFNI timing files
	- Running the AFNI processing pipeline 
- Running the group analysis
- Creating tables and figures


The suggested approach is to write each piece, test it on one subject, and write the next piece. Once you have each piece working for a single subject, combine them into one script and try running the same subject again (take care to delete any files from the previous test). When that works, try using a list of a few subjects and testing the group analysis. Finally, run the whole dataset when everything seems to work. Your scripts can be written in any reasonably common language. Using an HPC job array to process subject is recommended.

You must provide a `scripts/README.md` Markdown file that documents the use of your processing script in sufficient detail for me to run your analysis in a sleep-deprived state in the wee hours of the night. Your scripts should use relative paths or variables so that they can be run by someone who has the same relative directory structure, but not necessarily the same absolute paths. For example, replace references to `/scratch/netid/...` with something else, since this depends on the person running the script to have access to your directory. The repository directory structure will facilitate this if you assume that your main script is run from the top level of the repository (e.g. as `./scripts/run_analysis.sh`).

### Results

In addition to your scripts, your submission must include a PDF document, `results/results.pdf`, containing a short methods section and figures/tables. Your figures and tables should be manuscript quality. The methods should be written in your own words and describe your analysis in sufficient detail to be reproduced.



## Option 1: Reproduce Rong et al. (2018)

*For the good listener who wants to rehearse their AFNI skills.*

The data for this option are at `/scratch/psyc5171/ds001525`. The dataset is documented at [https://openneuro.org/datasets/ds001525/versions/1.1.0](https://openneuro.org/datasets/ds001525/versions/1.1.0).

This option uses the standard AFNI pipeline, but requires analyzing two different tasks (a functional localizer followed by the main task).

Replicate<sup>*</sup> the analysis of Rong, Isenberg, Sun and Hickok (2018).

Your results should include:

- A print-ready table similar to that of Table 1. 
- Statistic maps showing significant clusters as in Figure 2. It is not necessary to plot the RS/RU signal change within the ROIs, but your images should be combined into a single labeled figure. 
- For table 2, calculate the t-statistic associated with difference between the RS and RU beta values, based on the mean values within each ROI defined by the localizer. 


<sup>*</sup>You should deviate from an exact replication in the following ways:

- Rong et al. used an intermediate group-specific template during normalization. This step can control normalization error to some extent. Since constructing such templates was not covered in class, simply use a direct linear alignment to the MNI template.
- Rong et al. filtered the data at 0.008 Hz highpass. Although it is possible to control the filter parameters using a `3dbandpass` or `3dFourier` step, you can stick to using the filtering approach of including regressors at the `3dDeconvolve` stage. Using `-polort 4` will give a highpass filter of ~0.007 Hz. We can only hope that the critical signal is not inside the .001 Hz gap.

## Option 2: Reproduce Pernet et al. (2015)

*For those seeking more variability*

The data for this option are at `/scratch/psyc5171/ds000158`. The dataset is documented at [https://openneuro.org/datasets/ds000158/versions/00001](https://openneuro.org/datasets/ds000158/versions/00001).

This option requires translating methods from another software package (SPM). The steps are basically the same as those in the standard AFNI pipeline. You only need to recreate the basic group analysis on the **block** design, up to and including the section *Random-effect analyses* in the methods.

Translation hints:

- *auto-correlation was modeled using an auto-regressive matrix of order 1* means that you need to use the `3dREMLfit` program to obtain comparable results. 
- *auto-correlation was modeled using an auto-regressive matrix of order 1* means that you need to use the `3dREMLfit` program to obtain comparable results (this uses an ARMA(1,1) model). If you use an `afni_proc.py` based pipeline, add the option `-regress_reml_exec` and use the results stats BRIK files with `REML` in the name.
- *the SPM average 152 T1 image* = use the MNI template for normalization.
- The final voxel size is specified using the `-volreg_warp_dxyz` option, e.g. `-volreg_warp_dxyz 2` will produce preprocessed functional data at 2mm isotropic resolution.
- Pernet et al. aligned the T1 to MNI space using *diffeomorphic normalization using the forward deformation field computed during segmentation*. You should use the default linear alignment option. For reference, there is nonlinear warping in AFNI. The `-tlrc_NL_warp` `afni_proc.py` option will implement this, although it is not segmentation based. Nonlinear warping can greatly improve alignment across subjects, but it is also significantly slower and provides rich opportunities for things to go wrong.
- AFNI does not implement Gaussian random field theory for multiple comparison correction. Use cluster-based correction instead.
- The Eickoff atlases are known to `whereami`

Your results should include:

- A print-ready figure showing clusters significant in the vocal vs nonvocal contrast, similar to the upper left of Figure 2.
- A result table similar to Table 2. It is not necessary to calculate bootstrap confidence intervals for PSC, but you should report the mean PSC in each cluster. You can report a single value for each statistically-defined cluster, rather than subdividing as they have done for TVA.
