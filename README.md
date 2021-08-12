# OoVRE

## Working directory
    /Users/abuultjens/Google Drive/OneDrive/PhD/Bioinformatics/2021_Microbiome/Nicole_Isles

## Subsetting feature matrices

    sh subsetter.sh [feature_matrix.csv] [OUTFILE.csv] [fofn.txt]
    
    sh subsetter.sh ORIGINAL_397_16s_presence-absence.csv 353_16s_presence-absence.csv 353_cardiac_cc2-CASE-CONTROL_fofn.txt
    
## Binarising feature matrix
    
    python3 ~/github/binarise_floats/binarise_floats.py 353_16s_presence-absence.csv 353_16s_presence-absence.BIN-1 1
    
## Running random forest feature importance    
    
    python RFR_feat_imp.py target_353_cardiac_cc2-CASE-CONTROL.csv 353_16s_presence-absence.BIN-1.csv RFR_feat_imp_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL.csv
    
## Randomising target files

    python ~/github/randomise_target/randomise_target.py target_353_cardiac_cc2-CASE-CONTROL.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-1.csv
    
    
    


















