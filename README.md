# OoVRE

## Working directory
    /Users/abuultjens/Google Drive/OneDrive/PhD/Bioinformatics/2021_Microbiome/Nicole_Isles
    
## Labels

    CONTROL: 251
    CASES: 102

## Subsetting feature matrices

    sh subsetter.sh [feature_matrix.csv] [OUTFILE.csv] [fofn.txt]
    
    sh subsetter.sh ORIGINAL_397_16s_presence-absence.csv 353_16s_presence-absence.csv 353_cardiac_cc2-CASE-CONTROL_fofn.txt
    
## Binarising feature matrix
    
    python3 ~/github/binarise_floats/binarise_floats.py 353_16s_presence-absence.csv 353_16s_presence-absence.BIN-1 1
    
## Running random forest feature importance    
    
    python RFR_feat_imp.py target_353_cardiac_cc2-CASE-CONTROL.csv 353_16s_presence-absence.BIN-1.csv RFR_feat_imp_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL.csv
    
## Randomising target files

    # General command
    python randomise_target.py [target_file.csv] [target_file_RAND-1.csv]

    # Command
    python ~/github/randomise_target/randomise_target.py target_353_cardiac_cc2-CASE-CONTROL.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-1.csv
    
    # Running the command in a loop to make 100 random target files
    for NUMBER in $(seq 1 100); do
        python ~/github/randomise_target/randomise_target.py target_353_cardiac_cc2-CASE-CONTROL.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}.csv
    done
    
    
## Running analyses with randomised target files

### WD
    /home/buultjensa/Nicole_Isles/rand_353_OoVRE_relative_freq_merged
    /home/buultjensa/Nicole_Isles/rand_353_OoVRE_all_count_merged
    /home/buultjensa/Nicole_Isles/rand_353_16s_presence-absence
    
### Run the 100 random runs
    sh fofn-checker.sh ../seq_1-100.txt 
    
    for NUMBER in $(seq 1 100); do
        python RFC_replicator_CLASSIFICATION.py ../353_OoVRE_relative_freq_merged.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}.csv RFC_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all
       done    
       
### Running with actual target file

####  353_OoVRE_relative_freq_merged

    python RFC_replicator_CLASSIFICATION.py ../353_OoVRE_relative_freq_merged.csv ../target_353_cardiac_cc2-CASE-CONTROL.csv RFC_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all
    accuracy:
    [[0.73888889]]
    
#### 353_OoVRE_all_count_merged
    
    python RFC_replicator_CLASSIFICATION.py ../353_OoVRE_all_count_merged.csv ../target_353_cardiac_cc2-CASE-CONTROL.csv RFC_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all
    accuracy:
    [[0.73333333]]
    
#### 353_16s_presence-absence
    
    python RFC_replicator_CLASSIFICATION.py ../353_16s_presence-absence.csv ../target_353_cardiac_cc2-CASE-CONTROL.csv RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all

    cat RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all_balanced_accuracy.csv
    0.5735616216978245
    
    cat RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all_confusion_matrix.csv
    2532,57
    840,171
    
    TN FP
    FN TP
    
    # TN: 2532
    # TP: 171
    # FP: 57
    # FN: 840  
    
### True negatives: random vs actual    

    for NUMBER in $(seq 1 100); do
        TN=`cut -f 1 -d ',' RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all_confusion_matrix.csv | head -1`
        echo "${TP}" >> rand_353_16s_presence-absence_TP_1-100.csv
        TP=`cut -f 2 -d ',' RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all_confusion_matrix.csv | tail -1`       
        echo "${TN}" >> rand_353_16s_presence-absence_TN_1-100.csv        
        FP=`cut -f 2 -d ',' RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all_confusion_matrix.csv | head -1`
        echo "${FP}" >> rand_353_16s_presence-absence_FP_1-100.csv
        FN=`cut -f 1 -d ',' RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all_confusion_matrix.csv | tail -1`        
        echo "${FN}" >> rand_353_16s_presence-absence_FN_1-100.csv
    done        
    
### Density plots

    # Import the libraries
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    # Read in the data
    data = pd.read_csv("rand_353_16s_presence-absence_TN_1-100.csv", header=None, index_col=None)
    
    # Density Plot and Histogram of values from random data labels
    sns.distplot(data[0], hist=True, kde=True, bins=int(180/5), color = 'darkblue', hist_kws={'edgecolor':'black'},kde_kws={'linewidth': 4})
    
    # Plot main title
    plt.suptitle('True negatives from random data labels', fontsize=16)       
   
    # Plot subtitle
    plt.title('Red dashed line is from the actual data', fontsize=14)
    
    # Add X-axis label
    plt.xlabel('Number of true negatives', fontsize=14)
    
    # Add Y-axis label
    plt.ylabel('Density', fontsize=14)
    
    # Add vertical line for value from actual data labels
    plt.axvline(x=171, color='red', linestyle='dashed')
    
    # Save plot
    plt.savefig('rand_353_16s_presence-absence_TN_1-100_plot.png')
    
    # Clear plot. Needed if you try to make another plot with the same python instance
    plt.close()
    
#### Balanced accuracy
![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_balanced_accuracy_1-100_plot.png)    
    
#### True positives
![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_TP_1-100_plot.png)

#### True negatives
![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_TN_1-100_plot.png)

#### False positives
![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_FP_1-100_plot.png)

#### False negatives
![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_FN_1-100_plot.png)
    
    
      
    
    
    
    


















