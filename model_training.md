# Training prediction model with PriLer

## Paths to input Files
- **Gene expression matrix** (*--geneExp_file*): preprocessed gene expression (genes x samples). First column refers to gene names (ensembl annotation or HUGO nomenclature). \
*Path*:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/RNAseq_data/${tissue}/RNAseq_`

- **Genotype matrix** (*--genoDat_file*): dosages for each chromosome (compressed txt) without variants name/position (variants x samples). *NOTE: the file must end with chr<>_matrix.txt.gz* \
*Path*:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/Genotype_data/Genotype_dosage_chr{chr}_matrix.txt.gz`

- **Genotype info matrix** (*--VarInfo_file*): contains variants position, name and other info, must contain columns `CHR` and `POS` and match with Genotype matrix. *NOTE: the file must end with  chr<>.txt* \
*Path*:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/Genotype_data/Genotype_VariantsInfo_CMC-PGCgwas-CADgwas_`

- **Covariate matrix** (*--covDat_file*): columns must contain `Individual_ID`, `genoSample_ID` and `RNASample_ID` to match genotype and gene expression plus covariates to be used in the regression model. Column `Dx` (0 control 1 case) is optional as well as it's usage. *Note: Samples in genotype and gene expression matrix are filtered based on covariate matrix* \
*Path*:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/Covariates/${tissue}/covariates_EuropeanSamples.txt`

- **Prior matrix** (*--priorDat_file*): prior information for variants (variants x prior features). It doesn't include variant name and MUST match genotype matrix. The columns can be binary (one-hot encoding for intersection) or continuous.  \
*Path*:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/OUTPUT_SCRIPTS_v2/priorMatrix_*`

- **List heritable genes** (*--geneList_file*): usually obtained from TWAS heritable analysis: list of genes, match external_gene_name or ensembl_gene_id. Set of heritable genes being regulated by cis-variants. \
*Path*:`/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/INPUT_DATA/TWAS/GTEx_v7/list_heritableGenes_${tissue}.txt` 

- **Gene annotation files of TSS and position** (*--biomartTSS_file --biomartGenePos_file*) obtained using *PrepareData_biomart_TSS.R* script. Possibility of recomputing or use provided fixed version
*Path*:`${CASTom-iGEx_fold}refData/hg19.ENSEMBL_geneTSS_biomart_correct.txt`

## Conda Environment
```bash
# Start a job
salloc -c 2 --mem-per-cpu 16G -p normal -t 04:00:00 
module --force purge
module load palma/2022a Miniconda3/4.12.0

# Create a conda envrionment
conda create -n castom -c conda-forge -c bioconda r-base r-essentials jq r-biocmanager
conda init
conda activate castom

# Install required packages
conda install bioconda::bioconductor-biomart
....                                                    
```


## Workflow
### Pre-processing:
https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/preProcessing_data_allTissues.sh

```bash
# module load palma/2023b GCC/13.2.0 R/4.4.1
f=/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/PriLer_PROJECT_GTEx/
#f=/scratch/tmp/rguo/CASTOMiGEx/PriLer_PROJECT_GTEx/
git_fold=/home/r/rguo/scripts/castom-igex/
t=Brain_Cortex # as an example
```


```bash
Rscript ${git_fold}Software/model_training/preProcessing_data_run.R  \
	--geneExp_file ${f}INPUT_DATA/RNAseq_data/${t}/RNAseq_norm.txt.gz\
	--geneList_file ${f}INPUT_DATA/TWAS/GTEx_v7/list_heritableGenes_${t}.txt \
	--VarInfo_file ${f}INPUT_DATA/Genotype_data/Genotype_VariantsInfo_matched_PGCgwas-CADgwas_ \
	--biomartGenePos_file ${git_fold}refData/hg19.ENSEMBL_genes_biomart.txt \
	--biomartTSS_file ${git_fold}refData/hg19.ENSEMBL_geneTSS_biomart_correct.txt \
	--outFold ${f}OUTPUT_SCRIPTS_v2/${t}/ \
	--outFold_geneExp ${f}INPUT_DATA/RNAseq_data/${t}/ \
	--outFold_snps ${f}OUTPUT_SCRIPTS_v2/
```

### Step 1
https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/ElNet_withPrior_part1_200kb_tissue.sh

```bash
mkdir -p ${f}PriLer_PROJECT_GTEx/OUTPUT_SCRIPTS_v2/${t}/200kb

for i in $(seq 22)
do
	echo 'chr' $i

	Rscript ${git_fold}Software/model_training/PriLer_part1_run.R \
		--curChrom chr$i \
		--covDat_file ${f}INPUT_DATA/Covariates/${t}/covariates_EuropeanSamples.txt \
		--genoDat_file ${f}INPUT_DATA/Genotype_data/Genotype_dosage_ \
		--geneExp_file ${f}INPUT_DATA/RNAseq_data/${t}/RNAseq_filt.txt \
		--ncores 30 \
		--outFold ${f}OUTPUT_SCRIPTS_v2/${t}/200kb/ \
		--InfoFold ${f}OUTPUT_SCRIPTS_v2/${t}/ \
		--functR ${git_fold}Software/model_training/PriLer_functions.R 
done

```

### Step 2
https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/ElNet_withPrior_part2_200kb_tissue_PGCgwas1e-2.sh

```bash
priorInd=$(awk '{print $1}' ${f}OUTPUT_SCRIPTS_v2/${t}/priorName_PGCgwas_withIndex.txt)

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
    --E_set 5 7
```


### Step 3
https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/ElNet_withPrior_part3_200kb_tissue_PGCgwas1e-2.sh

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
https://github.com/zillerlab/CASTom-iGEx_Paper/blob/main/Application/PriLer/GTEx_v6p/ElNet_withPrior_part4_200kb_tissue_PGCgwas1e-2.sh
```bash
for i in $(seq 22)
do
	echo 'chr' $i

	${git_fold}Software/model_training/PriLer_part4_run.R \
		--curChrom chr${i} \
		--covDat_file ${f}INPUT_DATA/Covariates/${t}/covariates_EuropeanSamples.txt \
		--genoDat_file ${f}INPUT_DATA/Genotype_data/Genotype_dosage_ \
		--geneExp_file ${f}INPUT_DATA/RNAseq_data/${t}/RNAseq_filt.txt \
		--ncores 32 \
		--outFold ${f}OUTPUT_SCRIPTS_v2/${t}/200kb/PGC_GWAS_bin1e-2/ \
		--InfoFold ${f}OUTPUT_SCRIPTS_v2/${t}/ \
		--functR ${git_fold}Software/model_training/PriLer_functions.R \
		--part1Res_fold ${f}OUTPUT_SCRIPTS_v2/${t}/200kb/ \
		--part2Res_fold ${f}OUTPUT_SCRIPTS_v2/${t}/200kb/PGC_GWAS_bin1e-2/ \
		--part3Res_fold ${f}OUTPUT_SCRIPTS_v2/${t}/200kb/PGC_GWAS_bin1e-2/ \
		--priorDat_file ${f}OUTPUT_SCRIPTS_v2/priorMatrix_ \
		--priorInf ${priorInd[@]}
done
```