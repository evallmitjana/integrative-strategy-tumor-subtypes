README: integrative-strategy-tumor-subtypes
Este repositorio contiene un pipeline reproducible en R para el análisis de expresión diferencial y exploración transcriptómica entre muestras tumorales de colon (proyecto TCGA-COAD) y muestras normales de colon (GTEx), utilizando datos descargados desde el recurso recount3. El enfoque es en capas, abordando tanto expresión génica como isoformas, con el objetivo de identificar patrones transcriptómicos característicos de distintos subtipos tumorales.
Estructura del pipeline
El análisis está organizado en múltiples scripts .Rmd, cada uno correspondiente a una etapa del flujo de trabajo:

1- Preprocesamiento y descarga de datos
 1.1 Descarga_Genes_Metadata.Rmd
  Descarga los objetos RSE para genes de TCGA-COAD y GTEx-Colon.
  Limpieza de duplicados, filtrado de metadatos y creación de tablas clínicas.
  
  Input:
  Datos descargados desde recount3 para proyectos COAD (TCGA) y Colon (GTEx).
  
  Output:
  rse_gene_coad y rse_gene_colon (objetos SummarizedExperiment)
  metadata_coad.RData
  metadata_colon.RData
  genMatrix.RData
  genes_recount.RData (lista de genes en común)

1.2.Descarga_Isoformas.Rmd
Carga los objetos RSE a nivel de isoformas para los mismos proyectos.
Aplica los mismos criterios de limpieza y selección de muestras.

Input:

GTEx_Analysis_2017-06-05_v8_RSEMv1.3.0_transcript_expected_count.gct.gz

GTEx_Analysis_v8_Annotations_SampleAttributesDS.txt

TCGA_COAD_counts.tsv.gz

Output:

GTex_Colon_iso.gct (expresión de isoformas de colon normal)

genesFromIso.RData (genes en común a partir de isoformas)



2. Filtrado por genes y muestras en común
2.Genes_y_Muestras_en_comun.Rmd
Unifica los metadatos de TCGA y GTEx en una sola tabla.
Identifica las muestras y genes que están presentes tanto en las matrices de expresión génica como en las de isoformas.

Input:

metadata_coad.RData

metadata_colon_v1.RData

GTex_Colon_iso.gct

TCGA_COAD_counts.tsv.gz

genes_recount.RData

genesFromIso.RData

Output:

metadataCOADunificada.RData (metadata de muestras en común)

commonGenesCOAD.RData (genes en común entre genes e isoformas)





3. Unificación de matrices de conteo
3.1.Unificacion_Matrices_Genes.Rmd
Filtra los objetos RSE de expresión génica (TCGA y GTEx) para conservar solo genes y muestras comunes.
Genera una matriz unificada de conteo de reads por gen y reordena filas y columnas según los metadatos.

Input:

genMatrix.RData

metadataCOADunificada.RData

commonGenesCOAD.RData

Output:

gene_counts.RData (matriz final de conteos por gen, unificada)


3.2.Unificacion_Matrices_Isoformas.Rmd
Filtra y une las matrices de expresión de isoformas (TCGA y GTEx) para conservar solo aquellas correspondientes a los genes y muestras comunes.
Renombra las muestras tumorales con sus códigos estándar y genera una matriz unificada de isoformas.

Input:

metadataCOADunificada.RData

commonGenesCOAD.RData

TCGA_COAD_counts.tsv.gz

GTex_Colon_iso.gct

Output:

COAD_isoData_unificada.RData (matriz final de conteos por isoforma, unificada)

4. Análisis exploratorio y corrección por batch
4.1.AnálisisExploratorio.Rmd
Explora la expresión génica e isoforme a través de gráficos descriptivos y PCA.
Filtra genes e isoformas poco expresadas, transforma las matrices a TPM + log2 y evalúa agrupamientos según metadatos.

Input:

gene_counts.RData

COAD_isoData_unificada.RData

metadataCOADunificada.RData

Output:

geneExp_filt_counts.RData (matriz filtrada de genes)

isoData_filt_counts.RData (matriz filtrada de isoformas)

geneIso_f.RData (relación gen–transcripto filtrada)

pca_res_gene.RData, pca_res_iso.RData (resultados de PCA por capa)



4.2.Correccion_Efecto_Batch.Rmd
Aplica corrección de batch con ComBat-seq para eliminar efectos del proyecto (TCGA vs GTEx) en matrices de expresión de genes e isoformas.
Evalúa los resultados mediante PCA antes y después de la corrección.

Input:

geneExp_filt_counts.RData

isoData_filt_counts.RData

metadataCOADunificada.RData

Output:

corrected_gene_counts.RData

corrected_iso_counts.RData

data_pca_gene_corrected.RData

data_pca_iso_corrected.RData

pca_res_gene_corrected.RData

pca_res_iso_corrected.RData


5. Análisis de expresión diferencial
5.1.Expresion_Diferencial_Genes.Rmd
Realiza el análisis de expresión diferencial a nivel génico entre muestras tumorales y normales.
Filtra genes por baja expresión, aplica normalización TMM, ajusta un modelo lineal con edgeR, y selecciona genes con FDR < 0.01 y |logFC| > 2.

Input:

corrected_gene_counts.RData

metadataCOADunificada.RData

Output:

gen_data_deg_raw.RData (matriz de genes diferencialmente expresados)

res_signif (data.frame interno con resultados DEG filtrados)

5.2.Genes_con_splicing_diferencial.Rmd
Identifica genes con splicing diferencial entre muestras normales (GTEx) y tumorales (TCGA-COAD), usando modelos basados en NBSplice. Filtra genes con múltiples isoformas, detecta isoformas de baja expresión y selecciona los genes con evidencia estadística de splicing diferencial.

Input:

geneIso_f.RData (relación entre genes e isoformas)

corrected_iso_counts.RData (matriz de isoformas corregida por batch)

metadataCOADunificada.RData (metadatos unificados de muestras normales y tumorales)

Output:

isoData_DSG.RData (matriz reducida a genes con splicing diferencial - DSG)

isoData_f_cpm.RData (matriz de isoformas filtrada, expresada como CPM, para análisis posteriores como clustering)


Requisitos
Versión de R recomendada:
R ≥ 4.2

Paquetes utilizados:

recount3 – Descarga y manipulación de objetos RSE desde bases públicas

SummarizedExperiment – Manejo de datos transcriptómicos en formato estructurado

edgeR, limma – (utilizados en otros scripts para análisis de expresión diferencial)

dplyr, stringr, tibble – Manipulación de datos

data.table – Lectura eficiente de archivos grandes (fread)

NBSplice – Detección de splicing diferencial

BiocParallel – Paralelización en análisis con NBSplice

ggplot2 – Visualización (presumiblemente en scripts de análisis)


 Instalación de dependencias
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
