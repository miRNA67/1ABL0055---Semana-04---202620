# Semana 04: Basecalling de los datos de secuenciación - Visualización de la calidad y limpieza de los archivos FASTQ

## Logro de la sesión:

Al finalizar la sesión, el estudiante realiza el basecalling de los archivos POD5 y la limpieza de los archivos FASTQ con diferentes herramientas bioinformáticas, interpreta las métricas de calidad de los datos Illumina y Nanopore, cuantifica el efecto de cada paso de limpieza sobre el número de lecturas y de bases, y evalúa la presencia de contaminación en los datos de Nanopore con Kraken2.

## Estructura de la práctica:

1. Acceso al servidor de cómputo
2. Análisis de calidad de archivos FASTQ de Illumina
3. Limpieza de los archivos FASTQ de Illumina
4. Basecalling de los archivos POD5 de Nanopore
5. Análisis de calidad de archivos FASTQ de Nanopore
6. Limpieza de los archivos FASTQ de Nanopore
7. Análisis de contaminación con Kraken2
8. Análisis de calidad, limpieza y contaminación de los datos de secuenciación Nanopore generados en el curso

## Flujo de trabajo:

### Illumina:

```mermaid
flowchart LR
    subgraph ILL["Illumina (CAT_R1 / CAT_R2)"]
        direction LR
        A1["FASTQ crudos"] --> A2["FastQC + MultiQC"]
        A2 --> A3["Trim Galore"]
        A2 --> A4["Trimmomatic"]
        A3 --> A5["FastQC + MultiQC + seqkit stats"]
        A4 --> A5
    end
```
### Nanopore:

```mermaid
flowchart LR
    subgraph NAN["Nanopore (barcode asignado)"]
        direction LR
        B1["POD5"] --> B2["Dorado (sup)"]
        B2 --> B3["BAM a FASTQ"]
        B3 --> B4["NanoPlot (calidad cruda)"]
        B4 --> B5["Porechop"]
        B5 --> B6["NanoFilt"]
        B6 --> B7["seqkit stats"]
        B7 --> B8["Kraken2 (contaminación)"]
    end
```

## Programas requeridos:

### Programas de acceso al servidor:

PuTTY v0.79 https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
   - **Descripción:** PuTTY es un cliente SSH, Telnet y Rlogin gratuito y de código abierto para Windows y sistemas Unix. Se utiliza principalmente para establecer conexiones seguras de línea de comandos a servidores remotos.

WinSCP v6.1 https://winscp.net/eng/download.php
   - **Descripción:** WinSCP es un cliente SFTP, FTP, WebDAV, Amazon S3 y SCP gratuito y de código abierto para Windows. Permite la transferencia segura de archivos entre un ordenador local y servidores remotos mediante una interfaz gráfica de usuario. En esta práctica se usará para descargar los reportes HTML (FastQC, MultiQC, NanoPlot) y visualizarlos en su computadora.

### Programas bioinformáticos:

Dorado v0.9.1 https://github.com/nanoporetech/dorado
   - **Descripción:** Dorado es una herramienta de Oxford Nanopore Technologies, sucesora de Guppy, para la llamada de bases (basecalling) de las señales eléctricas generadas por sus secuenciadores de ADN. Al igual que Guppy, también incluye funcionalidades para la clasificación de barcodes (multiplexado), el recorte de adaptadores y el filtrado de reads por calidad, pero con mejoras en rendimiento y precisión.

FastQC v0.12.1 http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
   - **Descripción:** FastQC es una herramienta de control de calidad para datos de secuenciación de alto rendimiento. Proporciona un informe detallado que ayuda a identificar posibles problemas en los datos brutos antes del análisis posterior.

Kraken2 v2.1.3 https://github.com/DerrickWood/kraken2
   - **Descripción:** Kraken2 es un clasificador taxonómico de secuencias basado en k-mers: compara cada lectura con una base de datos de genomas de referencia y le asigna un taxón. Aquí se usa para detectar contaminación en los FASTQ de Nanopore, con la base de datos PlusPF (arqueas, bacterias, virus, plásmidos, humano, protozoos y hongos).

MultiQC v1.28.0 https://multiqc.info
   - **Descripción:** MultiQC es una herramienta que agrega informes de control de calidad de múltiples herramientas de análisis bioinformático en un único informe HTML interactivo. Es compatible con una amplia gama de herramientas, incluyendo FastQC, Cutadapt/Trim Galore! y Trimmomatic, facilitando la revisión y comparación de los resultados de control de calidad de múltiples muestras.

NanoFilt v2.8.0 https://github.com/wdecoster/nanofilt
   - **Descripción:** NanoFilt es una herramienta para filtrar datos de secuenciación de Nanopore basándose en la calidad y la longitud de los reads. Permite seleccionar reads de alta calidad para análisis posteriores.

NanoPlot v1.41.6 https://github.com/wdecoster/NanoPlot
   - **Descripción:** NanoPlot es una herramienta para la visualización de datos de secuenciación de Nanopore. Genera varios tipos de gráficos para evaluar la calidad y las características de los reads, como la distribución de longitudes y la calidad a lo largo de los reads.

Porechop v0.2.4 https://github.com/rrwick/Porechop
   - **Descripción:** Porechop es una herramienta para identificar y recortar adaptadores en los extremos de los reads de Oxford Nanopore. Cuando detecta un adaptador en el interior de un read, lo considera quimérico y lo divide en reads separados. *Nota: su autor lo declaró oficialmente sin mantenimiento en 2018; se usa aquí con fines didácticos para comparar con el recorte que ya realiza Dorado.*

SeqKit v2.13.0 https://bioinf.shenwei.me/seqkit/
   - **Descripción:** Kit de herramientas para manipular y resumir archivos FASTA/FASTQ. Aquí se usa para calcular estadísticas de los FASTQ (`stats`) y, antes de Kraken2, para renombrar las lecturas (`rename`).

TrimGalore v0.6.10 https://github.com/FelixKrueger/TrimGalore
   - **Descripción:** Trim Galore! es un wrapper alrededor de Cutadapt y FastQC para realizar el recorte de adaptadores y el control de calidad en datos de secuenciación de alto rendimiento. Automatiza el proceso de recorte y genera informes de calidad.

Trimmomatic v0.39 http://www.usadellab.org/cms/?page=trimmomatic
   - **Descripción:** Trimmomatic es una herramienta flexible y rápida para realizar el recorte de adaptadores y el filtrado de calidad en datos de secuenciación de Illumina. Permite eliminar secuencias de adaptadores, bases de baja calidad y reads demasiado cortos.

## Metodología:

## 1. Acceso al servidor de cómputo:

### Abrir el programa PuTTY, colocar el hostname ( 10.142.250.66 ) y port ( 22 ), y dar clic en Open:

<img width="500" alt="image" src="https://github.com/user-attachments/assets/92f89dbb-1a21-411d-adb5-38fe486a5567" />



### En la terminal abierta, escribir su usuario y contraseña correspondiente para tener acceso al servidor de cómputo Tensor:
 
<img width="700" alt="image" src="https://github.com/user-attachments/assets/4d246e93-c59c-4749-a2dd-03db25c53654" />



### Abrir el programa WinSCP, colocar el hostname ( 10.142.250.66 ) y port ( 22 ), escribir su usuario y contraseña correspondiente para tener acceso al servidor de cómputo Tensor, y hacer clic en Login:

<img width="500" alt="image" src="https://github.com/user-attachments/assets/ef4dc253-ce4a-417d-b761-39692d2a011a" />

<img width="500" alt="image" src="https://github.com/user-attachments/assets/577debca-6085-47c5-9bbd-73688bfa8bb0" />

### Crear la estructura de carpetas de trabajo

Una vez dentro del servidor, crear **de una sola vez** todas las carpetas que se usarán durante la práctica:

```bash
cd

mkdir -p ~/genomics/{basecalling/pod5_db_sup,quality/{illumina,nanopore},trimming/{illumina/{trim_galore,trimmomatic},nanopore},contamination}

tree ~/genomics
```

> **Comentario:**
> - `mkdir -p`: crea las carpetas indicadas y también las carpetas intermedias que falten; si ya existen, no da error.
> - `{a,b}`: es una expansión de llaves de Bash; `quality/{illumina,nanopore}` equivale a escribir `quality/illumina quality/nanopore`.
> - `tree ~/genomics`: muestra el árbol de carpetas para verificar que quedó como en la sección 7.

> **Importante:** Las rutas de los comandos siguientes asumen esta estructura. Si cierra y vuelve a abrir PuTTY, siempre debe volver a activar el entorno conda que corresponda (`conda activate ...`).

## 2. Análisis de calidad de archivos FASTQ de Illumina

```bash
cd ~/genomics/quality/illumina

conda activate quality

fastqc -t 10 /data/2025_1/database/illumina/CAT_R1.fastq.gz /data/2025_1/database/illumina/CAT_R2.fastq.gz -o .
```

> **Comentario:** 
> - `-t 10`: Esta opción especifica el número de hilos (threads) que FastQC debe utilizar. FastQC procesa un archivo por hilo, por lo que con 10 hilos analiza R1 y R2 al mismo tiempo.
> - `/data/2025_1/database/illumina/CAT_R1.fastq.gz` y `.../CAT_R2.fastq.gz`: Rutas de los archivos que FastQC debe analizar (lecturas R1 y R2 del par).
> - `-o .`: Esta opción define el directorio de salida. El punto "." representa el directorio actual. Esto significa que los informes HTML generados por FastQC se guardarán en el mismo directorio donde se ejecuta el comando.

```bash
multiqc -o raw_illumina .
```
> **Comentario:**
> - `-o raw_illumina`: Esta opción especifica el directorio de salida donde se guardará el informe HTML generado por MultiQC.
> - `.`: Representa el directorio actual. Esto le dice a MultiQC que busque archivos de resultados (los que ha generado FastQC por ejemplo) dentro del directorio en el que estás ejecutando el comando. MultiQC buscará automáticamente archivos de salida de las herramientas de control de calidad compatibles que se encuentren en el directorio actual y sus subdirectorios.

> **Punto de control:** Descargue `raw_illumina/multiqc_report.html` con WinSCP y revise, para R1 y R2: *Per base sequence quality*, *Adapter content*, *Per base N content* y *Sequence duplication levels*. ¿Baja la calidad hacia el extremo 3' de las lecturas? ¿Hay adaptadores presentes?

## 3. Limpieza de los archivos FASTQ de Illumina

### Limpieza con trim galore

```bash
cd ~/genomics/trimming/illumina/trim_galore

trim_galore --quality 30 --length 50 --phred33 --cores 2 --fastqc --paired --output_dir . /data/2025_1/database/illumina/CAT_R1.fastq.gz /data/2025_1/database/illumina/CAT_R2.fastq.gz
```

```bash
AUTO-DETECTING ADAPTER TYPE
===========================
Attempting to auto-detect adapter type from the first 1 million sequences of the first file (>> /data/2025_1/database/illumina/CAT_R1.fastq.gz <<)

Found perfect matches for the following adapter sequences:
Adapter type    Count   Sequence        Sequences analysed      Percentage
Nextera 466837  CTGTCTCTTATA    1000000 46.68
smallRNA        8       TGGAATTCTCGG    1000000 0.00
Illumina        0       AGATCGGAAGAGC   1000000 0.00
Using Nextera adapter for trimming (count: 466837). Second best hit was smallRNA (count: 8)

Writing report to '/home/alumno01/genomics/trimming/illumina/trim_galore/CAT_R1.fastq.gz_trimming_report.txt'
```

> **Comentario:**
> - `--quality 30`: Esta opción especifica el umbral de calidad para el recorte de bases. Las bases con una calidad inferior a 30 (en escala Phred33) serán recortadas del extremo 3' de las lecturas. Es un umbral estricto (el valor por defecto de Trim Galore! es 20).
> - `--length 50`: Esta opción establece la longitud mínima de las lecturas después del recorte. Las lecturas que sean más cortas que 50 bases serán descartadas.
> - `--phred33`: Esta opción indica que los datos de calidad de las bases están en la escala Phred33, que es la escala más común utilizada en la secuenciación Illumina.
> - `--cores 2`: Esta opción especifica el número de núcleos de procesamiento que Trim Galore! debe utilizar. En este caso, se están utilizando 2 núcleos para acelerar el procesamiento.
> - `--fastqc`: Esta opción le indica a Trim Galore! que ejecute FastQC automáticamente después del recorte para generar informes de control de calidad de los datos recortados.
> - `--paired`: Esta opción indica que los datos son pareados (paired-end), lo que significa que las lecturas vienen en pares (R1 y R2). Si una lectura se descarta, su pareja también se descarta para mantener el emparejamiento.
> - `--output_dir .`: Guarda los resultados en el directorio actual (por defecto ya se guardan ahí; se indica explícitamente para evitar confusiones).
> - `/data/2025_1/database/illumina/CAT_R1.fastq.gz`: Esta es la ruta del archivo FASTQ comprimido que contiene las lecturas R1 (la primera lectura del par).
> - `/data/2025_1/database/illumina/CAT_R2.fastq.gz`: Esta es la ruta del archivo FASTQ comprimido que contiene las lecturas R2 (la segunda lectura del par).
>
> Trim Galore! detecta automáticamente el tipo de adaptador (Illumina, Nextera o small RNA). Los archivos resultantes se llaman `CAT_R1_val_1.fq.gz` y `CAT_R2_val_2.fq.gz`, y los reportes de recorte terminan en `_trimming_report.txt`.

```bash
multiqc -o trimming_trim_galore .
```

### Limpieza con trimmomatic

```bash
cd ~/genomics/trimming/illumina/trimmomatic
```

> **Comentario:** Crear el archivo de adaptadores NexteraPE.fa. Trimmomatic incluye este mismo archivo (`NexteraPE-PE.fa`) en su carpeta `adapters/`; aquí lo crearemos manualmente para ver cómo es su formato.

```bash
nano NexteraPE.fa

>PrefixNX/1
AGATGTGTATAAGAGACAG
>PrefixNX/2
AGATGTGTATAAGAGACAG
>Trans1
TCGTCGGCAGCGTCAGATGTGTATAAGAGACAG
>Trans1_rc
CTGTCTCTTATACACATCTGACGCTGCCGACGA
>Trans2
GTCTCGTGGGCTCGGAGATGTGTATAAGAGACAG
>Trans2_rc
CTGTCTCTTATACACATCTCCGAGCCCACGAGAC

cat NexteraPE.fa
```

```bash
trimmomatic PE -threads 10 -phred33 /data/2025_1/database/illumina/CAT_R1.fastq.gz /data/2025_1/database/illumina/CAT_R2.fastq.gz CAT_R1.trim.fastq.gz CAT_R1.unpaired.fastq.gz CAT_R2.trim.fastq.gz CAT_R2.unpaired.fastq.gz ILLUMINACLIP:NexteraPE.fa:2:30:10 SLIDINGWINDOW:4:30 MINLEN:50 2> trimmomatic_CAT.log

cat trimmomatic_CAT.log
```

> **Comentario:**
> - `PE`: Indica que los datos son pareados (paired-end).
> - `-threads 10`: Número de hilos. 
> - `-phred33`: Indica la codificación de calidad de las bases.
> - `/data/2025_1/database/illumina/CAT_R1.fastq.gz`: Esta es la ruta del archivo FASTQ comprimido que contiene las lecturas R1 (la primera lectura del par).
> - `/data/2025_1/database/illumina/CAT_R2.fastq.gz`: Esta es la ruta del archivo FASTQ comprimido que contiene las lecturas R2 (la segunda lectura del par).
> - `CAT_R1.trim.fastq.gz`: Archivo de salida para las lecturas R1 recortadas y emparejadas.
> - `CAT_R1.unpaired.fastq.gz`: Archivo de salida para las lecturas R1 que quedaron sin par después del recorte.
> - `CAT_R2.trim.fastq.gz`: Archivo de salida para las lecturas R2 recortadas y emparejadas.
> - `CAT_R2.unpaired.fastq.gz`: Archivo de salida para las lecturas R2 que quedaron sin par después del recorte.
> - `ILLUMINACLIP:NexteraPE.fa:2:30:10`: Son los parámetros para el recorte de adaptadores (seed mismatches:umbral de coincidencia palindrómica:umbral de coincidencia simple).
> - `SLIDINGWINDOW:4:30`: Se analiza la lectura con una ventana deslizante de 4 bases; cuando la calidad promedio dentro de la ventana cae por debajo de 30, se recorta la lectura desde ese punto hasta el extremo 3'.
> - `MINLEN:50`: Esta opción establece la longitud mínima de las lecturas después del recorte. Las lecturas que sean más cortas que 50 bases serán descartadas.
> - `2> trimmomatic_CAT.log`: Trimmomatic imprime su resumen (pares de entrada, pares que sobreviven, etc.) por la salida de error; se guarda en un archivo para poder usarlo en la bitácora y para que MultiQC lo lea.

```bash
fastqc -t 10 *.trim.fastq.gz -o .

multiqc -o trimming_trimmomatic .
```

### Comparación entre los datos crudos y las dos limpiezas

```bash
cd ~/genomics/trimming/illumina

seqkit stats -a -j 10 /data/2025_1/database/illumina/CAT_R1.fastq.gz /data/2025_1/database/illumina/CAT_R2.fastq.gz trim_galore/CAT_R1_val_1.fq.gz trim_galore/CAT_R2_val_2.fq.gz trimmomatic/CAT_R1.trim.fastq.gz trimmomatic/CAT_R2.trim.fastq.gz > stats_illumina.txt

cat stats_illumina.txt
```

> **Punto de control:** Compare `num_seqs`, `sum_len`, `avg_len` y `Q30(%)` entre los datos crudos, Trim Galore! y Trimmomatic. ¿Qué herramienta conservó más lecturas? ¿Cuál mejoró más la calidad? ¿Por qué los resultados no son idénticos si se usaron los mismos umbrales de calidad y longitud?

## 4. Basecalling de los archivos POD5 de Nanopore

#### Verificación previa

```bash
dorado --version

nvidia-smi

ls /data/software/dorado-0.9.1-linux-x64/models
```

> **Comentario:**
> - `dorado --version`: confirma la versión de Dorado que se usará (anótela en su bitácora).
> - `nvidia-smi`: muestra las GPU del servidor y la memoria en uso. Verifique que la GPU esté disponible antes de lanzar el basecalling, porque el servidor es compartido.
> - `ls .../models`: lista los modelos de basecalling ya descargados en el servidor.

#### Basecalling

```bash
cd ~/genomics/basecalling/pod5_db_sup

dorado basecaller sup --kit-name SQK-NBD114-24 --min-qscore 10 --device "cuda:0" --barcode-both-ends --models-directory /data/software/dorado-0.9.1-linux-x64/models /data/2025_1/database/nanopore/pod5/ > wasp_calls.bam
```

> **Comentario:** 
> Al iniciar, Dorado imprime el nombre del modelo que está usando (por ejemplo, `dna_r10.4.1_e8.2_400bps_sup@v5.0.0`).
> - `sup`: Esta opción indica que se debe utilizar el modelo de basecalling de "super precisión" (super accuracy). Estos modelos están entrenados para ofrecer una mayor exactitud en la llamada de bases, a costa de un mayor tiempo de cómputo. Dorado elige automáticamente el modelo `sup` que corresponde a los datos.
> - `--kit-name SQK-NBD114-24`: Este parámetro especifica el nombre del kit de preparación de librería utilizado (Native Barcoding Kit 24 V14). Activa la **clasificación de barcodes** y le indica a Dorado qué adaptadores y barcodes buscar; por defecto Dorado también recorta los adaptadores, primers y barcodes que detecta (esto se desactiva con `--no-trim`). **No** sirve para elegir el modelo de basecalling.
> - `--min-qscore 10`: Esta opción establece un umbral de calidad mínima **para cada lectura completa**: Dorado descarta las lecturas cuyo Q-score **medio** sea menor que 10. No filtra bases individuales. El Q-score es una medida de la probabilidad de que una base llamada sea incorrecta: un Q-score de 10 significa una probabilidad de error de 1 en 10 (10%); Q20, 1 en 100 (1%); Q30, 1 en 1000 (0,1%).
> - `--device "cuda:0"`: Esto indica que Dorado debe utilizar la primera GPU habilitada para CUDA (índice 0) disponible en tu sistema para acelerar el proceso de basecalling. El uso de la GPU puede reducir significativamente el tiempo de procesamiento.
> - `--barcode-both-ends`: Esta opción le dice a Dorado que exija códigos de barras (barcodes) en ambos extremos de la lectura. Es útil si tu protocolo de secuenciación incluyó barcodes en ambos extremos, pero es más estricto: una lectura con barcode en un solo extremo no se clasifica.
> - `--models-directory /data/software/dorado-0.9.1-linux-x64/models`: Este parámetro especifica el directorio donde Dorado busca modelos ya descargados (o donde descargaría los que falten). Es importante que la ruta sea correcta para que Dorado pueda cargar los modelos necesarios.
> - `/data/2025_1/database/nanopore/pod5/`: Esta es la ruta de la carpeta de entrada con los archivos POD5 que contienen los datos de señal sin procesar de la corrida (de todos los barcodes). Dorado realizará el basecalling de todos los archivos de la carpeta.
> - `> wasp_calls.bam`: Esto redirige la salida estándar (stdout) del comando Dorado al archivo llamado wasp_calls.bam. Como no se indicó un genoma de referencia (`--reference`), Dorado genera un archivo BAM **sin alinear** (uBAM) con las secuencias base-llamadas y sus etiquetas (por ejemplo, la clasificación de barcode).

#### Conversión de bam a fastq

```bash
dorado demux --kit-name SQK-NBD114-24 --output-dir demux_fastq --emit-fastq wasp_calls.bam

seqkit stats -a -j 10 demux_fastq/*.fastq > stats_fastq.txt
```

> **Comentario:** 
> - `dorado demux`: separa las lecturas del BAM según la clasificación de barcode que Dorado les asignó durante el basecalling.
> - `--output-dir demux_fastq`: carpeta donde se guarda un archivo por barcode (y uno `unclassified` con las lecturas sin barcode).
> - `--emit-fastq`: escribe los archivos en formato FASTQ (por defecto Dorado escribe BAM).
> - `wasp_calls.bam`: BAM sin alinear generado en el paso anterior, con las lecturas de todos los barcodes.
> - `seqkit stats`: calcula las estadísticas de los FASTQ separados por barcode (-a: todas las estadísticas y -j: número de hilos).
>

```bash
cat stats_fastq.txt

file                                                                                format  type  num_seqs    sum_len  min_len  avg_len  max_len     Q1     Q2       Q3  sum_gap    N50  N50_num  Q20(%)  Q30(%)  AvgQual  GC(%)  sum_n
demux_fastq/bcf4b7732185c1a3353d1b4fe80266cd3ac60162_SQK-NBD114-24_barcode13.fastq  FASTQ   DNA          8      3,457      205    432.1      796  242.5    383    602.5        0    523        3   82.38   71.19    17.53  38.62      0
demux_fastq/bcf4b7732185c1a3353d1b4fe80266cd3ac60162_SQK-NBD114-24_barcode14.fastq  FASTQ   DNA      2,789  2,835,518       13  1,016.7   20,783    428    682    1,145        0  1,381      461   89.65   80.68    19.59  45.73      0
demux_fastq/bcf4b7732185c1a3353d1b4fe80266cd3ac60162_SQK-NBD114-24_barcode15.fastq  FASTQ   DNA          3      3,778    1,087  1,259.3    1,356  1,211  1,335  1,345.5        0  1,335        2      95   86.98    24.74  50.95      0
demux_fastq/bcf4b7732185c1a3353d1b4fe80266cd3ac60162_SQK-NBD114-24_barcode17.fastq  FASTQ   DNA          1        417      417      417      417    417    417      417        0    417        1   84.89   77.22    17.72  36.21      0
demux_fastq/bcf4b7732185c1a3353d1b4fe80266cd3ac60162_unclassified.fastq             FASTQ   DNA      1,227  1,526,649       26  1,244.2   26,983  592.5    864    1,394        0  1,581      245   90.52   81.78    20.14  45.71      0```

> **Comentario:** En `seqkit stats`, `Q20(%)` y `Q30(%)` son el **porcentaje de bases** con calidad ≥ Q20 y ≥ Q30, y `AvgQual` es la calidad media por lectura. No confundir con el resumen de NanoPlot (sección 5), donde `>Q20` y `>Q30` son el **número (y %) de lecturas** cuya calidad media supera ese umbral.

```bash
mv demux_fastq/bcf4b7732185c1a3353d1b4fe80266cd3ac60162_SQK-NBD114-24_barcode14.fastq b14.fastq
```

```bash
head b14.fastq

@00394845-e64b-49a7-af5a-af1b5b008917   qs:f:28.6034    st:Z:2024-04-28T00:01:23.062+00:00      RG:Z:bcf4b7732185c1a3353d1b4fe80266cd3ac60162_dna_r10.4.1_e8.2_400bps_sup@v5.0.0
TTCCCGATCGGCGGATGTTCAGGTATGTCATCTCCCGCAGAAATCGTATACGTTGAAACCACGTGGGTTTCTGACGGGCCGTAATGATTATGCAGCTGCATACCGTGTAAACGGAGCGTCTGTCTGAACAAACGTGAGATAGTCAGCTGCTCCCCGGCCGTAATGACATGCTTCACACAGCGGGGAAAAGACTGTGCATAACCTTCTTCATTGAAAAGCATCTTGACAAACGCAGTCGGGAAAAACACCACTTCCGTTTTGTGCTGATCGATAAACGAATACAGCTGAGAAACATCCCGTTTGATAGATTCGGGCACGATACAAAGGGTGCCGCCGCTCGAGAGTACTGAAAATAACTCCTGATAGCAGACGTCAAACGCCAAAGATGCGTACTGCAGCACATTTGTGCAAAAATCAATGTCTGTGTTCGTCAATTGGTCAGAAAGAAGGTTGGCCATATTTTTATGTTCGAGCAGCACGCCCCTTCGGTTTCCCTGTCGTACCGGATGTATAAATCATATAAAGGAGGTCGTCCGCCGTATTAATGGATTGCACG
+
JGDIG5DB@>CEEJHHNKKPIKOJLKKD;==;=;;999::BANJSRSLSSSNSSRSSOIGFGDSQSSMSSSSMOLIMSKJPSNSSOSLNSSKSSSSSMSSSSSSSSOSQSSSMSSSSSSSSRSOSSSSSOSSSQSSSSSSSLILHGEHGHGGGA;9<=>MISSSNSSNSSSSSOSSNSLSSNSSSSSSSSSSSLIPLNSNSSSSSSSSSSSSSSSSSSSSSNSSMSSKSSSNSSSSIJMSSSSSSSJMISSSSQSPSSSSPSSQMJISOGGGHFSSRSSSOSFFHIIJNSSSSQSOSSMKHHKPSSSSSSSSSSSPSSSSSSIDEGGSSKSFIGOHFED,++++4334<=>DFSMSSSNSSOSSOSSMSKHSSLSSSSOSSNSSSSSMSOSPSSSSIIJSSJINFAFGFNSSSSSSSSSLSSSRSRLLSMNNHJJKGIHNHSSSQNSSSSSSQSSSSSSSSSSSSSSJHJIRSSQSSSSRMSA8+J>>HMSSSSSFQSLIGISSQSSSKMJSNSQJKPILHHFEB9IQKSLMSLS==H@?LFEEFGHFGLMNSDA4
@0cb29fbb-1cad-4984-b8dd-04a930a3f387   qs:f:23.3482    st:Z:2024-04-28T00:00:33.395+00:00      RG:Z:bcf4b7732185c1a3353d1b4fe80266cd3ac60162_dna_r10.4.1_e8.2_400bps_sup@v5.0.0
CTTCCAGTCTCGCGAAGGCGGAAATGCTGTCTGCGTGCGTTTCAAATGCCTTCCGGCTGTAAGAGATGATGCTCGATTTCTTTTGAAAATCTGTTACATTCAACGGGCTTGAAAACCGCGCCGTTCCGTTTGTCGGCAAAACGTGATTCGGTCCGGCAAAATAATCGCCGACCGGTTCAGCGCTGTATCGTCCTAAGAAAATCGCTCCCGCGTGTCTGATGCTTCCAAGCAAAGCCTCCGGCGAATGCGTCATAATTTCTAAATGCTCGGGCGCCAGCGCATTCACCGTGTCCACCGCATCTTCCATCGATTCCGTAATATAAATGCGGCCGTGATCCTTGATCGACCCTTCGGCGATTTCTTTACGCGGCAGTGTCTGAAGCTGTTTATTCACCTCATCTGATACGGCTTCGGCAAGCTTCCGTGAGTCGGTGACAAGCACGCTTGAGCTGAGCGTATCGTGTTCGGCCTGTGAAAGCAGG
+
=@GSSGMINSNSSLKKIEFFELNSSIOSSSSSSSNNSSSIGEAABABHSSMJSSSSSSQSPSSSSSSNSSSSSHJHQJJKOJJC9MOSFSCBBBCBABEJHHHHSSSSSSKMLKHGFCCDHJNLJLNMSSHSSLJMSSSSSSKI==<;>>>>?AEEGIILOD?<=<<FFHGGEEJBA>:;;<=HGGHFJFLIKSDFMEQSSSNORQSIKLJKSSSSSSMJPPJHNJ@:;;;HSRISSOGEFGOSSHLSSSNMH+++++?H?@@SSSSHHFDDLOPOJSSSIIIPJKSQSC==9/..**,,+,,,,,3M@@@@@SSSGGGISHKSGA@-,,,,2226CDFG>>?CDGFFKSSHHSQLKPSSSSSSNSOSQSSOMJLOHIFGGSJISSSSOSSJJISSJPLQHFCBCEDDFIFIHGFJMMHLLGCCCCFSSQGKHSNKSMKMFSIGGGNSSNKCBBCAJSSKSSSSSSSSSSSNNCBB@@@<75
@0b941ebe-49bd-45ca-af16-9bbc5491a523   qs:f:29.7949    st:Z:2024-04-27T23:58:53.183+00:00      RG:Z:bcf4b7732185c1a3353d1b4fe80266cd3ac60162_dna_r10.4.1_e8.2_400bps_sup@v5.0.0
CTAGTGATAGATACACCGTTTCTTGGTTCGTAGTAGCGGGCCATGAGATAGTACAGGCCGGTTTCTTCGTCGTATTGGTAGCCTGCGTAGCGGTAGCGGTTGTCTTTTACTTCGTCGCTTGCTTCGGTTTTTGTCGGATTTCCCCATGCATCGTACTGATATTTGGCAACGGTTTTTCCGGTGCTGTCTGAGATGGCGATGACGTCGCCGTGTGCGTTGTAGTGATAGAAATATTTCTTGCCGTTCTCTGTATAGGATAACAGCTGGCCGCTGTCACCGTACGTGTATGATTTTGTGACGTTGTTGTCGGCGTCTGTTTCATACAGGACGTTCAGGCTGTCTCCGTCGTAGAAGTAGTTCGTAACTTTTCCGTTGACGGTTTTTTGGATTCTGTTTCCTTTTTCATCGTATTTGTATGTTGCGAACGGCTTGTCTTCGCCTTTTTTCGTGACGGCGGTCAGGTTGTCTTCCGCGTCCCACGTATATGTGTATTTGCCGTCCGATGTGCGGTTGCCGTTTTTATCATAGGACAGGTCTTCGTCATTCACCTTCGTCAGCTGGTTCATGATGTTGAAGGATGCGTTGACTGATTTGCTTGCGCCGTCCTTTGTGGTGGTGACGTTTGTCCGGTTGCCGAAGCCGTCGTACGTATATTGAATGACGGTGCCGTCTTCATGAGTTTCTTTGACGAGCTGGTTGAGCTTGCCGTATTCGTACTGCACCTTTCCGCCCGCTGAGCTGTCGATGACCGTGCGGTTGCCGTTGGCGTCATATTCATAGCTTTCCCGGAAGATGCTGCCGCCGTTTTTCGTTCCGATATGAAGAGAGCTGACGAGATTCCGCTCATCATATGAAAAACTCGTGCCGACTTCATTGCCGCTGATGAATGTCTGGACATTGCCGTTTTCATCATAATCAAATGTATAGGCTGAAGTGCCGTCTTTCATTTCAATCATTTGATCAAGTTTGTTGTACGTAAAGCTGTTTGTTCCTTTTT
```

```bash
ls -lh

gzip b14.fastq

ls -lh
```

## 5. Análisis de calidad de archivos FASTQ de Nanopore

### Visualización de la calidad

```bash
cd ~/genomics/quality/nanopore

NanoPlot -t 10 --fastq ~/genomics/basecalling/pod5_db_sup/b14.fastq.gz -p b14_sup_raw_ -o b14_sup_raw --maxlength 1000000 --only-report
```

> **Comentario:** 
> - `-t 10`: Número de hilos que usará NanoPlot.
> - `--fastq ~/genomics/basecalling/pod5_db_sup/b14.fastq.gz`: Indica la ruta del archivo FASTQ que contiene las lecturas de secuenciación de ONT que se van a analizar.
> - `-p b14_sup_raw_`: Define el prefijo que se usará para los nombres de los archivos de salida. En este caso, todos los gráficos generados comenzarán con "b14_sup_raw_".
> - `-o b14_sup_raw`: Especifica el directorio de salida donde se guardarán los gráficos. Si el directorio no existe, NanoPlot lo creará.
> - `--maxlength 1000000`: Oculta las lecturas más largas que este valor (1 Mb). Esas lecturas **se excluyen** de los gráficos y de las estadísticas (no se truncan). Sirve para que unas pocas lecturas extremadamente largas no distorsionen la visualización.
> - `--only-report`: Reduce los archivos de salida; el resumen queda en el reporte HTML y en el archivo `NanoStats.txt`.

> **Cómo leer el resumen:**
> - **N50**: longitud tal que las lecturas de esa longitud o mayores suman la mitad de las bases totales. Es mayor que la mediana porque da más peso a las lecturas largas.
> - **Mean read quality** vs **Median read quality**: promedio y mediana de la calidad media de cada lectura.
> - **>Q10, >Q15, >Q20...**: número, porcentaje y megabases de lecturas cuya calidad media supera ese umbral (es un conteo de *lecturas*, no de bases).

``bash
cat b14_sup_raw/b14_sup_raw_NanoStats.txt

General summary:         
Mean read length:              1,016.7
Mean read quality:                19.7
Median read length:              682.0
Median read quality:              21.1
Number of reads:               2,789.0
Read length N50:               1,381.0
STDEV read length:             1,206.3
Total bases:               2,835,518.0
Number, percentage and megabases of reads above quality cutoffs
>Q10:   2788 (100.0%) 2.8Mb
>Q15:   2655 (95.2%) 2.7Mb
>Q20:   1731 (62.1%) 1.8Mb
>Q25:   539 (19.3%) 0.4Mb
>Q30:   147 (5.3%) 0.1Mb
Top 5 highest mean basecall quality scores and their read lengths
1:      48.5 (20)
2:      47.4 (147)
3:      46.7 (180)
4:      46.4 (111)
5:      45.3 (161)
Top 5 longest reads and their mean basecall quality score
1:      20783 (26.5)
2:      14487 (23.3)
3:      13443 (15.4)
4:      12216 (16.7)
5:      9918 (16.9)
```

> **Puntos de control:** Descargue `b14_sup_raw/b14_sup_raw_NanoPlot-report.html` con WinSCP y responda: (1) ¿Qué porcentaje de lecturas supera Q20? (2) ¿Es simétrica la distribución de longitudes? ¿Qué indican la media y la mediana? (3) ¿Por qué el Q30(%) de `seqkit stats` (80,68 %) es mucho mayor que el porcentaje de lecturas >Q30 de NanoPlot (5,3 %)?

## 6. Limpieza de los archivos FASTQ de Nanopore

### Eliminación de adaptadores

```bash
cd ~/genomics/trimming/nanopore

porechop -t 10 -i ~/genomics/basecalling/pod5_db_sup/b14.fastq.gz -o b14_sup_porechop.fastq.gz > b14_porechop.log 2> b14_porechop.err
```

> **Comentario:**
> - `-t 10`: Número de hilos que usará Porechop.
> - `-i ~/genomics/basecalling/pod5_db_sup/b14.fastq.gz`: Esta opción indica la ruta del archivo FASTQ de entrada. Este es el archivo que contiene las lecturas de secuenciación de ONT que se van a procesar.
> - `-o b14_sup_porechop.fastq.gz`: Esta opción especifica el nombre del archivo FASTQ de salida comprimido con gzip. Este archivo contendrá las lecturas después de que Porechop haya recortado los adaptadores de los extremos y dividido las lecturas que tenían un adaptador en su interior (quimeras). Los fragmentos resultantes menores a 1000 pb se descartan por defecto.
> - `> b14_porechop.log 2> b14_porechop.err`: Guarda el resumen del proceso y los posibles errores en archivos separados. `tail` permite ver el resumen: cuántas lecturas tenían adaptadores recortados y cuántas fueron divididas.

```bash
cat b14_porechop.log

Loading reads
/home/alumno01/genomics/basecalling/pod5_db_sup/b14.fastq.gz
2,789 reads loaded


Looking for known adapter sets
2,789 / 2,789 (100.0%)
                                        Best               
                                        read       Best    
                                        start      read end
  Set                                   %ID        %ID     
  SQK-NSK007                                73.3       79.2
  Rapid                                     68.5        0.0
  RBK004_upstream                           68.3        0.0
  SQK-MAP006                                75.9       77.3
  SQK-MAP006 short                          76.9       75.0
  PCR adapters 1                            78.3       78.3
  PCR adapters 2                            79.2       77.3
  PCR adapters 3                            76.0       76.9
  1D^2 part 1                               71.4       73.3
  1D^2 part 2                               72.2       73.3
  cDNA SSP                                  73.2       70.0
  Barcode 1 (reverse)                       75.0       76.9
  Barcode 2 (reverse)                       76.9       79.2
  Barcode 3 (reverse)                       75.0       73.1
  Barcode 4 (reverse)                       75.0       76.0
  Barcode 5 (reverse)                       73.1       74.1
  Barcode 6 (reverse)                       75.0       76.9
  Barcode 7 (reverse)                       83.3       75.0
  Barcode 8 (reverse)                       76.0       77.8
  Barcode 9 (reverse)                       75.0       74.1
  Barcode 10 (reverse)                      76.0       76.0
  Barcode 11 (reverse)                      76.0       76.9
  Barcode 12 (reverse)                      76.9       84.0
  Barcode 1 (forward)                       79.2       76.0
  Barcode 2 (forward)                       76.9       80.0
  Barcode 3 (forward)                       75.0       75.0
  Barcode 4 (forward)                       77.8       84.6
  Barcode 5 (forward)                       77.8       77.8
  Barcode 6 (forward)                       76.9       76.0
  Barcode 7 (forward)                       76.0       76.9
  Barcode 8 (forward)                       76.9       76.9
  Barcode 9 (forward)                       74.1       75.0
  Barcode 10 (forward)                      79.2       76.9
  Barcode 11 (forward)                      76.0       76.0
  Barcode 12 (forward)                      76.9       80.0
  Barcode 13 (forward)                      75.0       79.2
  Barcode 14 (forward)                      92.0      100.0
  Barcode 15 (forward)                      76.0       73.3
  Barcode 16 (forward)                      75.0       75.0
  Barcode 17 (forward)                      76.0       76.9
  Barcode 18 (forward)                      72.4       76.0
  Barcode 19 (forward)                      76.0       76.9
  Barcode 20 (forward)                      75.0       76.0
  Barcode 21 (forward)                      75.0       76.0
  Barcode 22 (forward)                      74.1       76.9
  Barcode 23 (forward)                      76.0       74.1
  Barcode 24 (forward)                      76.9       80.8
  Barcode 25 (forward)                      76.9       74.1
  Barcode 26 (forward)                      79.2       76.0
  Barcode 27 (forward)                      76.0       75.0
  Barcode 28 (forward)                      76.9       79.2
  Barcode 29 (forward)                      74.1       73.1
  Barcode 30 (forward)                      72.0       75.0
  Barcode 31 (forward)                      76.9       76.0
  Barcode 32 (forward)                      76.0       75.0
  Barcode 33 (forward)                      76.7       72.0
  Barcode 34 (forward)                      78.6       80.0
  Barcode 35 (forward)                      76.0       75.0
  Barcode 36 (forward)                      76.9       76.0
  Barcode 37 (forward)                      79.2       75.0
  Barcode 38 (forward)                      75.9       76.0
  Barcode 39 (forward)                      76.0       75.0
  Barcode 40 (forward)                      75.0       73.1
  Barcode 41 (forward)                      76.0       76.0
  Barcode 42 (forward)                      76.0       75.0
  Barcode 43 (forward)                      76.0       79.2
  Barcode 44 (forward)                      76.0       80.8
  Barcode 45 (forward)                      76.9       76.0
  Barcode 46 (forward)                      77.8       80.0
  Barcode 47 (forward)                      75.0       74.1
  Barcode 48 (forward)                      76.0       76.0
  Barcode 49 (forward)                      75.0       76.0
  Barcode 50 (forward)                      75.0       75.0
  Barcode 51 (forward)                      76.9       75.0
  Barcode 52 (forward)                      76.0       76.0
  Barcode 53 (forward)                      80.8       80.0
  Barcode 54 (forward)                      76.0       75.9
  Barcode 55 (forward)                      75.0       76.0
  Barcode 56 (forward)                      76.0       76.0
  Barcode 57 (forward)                      75.0       76.9
  Barcode 58 (forward)                      76.0       75.0
  Barcode 59 (forward)                      79.2       79.2
  Barcode 60 (forward)                      75.0       72.0
  Barcode 61 (forward)                      76.0       79.2
  Barcode 62 (forward)                      75.0       77.8
  Barcode 63 (forward)                      75.0       76.9
  Barcode 64 (forward)                      79.2       79.2
  Barcode 65 (forward)                      75.0       80.0
  Barcode 66 (forward)                      76.0       75.0
  Barcode 67 (forward)                      75.0       76.0
  Barcode 68 (forward)                      72.0       73.1
  Barcode 69 (forward)                      80.0       76.0
  Barcode 70 (forward)                      73.1       76.0
  Barcode 71 (forward)                      74.1       76.0
  Barcode 72 (forward)                      76.9       81.5
  Barcode 73 (forward)                      76.9       77.8
  Barcode 74 (forward)                      74.1       76.0
  Barcode 75 (forward)                      76.0       80.0
  Barcode 76 (forward)                      75.0       76.0
  Barcode 77 (forward)                      76.9       76.0
  Barcode 78 (forward)                      80.0       76.9
  Barcode 79 (forward)                      76.0       76.0
  Barcode 80 (forward)                      75.0       80.0
  Barcode 81 (forward)                      76.9       76.0
  Barcode 82 (forward)                      75.0       77.8
  Barcode 83 (forward)                      76.9       75.0
  Barcode 84 (forward)                      73.1       73.1
  Barcode 85 (forward)                      76.9       74.1
  Barcode 86 (forward)                      73.1       72.0
  Barcode 87 (forward)                      83.3       73.1
  Barcode 88 (forward)                      76.9       75.0
  Barcode 89 (forward)                      75.0       75.0
  Barcode 90 (forward)                      74.1       75.0
  Barcode 91 (forward)                      75.0       75.0
  Barcode 92 (forward)                      76.0       76.9
  Barcode 93 (forward)                      75.0       79.2
  Barcode 94 (forward)                      75.0       73.1
  Barcode 95 (forward)                      80.0       76.9
  Barcode 96 (forward)                      76.0       75.0


Trimming adapters from read ends
      BC14: AACGAGTCTCTTGGGACCCATAGA
  BC14_rev: TCTATGGGTCCCAAGAGACTCGTT

2,789 / 2,789 (100.0%)

  152 / 2,789 reads had adapters trimmed from their start (1,733 bp removed)
  120 / 2,789 reads had adapters trimmed from their end (1,278 bp removed)


Splitting reads containing middle adapters
2,789 / 2,789 (100.0%)

0 / 2,789 reads were split based on middle adapters


Saving trimmed reads to file
pigz found - using it to compress instead of gzip

Saved result to /home/alumno01/genomics/trimming/nanopore/b14_sup_porechop.fastq.gz
```

### Eliminación de lecturas considerando su calidad y longitud

```bash
gunzip -c ~/genomics/trimming/nanopore/b14_sup_porechop.fastq.gz | NanoFilt -q 10 --length 1000 | gzip > b14_sup_nanofilt.fastq.gz
```

> **Comentario:**
> - `gunzip -c ~/genomics/trimming/nanopore/b14_sup_porechop.fastq.gz`: Descomprime el archivo "b14_sup_porechop.fastq.gz".
> - `NanoFilt -q 10 --length 1000`: Filtra las lecturas descomprimidas utilizando NanoFilt, manteniendo solo aquellas que tengan un puntaje de calidad medio mínimo de 10 y una longitud mínima de 1000 bases. Recuerde que Dorado ya descartó las lecturas con Q medio menor a 10 (`--min-qscore 10`), por lo que en este dato el filtro de calidad casi no elimina lecturas: el efecto principal es el de la longitud.
> - `gzip > b14_sup_nanofilt.fastq.gz`: Comprime las lecturas filtradas y las guarda en un nuevo archivo llamado "b14_sup_nanofilt.fastq.gz". Este es el **FASTQ final de la limpieza**, listo para ser utilizado en el ensamblaje de genomas.
> - `|`: Este símbolo es una "tubería" (pipe) que conecta la salida de un comando a la entrada de otro.

### Estadísticas de cada etapa y porcentaje de lecturas perdidas

```bash
seqkit stats -a -T -j 10 ~/genomics/basecalling/pod5_db_sup/b14.fastq.gz b14_sup_porechop.fastq.gz b14_sup_nanofilt.fastq.gz > b14_stats_fastq.tsv
```

> **Comentario:**
> - `seqkit stats -a -T`: calcula todas las estadísticas y las entrega en formato tabular (TSV). Se incluyen, **en orden**, el FASTQ crudo y el resultado de cada etapa de limpieza.

```bash
cat b14_stats_fastq.tsv

file    format  type    num_seqs        sum_len min_len avg_len max_len Q1      Q2      Q3      sum_gap N50     N50_num Q20(%)  Q30(%)  AvgQual GC(%)   sum_n
/home/alumno01/genomics/basecalling/pod5_db_sup/b14.fastq.gz    FASTQ   DNA     2789    2835518 13      1016.7  20783   428.0   682.0   1145.0  0       1381    461     89.65   80.68   19.59     45.73   0
b14_sup_porechop.fastq.gz       FASTQ   DNA     2785    2832515 13      1017.1  20783   428.0   682.0   1145.0  0       1381    460     89.67   80.70   19.61   45.74   0
b14_sup_nanofilt.fastq.gz       FASTQ   DNA     852     1804816 1000    2118.3  20783   1210.5  1556.0  2287.0  0       2255    209     89.83   81.04   19.54   45.54   0
```

## 7. Análisis de contaminación con Kraken2

Antes de usar las lecturas limpias en un ensamblaje conviene verificar que provienen del organismo esperado. Kraken2 clasifica cada lectura contra una base de datos de referencia y permite detectar contaminación (por ejemplo, ADN humano, bacterias de laboratorio u otros organismos).

```bash
cd ~/genomics/contamination

conda activate shotgun

kraken2 -db /data/db/kraken2/k2_pluspf/ --threads 30 --use-names ~/genomics/trimming/nanopore/b14_sup_nanofilt.fastq.gz --output b14.kraken --report b14.report
```

> **Comentario:**
> - `conda activate shotgun`: cambia de entorno conda a `shotgun`.
> - `-db /data/db/kraken2/k2_pluspf/`: ruta de la base de datos de Kraken2. La base **PlusPF** incluye arqueas, bacterias, virus, plásmidos, humano, protozoos y hongos; **no incluye plantas** ni la mayoría de animales, por lo que las lecturas de esos organismos pueden quedar sin clasificar.
> - `--threads 30`: número de hilos de cómputo. El servidor es compartido, así que no aumente este valor.
> - `--use-names`: muestra el nombre científico de cada taxón (y no solo su código de taxonomía) en los resultados.
> - `b14_sup_nanofilt.fastq.gz`: archivo FASTQ de entrada con las lecturas que se van a clasificar.
> - `--output b14.kraken`: archivo con la clasificación de **cada lectura** (clasificada `C` o no clasificada `U`, identificador de la lectura, taxón asignado y longitud).
> - `--report b14.report`: reporte resumen por taxón (porcentaje de lecturas, número de lecturas, rango taxonómico, código de taxonomía y nombre).

> **Nota:** El comando puede tardar varios minutos, porque Kraken2 carga en memoria la base de datos (varias decenas de GB): no lo ejecute varias veces en paralelo.

### Visualización de los resultados

```bash
cat b14.report

  0.22  93      93      U       0       unclassified
 99.78  43051   3       R       1       root
 99.64  42988   2       R1      131567    cellular organisms
 99.60  42971   13      D       2           Bacteria
 95.54  41220   3       D1      1783272       Terrabacteria group
 95.47  41190   2       P       1239            Bacillota
 95.46  41185   14      C       91061             Bacilli
 94.60  40814   23      O       186826              Lactobacillales
 90.34  38975   12      F       1300                  Streptococcaceae
 80.46  34715   574     G       1357                    Lactococcus
 64.74  27932   8223    S       1358                      Lactococcus lactis
 45.68  19709   14777   S1      1360                        Lactococcus lactis subsp. lactis
  3.69  1594    1594    S2      1046624                       Lactococcus lactis subsp. lactis IO-1
  3.60  1554    1554    S2      44688                         Lactococcus lactis subsp. lactis bv. diacetylactis
  1.28  552     552     S2      1117941                       Lactococcus lactis subsp. lactis NCDO 2118
  1.23  532     532     S2      684738                        Lactococcus lactis subsp. lactis KF147
  0.82  353     353     S2      272623                        Lactococcus lactis subsp. lactis Il1403
  0.45  192     192     S2      889971                        Lactococcus lactis subsp. lactis K214
  0.36  155     155     S2      929102                        Lactococcus lactis subsp. lactis CV56
  9.01  3889    0       G1      2643510                   unclassified Lactococcus
  8.95  3862    3862    S       44273                       Lactococcus sp.
  0.05  20      20      S       2879149                     Lactococcus sp. NH2-7C
  0.01  6       6       S       3037457                     Lactococcus sp. bn62
  0.00  1       1       S       2816912                     Lactococcus sp. LG606
  3.20  1381    1071    S       1359                      Lactococcus cremoris
  0.72  310     2       S1      2816960                     Lactococcus cremoris subsp. cremoris
  0.37  158     158     S2      1449093                       Lactococcus cremoris subsp. cremoris IBB477
  0.24  102     102     S2      1104322                       Lactococcus cremoris subsp. cremoris A76
  0.06  26      26      S2      1295826                       Lactococcus cremoris subsp. cremoris KW2
  0.02  10      10      S2      272622                        Lactococcus cremoris subsp. cremoris SK11
  0.02  10      10      S2      1111678                       Lactococcus cremoris subsp. cremoris UC509.9
  0.00  2       2       S2      416870                        Lactococcus cremoris subsp. cremoris MG1363
  1.30  561     559     S       1363                      Lactococcus garvieae
  0.00  2       2       S1      1890280                     Lactococcus garvieae subsp. garvieae
  0.32  139     139     S       1940789                   Lactococcus petauri
  0.26  114     0       S       1364                      Lactococcus piscium
  0.26  114     114     S1      297352                      Lactococcus piscium MKFS47
  0.15  63      1       S       1281486                   Lactococcus formosensis
  0.14  62      62      S1      2906461                     Lactococcus formosensis subsp. formosensis
  0.10  42      42      S       1366                      Lactococcus raffinolactis
  0.04  16      16      S       1151742                   Lactococcus taiwanensis
  0.00  2       2       S       2592653                   Lactococcus protaetiae
  0.00  1       1       S       2419773                   Lactococcus allomyrinae
  0.00  1       1       S       2749962                   Lactococcus paracarnosus
  9.85  4248    16      G       1301                    Streptococcus
  8.98  3875    3104    S       1308                      Streptococcus thermophilus
  0.33  144     144     S1      1435974                     Streptococcus thermophilus TH982
  0.31  132     132     S1      1433288                     Streptococcus thermophilus MTH17CL396
  0.22  94      94      S1      1433289                     Streptococcus thermophilus M17PTZA496
  0.15  66      66      S1      1435981                     Streptococcus thermophilus 1F8CT
  0.15  63      63      S1      1435972                     Streptococcus thermophilus TH985
  0.14  62      62      S1      1423145                     Streptococcus thermophilus TH1436
  0.10  44      44      S1      1051074                     Streptococcus thermophilus JIM 8232
  0.10  41      41      S1      1436725                     Streptococcus thermophilus TH1477
  0.09  37      37      S1      264199                      Streptococcus thermophilus LMG 18311
  0.09  37      37      S1      1408178                     Streptococcus thermophilus ASCC 1275
  0.05  23      23      S1      322159                      Streptococcus thermophilus LMD-9
  0.03  11      11      S1      299768                      Streptococcus thermophilus CNRZ1066
  0.02  10      10      S1      767463                      Streptococcus thermophilus ND03
  0.01  5       5       S1      1415776                     Streptococcus thermophilus TH1435
  0.00  2       2       S1      1187956                     Streptococcus thermophilus MN-ZLW-002
  0.42  182     182     S       1501662                   Streptococcus parasuis
  0.15  66      4       S       1348                      Streptococcus parauberis
  0.14  62      62      S1      873447                      Streptococcus parauberis NCFD 2020
  0.09  37      37      S       59310                     Streptococcus macedonicus
  0.04  17      17      S       82348                     Streptococcus pluranimalium
  0.03  13      11      S       315405                    Streptococcus gallolyticus
  0.00  2       0       S1      53354                       Streptococcus gallolyticus subsp. gallolyticus
  0.00  2       2       S2      990317                        Streptococcus gallolyticus subsp. gallolyticus ATCC BAA-2069
  0.02  8       8       S       113107                    Streptococcus australis
  0.01  6       6       S       1307                      Streptococcus suis
  0.01  4       4       S       1311                      Streptococcus agalactiae
  0.01  4       4       S       102684                    Streptococcus infantarius
  0.00  2       2       S       1314                      Streptococcus pyogenes
  0.00  2       0       S       45634                     Streptococcus cristatus
  0.00  2       2       S1      1302863                     Streptococcus cristatus AS 1.3089
  0.00  2       2       S       1335                      Streptococcus equinus
  0.00  2       0       G1      119603                    Streptococcus dysgalactiae group
  0.00  2       0       S       1334                        Streptococcus dysgalactiae
  0.00  2       2       S1      99822                         Streptococcus dysgalactiae subsp. dysgalactiae
  0.00  2       1       S       197614                    Streptococcus pasteurianus
  0.00  1       1       S1      981540                      Streptococcus pasteurianus ATCC 43144
  0.00  2       2       S       1304                      Streptococcus salivarius
  0.00  2       2       S       684066                    Streptococcus lactarius
  0.00  1       1       S       102886                    Streptococcus didelphis
  0.00  1       1       S       1349                      Streptococcus uberis
  0.00  1       1       S       149016                    Streptococcus urinalis
  0.00  1       0       S       1309                      Streptococcus mutans
  0.00  1       1       S1      511691                      Streptococcus mutans NN2025
  0.00  1       1       S       1310                      Streptococcus sobrinus
  0.00  1       1       S       1329                      Streptococcus canis
  2.53  1093    3       F       33958                 Lactobacillaceae
  1.85  798     3       G       1243                    Leuconostoc
  1.56  674     406     S       1245                      Leuconostoc mesenteroides
  0.48  207     203     S1      33967                       Leuconostoc mesenteroides subsp. mesenteroides
  0.01  4       4       S2      203120                        Leuconostoc mesenteroides subsp. mesenteroides ATCC 8293
  0.13  55      55      S1      2026657                     Leuconostoc mesenteroides subsp. jonggajibkimchii
  0.01  3       3       S1      33966                       Leuconostoc mesenteroides subsp. dextranicum
  0.01  3       3       S1      427140                      Leuconostoc mesenteroides KFRI-MG
  0.07  32      32      S       33968                     Leuconostoc pseudomesenteroides
  0.07  31      31      S       2766470                   Leuconostoc falkenbergense
  0.04  17      9       S       33964                     Leuconostoc citreum
  0.02  8       8       S1      349519                      Leuconostoc citreum KM20
  0.03  14      10      S       1252                      Leuconostoc carnosum
  0.01  4       4       S1      1229758                     Leuconostoc carnosum JB16
  0.03  13      13      S       1511761                   Leuconostoc suionicum
  0.01  5       0       G1      3016637                   Leuconostoc gelidum group
  0.01  5       3       S       115778                      Leuconostoc gasicomitatum
  0.00  1       1       S1      762550                        Leuconostoc gasicomitatum LMG 18811
  0.00  1       1       S1      1165892                       Leuconostoc gasicomitatum KG16-1
  0.01  4       4       S       1246                      Leuconostoc lactis
  0.01  3       0       S       136609                    Leuconostoc kimchii
  0.01  3       3       S1      762051                      Leuconostoc kimchii IMSNU 11154
  0.00  2       0       G1      2685106                   unclassified Leuconostoc
  0.00  2       2       S       2698683                     Leuconostoc sp. MTCC 10508
```

> **Punto de control:** Responda con sus datos: (1) ¿Qué porcentaje de lecturas fue clasificado y cuál quedó sin clasificar? (2) ¿El taxón con más lecturas corresponde al organismo esperado de la muestra? (3) ¿Qué otros taxones aparecen (por ejemplo, humano u otras bacterias) y con qué porcentaje? Tenga en cuenta que una lectura **no clasificada no es necesariamente un contaminante**: puede pertenecer a un organismo que no está en la base de datos, y la exactitud de las lecturas de Nanopore puede reducir la fracción que Kraken2 logra clasificar.

## 8. Análisis de calidad, limpieza y contaminación de los datos de secuenciación Nanopore generados en el curso

> - La bitácora se centra **únicamente en los datos de Nanopore** (secciones 4 a 7). Los análisis de Illumina (secciones 2 y 3) forman parte de la práctica, pero no se incluyen en la bitácora.
> - Realizar todo el proceso de basecalling, visualización de calidad, limpieza y análisis de contaminación del FASTQ de su respectivo barcode (secciones 4 a 7), cambiando `b14` por el código de su barcode en los nombres de archivos.
> - Localización de los archivos FASTQ:

```bash
tree /data/2026_2/genomics/fastq

/data/2026_2/genomics/fastq
├── b13.fastq.gz
├── b14.fastq.gz
├── b15.fastq.gz
├── b16.fastq.gz
├── b17.fastq.gz
└── b18.fastq.gz
```

> - Mantener la siguiente estructura de carpetas:

```bash
~/genomics/
├── basecalling/          # Archivos BAM y FASTQ iniciales (Dorado)
│   └── pod5_db_sup/
├── quality/              # Informes de FastQC, NanoPlot y MultiQC
│   ├── illumina/
│   └── nanopore/
└── trimming/             # Archivos procesados y limpios
    ├── illumina/
    │   ├── trim_galore/
    │   └── trimmomatic/
    └── nanopore/         # Resultados de Porechop, NanoFilt y Kraken2
```

### Bitácora bioinformática:

Debe incluir las siguientes secciones (solo con los datos de **Nanopore**):

1. **Carátula** (usar la carátula del modelo de bitácora, con los 6 integrantes del grupo)
2. **Título**
3. **Objetivo de la práctica**
4. **Metodología:** flujograma de los análisis realizados con los datos de Nanopore (basecalling, análisis de calidad, limpieza y contaminación)
5. **Metodología:** estructura de las carpetas
6. **Metodología:** versión de cada programa utilizado (`programa --version`), modelo de Dorado y base de datos de Kraken2 empleados
7. **Resultados:** análisis de calidad de los datos crudos de Nanopore
8. **Resultados:** estadísticas de los FASTQ en cada etapa de la limpieza (tabla generada con `seqkit stats`)
9. **Resultados:** número total y porcentaje (%) de lecturas y de bases que se perdieron en el proceso de limpieza (archivo `b14_perdidas.txt`, adaptado a su barcode)
10. **Resultados:** análisis de contaminación con Kraken2 (porcentaje de lecturas clasificadas y no clasificadas, tabla con los 10 taxones con más lecturas e interpretación)
11. **Discusión:** responder las preguntas siguientes.

### Preguntas para la discusión:

1. ¿Qué importancia tiene la limpieza de lecturas de Nanopore para los análisis posteriores (por ejemplo, el ensamblaje de genomas)?
2. ¿Qué etapa de la limpieza eliminó más lecturas y más bases? ¿Es una pérdida esperable? Justifique con sus datos.
3. Si aumentara el umbral de calidad de NanoFilt a `-q 12` o `-q 15`, o el de longitud a `--length 2000`, ¿cómo cambiaría el número de lecturas y de bases? Explique el compromiso entre calidad y cantidad de datos.
4. ¿Qué organismos identificó Kraken2 en su FASTQ? ¿Qué organismo sería el que corresponde a su muestra? ¿Qué implican el porcentaje de lecturas no clasificadas y la presencia de posibles contaminantes para un ensamblaje posterior?
