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
    
    
## Running analyses with randomised target files

### WD
    /home/buultjensa/Nicole_Isles/rand_353_OoVRE_relative_freq_merged
    /home/buultjensa/Nicole_Isles/rand_353_OoVRE_all_count_merged
    /home/buultjensa/Nicole_Isles/rand_353_16s_presence-absence
    
### CMDs
    sh fofn-checker.sh ../seq_1-100.txt 
    
    for TAXA in $(cat $1); do
        python RFC_replicator_CLASSIFICATION.py ../353_OoVRE_relative_freq_merged.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-${TAXA}.csv RFC_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL_RAND-${TAXA}_COR-0.0_chi2-all
       done
       
       
       
    # using BIN-1
    python RFC_replicator_CLASSIFICATION.py ../353_16s_presence-absence.BIN-1.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-${TAXA}.csv RFC_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL_RAND-${TAXA}_COR-0.0_chi2-all
       
### Running with actual target file

    #  353_OoVRE_relative_freq_merged

    python RFC_replicator_CLASSIFICATION.py ../353_OoVRE_relative_freq_merged.csv ../target_353_cardiac_cc2-CASE-CONTROL.csv RFC_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all
    accuracy:
    [[0.73888889]]
    
    # 353_OoVRE_all_count_merged
    
    python RFC_replicator_CLASSIFICATION.py ../353_OoVRE_all_count_merged.csv ../target_353_cardiac_cc2-CASE-CONTROL.csv RFC_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all
    accuracy:
    [[0.73333333]]
    
    # 353_16s_presence-absence
    
    python RFC_replicator_CLASSIFICATION.py ../353_16s_presence-absence.csv ../target_353_cardiac_cc2-CASE-CONTROL.csv RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all
    accuracy:
    [[0.75083333]]   
    
    cat RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all_confusion_matrix.csv
    
    # 353_16s_presence-absence.BIN-1
    # actually using BIN-1    
    
    
### Density plot

    # Import the libraries
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # Density Plot and Histogram of values from random data labels
    sns.distplot(data[0], hist=True, kde=True, bins=int(180/5), color = 'darkblue', hist_kws={'edgecolor':'black'},kde_kws={'linewidth': 4})
    
    # Add vertical line for value from actual data labels
    plt.axvline(x=0.85)
    
    # Display plot
    plt.show()
    
### True negatives: random vs actual    
    for NUMBER in $(seq 1 100); do
       TN=`cut -f 2 -d ',' RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all_confusion_matrix.csv | tail -1`
       echo "${TN}" >> rand_353_16s_presence-absence_TN_1-100.csv
    done    
    
    
      
    
    
    
    


















