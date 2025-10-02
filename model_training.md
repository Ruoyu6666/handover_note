# Training prediction model with PriLer

## 1. Paths to input Files
This part provides the input file paths for model training, which I infer from the [bash scripts](https://github.com/zillerlab/CASTom-iGEx_Paper/tree/main/Application/PriLer/GTEx_v6p) for model training. Better get further confirmed.

- **Gene expression matrix** (*--geneExp_file*): preprocessed gene expression (genes x samples). First column refers to gene names (ensembl annotation or HUGO nomenclature)
***Path***:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/RNAseq_data/${tissue}/RNAseq_`

- **Genotype matrix** (*--genoDat_file*): dosages for each chromosome (compressed txt) without variants name/position (variants x samples). *NOTE: the file must end with chr<>_matrix.txt.gz* \
***Path***:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/Genotype_data/Genotype_dosage_chr{chr}_matrix.txt.gz`

- **Genotype info matrix** (*--VarInfo_file*): contains variants position, name and other info, must contain columns `CHR` and `POS` and match with Genotype matrix. *NOTE: the file must end with  chr<>.txt* \
***Path***:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/Genotype_data/Genotype_VariantsInfo_CMC-PGCgwas-CADgwas_`

- **Covariate matrix** (*--covDat_file*): columns must contain `Individual_ID`, `genoSample_ID` and `RNASample_ID` to match genotype and gene expression plus covariates to be used in the regression model. Column `Dx` (0 control 1 case) is optional as well as it's usage. *Note: Samples in genotype and gene expression matrix are filtered based on covariate matrix* \
***Path***:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/Covariates/${tissue}/covariates_EuropeanSamples.txt`

- **Prior matrix** (*--priorDat_file*): prior information for variants (variants x prior features). It doesn't include variant name and MUST match genotype matrix. The columns can be binary (one-hot encoding for intersection) or continuous.  \
***Path***:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/OUTPUT_SCRIPTS_v2/priorMatrix_*`

- **List heritable genes** (*--geneList_file*): usually obtained from TWAS heritable analysis: list of genes, match external_gene_name or ensembl_gene_id. Set of heritable genes being regulated by cis-variants. \
***Path***:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/TWAS/GTEx_v7/list_heritableGenes_${tissue}.txt` 

- **Gene annotation files of TSS and position** (*--biomartTSS_file --biomartGenePos_file*) obtained using *PrepareData_biomart_TSS.R* script. Possibility of recomputing or use provided fixed version \
***Path***:`${CASTom-iGEx_fold}refData/hg19.ENSEMBL_geneTSS_biomart_correct.txt`

## 2. Conda Environment
Prefer to use conda for package management. The following steps are for setting up conda environment:
```bash
# Start a job
salloc -c 2 --mem-per-cpu 16G -p normal -t 04:00:00 # for example
module --force purge
# Load Miniconda
module load palma/2022a Miniconda3/4.12.0

# Create a conda envrionment named castom
conda create -n castom -c conda-forge -c bioconda r-base r-essentials jq r-biocmanager
conda init
conda activate castom

# Install required packages
conda install bioconda::bioconductor-biomart
.... 
                      
```
Install the other required packages using the [R script](https://github.com/zillerlab/CASTom-iGEx/blob/master/Software/install_requirements.R).

## 3. Model Training
### Pre-processing:
```bash
f=/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/
git_fold=/home/r/rguo/scripts/castom-igex/ # the path of CASTom-iGEx folder
t=Brain_Cortex # as an example
```
Bash Script:
https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/preProcessing_data_allTissues.sh


### Step 1
https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/ElNet_withPrior_part1_200kb_tissue.sh


### Step 2
An example bash script: https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/ElNet_withPrior_part2_200kb_tissue_PGCgwas1e-2.sh
```bash
Rscript ${git_fold}Software/model_training/PriLer_part2_run.R \
	--covDat_file ${f}INPUT_DATA/Covariates/${t}/covariates_EuropeanSamples.txt \
	--genoDat_file ${f}INPUT_DATA/Genotype_data/Genotype_dosage_ \
	--geneExp_file ${f}INPUT_DATA/RNAseq_data/${t}/RNAseq_filt.txt \
	--ncores 15 \
	--outFold ${f}OUTPUT_SCRIPTS_v2/${t}/200kb/PGC_GWAS_bin1e-2/ \
    --InfoFold ${f}OUTPUT_SCRIPTS_v2/${t}/ \
    --functR ${git_fold}Software/model_training/PriLer_functions.R \
    --part1Res_fold ${f}OUTPUT_SCRIPTS_v2/${t}/200kb/ \
    --priorDat_file ${f}OUTPUT_SCRIPTS_v2/priorMatrix_ \
    --priorInf ${priorInd[@]} \
    --E_set 4 5 6 7 8 9 10 11 12 13 14 15 17.5 20 25
```

### Step 3
An example bash script: https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/ElNet_withPrior_part3_200kb_tissue_PGCgwas1e-2.sh

```bash
Rscript ${git_fold}Software/model_training/PriLer_part3_run.R \
    --covDat_file ${f}INPUT_DATA/Covariates/${t}/covariates_EuropeanSamples.txt \
    --genoDat_file ${f}INPUT_DATA/Genotype_data/Genotype_dosage_ \
    --geneExp_file  ${f}INPUT_DATA/RNAseq_data/${t}/RNAseq_filt.txt \
    --ncores 30 \
    --outFold  ${f}OUTPUT_SCRIPTS_v2/${t}/200kb/PGC_GWAS_bin1e-2/ \
    --InfoFold  ${f}OUTPUT_SCRIPTS_v2/${t}/ \
    --functR ${git_fold}Software/model_training/PriLer_functions.R \
    --part2Res_fold ${f}OUTPUT_SCRIPTS_v2/${t}/200kb/PGC_GWAS_bin1e-2/ \
    --priorDat_file ${f}OUTPUT_SCRIPTS_v2/priorMatrix_ \
    --priorInf ${priorInd[@]}
```

### Step 4
An example bash script: https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/ElNet_withPrior_part4_200kb_tissue_PGCgwas1e-2.sh