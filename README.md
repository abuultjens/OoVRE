# OoVRE

## Overview of basic files for machine learning

### target file

Is a two column matrix with a header (first row) and index (the first column). The index contains the observation names and the second column contains the labels. File is comma separated. 

    # Structure of a target file (three observations)
    INDEX,CLASS
    obs-1,0
    obs-2,0
    obs-3,1
    
    # Example with your data (three observations)
    SampleID,cardiac_cc2
    13724.OoVRE001.14062012,0
    13724.OoVRE001.24062012,0
    13724.OoVRE002.09082013,1
    
### data feature matrix file

The data feature matrix file has both a header line (first row) and an index (first column). There can be an unlimited number of columns beyond the index column for n many features. The header line contains the observations and the rows below the header (first row) contain the features, Eg 16s sequences. File is comma separated.

    # Structure of a data matrix file (three observations and three features)
    INDEX,obs-1,obs-2,obs-3
    feat-1,0,0,1
    feat-2,0,1,1
    feat-3,1,1,0
    
    # Example with your data (three observations and three features)
    ID,13724.OoVRE001.14062012,13724.OoVRE001.24062012,13724.OoVRE002.09082013
    AACATAG,0,0,0
    AACATAG,0,0,0
    AACATAG,0,0,0

## Running with 353 observations (cardiac_cc2-CASE-CONTROL labels)

### Working directory
    /home/buultjensa/Nicole_Isles  

### Subsetting feature matrices

I have a script that uses the original data feature files to generate subsets feature matrices. For example, the original 16s_presence-absence.csv file has a total of 397 observations but when using the cardiac_cc2 labels there are only 353 observations. This script allows you to subset the larger original feature matrix into a subset matrix that only contains the 353 observations of interest. You just need to provide it with three inputs: the original feature matrix, an ordered list of observation names (called a "file-of-file-names" or fofn) and a name for the new subset matrix outfile.

    # General command
    sh subsetter.sh [feature_matrix.csv] [OUTFILE.csv] [fofn.txt]
    
    # Command
    sh subsetter.sh ORIGINAL_397_16s_presence-absence.csv 353_16s_presence-absence.csv 353_cardiac_cc2-CASE-CONTROL_fofn.txt
    
### Checking that the observation names match between target and data files

    sh index-checker.sh 353_16s_presence-absence.csv target_353_cardiac_cc2-CASE-CONTROL.csv 
    INDEXES ARE THE SAME   
    
### Class labels

    CONTROL: 251
    CASES: 102    
    
### Running a random forest classifier
    
I have a script that that iteratively splits the observations into train (90% of observations) and test (10% of observations) partitions 100 times. In each iteration it builds a model using the train partition and then uses that model to predict the class of the test partition. The value in doing this is that it provides a measure of how well the labels associate with the features of the data. The metrics that are used to assess the predictions are the balanced accuracy, as this takes into account any imbalances in the classes, and the confusion matrix. 
    
#### 353_16s_presence-absence
    
##### Random forest classifier

    # General command
    python RFC_replicator_CLASSIFICATION.py [data_matrix.csv] [target_label.csv] [outfile_name]

    # Command
    python RFC_replicator_CLASSIFICATION.py 353_16s_presence-absence.csv target_353_cardiac_cc2-CASE-CONTROL.csv RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all

    # Balanced accuracy
    cat RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all_balanced_accuracy.csv
    0.5735616216978245
    
    # Confusion matrix
    cat RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_ACTUAL_COR-0.0_chi2-all_confusion_matrix.csv
    2532,57
    840,171
    
    # Structure of a confusion matrix
    TN FP
    FN TP
    
    # Confusion matrix breakdown
    # TN: 2532
    # TP: 171
    # FP: 57
    # FN: 840      
    
## Running classifier with randomised target files

In order to assess how impressive the balanced accuracy and confusion matrix is, an expectation for what we would get from random chance for this data and set of labels is needed. To do this the labels are randomly reassigned to observations and the same analysis as above is rerun. 

### WD

    /home/buultjensa/Nicole_Isles/rand_353_16s_presence-absence
    
### Running the classifier with randomly reshuffled target files

A python script is used to generate 100 new target files each with a random reshuffle of the class labels.

    # General command
    python randomise_target.py [target_file.csv] [target_file_RAND-1.csv]

    # Command
    python randomise_target.py target_353_cardiac_cc2-CASE-CONTROL.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-1.csv
    
    # Running the command in a loop to make 100 random target files and run the classifier
    for NUMBER in $(seq 1 100); do
        # make the random reshuffled target file
        python randomise_target.py target_353_cardiac_cc2-CASE-CONTROL.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}.csv
        # run the classifier with the random reshuffled target file
        python RFC_replicator_CLASSIFICATION.py ../353_OoVRE_relative_freq_merged.csv target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}.csv RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all
    done             
    
### Combine outfile data to make density plots 

    # make balanced_accuracy combined outfile
    cat RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-*_COR-0.0_chi2-all_balanced_accuracy.csv > RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-1-100_COR-0.0_chi2-all_balanced_accuracy.csv

    # make combined outfiles from the confusion matrix outfiles
    for NUMBER in $(seq 1 100); do
        # make TN combined outfile
        TN=`cut -f 1 -d ',' RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all_confusion_matrix.csv | head -1`
        echo "${TP}" >> rand_353_16s_presence-absence_TP_1-100.csv       
        # make TP combined outfile
        TP=`cut -f 2 -d ',' RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all_confusion_matrix.csv | tail -1`       
        echo "${TN}" >> rand_353_16s_presence-absence_TN_1-100.csv          
        # make FP combined outfile
        FP=`cut -f 2 -d ',' RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all_confusion_matrix.csv | head -1`
        echo "${FP}" >> rand_353_16s_presence-absence_FP_1-100.csv        
        # make FN combined outfile
        FN=`cut -f 1 -d ',' RFC_data_353_16s_presence-absence_target_353_cardiac_cc2-CASE-CONTROL_RAND-${NUMBER}_COR-0.0_chi2-all_confusion_matrix.csv | tail -1`        
        echo "${FN}" >> rand_353_16s_presence-absence_FN_1-100.csv
    done        
    
### Density plots

Density plots are used to graphically show how different or similar the values from the actual labels are from what is obtained by the random reshuffles. 

#### Python script to generate density plots  

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

    # The balanced accuracy is better than what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_balanced_accuracy_1-100_plot.png)    
    
#### True positives

    # The TPs are better than what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_TP_1-100_plot.png)

#### True negatives

    # The TNs are around what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_TN_1-100_plot.png)

#### False positives

    # The FPs are worse than what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_FP_1-100_plot.png)

#### False negatives

    # The FNs are better than what is expected by random chance

![Image description](https://github.com/abuultjens/OoVRE/blob/main/rand_353_16s_presence-absence_FN_1-100_plot.png)
      
## Investigating the features driving the model

I used the binary allele frequencies to sort the features according to the largest differences between the classes. Below is a screenshot of the top features ordered horizontally by observation class. The top row is coloured to denote the class (red = control and blue = case).

![Image description](https://github.com/abuultjens/OoVRE/blob/main/Screen%20Shot%202021-08-16%20at%205.38.22%20pm.png)
    
There is a slight visual pattern that differs between the classes, with more red (zero 16s count) in the cases class while more yellow to green in the control class (non-zero 16s count).

### PCA on top features    
    
I took the top 30 features and ran PCA to reduce the dimensionality. I plotted the first two principal components and coloured the points according to observation class (orange = case, blue = control). The cases are mixed within the controls, however there are some controls that sit out away from where the cases overlap.

![Image description](https://github.com/abuultjens/OoVRE/blob/main/Screen%20Shot%202021-08-18%20at%2010.02.35%20am.png) 

This two-dimension reduction of the data is representing the interacting effects of the 30 top features and is a good graphic to try to understand how a classifier is making it's decisions. I have drawn two class decision zones on the PCA plot depicting how I think the RFC is defining it's class decision boundaries, given the confusion matrix we get with the actual labels.

![Image description](https://github.com/abuultjens/OoVRE/blob/main/Screen%20Shot%202021-08-18%20at%2010.02.35%20am%20copy.png) 

Here, the zone aimed at capturing the cases (red) does quite a good job, as most cases are densely clustered together to the left, providing a high true positive count. However, due to the overlap of cases and controls in this section, the zone also captures a lot of controls, providing a high false positive count. The zone aimed at capturing the controls (green) try's to contain as many controls as possible with few cases. In doing so it focuses on capturing the controls that are not overlapped with cases. This zone placement reduces the number of cases captured, providing a low false negative count and low true negatives, as there are many controls overlapped by cases that are in the other zone (red). This is fitting with the good TP and FN and bad FP and TN counts from the actual labels vs the simulations with random labels.
    
## Summary

The fact that the balanced accuracy and several other values from the confusion matrix are different to what is expected from random chance indicate that there is an association between the labels of case and control and the features of 16s presence/absence. When looking at the features that best associate with the classes there is a slight pattern. Using the data there are some controls that can be distinguished from other controls that are indistinguishable to cases. 
    
## 30 16s sequences that best accociate with the labels

Below are the 30 16s sequences that best accociate with the labels:

    TACGTAGGGGGCAAGCGTTATCCGGAATTACTGGGTGTAAAGGGTGCGTAGGTGGTATGGCAAGTCAGAAGTGAAAACCCAGGGCTTAACTCTGGGACTGCTTTTGAAACTGTCAGACTGGAGTGCAGGAGAGGTAAGCGGAATTCCTAG
    TACGTAGGGGGCAAGCGTTATCCGGAATTATTGGGCGTAAAGAGTACGTAGGTGGTTTTCTAAGCACGGGGTTTAAGGCAATGGCTTAACCATTGTTCGCCTTGTGAACTGGAAGACTTGAGTGCAGGAGAGGAAAGCGGAATTCCTAGT
    TACGTAGGGGGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGAGCGTAGACGGCACGGCAAGCCAGATGTGAAAGCCCGGGGCTCAACCCCGGGACTGCATTTGGAACTGCTGAGCTAGAGTGTCGGAGAGGCAAGTGGAATTCCTAG
    TACGGAGGATTCAAGCGTTATCCGGATTTATTGGGTTTAAAGGGTGCGTAGGCGGTTTGATAAGTTAGAGGTGAAATTTCGGGGCTCAACCCTGAACGTGCCTCTAATACTGTTGAGCTAGAGAGTAGTTGCGGTAGGCGGAATGTATGG
    TACGTAGGGGGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGAGCGTAGACGGCAAGGCAAGTCTGATGTGAAAACCCAGGGCTTAACCCTGGGACTGCATTGGAAACTGTCTGGCTCGAGTGCCGGAGAGGTAAGCGGAATTCCTAG
    TACGGAGGATCCAAGCGTTATCCGGATTTATTGGGTTTAAAGGGTGCGTAGGCGGTTTGATAAGTTAGAGGTGAAATACCGGGGCTCAACTCCGGAACTGCCTCTAATACTGTTGAACTAGAGAGTAGTTGCGGTAGGCGGAATGTATGG
    TACGTAGGGAGCAAGCGTTGTCCGGATTTACTGGGTGTAAAGGGTGCGTAGGCGGCTAGGCAAGTCAGACGTGAAAACCATGGGCTCAACCTGTGGACTGCGTTTGAAACTGTTTAGCTTGAGTGAAGTAGAGGTAGGCGGAATTCCCGG
    TACGTAGGGAGCGAGCGTTGTCCGGATTTACTGGGTGTAAAGGGTGCGTAGGCGGCGAGGCAAGTCAGGCGTGAAATCTATGGGCTTAACCCATAAACTGCGCTTGAAACTGTCTTGCTTGAGTGAAGTAGAGGTAGGCGGAATTCCCGG
    AACGTAGGTCACAAGCGTTGTCCGGAATTACTGGGTGTAAAGGGAGCGCAGGCGGGAGAACAAGTTGGAAGTGAAATCCATGGGCTCAACCCATGAACTGCTTTCAAAACTGTTTTTCTTGAGTAGTGCAGAGGTAGGCGGAATTCCCGG
    AACGTAGGGTGCAAGCGTTGTCCGGAATTACTGGGTGTAAAGGGAGCGCAGGCGGGAAGACAAGTTGGAAGTGAAAACCATGGGCTCAACCCATGAATTGCTTTCAAAACTGTTTTTCTTGAGTAGTGCAGAGGTAGATGGAATTCCCGG
    TACGTATGGTGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGAGCGTAGACGGAGTGGCAAGTCTGATGTGAAAACCCGGGGCTCAACCCCGGGACTGCATTGGAAACTGTCAATCTAGAGTACCGGAGAGGTAAGCGGAATTCCTAG
    TACGTATGGAGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGTGCGTAGGTGGCAGTGCAAGTCAGATGTGAAAGGCCGGGGCTCAACCCCGGAGCTGCATTTGAAACTGCTCGGCTAGAGTACAGGAGAGGCAGGCGGAATTCCTAG
    TACGTAGGGGGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGAGCGTAGACGGTTATGTAAGTCTGATGTGAAAACCCGGGGCTCAACCCCGGGACTGCATTGGAAACTATGTAACTAGAGTGTCGGAGAGGTAAGTGGAATTCCTAG
    TACGTAGGGGGCTAGCGTTATCCGGAATTACTGGGCGTAAAGGGTGCGTAGGTGGTTTCTTAAGTCAGAGGTGAAAGGCTACGGCTCAACCGTAGTAAGCCTTTGAAACTGGGAAACTTGAGTGCAGGAGAGGAGAGTGGAATTCCTAGT
    TACGGAGGATCCGAGCGTTATCCGGATTTATTGGGTTTAAAGGGTGCGTAGGCTGTTTTTTAAGTTAGAGGTGAAAGCTCGACGCTCAACGTCGAAATTGCCTCTGATACTGAGAGACTAGAGTGTAGTTGCGGAAGGCGGAATGTGTGG
    TACGGAGGATTCAAGCGTTATCCGGATTTATTGGGTTTAAAGGGTGCGTAGGCGGTTTGATAAGTTAGAGGTGAAATCCCGGGGCTTAACTCCGGAACTGCCTCTAATACTGTTAGACTAGAGAGTAGTTGCGGTAGGCGGAATGTATGG
    TACGGAGGATCCGAGCGTTATCCGGATTTATTGGGTTTAAAGGGAGCGTAGGTGGACAGTTAAGTCAGTTGTGAAAGTTTGCGGCTCAACCGTAAAATTGCAGTTGATACTGGCTGTCTTGAGTACAGTAGAGGTGGGCGGAATTCGTGG
    TACGTAGGGGGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGAGCGTAGACGGTTAAGCAAGTCTGAAGTGAAAGCCCGGGGCTCAACCCCGGTACTGCTTTGGAAACTGTTTGACTTGAGTGCAGGAGAGGTAAGTGGAATTCCTAG
    TACGTAGGGGGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGAGCGTAGACGGCATGACAAGCCAGATGTGAAAACCCAGGGCTCAACCCTGGGACTGCATTTGGAACTGCCAGGCTGGAGTGCAGGAGAGGTAAGCGGAATTCCTAG
    TACGGAGGATCCGAGCGTTATCCGGATTTATTGGGTTTAAAGGGAGCGTAGGCGGATTGTTAAGTCAGTTGTGAAAGTTTGCGGCTCAACCGTAAAATTGCAGTTGATACTGGCAGTCTTGAGTGCAGTAGAGGTGGGCGGAATTCGTGG
    TACGGAGGATCCGAGCGTTATCCGGATTTATTGGGTTTAAAGGGAGCGTAGGCGGACTATTAAGTCAGCTGTGAAAGTTTGCGGCTCAACCGTAAAATTGCAGTTGATACTGGTCGTCTTGAGTGCAGTAGAGGTAGGCGGAATTCGTGG
    TACGTAGGGGGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGAGCGTAGACGGCTCTGCAAGTCTGGAGTGAAAACCCGGGGCTCAACCCTGGGACTGCTTTGGAAACTGTGGAGCTTGAGTACCGGAGGGGTAAGCGGAATTCCTAG
    TACGGAGGGTGCGAGCGTTAATCGGAATAACTGGGCGTAAAGGGCACGCAGGCGGTGACTTAAGTGAGGTGTGAAAGCCCCGGGCTTAACCTGGGAATTGCATTTCATACTGGGTCGCTAGAGTACTTTAGGGAGGGGTAGAATTCCACG
    TACGTAGGTGGCAAGCGTTGTCCGGATTTACTGGGTGTAAAGGGCGTGCAGCCGGGCCGGCAAGTCAGATGTGAAATCTGGAGGCTTAACCTCCAAACTGCATTTGAAACTGTAGGTCTTGAGTACCGGAGAGGTTATCGGAATTCCTTG
    TACGTAGGGAGCAAGCGTTGTCCGGATTTACTGGGTGTAAAGGGTGCGTAGGCGGATTGGCAAGTCAGAAGTGAAATCCATGGGCTTAACCCATGAACTGCTTTTGAAACTGTTAGTCTTGAGTGAAGTAGAGGTAGGCGGAATTCCCGG
    TACGTAGGGAGCGAGCGTTGTCCGGATTTACTGGGTGTAAAGGGTGCGTAGGCGGATAGGCAAGTCAGTGGTGAAATCTATGGGCTTAACCCATAAACTGCCATTGAAACTGTTTATCTTGAGTGGAGTAGAGGCAGGCGGAATTCCCGG
    TACGTAGGGGGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGAGCGCAGACGGCACTGCAAGTCTGAAGTGAAAGCCCGGGGCTCAACCCCGGGACTGCTTTGGAAACTGTAGAGCTAGAGTGCTGGAGAGGCAAGCGGAATTCCTAG
    TACGGAGGATCCGAGCGTTATCCGGATTTATTGGGTTTAAAGGGTGCGTAGGCGGCCTTTTAAGTCAGCGGTGAAAGTCTGTGGCTCAACCATAGAATTGCCGTTGAAACTGGGGGGCTTGAGTATGTTTGAGGCAGGCGGAATGCGTGG
    TACGTAGGGGGCAAGCGTTATCCGGATTTACTGGGTGTAAAGGGAGCGTAGACGGATGGACAAGTCTGATGTGAAAGGCTGGGGCTCAACCCCGGGACTGCATTGGAAACTGCCCGTCTTGAGTGCCGGAGAGGTAAGCGGAATTCCTAG
    TACGGAGGATCCGAGCGTTATCCGGATTTATTGGGTTTAAAGGGAGCGTAGGCGGACGCTTAAGTCAGTTGTGAAAGTTTGCGGCTCAACCGTAAAATTGCAGTTGATACTGGGTGTCTTGAGTACAGTAGAGGCAGGCGGAATTCGTGG


















