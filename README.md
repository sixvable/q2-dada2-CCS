# q2-dada2-CCS

make q2-dada2 support Pacbio CCS

test at **qiime2-2020.8**

by changing those files in `/path/to/your/conda/envs/qiime2-version/lib/python3.6/site-packages/q2_dada2/`

*\_\_init\_\_.py* : add a new function `denoise_ccs`

*plugin_setup.py* : 

  1. register the new function `q2_dada2.denoise_ccs` 
  2. add some explicit parameters such as `min_len` or change the default explicit value such as `min_fold_parent_over_abundance` to **3.5**

*\_denoise.py* : 

  1. add new parameters to the `_valid_inputs`.
  2. add new `denoise_ccs` and `_denoise_ccs`(which can be integrated into `denoise_ccs`).
  3. add new parameters and change the default value.

then add a new Rscript in `/path/to/your/conda/envs/qiime2-version/bin`

*run_dada_ccs.R*(which is modified by *run_dada_single.R*) : 

  1. add new parameters such as `minLen`
  2. change some implicit in function `filterAndTrim`: 
  
      1. *rm.phix*=*FALSE*
      2. add *minLen*=*minLen*, *minQ*=*3*
      
  3. change some implicit in function `filterAndTrim`: add *errorEstimationFunction*=*dada2:::PacBioErrfun*

All parameters used in those files are referenced from [Callahan et al., 2019](https://doi.org/10.1093/nar/gkz569) and https://github.com/benjjneb/LRASManuscript
