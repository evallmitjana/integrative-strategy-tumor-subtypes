README: integrative-strategy-tumor-subtypes
Este repositorio contiene un pipeline reproducible en R para el análisis de expresión diferencial y exploración transcriptómica entre muestras tumorales de colon (proyecto TCGA-COAD) y muestras normales de colon (GTEx), utilizando datos descargados desde el recurso recount3. El enfoque es en capas, abordando tanto expresión génica como isoformas, con el objetivo de identificar patrones transcriptómicos característicos de distintos subtipos tumorales.
Estructura del pipeline
El análisis está organizado en múltiples scripts .Rmd, cada uno correspondiente a una etapa del flujo de trabajo:

1- Preprocesamiento y descarga de datos
1.1 Descarga\_Genes\_Metadata.Rmd
Descarga los objetos RSE para genes de TCGA-COAD y GTEx-Colon.
Limpieza de duplicados, filtrado de metadatos y creación de tablas clínicas.

Input:
Datos descargados desde recount3 para proyectos COAD (TCGA) y Colon (GTEx).

Output:
rse\_gene\_coad y rse\_gene\_colon (objetos SummarizedExperiment)
metadata\_coad.RData
metadata\_colon.RData
genMatrix.RData
genes\_recount.RData (lista de genes en común)

1.2.Descarga\_Isoformas.Rmd
Carga los objetos RSE a nivel de isoformas para los mismos proyectos.
Aplica los mismos criterios de limpieza y selección de muestras.

Input:

GTEx\_Analysis\_2017-06-05\_v8\_RSEMv1.3.0\_transcript\_expected\_count.gct.gz

GTEx\_Analysis\_v8\_Annotations\_SampleAttributesDS.txt

TCGA\_COAD\_counts.tsv.gz

Output:

GTex\_Colon\_iso.gct (expresión de isoformas de colon normal)

genesFromIso.RData (genes en común a partir de isoformas)



2. Filtrado por genes y muestras en común
   2.Genes\_y\_Muestras\_en\_comun.Rmd
   Unifica los metadatos de TCGA y GTEx en una sola tabla.
   Identifica las muestras y genes que están presentes tanto en las matrices de expresión génica como en las de isoformas.

Input:

metadata\_coad.RData

metadata\_colon\_v1.RData

GTex\_Colon\_iso.gct

TCGA\_COAD\_counts.tsv.gz

genes\_recount.RData

genesFromIso.RData

Output:

metadataCOADunificada.RData (metadata de muestras en común)

commonGenesCOAD.RData (genes en común entre genes e isoformas)





3. Unificación de matrices de conteo
   3.1.Unificacion\_Matrices\_Genes.Rmd
   Filtra los objetos RSE de expresión génica (TCGA y GTEx) para conservar solo genes y muestras comunes.
   Genera una matriz unificada de conteo de reads por gen y reordena filas y columnas según los metadatos.

Input:

genMatrix.RData

metadataCOADunificada.RData

commonGenesCOAD.RData

Output:

gene\_counts.RData (matriz final de conteos por gen, unificada)



3.2.Unificacion\_Matrices\_Isoformas.Rmd
Filtra y une las matrices de expresión de isoformas (TCGA y GTEx) para conservar solo aquellas correspondientes a los genes y muestras comunes.
Renombra las muestras tumorales con sus códigos estándar y genera una matriz unificada de isoformas.

Input:

metadataCOADunificada.RData

commonGenesCOAD.RData

TCGA\_COAD\_counts.tsv.gz

GTex\_Colon\_iso.gct

Output:

COAD\_isoData\_unificada.RData (matriz final de conteos por isoforma, unificada)



4. Análisis exploratorio y corrección por batch



4.1.AnálisisExploratorio.Rmd
Explora la expresión génica e isoforme a través de gráficos descriptivos y PCA.
Filtra genes e isoformas poco expresadas, transforma las matrices a TPM + log2 y evalúa agrupamientos según metadatos.

Input:

gene\_counts.RData

COAD\_isoData\_unificada.RData

metadataCOADunificada.RData

Output:

geneExp\_filt\_counts.RData (matriz filtrada de genes)

isoData\_filt\_counts.RData (matriz filtrada de isoformas)

geneIso\_f.RData (relación gen–transcripto filtrada)

pca\_res\_gene.RData, pca\_res\_iso.RData (resultados de PCA por capa)



4.2.Correccion\_Efecto\_Batch.Rmd
Aplica corrección de batch con ComBat-seq para eliminar efectos del proyecto (TCGA vs GTEx) en matrices de expresión de genes e isoformas.
Evalúa los resultados mediante PCA antes y después de la corrección.

Input:

geneExp\_filt\_counts.RData

isoData\_filt\_counts.RData

metadataCOADunificada.RData

Output:

corrected\_gene\_counts.RData

corrected\_iso\_counts.RData

data\_pca\_gene\_corrected.RData

data\_pca\_iso\_corrected.RData

pca\_res\_gene\_corrected.RData

pca\_res\_iso\_corrected.RData



5. Análisis de expresión diferencial
   5.1.Expresion\_Diferencial\_Genes.Rmd
   Realiza el análisis de expresión diferencial a nivel génico entre muestras tumorales y normales.
   Filtra genes por baja expresión, aplica normalización TMM, ajusta un modelo lineal con edgeR, y selecciona genes con FDR < 0.01 y |logFC| > 2.

Input:

corrected\_gene\_counts.RData

metadataCOADunificada.RData

Output:

gen\_data\_deg\_raw.RData (matriz de genes diferencialmente expresados)

res\_signif (data.frame interno con resultados DEG filtrados)



5.2.Genes\_con\_splicing\_diferencial.Rmd
Identifica genes con splicing diferencial entre muestras normales (GTEx) y tumorales (TCGA-COAD), usando modelos basados en NBSplice. Filtra genes con múltiples isoformas, detecta isoformas de baja expresión y selecciona los genes con evidencia estadística de splicing diferencial.

Input:

geneIso\_f.RData (relación entre genes e isoformas)

corrected\_iso\_counts.RData (matriz de isoformas corregida por batch)

metadataCOADunificada.RData (metadatos unificados de muestras normales y tumorales)

Output:

isoData\_DSG.RData (matriz reducida a genes con splicing diferencial - DSG)

isoData\_f\_cpm.RData (matriz de isoformas filtrada, expresada como CPM, para análisis posteriores como clustering)



6\. Clustering

Agrupa muestras tumorales utilizando matrices de expresión de genes, isoformas o la combinación de ambas. Aplica selección adaptativa de genes, determina el número óptimo de clusters y genera una clasificación basada en el consenso de los dos agrupamientos (con y sin isoformas).



Input:

metadataCOADunificada.RData (metadatos de muestras tumorales)

corrected\_gene\_counts.RData (expresión de genes corregida)

gen\_data\_deg\_raw.RData (genes diferencialmente expresados)

isoData\_f\_cpm.RData (isoformas filtradas en CPM)

isoData\_DSG.RData (isoformas de genes con splicing diferencial)



Output:

Clustering01.RData (agrupamiento basado solo en genes)

Clustering02.RData (agrupamiento basado en genes + isoformas)

consenso\_sampleGroup.RData (clasificación final por consenso con subtipos tumorales)



7\. Análisis por subtipo

7.1.Expresion\_Diferencial\_Genes\_Subtipo\_vs\_Normal.Rmd

Realiza análisis de expresión diferencial entre cada subtipo tumoral y las muestras normales.

Genera contrastes independientes para cada subtipo, filtra genes por FDR < 0.05 y |logFC| > 2, y obtiene listas de genes exclusivos de cada subtipo.



Input:

metadataCOADunificada.RData

corrected\_gene\_counts.RData

consenso\_sampleGroup.RData (asignaciones de subtipos)

Output:

DEGRes\_SubtipeVsN.RData (resultados DEG por subtipo)

exclusiveDEGxSubtype.RData (genes exclusivos por subtipo)



7.2.Expresión\_Diferencial\_Isoformas\_Subtipo\_vs\_Normal.Rmd

Detecta genes con splicing diferencial entre cada subtipo tumoral (definido por consenso de clustering) y las muestras normales, utilizando modelos estadísticos de NBSplice. Filtra por FDR y cambio absoluto, y obtiene los genes exclusivos de cada subtipo.



Input:

geneIso\_f.RData (relación entre genes e isoformas)

corrected\_iso\_counts.RData (expresión de isoformas corregida por batch)

consenso\_sampleGroup.RData (asignación de muestras a subtipos tumorales)

metadataCOADunificada.RData (metadatos combinados de muestras tumorales y normales)



Output:

DSRes\_SubtipeVsN.RData (resultados de splicing diferencial por subtipo)

exclusiveDS (lista en entorno de trabajo con genes con splicing diferencial exclusivos por subtipo)





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

