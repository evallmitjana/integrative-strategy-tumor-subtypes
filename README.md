\# integrative-strategy-tumor-subtypes



Este repositorio contiene un pipeline reproducible en R para el análisis de expresión diferencial y exploración transcriptómica entre muestras tumorales de colon (proyecto TCGA-COAD) y muestras normales de colon (GTEx), utilizando datos descargados desde el recurso \*\*recount3\*\*. El enfoque es en capas, abordando tanto expresión génica como isoformas, con el objetivo de identificar patrones transcriptómicos característicos de distintos subtipos tumorales.



---



\## Estructura del pipeline



El análisis está organizado en múltiples scripts `.Rmd`, cada uno correspondiente a una etapa del flujo de trabajo:



---



\### 1. Preprocesamiento y descarga de datos



\#### 1.1 Descarga\_Genes\_Metadata.Rmd



Descarga los objetos RSE para genes de TCGA-COAD y GTEx-Colon. Limpieza de duplicados, filtrado de metadatos y creación de tablas clínicas.



\*\*Input:\*\*



\- Datos descargados desde recount3 para proyectos COAD (TCGA) y Colon (GTEx)



\*\*Output:\*\*



\- `rse\_gene\_coad`, `rse\_gene\_colon` (SummarizedExperiment)

\- `metadata\_coad.RData`

\- `metadata\_colon.RData`

\- `genMatrix.RData`

\- `genes\_recount.RData` (genes en común)



\#### 1.2 Descarga\_Isoformas.Rmd



Carga los objetos RSE a nivel de isoformas para los mismos proyectos. Aplica los mismos criterios de limpieza y selección de muestras.



\*\*Input:\*\*



\- `GTEx\_Analysis\_2017-06-05\_v8\_RSEMv1.3.0\_transcript\_expected\_count.gct.gz`

\- `GTEx\_Analysis\_v8\_Annotations\_SampleAttributesDS.txt`

\- `TCGA\_COAD\_counts.tsv.gz`



\*\*Output:\*\*



\- `GTex\_Colon\_iso.gct`

\- `genesFromIso.RData`



---



\### 2. Filtrado por genes y muestras en común



\#### 2.Genes\_y\_Muestras\_en\_comun.Rmd



Unifica los metadatos de TCGA y GTEx. Identifica las muestras y genes presentes tanto en las matrices de expresión génica como de isoformas.



\*\*Input:\*\*



\- `metadata\_coad.RData`

\- `metadata\_colon\_v1.RData`

\- `GTex\_Colon\_iso.gct`

\- `TCGA\_COAD\_counts.tsv.gz`

\- `genes\_recount.RData`

\- `genesFromIso.RData`



\*\*Output:\*\*



\- `metadataCOADunificada.RData`

\- `commonGenesCOAD.RData`



---



\### 3. Unificación de matrices de conteo



\#### 3.1 Unificacion\_Matrices\_Genes.Rmd



Filtra los objetos RSE de expresión génica para conservar solo genes y muestras comunes. Genera una matriz unificada.



\*\*Input:\*\*



\- `genMatrix.RData`

\- `metadataCOADunificada.RData`

\- `commonGenesCOAD.RData`



\*\*Output:\*\*



\- `gene\_counts.RData`



\#### 3.2 Unificacion\_Matrices\_Isoformas.Rmd



Une las matrices de expresión de isoformas (TCGA y GTEx) con nombres armonizados.



\*\*Input:\*\*



\- `metadataCOADunificada.RData`

\- `commonGenesCOAD.RData`

\- `TCGA\_COAD\_counts.tsv.gz`

\- `GTex\_Colon\_iso.gct`



\*\*Output:\*\*



\- `COAD\_isoData\_unificada.RData`



---



\### 4. Análisis exploratorio y corrección por batch



\#### 4.1 AnálisisExploratorio.Rmd



Explora la expresión génica e isoforme con PCA y filtros por baja expresión. Transforma a TPM + log2.



\*\*Input:\*\*



\- `gene\_counts.RData`

\- `COAD\_isoData\_unificada.RData`

\- `metadataCOADunificada.RData`



\*\*Output:\*\*



\- `geneExp\_filt\_counts.RData`

\- `isoData\_filt\_counts.RData`

\- `geneIso\_f.RData`

\- `pca\_res\_gene.RData`, `pca\_res\_iso.RData`



\#### 4.2 Correccion\_Efecto\_Batch.Rmd



Corrige por batch usando ComBat-seq y evalúa por PCA.



\*\*Input:\*\*



\- `geneExp\_filt\_counts.RData`

\- `isoData\_filt\_counts.RData`

\- `metadataCOADunificada.RData`



\*\*Output:\*\*



\- `corrected\_gene\_counts.RData`

\- `corrected\_iso\_counts.RData`

\- `data\_pca\_gene\_corrected.RData`

\- `data\_pca\_iso\_corrected.RData`

\- `pca\_res\_gene\_corrected.RData`

\- `pca\_res\_iso\_corrected.RData`



---



\### 5. Análisis de expresión diferencial



\#### 5.1 Expresion\_Diferencial\_Genes.Rmd



Análisis de expresión diferencial con `edgeR`. Selección por FDR < 0.01 y |logFC| > 2.



\*\*Input:\*\*



\- `corrected\_gene\_counts.RData`

\- `metadataCOADunificada.RData`



\*\*Output:\*\*



\- `gen\_data\_deg\_raw.RData`

\- `res\_signif` (data.frame interno)



\#### 5.2 Genes\_con\_splicing\_diferencial.Rmd



Identifica genes con splicing diferencial con `NBSplice`.



\*\*Input:\*\*



\- `geneIso\_f.RData`

\- `corrected\_iso\_counts.RData`

\- `metadataCOADunificada.RData`



\*\*Output:\*\*



\- `isoData\_DSG.RData`

\- `isoData\_f\_cpm.RData`



---



\### 6. Clustering



Agrupa muestras tumorales usando matrices de expresión de genes y/o isoformas, y combina ambos resultados en un consenso.



\*\*Input:\*\*



\- `metadataCOADunificada.RData`

\- `corrected\_gene\_counts.RData`

\- `gen\_data\_deg\_raw.RData`

\- `isoData\_f\_cpm.RData`

\- `isoData\_DSG.RData`



\*\*Output:\*\*



\- `Clustering01.RData`

\- `Clustering02.RData`

\- `consenso\_sampleGroup.RData`



---



\### 7. Análisis por subtipo



\#### 7.1 Expresion\_Diferencial\_Genes\_Subtipo\_vs\_Normal.Rmd



Realiza DEG para cada subtipo vs muestras normales.



\*\*Input:\*\*



\- `metadataCOADunificada.RData`

\- `corrected\_gene\_counts.RData`

\- `consenso\_sampleGroup.RData`



\*\*Output:\*\*



\- `DEGRes\_SubtipeVsN.RData`

\- `exclusiveDEGxSubtype.RData`



\#### 7.2 Expresión\_Diferencial\_Isoformas\_Subtipo\_vs\_Normal.Rmd



Análisis de splicing diferencial entre subtipos y normales. Filtra por FDR y cambio absoluto.



\*\*Input:\*\*



\- `geneIso\_f.RData`

\- `corrected\_iso\_counts.RData`

\- `consenso\_sampleGroup.RData`

\- `metadataCOADunificada.RData`



\*\*Output:\*\*



\- `DSRes\_SubtipeVsN.RData`

\- `exclusiveDS` (lista interna con genes exclusivos por subtipo)



---



\## Requisitos



\*\*Versión de R recomendada:\*\*  

R ≥ 4.2



\*\*Paquetes utilizados:\*\*



\- `recount3` – Descarga de datos de expresión

\- `SummarizedExperiment` – Formato estructurado de expresión

\- `edgeR`, `limma` – Análisis de expresión diferencial

\- `NBSplice` – Splicing diferencial

\- `dplyr`, `stringr`, `tibble`, `data.table` – Manipulación de datos

\- `BiocParallel` – Paralelización

\- `ggplot2` – Visualización

\- `multiClust` – Clustering de datos transcriptómicos



---



\## Instalación de dependencias



```r

\# CRAN

install.packages(c("dplyr", "stringr", "ggplot2", "tibble", "data.table"))



\# Bioconductor

if (!requireNamespace("BiocManager", quietly = TRUE)) 

&nbsp;   install.packages("BiocManager")



BiocManager::install(c(

&nbsp; "recount3", 

&nbsp; "SummarizedExperiment", 

&nbsp; "edgeR", 

&nbsp; "limma", 

&nbsp; "NBSplice", 

&nbsp; "BiocParallel"

))



