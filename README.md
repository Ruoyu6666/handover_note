# Handover Note
## 0) Preparation
1. Project description *Genetic imputation of multi-omics profiles for patient stratification and phenotype prediction*
2. Paper [Distinct genetic liability profiles define clinically relevant patient strata across common diseases](https://www.nature.com/articles/s41467-024-49338-2), for which the code is available in the [CASTom-iGEx](https://github.com/zillerlab/CASTom-iGEx/tree/master) repository.
3. Read [HPC Documentation](https://palma.uni-muenster.de/documentation/) to set up the HPC environment. 
First the [Registration](https://palma.uni-muenster.de/documentation/quick-intro/registration/) and [SSH Access](https://palma.uni-muenster.de/documentation/quick-intro/ssh-access/) section to access **PALMA**. Then read the other sections for further use.

## 1) Implement the imputation of gene expression pipeline managed with Nextflow
In the project *Genetic imputation of multi-omics profiles for patient stratification and phenotype prediction*, the goal of Task 1.1 (Aim 1) is to implement an imputation framework from genotype data. 
The [eCASTom-iGEx](https://github.com/Ruoyu6666/eCASTom-iGEx) repository is based on the [genetic data processing guide](https://github.com/zillerlab/CASTom-iGEx/wiki/Processing-genetic-data-to-work-with-CASTom%E2%80%90iGEx), which prepares genetic data for use with PriLer. The new repository manages this processing and imputation pipeline using **Nextflow** on top of the guide.

Each branch includes a README.md file with setup instructions:
- Branch `Master`: draft.
- Branch `nextflow`: how to run the Nextflow pipeline on Palma.
- Branch `apptainer`: how to run the Nextflow pipeline using containers on Palma. Unlike the `nextflow` branch, where Plink and the R packages are installed in the home directory, in this branch they are packed in two containers separately. 

## 2) Note regarding repo [CASTom-iGEx](https://github.com/zillerlab/CASTom-iGEx/tree/master).

### Module 1 [Model Training](https://github.com/zillerlab/CASTom-iGEx/tree/master/Software/model_training)
Task 1.2 of Aim 1 focuses on training modality-specific (transcriptome, proteome and metabolome) prediction models using harmonized SNP datasets. The training process of PriLer in the **proteome** setting is similar: here, protein abundance is predicted instead of gene expression. Therefore, it is helpful to understand the model training process. For the **metabolome**, it's more complex.

(Paper recommendation for protein imputation [OmicsPred](https://www.nature.com/articles/s41586-023-05844-9))

There is a [`README.md`](https://github.com/zillerlab/CASTom-iGEx/tree/master/Software/model_training) file describing the steps of model training. Supplementary Fig. 2 in the [Supplementary Information](https://static-content.springer.com/esm/art%3A10.1038%2Fs41467-024-49338-2/MediaObjects/41467_2024_49338_MOESM1_ESM.pdf) provides a visualisation of the model training overview.

For additional personal notes on this part, please refer to [`model_training.md`](https://github.com/Ruoyu6666/handover_note/blob/main/model_training.md) in this repository.


### Module 2 [Model Prediction](https://github.com/zillerlab/CASTom-iGEx/tree/master/Software/model_prediction)
##

### 3) Ideas regarding interdediate phenotyes convergence
