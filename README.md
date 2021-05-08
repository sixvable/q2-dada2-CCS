# q2-dada2-CCS

Implement `qiime dada denoise_ccs` in [q2-dada2 plugin](https://github.com/qiime2/q2-dada2) for denoising Pacbio CCS long reads.

Test on **qiime2-2021.4** 

## Usage

1. Overwrite the file **\_\_init\_\_.py**, **plugin_setup.py**, and **\_denoise.py** in */path/to/your/conda/envs/qiime2-version/lib/python3.6/site-packages/q2_dada2/* with the file in this repo.
2. put a new file **run_dada_ccs.R** in */path/to/your/conda/envs/qiime2-version/bin* and use command `chmod u+x ./run_dada_ccs.R` to make sure you have execution authority.
3. use command `qiime dev refresh-cache` to refresh your QIIME2 environment.

## Detail 

Changing following files in ***/path/to/your/conda/envs/qiime2-version/lib/python3.8/site-packages/q2_dada2/***

1. **\_\_init\_\_.py** : import a new function `denoise_ccs` from **\_denoise.py**

2. **plugin_setup.py** : 

   1. Register the new function `denoise_ccs` based on `denoise_pyro`
   
   2. Add following explicit parameters :
      - ***front*** and ***adapter*** for primer removing
      - ***max_mismatch*** and ***indels*** for primer matching 
      - ***min_len*** to filter the min length of CCS reads
   
   3. Change the default value of following explicits parameters :
      + ***trunc_len*** : default **0**
      + ***min_fold_parent_over_abundance*** from **1** to **3.5**
      + ***n_reads_learn*** from **250000** to **1000000** (same with `denoise_single`)

3. **\_denoise.py** : 
   1. define the new function `denoise_ccs` based on `denoise_pyro` and `_denoise_single` :
   
      1. add new parameters :
         - ***front*** and ***adapter*** : string
         - ***max_mismatch*** : interger, default  **2**
         - ***indels*** : boolean, default **False**
         - ***min_len*** : interger, default  **20**
   
      2. change the default value of following parameters :
         - ***trunc_len*** : default **0**
         - ***min_fold_parent_over_abundance*** from **1** to **3.5**
         - ***n_reads_learn*** from **250000** to **1000000** (same with `denoise_single`)
   
      3. remove implicit parameters ***homopolymer_gap_penalty*** and ***band_size*** , re-define their default values in **run_dada_ccs.R**
   
      4. change the command line `run_dada_ccs.R` to work with previous changes :
         - add new temporary directory *nop_fp* for *PRIMER REMOVING* step 
         - change the path of  temporary directory *filt_fp*
         - add new arguments in `run_dada_ccs.R` for the new parameters in `denoise_ccs`
   
      5. return the stats log of primer removing step to function `_denoise_helper`

   2. change the function `_denoise_helper` for exposing the *PRIMER REMOVING* step stat in final **denoise_stats.qza**, including *primer-removed* and *percentage of input primer-removed*
   
   3. add new parameters to the `_valid_inputs` for paramters validation:
      - ***max_mismatch*** : _WHOLE_NUM
      - ***min_len*** : _WHOLE_NUM
      - ***front***, ***adapter*** and ***indels*** : _SKIP
   

Then add a new Rscript *run_dada_ccs.R* based on *run_dada_single.R* in `/path/to/your/conda/envs/qiime2-version/bin`


  1. assign the new arguments and reorder all the arguments :
     - ***primer.removed.dir*** for the path of temporary directory in *PRIMER REMOVING* step , corresponding to *nop_fp* in `denoise_ccs`
     - ***primerF*** and ***primerR*** for primer sequences, corresponding to ***front*** and ***adapter*** in `denoise_ccs`
     - ***maxMismatch*** and ***indels*** for primer matching, corresponding to ***max_mismatch*** and ***indels*** in `denoise_ccs`
     - ***minLen** for min length CCS reads filtering, corresponding to ***min_len*** in `denoise_ccs`
   
  2. add a new *PRIMER REMOVING* step before *TRIM AND FILTER* step :

  3. modify the *TRIM AND FILTER* step :
     1. change the default value of implicit parameters *rm.phix* from *TRUE* to *FALSE* in function `filterAndTrim`  
     2. add paramters in function `filterAndTrim` :
        - add *minLen* = *minLen*
        - *minQ* = *3*
      
  4. modify the *LEARN EROOR RATES* step :
     1. change the default value of following implicit parameters :
        - *errorEstimationFunction* from *dada2:::loessErrfun* to *dada2:::PacBioErrfun*
        - add an explicit default value of *BAND_SIZE* to *32*
        - remove the *HOMOPOLYMER_GAP_PENALTY* parameters so its default value change to *NULL*

  5. modify the *REPORT READ FRACTIONS THROUGH PIPELINE* step  to add statistics of *PRIMER REMOVING* step in **denoise_stats.qza**

Enter `qiime dev refresh-cache` after changes those files.

Note: Still lack of auto test of those new parameters so make sure your input is valid.

All parameters used in those files are referenced from [Callahan et al., 2019](https://doi.org/10.1093/nar/gkz569) and https://github.com/benjjneb/LRASManuscript

PR to [qiime/q2-dada2](https://github.com/qiime2/q2-dada2/pull/135 "add support for Pacbio CCS reads"), waiting for merging.