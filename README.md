# OoVRE

## Working directory
    /Users/abuultjens/Google Drive/OneDrive/PhD/Bioinformatics/2021_Microbiome/Nicole_Isles
    
## Labels

    CONTROL: 251
    CASES: 102

## Subsetting feature matrices

    # General command
    sh subsetter.sh [feature_matrix.csv] [OUTFILE.csv] [fofn.txt]
    
    # Command
    sh subsetter.sh ORIGINAL_397_16s_presence-absence.csv 353_16s_presence-absence.csv 353_cardiac_cc2-CASE-CONTROL_fofn.txt
    
### Checking that the observation names match between target and data files

    sh ~/github/index-checker/index-checker.sh 353_16s_presence-absence.csv target_353_cardiac_cc2-CASE-CONTROL.csv 
    INDEXES ARE THE SAME
    
## Running random forest classifier feature importance    
    
    python RFC_feat_imp.py target_353_cardiac_cc2-CASE-CONTROL.csv 353_16s_presence-absence.BIN-1.csv RFC_feat_imp_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL.csv
    
## Randomising target files

    # General command
    python randomise_target.py [target_file.csv] [target_file_RAND-1.csv]

    # Command
    python randomise_target.py target_353_cardiac_cc2-CASE-CONTROL.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-1.csv
    
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

    for NUMBER in $(seq 1 100); do
        python RFC_replicator_CLASSIFICATION.py ../353_OoVRE_relative_freq_merged.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}.csv RFC_data_353_16s_presence-absence.BIN-1_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all
    done    
       
### Running with actual target file
    
#### 353_16s_presence-absence

##### Logistic regression

    cat LGR_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all_balanced_accuracy.csv
    0.7018554876658036

    cat LGR_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all_confusion_matrix.csv
    2131,458
    424,587
    
##### Random forest classifier

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
    
### Combine outfile data to make density plots 

    cat RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-*_COR-0.0_chi2-all_balanced_accuracy.csv > RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-1-100_COR-0.0_chi2-all_balanced_accuracy.csv

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

    # much better than what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_balanced_accuracy_1-100_plot.png)    
    
#### True positives

    # much better than what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_TP_1-100_plot.png)

#### True negatives

    # around what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_TN_1-100_plot.png)

#### False positives

    # worse than what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_FP_1-100_plot.png)

#### False negatives

    # better than what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_FN_1-100_plot.png)
    
    
      
    
    
    
    


















