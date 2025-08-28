# integrative-strategy-tumor-subtypes

Este repositorio contiene un flujo de análisis  en R para la identificación de potenciales subtipos tumorales a partir del análisis de expresión diferencial de genes y splicing diferencial. Este flujo de análisis se ha aplicado al estudio de cáncer colon mediante el procesamiento de datos transcriptómicos de  muestras tumorales  provenientes del proyecto [TCGA-COAD](https://portal.gdc.cancer.gov/projects/TCGA-COAD) y muestras de tejido sano de [GTEx](https://gtexportal.org/home/)
. Los datos utilizados han sido uniformemente procesados y están libremente disponibles en el repositorio**recount3**. El enfoque de análisis es integrativo, combinando el estudio de  tanto expresión génica como isoformas, con el objetivo de identificar patrones transcriptómicos característicos de distintos subtipos tumorales.


---

## Estructura del pipeline

El análisis involucra diferentes etapas, cada una de las cuales ha sido implementada en forma independiente. Este repositorio contiene los correspondientes scripts en formato R notebook (`.Rmd`)  para su reutilización. 

Etapas de análisis:

---

### 1. Preprocesamiento y descarga de datos

#### 1.1 Descarga_Genes_Metadata.Rmd

Se trabaja con objetos de la clase RangedSummarizedExperiment (RSE). Se trata de un contenedor de Bioconductor que integra en un solo objeto las matrices de conteos, los metadatos de las muestras y la anotación genómica.

Descarga los objetos RSE para genes de TCGA-COAD y GTEx-Colon. Limpieza de duplicados, filtrado de metadatos y creación de tablas clínicas.

**Input:**

- Datos descargados desde recount3 para proyectos COAD (TCGA) y Colon (GTEx)

**Output:**

- `genMatrix.RData`: contiene `rse_gene_coad`, `rse_gene_colon` que son objetos [SummarizedExperiment](https://bioconductor.org/packages/devel/bioc/manuals/SummarizedExperiment/man/SummarizedExperiment.pdf) para muestras tumorales de COAD y COLON.
- `metadata_coad.RData`: dataframe con metadatos biológicos y clínicos de COAD (ej. sexo, edad, tejido, estado tumoral).
- `metadata_colon.RData`: dataframe con metadatos de GTEx Colon (ej. sexo, tipo de tejido, edad).
- `genes_recount.RData`: lista de genes comunes presentes en ambos proyectos dentro de recount (COAD-COLON)

#### 1.2 Descarga_Isoformas.Rmd

Carga los objetos RSE a nivel de isoformas para los mismos proyectos. Aplica los mismos criterios de limpieza y selección de muestras.

**Input:**

- `GTEx_Analysis_2017-06-05_v8_RSEMv1.3.0_transcript_expected_count.gct.gz`
- `GTEx_Analysis_v8_Annotations_SampleAttributesDS.txt`
- `TCGA_COAD_counts.tsv.gz`

**Output:**

- `GTex_Colon_iso.gct`: conteos esperados de los transcriptos, para tejido normal de colon --> REVISAR: ELI DOWNLOAD DATA
- `genesFromIso.RData`:

---

### 2. Filtrado por genes y muestras en común

#### 2.Genes_y_Muestras_en_comun.Rmd

Unifica los metadatos de TCGA y GTEx. Identifica las muestras y genes presentes tanto en las matrices de expresión génica como de isoformas.

**Input:**

- `metadata_coad.RData`
- `metadata_colon.RData`
- `GTex_Colon_iso.gct`
- `TCGA_COAD_counts.tsv.gz`
- `genes_recount.RData`
- `genesFromIso.RData`

**Output:**

- `metadataCOADunificada.RData`:  metadatos combinados de TCGA y GTEx.
- `commonGenesCOAD.RData`: genes en comun entre la matriz de expresión de genes y la matriz de expresión de isoformas

---

### 3. Unificación de matrices de conteo

#### 3.1 Unificacion_Matrices_Genes.Rmd

Filtra los objetos RSE de expresión génica para conservar solo genes y muestras comunes entre lo reportado de isoformas como de genes. Genera una matriz unificada.

**Input:**

- `genMatrix.RData`: matriz inicial de conteos de genes (rse_gene_coad, rse_gene_coad)
- `metadataCOADunificada.RData`: metadatos combinados de TCGA y GTEx.
- `commonGenesCOAD.RData`: conjunto de genes presentes en ambos proyectos.

**Output:**

- `gene_counts.RData`: matriz de conteos de genes unificada (genes × muestras comunes).

#### 3.2 Unificacion_Matrices_Isoformas.Rmd

Une las matrices de expresión de isoformas (TCGA y GTEx) con nombres armonizados.

**Input:**

- `metadataCOADunificada.RData`
- `commonGenesCOAD.RData`
- `TCGA_COAD_counts.tsv.gz`
- `GTex_Colon_iso.gct`

**Output:**

- `COAD_isoData_unificada.RData`

---

### 4. Análisis exploratorio y corrección por batch

#### 4.1 AnálisisExploratorio.Rmd

Explora la expresión génica e isoforme con PCA y filtros por baja expresión. Transforma a TPM + log2.

**Input:**

- `gene_counts.RData`
- `COAD_isoData_unificada.RData`
- `metadataCOADunificada.RData`

**Output:**

- `geneExp_filt_counts.RData`
- `isoData_filt_counts.RData`
- `geneIso_f.RData`
- `pca_res_gene.RData`, `pca_res_iso.RData`

#### 4.2 Correccion_Efecto_Batch.Rmd

Corrige por batch usando ComBat-seq y evalúa por PCA.

**Input:**

- `geneExp_filt_counts.RData`
- `isoData_filt_counts.RData`
- `metadataCOADunificada.RData`

**Output:**

- `corrected_gene_counts.RData`: matriz de conteos de genes corregida por batch.
- `corrected_iso_counts.RData`: matriz de conteos de genes corregida por batch.
- `data_pca_gene_corrected.RData`: datos PCA sobre genes corregidos.
- `data_pca_iso_corrected.RData`: datos PCA sobre isoformas corregidas.
- `pca_res_gene_corrected.RData`: resultados PCA de genes sin corrección.
- `pca_res_iso_corrected.RData`: resultados PCA de isoformas sin corrección.

---

### 5. Análisis de expresión diferencial

#### 5.1 Expresion_Diferencial_Genes.Rmd

Análisis de expresión diferencial con `edgeR`. Selección por FDR < 0.01 y |logFC| > 2.

**Input:**

- `corrected_gene_counts.RData`: matriz de conteos de genes corregida por batch.
- `metadataCOADunificada.RData`: metadatos integrados.

**Output:**

- `gen_data_deg_raw.RData`
- `res_signif` (data.frame interno)

#### 5.2 Genes_con_splicing_diferencial.Rmd

Identifica genes con splicing diferencial con `NBSplice`.

**Input:**

- `geneIso_f.RData`
- `corrected_iso_counts.RData`: matriz de isoformas corregida por batch.
- `metadataCOADunificada.RData`: metadatos integrados de TCGA-COAD y GTEx-Colon.

**Output:**

- `isoData_DSG.RData`
- `isoData_f_cpm.RData`

---

### 6. Clustering

Agrupa muestras tumorales usando matrices de expresión de genes y/o isoformas, y combina ambos resultados en un consenso.

**Input:**

- `metadataCOADunificada.RData`: metadatos integrados.
- `corrected_gene_counts.RData`: matriz de conteos de genes corregida por batch.
- `gen_data_deg_raw.RData`
- `isoData_f_cpm.RData`
- `isoData_DSG.RData`

**Output:**

- `Clustering01.RData`
- `Clustering02.RData`
- `consenso_sampleGroup.RData`

---

### 7. Análisis por subtipo

#### 7.1 Expresion_Diferencial_Genes_Subtipo_vs_Normal.Rmd

Realiza DEG para cada subtipo vs muestras normales.

**Input:**

- `metadataCOADunificada.RData`: metadatos integrados.
- `corrected_gene_counts.RData`: matriz de conteos de genes corregida por batch.
- `consenso_sampleGroup.RData`

**Output:**

- `DEGRes_SubtipeVsN.RData`: resultados de expresión diferencial para cada subtipo comparado contra muestras normales.
- `exclusiveDEGxSubtype.RData`: genes diferencialmente expresados exclusivos de cada subtipo tumoral.

#### 7.2 Expresión_Diferencial_Isoformas_Subtipo_vs_Normal.Rmd

Análisis de splicing diferencial entre subtipos y normales. Filtra por FDR y cambio absoluto.

**Input:**

- `geneIso_f.RData`
- `corrected_iso_counts.RData`: matriz de conteos de isoformas corregida por batch.
- `consenso_sampleGroup.RData`
- `metadataCOADunificada.RData`: metadatos integrados.

**Output:**

- `DSRes_SubtipeVsN.RData`: resultados del análisis de splicing diferencial para cada subtipo frente a normales.
- `exclusiveDS`: isoformas diferencialmente expresadas exclusivas de cada subtipo tumoral.

---

## Requisitos

**Versión de R recomendada:**  
R ≥ 4.2

**Paquetes utilizados:**

- `recount3`
- `SummarizedExperiment`
- `edgeR`, `limma`
- `NBSplice`
- `dplyr`, `stringr`, `tibble`, `data.table`
- `BiocParallel`
- `ggplot2`
- `multiClust`

---

## Instalación de dependencias

```r
# CRAN
install.packages(c("dplyr", "stringr", "ggplot2", "tibble", "data.table"))

# Bioconductor
if (!requireNamespace("BiocManager", quietly = TRUE)) 
    install.packages("BiocManager")

BiocManager::install(c(
  "recount3", 
  "SummarizedExperiment", 
  "edgeR", 
  "limma", 
  "NBSplice", 
  "BiocParallel"
))
```
