/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/CAD_UKBB/eQTL_PROJECT/INPUT_DATA_GTEx/CAD/Covariates/UKBB
Is it phe

/cloud/wwu1/h_fungenpsy/AGZiller_data/CASTOMiGEx/CAD_UKBB/eQTL_PROJECT/OUTPUT_GTEx

## 1) Predict gene expression
```bash
./Priler_predictGeneExp_run.R \
    --genoDat_file \
    --covDat_file \
    --cis_thres (default = 200000) \
    --outFold \
    --outTrain_fold  \
    --InfoFold \
    --no_zip (default F)
```

## 2) Small dataset: T-scores and Pathway-scores computation
```bash
./Tscore_PathScore_diff_run.R \
    --input_file \
    --reactome_file (default = NULL)\
    --GOterms_file (default = NULL) \
    --originalRNA (default = F) \
    --thr_reliableGenes (default = c(0.01, 0)) \
    --covDat_file \
    --nFolds (default = 20) \
    --outFold
```

## 3) Small dataset: Association with phenotype of T-score and pathways
```bash
./pheno_association_smallData_run.R \
    --reactome_file (default = NULL) \
    --GOterms_file (default = NULL) \
    --sampleAnn_file \
    --thr_reliableGenes (default = c(0.01, 0)) \
    --covDat_file \
    --phenoDat_file \
    --names_file \
    --phenoAnn_file \
    --cov_corr (default = T) \
    --ncores (default = 0) \
    --geneAnn_file \
    --functR ./pheno_association_functions.R \
    --outFold
```
