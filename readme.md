# lfdcs2raven
Review LFDCS output in Raven Pro

## Summary

The Low-Frequency Detection and Classification System (LFDCS; see [Baumgartner & Mussoline 2011](https://www.whoi.edu/cms/files/JASMAN12952889_85804.pdf)) is a fantastic system for automatically detecting marine mammal vocalizations in large datasets. It is not perfect, however, and many analyses require an often lengthy manual review of the detector output. The custom software that is typically used for this review is written in IDL, which can be difficult for the average analyst to install and use. This R package converts LFDCS output into a format that can be more conviently reviewed using [Raven Pro sound analysis software](http://ravensoundsoftware.com/software/raven-pro/). It also contains some additional functions for adding timestamps to audio file names so that they can be read by Raven. This approach only requires an analyst to have a copy of Raven, the raw audio data, and the selection table file to review LFDCS output, rather than all of the computational infrastructure to run the detector. Hopefully this will make it much easier to involve students and collaborators in LFDCS analysis and validation.

For more information contact hansen.johnson@dal.ca

## Using the package

### Installation
```
library(devtools)
install_github('hansenjohnson/lfdcs2raven')
library(lfdcs2raven)
```
 
### Converting LFDCS autodetections to a selection table
```
lfdcs_to_raven(lfdcs_file = 'lfdcs_autodetections.csv',
               raven_file = 'raven_selections.txt',
               call_types = c(1,2,3,4,5,6),
               max_mdist = 5,
               audio_start_time = '2017-01-01 00:00:00')
```

### Adding timestamp to DMON wav files
```
# add timestamp to dmon audio files
timestamp_dmon(dmon_dir = 'data/dmon/')

# undo dmon timestamps
undo_timestamp_dmon(timestamp_file = 'data/dmon/dmon_filenames.rds')
```

## Running the LFDCS

This only applies to users that are running the LFDCS and then converting the autodetections to raven selections

1. Run the LFDCS
Create your `paramfile.txt` and run the detector using the following
```
# move to the main program directory
cd ~/Projects/Detectors/lfdcs/process

# switch from bash to tcsh shell
tcsh

# open idl
idl

# run the detector
reformat_detect_classify, paramfile.txt
```

2. Extract autodetections
Here's an example of how to extract autodetections of right whale upcalls (call types 5,6,7,8,9)
```
export_autodetections, 'lfdcs/lfdcs_file_index.nc', [5,6,7,8,9,10], 'lfdsc_autodetections.csv'
```

3. Convert LFDCS autodetections to Raven selection table
This is done in R using the package described here
```
lfdcs_to_raven(lfdcs_file = 'lfdcs_autodetections.csv',
               raven_file = 'raven_selections.txt',
               call_types = c(5,6,7,8,9,10),
               max_mdist = 3,
               audio_start_time = '2017-01-01 00:00:00')
```

## Known issues / limitations

- the `lfdcs_to_raven()` function is crude in that ignores the metadata in the header of the lfdcs autodetections file. Future versions may attempt to remedy this.

- make sure the `lfdcs_start_time` argument is set correctly. Our deployments always use the unix convention (1970-01-01 00:00:00), but others may not. You can check the LFDCS paramfile or header of the autodetection file.

- the dmon file renaming **only** works with continuous audio (i.e., gliders and not buoys)

- the timestamp extraction in `timestamp_dmon()` is also crude, as it simply pulls the first available timestamp rather than the time associated with a specific event. Limited testing has suggested this is sufficient, but there may be shortcomings

- there are no example data included with this package, as these data are large and often proprietary.

## sessionInfo

Output from my sessionInfo():
```
> sessionInfo()
R version 3.6.2 (2019-12-12)
Platform: x86_64-apple-darwin15.6.0 (64-bit)
Running under: macOS Mojave 10.14.6

Matrix products: default
BLAS:   /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/libBLAS.dylib
LAPACK: /Library/Frameworks/R.framework/Versions/3.6/Resources/lib/libRlapack.dylib

locale:
[1] en_CA.UTF-8/en_CA.UTF-8/en_CA.UTF-8/C/en_CA.UTF-8/en_CA.UTF-8

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base     

other attached packages:
[1] lfdcs2raven_0.1.0

loaded via a namespace (and not attached):
 [1] Rcpp_1.0.3        pillar_1.4.3      compiler_3.6.2    prettyunits_1.1.1 tools_3.6.2      
 [6] digest_0.6.25     testthat_2.3.2    pkgbuild_1.0.7    pkgload_1.0.2     tibble_3.0.1     
[11] lifecycle_0.2.0   pkgconfig_2.0.3   rlang_0.4.5       cli_2.0.2         rstudioapi_0.11  
[16] xfun_0.13         dplyr_0.8.5       withr_2.2.0       stringr_1.4.0     roxygen2_7.1.0   
[21] xml2_1.3.2        knitr_1.28        desc_1.2.0        vctrs_0.2.4       hms_0.5.3        
[26] tidyselect_1.0.0  rprojroot_1.3-2   glue_1.4.0        R6_2.4.1          processx_3.4.2   
[31] fansi_0.4.1       tuneR_1.3.3       callr_3.4.3       purrr_0.3.4       readr_1.3.1      
[36] magrittr_1.5      MASS_7.3-51.6     backports_1.1.6   ps_1.3.2          ellipsis_0.3.0   
[41] assertthat_0.2.1  stringi_1.4.6     signal_0.7-6      crayon_1.3.4    
```
