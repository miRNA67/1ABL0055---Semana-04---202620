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
    subgraph NAN["Nanopore (barcode asignado)"]
        direction LR
        B1["POD5"] --> B2["Dorado (sup)"]
        B2 --> B3["BAM a FASTQ"]
        B3 --> B4["NanoPlot (calidad cruda)"]
        B4 --> B5["Porechop"]
        B5 --> B6["minimap2 + YACRD"]
        B6 --> B7["NanoFilt"]
        B7 --> B8["seqkit stats"]
        B7 --> B9["Kraken2 (contaminación)"]
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

Kraken2 https://github.com/DerrickWood/kraken2
   - **Descripción:** Kraken2 es un clasificador taxonómico de secuencias basado en k-mers: compara cada lectura con una base de datos de genomas de referencia y le asigna un taxón. Aquí se usa para detectar contaminación en los FASTQ de Nanopore, con la base de datos PlusPF (arqueas, bacterias, virus, plásmidos, humano, protozoos y hongos). Verifique la versión instalada con `kraken2 --version`.

Minimap2 v2.28 https://github.com/lh3/minimap2
   - **Descripción:** Minimap2 es un alineador de secuencias versátil y de alta velocidad diseñado para lecturas largas (Nanopore y PacBio). En esta práctica se usa para calcular los solapamientos entre lecturas (all-vs-all) que necesita YACRD.

MultiQC v1.28.0 https://multiqc.info
   - **Descripción:** MultiQC es una herramienta que agrega informes de control de calidad de múltiples herramientas de análisis bioinformático en un único informe HTML interactivo. Es compatible con una amplia gama de herramientas, incluyendo FastQC, Cutadapt/Trim Galore! y Trimmomatic, facilitando la revisión y comparación de los resultados de control de calidad de múltiples muestras.

NanoFilt v2.8.0 https://github.com/wdecoster/nanofilt
   - **Descripción:** NanoFilt es una herramienta para filtrar datos de secuenciación de Nanopore basándose en la calidad y la longitud de los reads. Permite seleccionar reads de alta calidad para análisis posteriores.

NanoPlot v1.41.6 https://github.com/wdecoster/NanoPlot
   - **Descripción:** NanoPlot es una herramienta para la visualización de datos de secuenciación de Nanopore. Genera varios tipos de gráficos para evaluar la calidad y las características de los reads, como la distribución de longitudes y la calidad a lo largo de los reads.

PycoQC v2.5.2 https://github.com/a-slide/pycoQC *(opcional: no se utiliza en esta práctica)*
   - **Descripción:** PycoQC es una herramienta para el control de calidad de datos de secuenciación de Oxford Nanopore, similar a FastQC pero diseñada específicamente para este tipo de datos. Genera informes interactivos en HTML con diversas métricas de calidad.

Porechop v0.2.4 https://github.com/rrwick/Porechop
   - **Descripción:** Porechop es una herramienta para identificar y recortar adaptadores en los extremos de los reads de Oxford Nanopore. Cuando detecta un adaptador en el interior de un read, lo considera quimérico y lo divide en reads separados. *Nota: su autor lo declaró oficialmente sin mantenimiento en 2018; se usa aquí con fines didácticos para comparar con el recorte que ya realiza Dorado.*

TrimGalore v0.6.10 https://github.com/FelixKrueger/TrimGalore
   - **Descripción:** Trim Galore! es un wrapper alrededor de Cutadapt y FastQC para realizar el recorte de adaptadores y el control de calidad en datos de secuenciación de alto rendimiento. Automatiza el proceso de recorte y genera informes de calidad.

Trimmomatic v0.39 http://www.usadellab.org/cms/?page=trimmomatic
   - **Descripción:** Trimmomatic es una herramienta flexible y rápida para realizar el recorte de adaptadores y el filtrado de calidad en datos de secuenciación de Illumina. Permite eliminar secuencias de adaptadores, bases de baja calidad y reads demasiado cortos.

YACRD v0.6.2 https://github.com/natir/yacrd
   - **Descripción:** YACRD (Yet Another Chimeric Read Detector) es una herramienta especializada en la detección de lecturas quiméricas y regiones sin cobertura en datos de secuenciación de lecturas largas. Permite "limpiar" (scrubb) o dividir las lecturas donde se detectan uniones accidentales, mejorando significativamente la contigüidad y precisión de los ensamblajes genómicos posteriores.

### Herramientas auxiliares (ya instaladas en el servidor):

SAMtools https://www.htslib.org
   - **Descripción:** Conjunto de utilidades para manipular archivos SAM/BAM. Aquí se usa para ordenar los BAM generados por Dorado.

BEDTools https://bedtools.readthedocs.io
   - **Descripción:** Conjunto de utilidades para manipular archivos genómicos. Aquí se usa `bamtofastq` para convertir BAM a FASTQ.

SeqKit https://bioinf.shenwei.me/seqkit/
   - **Descripción:** Kit de herramientas para manipular y resumir archivos FASTA/FASTQ. Aquí se usa para calcular estadísticas de los FASTQ (`stats`) y, antes de Kraken2, para renombrar las lecturas (`rename`).

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

mkdir -p ~/genomics/{basecalling/pod5_db_sup,quality/{illumina,nanopore},trimming/{illumina/{trim_galore,trimmomatic},nanopore}}

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

fastqc -t 2 /data/2025_1/database/illumina/CAT_R1.fastq.gz /data/2025_1/database/illumina/CAT_R2.fastq.gz -o .
```

> **Comentario:** 
> - `-t 2`: Esta opción especifica el número de hilos (threads) que FastQC debe utilizar. FastQC procesa un archivo por hilo, por lo que con 2 hilos analiza R1 y R2 al mismo tiempo.
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
cat > NexteraPE.fa << 'EOF'
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
EOF

cat NexteraPE.fa
```

> **Comentario:** También puede crear el archivo con `nano NexteraPE.fa`, pegar el contenido (sin las líneas `cat > ...` ni `EOF`), guardar con `Ctrl+O` y salir con `Ctrl+X`.

```bash
trimmomatic PE -threads 2 -phred33 /data/2025_1/database/illumina/CAT_R1.fastq.gz /data/2025_1/database/illumina/CAT_R2.fastq.gz CAT_R1.trim.fastq.gz CAT_R1.unpaired.fastq.gz CAT_R2.trim.fastq.gz CAT_R2.unpaired.fastq.gz ILLUMINACLIP:NexteraPE.fa:2:30:10 SLIDINGWINDOW:4:30 MINLEN:50 2> trimmomatic_CAT.log

cat trimmomatic_CAT.log
```

> **Comentario:**
> - `PE`: Indica que los datos son pareados (paired-end).
> - `-threads 2`: Número de hilos. **Las opciones de Trimmomatic (`-threads`, `-phred33`) van antes de los archivos de entrada**; si se colocan al final, Trimmomatic las interpreta como un paso de limpieza y da error.
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
fastqc -t 2 *.trim.fastq.gz -o .

multiqc -o trimming_trimmomatic .
```

### Comparación entre los datos crudos y las dos limpiezas

```bash
cd ~/genomics/trimming/illumina

seqkit stats -a -j 2 /data/2025_1/database/illumina/CAT_R1.fastq.gz /data/2025_1/database/illumina/CAT_R2.fastq.gz trim_galore/CAT_R1_val_1.fq.gz trim_galore/CAT_R2_val_2.fq.gz trimmomatic/CAT_R1.trim.fastq.gz trimmomatic/CAT_R2.trim.fastq.gz > stats_illumina.txt

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

> **Tip:** Si la conexión de PuTTY se corta, el proceso se detiene. Para tareas largas, ejecute el comando dentro de una sesión `tmux` (si está instalado): `tmux new -s basecalling`; para salir sin detener el proceso, `Ctrl+b` y luego `d`; para volver, `tmux attach -t basecalling`.

#### Basecalling

```bash
cd ~/genomics/basecalling/pod5_db_sup

dorado basecaller sup \
  --kit-name SQK-NBD114-24 \
  --min-qscore 10 \
  --device 'cuda:0' \
  --barcode-both-ends \
  --models-directory /data/software/dorado-0.9.1-linux-x64/models \
  /data/2025_1/database/nanopore/pod5/barcode15.pod5 > b15_calls.bam
```

> **Comentario:** 
> Al iniciar, Dorado imprime el nombre del modelo que está usando (por ejemplo, `dna_r10.4.1_e8.2_400bps_sup@v5.0.0`). **Anótelo**: debe reportarse en la Metodología de la bitácora.
>
> - `sup`: Esta opción indica que se debe utilizar el modelo de basecalling de "super precisión" (super accuracy). Estos modelos están entrenados para ofrecer una mayor exactitud en la llamada de bases, a costa de un mayor tiempo de cómputo. Dorado elige automáticamente el modelo `sup` que corresponde a los datos.
> - `--kit-name SQK-NBD114-24`: Este parámetro especifica el nombre del kit de preparación de librería utilizado (Native Barcoding Kit 24 V14). Activa la **clasificación de barcodes** y le indica a Dorado qué adaptadores y barcodes buscar; por defecto Dorado también recorta los adaptadores, primers y barcodes que detecta (esto se desactiva con `--no-trim`). **No** sirve para elegir el modelo de basecalling.
> - `--min-qscore 10`: Esta opción establece un umbral de calidad mínima **para cada lectura completa**: Dorado descarta las lecturas cuyo Q-score **medio** sea menor que 10. No filtra bases individuales. El Q-score es una medida de la probabilidad de que una base llamada sea incorrecta: un Q-score de 10 significa una probabilidad de error de 1 en 10 (10%); Q20, 1 en 100 (1%); Q30, 1 en 1000 (0,1%).
> - `--device 'cuda:0'`: Esto indica que Dorado debe utilizar la primera GPU habilitada para CUDA (índice 0) disponible en tu sistema para acelerar el proceso de basecalling. El uso de la GPU puede reducir significativamente el tiempo de procesamiento.
> - `--barcode-both-ends`: Esta opción le dice a Dorado que exija códigos de barras (barcodes) en ambos extremos de la lectura. Es útil si tu protocolo de secuenciación incluyó barcodes en ambos extremos, pero es más estricto: una lectura con barcode en un solo extremo no se clasifica.
> - `--models-directory /data/software/dorado-0.9.1-linux-x64/models`: Este parámetro especifica el directorio donde Dorado busca modelos ya descargados (o donde descargaría los que falten). Es importante que la ruta sea correcta para que Dorado pueda cargar los modelos necesarios.
> - `/data/2025_1/database/nanopore/pod5/barcode15.pod5`: Esta es la ruta al archivo de entrada POD5 que contiene los datos de señal sin procesar para una muestra con el código de barras (barcode) número 15. Dorado realizará el basecalling de los datos contenidos en este archivo.
> - `> b15_calls.bam`: Esto redirige la salida estándar (stdout) del comando Dorado al archivo llamado b15_calls.bam. Como no se indicó un genoma de referencia (`--reference`), Dorado genera un archivo BAM **sin alinear** (uBAM) con las secuencias base-llamadas y sus etiquetas (por ejemplo, la clasificación de barcode).
> - `\` al final de cada línea: permite escribir un comando largo en varias líneas; no debe haber espacios después de la barra.

#### Conversión de bam a fastq

```bash
for file in *.bam; do prefix="${file%.bam}"; samtools sort -n "$file" -o "${prefix}_sorted.bam"; done

for file in *_sorted.bam; do prefix="${file%.bam}"; bedtools bamtofastq -i "${prefix}.bam" -fq "${prefix}.fastq"; done

seqkit stats -a -j 4 *_sorted.fastq > stats_sorted_fastq.txt
```

> **Comentario:** 
> - `samtools sort -n`: Ordena los archivos BAM por nombre de lectura.
> - `bedtools bamtofastq`: Convierte los archivos BAM ordenados a formato FASTQ.
> - `seqkit stats`: Calcula las estadísticas de archivos FASTQ que han sido previamente ordenados (-a: todas las estadísticas y -j: número de hilos).
>
> **Alternativa más corta (opcional):** para lecturas largas no emparejadas no es necesario ordenar por nombre; se puede convertir directamente con `samtools fastq -@ 4 b15_calls.bam > b15.fastq`. Dorado también puede escribir FASTQ directamente si se agrega `--emit-fastq` al comando de basecalling (en ese caso no se genera el BAM).

```bash
cat stats_sorted_fastq.txt

file                    format  type  num_seqs    sum_len  min_len  avg_len  max_len   Q1   Q2       Q3  sum_gap    N50  N50_num  Q20(%)  Q30(%)  AvgQual  GC(%)  sum_n
b15_calls_sorted.fastq  FASTQ   DNA      8,056  9,351,266       26  1,160.8   26,983  555  814  1,298.5        0  1,423      721   89.24   80.11    19.24   45.7      0
```

> **Comentario:** En `seqkit stats`, `Q20(%)` y `Q30(%)` son el **porcentaje de bases** con calidad ≥ Q20 y ≥ Q30, y `AvgQual` es la calidad media por lectura. No confundir con el resumen de NanoPlot (sección 5), donde `>Q20` y `>Q30` son el **número (y %) de lecturas** cuya calidad media supera ese umbral.

```bash
mv b15_calls_sorted.fastq b15.fastq
```

```bash
head b15.fastq

@0a3b5933-6ecd-4ab8-b6fe-323eeee7a012
TAAGGTTAAAACGAGTCTCTTGGGACCCATAGACAGCACCTCAAGAGCCGTGTCTCCTGTCCTTAGTGTAATCAAGCTTTTGTTTATACTTGTCAATCAGCCGCTCGTTTTCTTTGAAAATTCTGGCGGTATGAGGGCTGACCTGGTAACTTGCGATACTTGTCATTGAACGTTTTTTAAACATTTTGAACAGTTTCGCTTCTTGTTTCCGGCTGCCCCGTTTTGAAATGCCTGCTCCATTTAACCGTCACCTTCCTCTTCTATTGGCAGCATTAAATCATAAATGCTCGTTAGCGATGTGAAAAGCAAATAATCGAATTCCGTCAACAGGTTTTCCGACGTGAGTTTGATCACGTAATGGCGGTTCTGGACTGTAAACGGAATAAGCACAAGCCTGCCTTTTTGATCATAGTAGACGTCTTTACGGTTAAGACGGGACTGAACGTCTGCCGCATCAGGCATATGCTCCGTCAAGCGGTCCTTATCCAATTGTGCGGAATAGTCAAGGAAGCCGTGACATTCATTTTTTCGGCATAAGCGGCCAGCAGTCTGCT
+
EFDCDDFCEGFFGGKHJJPSE6666SLNSSSSJGGSRMPMJSSSSSSSSSSSSRSPSSNSLOKIIISSSRLS:4322/..:8=@CDSOSSSNSQSSSQSSMQNSSOSSSSSSNMRBSSSSSSOSNSSQSSNSNSSSSSSLSSQMKLISSSSSSSNQSSSLIQSSSSSSSSSQSSSSSSSSSSNSSSS////*,,,.B>>>=66:33K@22CBGSSSE@2+SSKSLNSSSSSSSMIINCBA@BSSSSSSSSSNQSSSSOSSSSRSSSMJSSSLSSMSSSSSSSSSSSSSSSSSPSSSSSSSLSSSSSSLOSSNSSSSSSSSOSSSSSKHSSSSSSSSSSLIOSSSMNSSMSSSSSSSMSSPSSSSSSOSSSSSSOSSSSSPSSSSLMOSDBBBBGDBBB@==QNPSSSSSSSSSSSSSSSSLSSSSNSSSSSSSSOSISJGNQSSJJLSOMLLJKKKOSSSSSSSQSSNSJQQKJEEFSSSSQPJJKKFHGFAA>=<?=?@3222***=BFGJKSSMMJHSLKE==GCBBCEHHISQKFEBA?==<<;643130)
@0a3d8a47-5c4d-4d13-913c-3b6f598d1449
TAAGGTTAAAACGAGTCTCTTGGGACCCATAGACAGCACCTTTTTTGTTTCATCGGAGCGCCGTTAGATTGGTTTTTATTTTTTCTGCTGCTCATTACGGGGCTTGCCAAAAAAATAAAGCATTGGCTGGAAGCAGCGGTGCGCTTTCGGGTGCTGCAAATCGTAAGCTTCGTATTTGTGATTTCACTCATCATTACGGTGGCTTCGCTTCGCTTGAGTGGATCGGATATCGGGTTTCGCTTGCGTATCATATTTCGACGCAGACAACAGCAAGCTGGATAAGAGACCATGTCATTGATTTCTGGATCAGCTTTCCGCTGTTTGCCGTGTGTGTGCTTGTGTTTTACTGGCTGATCACAAAGCATACAAAAAAATGGTGGTTTTACGCTTGGTGTAC
+
7OLSSMJJEEDE;:99>ED>AAABDISSJKIHFHMMNJIF@=666HG0+*):::;<CJJKLJOPIDDDDI>;:;;???@CIKSSSSSBC:=BB@@BDIIKSSOLNP:77788SSSSOSRNNKHGHHPMONKCARSMMSOOJKNA><<45555OONSSLNKLPOMORSQSSLCAD>@?)B@??@:*****))))))(()))43,,,++043-+,.33===,,+++?BBBPMJJSKSSSCA<3>AAAABHJ7656:=10000?CA@@@@KKLMPLSSSMIKGHECEGKKSHHID=<<::>>>IGGEEISJJJIJMSKMFFHHJJJNSSNMOSOSPSOQPSSSSSPMKSSSRSSSRPMSKK@@@@@RSNJ<4<5333634489::8988----,++++(&
@0a3f050d-7fd8-4d74-a730-655bedec67ec
AGGGCTTATTACTGATAAAAAAAGCAAAACGATTTATAAAATTGAGTCAAAACGGTCTTTGCAGCCGGGCATATACGCATTCAAGGTTTACAGACCTCTGAAGGGTACCCGGCCGACGAAGAAAAATTTGAGTGGTCAAAACCGATGAAACTCGTCAAATGCCGGGAACAGGCGACCGTTTCGAATATAAAACGGAGAAAGAGCCGGCTGAGCCGGTAAAAGAAAGCGGGGAAGAACATGAAAAAAACGCTGAAACTGATGAGTAATATTTTGTATGCCAGTCATCTTCAGCTTGATTATTGTGCTGGCGTTAACGGTGATTCAGACCCGGGCTTCCGGGGGTGAGCCGGCCATTTTCGGCTATACA
```

> **Pregunta guía:** Observe el inicio de las dos primeras lecturas. ¿Qué secuencia (de unas 40 bases) se repite exactamente igual en ambas? Una secuencia idéntica al inicio de lecturas independientes no puede ser genómica: piense de dónde proviene. Volverá a esta pregunta en la sección 6.

```bash
gzip b15.fastq
```

## 5. Análisis de calidad de archivos FASTQ de Nanopore

### Visualización de la calidad

```bash
cd ~/genomics/quality/nanopore

NanoPlot -t 2 --fastq ~/genomics/basecalling/pod5_db_sup/b15.fastq.gz -p b15_sup_raw_ -o b15_sup_raw --maxlength 1000000 --only-report

cat b15_sup_raw/b15_sup_raw_NanoStats.txt

General summary:         
Mean read length:              1,160.8
Mean read quality:                18.9
Median read length:              814.0
Median read quality:              20.3
Number of reads:               8,056.0
Read length N50:               1,423.0
STDEV read length:             1,257.0
Total bases:               9,351,266.0
Number, percentage and megabases of reads above quality cutoffs
>Q10:   8054 (100.0%) 9.4Mb
>Q15:   7484 (92.9%) 8.9Mb
>Q20:   4292 (53.3%) 5.4Mb
>Q25:   1024 (12.7%) 1.0Mb
>Q30:   162 (2.0%) 0.1Mb
Top 5 highest mean basecall quality scores and their read lengths
1:      43.1 (332)
2:      43.1 (332)
3:      41.9 (351)
4:      41.9 (351)
5:      41.3 (371)
Top 5 longest reads and their mean basecall quality score
1:      26983 (18.4)
2:      26983 (18.4)
3:      20990 (26.5)
4:      20990 (26.5)
5:      14663 (23.3)
```

> **Comentario:** 
> - `-t 2`: Número de hilos que usará NanoPlot.
> - `--fastq ~/genomics/basecalling/pod5_db_sup/b15.fastq.gz`: Indica la ruta del archivo FASTQ que contiene las lecturas de secuenciación de ONT que se van a analizar.
> - `-p b15_sup_raw_`: Define el prefijo que se usará para los nombres de los archivos de salida. En este caso, todos los gráficos generados comenzarán con "b15_sup_raw_".
> - `-o b15_sup_raw`: Especifica el directorio de salida donde se guardarán los gráficos. Si el directorio no existe, NanoPlot lo creará.
> - `--maxlength 1000000`: Oculta las lecturas más largas que este valor (1 Mb). Esas lecturas **se excluyen** de los gráficos y de las estadísticas (no se truncan). Sirve para que unas pocas lecturas extremadamente largas no distorsionen la visualización.
> - `--only-report`: Reduce los archivos de salida; el resumen queda en el reporte HTML y en el archivo `NanoStats.txt`.

> **Cómo leer el resumen:**
> - **N50**: longitud tal que las lecturas de esa longitud o mayores suman la mitad de las bases totales. Es mayor que la mediana porque da más peso a las lecturas largas.
> - **Mean read quality** vs **Median read quality**: promedio y mediana de la calidad media de cada lectura.
> - **>Q10, >Q15, >Q20...**: número, porcentaje y megabases de lecturas cuya calidad media supera ese umbral (es un conteo de *lecturas*, no de bases).

> **Puntos de control:** Descargue `b15_sup_raw/b15_sup_raw_NanoPlot-report.html` con WinSCP y responda: (1) ¿Qué porcentaje de lecturas supera Q20? (2) ¿Es simétrica la distribución de longitudes? ¿Qué indican la media y la mediana? (3) ¿Por qué el Q30(%) de `seqkit stats` (80,11 %) es mucho mayor que el porcentaje de lecturas >Q30 de NanoPlot (2,0 %)?

## 6. Limpieza de los archivos FASTQ de Nanopore

### Eliminación de adaptadores

```bash
cd ~/genomics/trimming/nanopore

porechop -t 10 -i ~/genomics/basecalling/pod5_db_sup/b15.fastq.gz -o b15_sup_porechop.fastq.gz > b15_porechop.log 2> b15_porechop.err

tail -n 25 b15_porechop.log
```

> **Comentario:**
> - `-t 10`: Número de hilos que usará Porechop.
> - `-i ~/genomics/basecalling/pod5_db_sup/b15.fastq.gz`: Esta opción indica la ruta del archivo FASTQ de entrada. Este es el archivo que contiene las lecturas de secuenciación de ONT que se van a procesar.
> - `-o b15_sup_porechop.fastq.gz`: Esta opción especifica el nombre del archivo FASTQ de salida comprimido con gzip. Este archivo contendrá las lecturas después de que Porechop haya recortado los adaptadores de los extremos y dividido las lecturas que tenían un adaptador en su interior (quimeras). Los fragmentos resultantes menores a 1000 pb se descartan por defecto.
> - `> b15_porechop.log 2> b15_porechop.err`: Guarda el resumen del proceso y los posibles errores en archivos separados. `tail` permite ver el resumen: cuántas lecturas tenían adaptadores recortados y cuántas fueron divididas.

> **Pregunta guía:** Compare el inicio de las lecturas antes y después de Porechop (`zcat archivo.fastq.gz | head -n 2`). ¿Desapareció la secuencia repetida que observó en la sección 4? Dorado ya recorta adaptadores y barcodes por defecto: ¿por qué cree que aún quedaron restos en algunas lecturas?

### Eliminación de quimeras

```bash
cd ~/genomics/trimming/nanopore

minimap2 -x ava-ont -g 500 -t 10 b15_sup_porechop.fastq.gz b15_sup_porechop.fastq.gz > b15_overlap.paf

yacrd -i b15_overlap.paf -o b15_report.yacrd -c 4 -n 0.4 scrubb -i b15_sup_porechop.fastq.gz -o b15_yacrd.fastq.gz

awk '{print $1}' b15_report.yacrd | sort | uniq -c
```

> **Comentario:** 
> - `minimap2 -x ava-ont`: Utiliza el algoritmo Minimap2 con el preajuste (preset) para solapamientos de lecturas largas de Nanopore.
> - `-g 500`: Establece la distancia máxima para el llenado de brechas (gaps) entre semillas en 500 bases. Es una configuración recomendada por los autores de YACRD para datos de Nanopore.
> - `-t 10`: Indica que el servidor utilizará 10 hilos (CPUs) para procesar el alineamiento de forma paralela y rápida.
> - `b15_sup_porechop.fastq.gz b15_sup_porechop.fastq.gz`: Realiza un mapeo de tipo "all-vs-all", comparando cada lectura contra todas las demás de la misma muestra para encontrar solapamientos consistentes.
> - `> b15_overlap.paf`: Redirige los resultados al archivo "b15_overlap.paf" en formato PAF (Pairwise Alignment Format).
> - `yacrd -i b15_overlap.paf`: Carga el archivo de solapamientos generado anteriormente como entrada para el detector de quimeras.
> - `-o b15_report.yacrd`: Crea un archivo de reporte con la clasificación de cada lectura (Chimeric, NotCovered o NotBad).
> - `-c 4`: Define el umbral de cobertura mínima. Las regiones de una lectura con cobertura menor o igual a 4 lecturas de soporte se consideran "regiones malas".
> - `-n 0.4`: Si más del 40% de la longitud de una lectura está formada por regiones malas, la lectura se marca como `NotCovered` y se elimina.
> - `scrubb`: Modo de operación que limpia la secuencia. En lugar de borrar la lectura completa si es quimérica, yacrd la corta en los puntos de unión falsos y conserva las partes reales.
> - `-i b15_sup_porechop.fastq.gz`: Indica el archivo FASTQ original que contiene las secuencias físicas que serán procesadas y cortadas.
> - `-o b15_yacrd.fastq.gz`: Genera el archivo comprimido con las lecturas sin quimeras, que se filtrará por calidad y longitud en el paso siguiente.
> - `awk '{print $1}' b15_report.yacrd | sort | uniq -c`: Cuenta cuántas lecturas fueron clasificadas como `Chimeric`, `NotCovered` o `NotBad`.

> **Importante:** Los autores de YACRD recomiendan `-c 4 -n 0.4` para conjuntos de datos con cobertura mayor a **30x**. Si la cobertura es baja (por ejemplo, pocos megabases de datos frente al tamaño del genoma de la muestra), casi todas las regiones tendrán cobertura ≤ 4 y `scrubb` eliminará o fragmentará una gran parte de las lecturas. Revise el conteo del comando `awk` y el número de lecturas resultantes antes de continuar.

### Eliminación de lecturas considerando su calidad y longitud

```bash
gunzip -c ~/genomics/trimming/nanopore/b15_yacrd.fastq.gz | NanoFilt -q 10 --length 1000 | gzip > b15_sup_nanofilt.fastq.gz
```

> **Comentario:**
> - `gunzip -c ~/genomics/trimming/nanopore/b15_yacrd.fastq.gz`: Descomprime el archivo "b15_yacrd.fastq.gz".
> - `NanoFilt -q 10 --length 1000`: Filtra las lecturas descomprimidas utilizando NanoFilt, manteniendo solo aquellas que tengan un puntaje de calidad medio mínimo de 10 y una longitud mínima de 1000 bases. Recuerde que Dorado ya descartó las lecturas con Q medio menor a 10 (`--min-qscore 10`), por lo que en este dato el filtro de calidad casi no elimina lecturas: el efecto principal es el de la longitud.
> - `gzip > b15_sup_nanofilt.fastq.gz`: Comprime las lecturas filtradas y las guarda en un nuevo archivo llamado "b15_sup_nanofilt.fastq.gz". Este es el **FASTQ final de la limpieza**, listo para ser utilizado en el ensamblaje de genomas.
> - `|`: Este símbolo es una "tubería" (pipe) que conecta la salida de un comando a la entrada de otro.

### Estadísticas de cada etapa y porcentaje de lecturas perdidas

```bash
cd ~/genomics/trimming/nanopore

seqkit stats -a -T -j 10 ~/genomics/basecalling/pod5_db_sup/b15.fastq.gz b15_sup_porechop.fastq.gz b15_yacrd.fastq.gz b15_sup_nanofilt.fastq.gz > b15_stats_fastq.tsv

column -t -s $'\t' b15_stats_fastq.tsv | less -S

awk -F'\t' 'NR==2{n0=$4; b0=$5} NR>1{printf "%-40s lecturas=%-7d (pérdida: %6.2f%%)  bases=%-10d (pérdida: %6.2f%%)\n", $1, $4, 100*(n0-$4)/n0, $5, 100*(b0-$5)/b0}' b15_stats_fastq.tsv > b15_perdidas.txt

cat b15_perdidas.txt
```

> **Comentario:**
> - `seqkit stats -a -T`: calcula todas las estadísticas y las entrega en formato tabular (TSV). Se incluyen, **en orden**, el FASTQ crudo y el resultado de cada etapa de limpieza.
> - `column -t -s $'\t' ... | less -S`: muestra la tabla alineada; use las flechas para desplazarse y `q` para salir.
> - `awk ...`: toma como referencia la primera fila (FASTQ crudo) y calcula, para cada etapa, el porcentaje de lecturas y de bases perdidas respecto al crudo. Si una etapa **divide** lecturas (como Porechop o YACRD), el número de lecturas puede aumentar y el porcentaje de pérdida saldrá negativo; por eso también se compara el número de bases.

## 7. Análisis de contaminación con Kraken2

Antes de usar las lecturas limpias en un ensamblaje conviene verificar que provienen del organismo esperado. Kraken2 clasifica cada lectura contra una base de datos de referencia y permite detectar contaminación (por ejemplo, ADN humano, bacterias de laboratorio u otros organismos).

```bash
cd ~/genomics/trimming/nanopore/

conda activate quality

seqkit rename -n b20_sup_nanofilt.fastq.gz -o b20_rename.fastq.gz

conda activate shotgun

kraken2 -db /data/db/kraken2/k2_pluspf/ --threads 30 --use-names b20_rename.fastq.gz --output b20.kraken --report b20.report
```

> **Comentario:**
> - `conda activate quality` / `conda activate shotgun`: cambia de entorno conda. SeqKit se ejecuta en el entorno `quality` y Kraken2 en el entorno `shotgun`.
> - `seqkit rename -n b20_sup_nanofilt.fastq.gz -o b20_rename.fastq.gz`: asigna identificadores únicos a las lecturas que tengan nombres repetidos, para evitar errores o ambigüedades en el análisis posterior. Se parte del FASTQ final de la limpieza (`b20_sup_nanofilt.fastq.gz`).
> - `-db /data/db/kraken2/k2_pluspf/`: ruta de la base de datos de Kraken2. La base **PlusPF** incluye arqueas, bacterias, virus, plásmidos, humano, protozoos y hongos; **no incluye plantas** ni la mayoría de animales, por lo que las lecturas de esos organismos pueden quedar sin clasificar.
> - `--threads 30`: número de hilos de cómputo. El servidor es compartido, así que no aumente este valor.
> - `--use-names`: muestra el nombre científico de cada taxón (y no solo su código de taxonomía) en los resultados.
> - `b20_rename.fastq.gz`: archivo FASTQ de entrada con las lecturas que se van a clasificar.
> - `--output b20.kraken`: archivo con la clasificación de **cada lectura** (clasificada `C` o no clasificada `U`, identificador de la lectura, taxón asignado y longitud).
> - `--report b20.report`: reporte resumen por taxón (porcentaje de lecturas, número de lecturas, rango taxonómico, código de taxonomía y nombre).

> **Nota:** En este ejemplo se usa el barcode 20 (`b20`); cambie `b20` por el código de su barcode. El comando puede tardar varios minutos, porque Kraken2 carga en memoria la base de datos (varias decenas de GB): no lo ejecute varias veces en paralelo.

### Visualización de los resultados

```bash
head -n 2 b20.report

awk -F'\t' '$4=="S"' b20.report | sort -t$'\t' -k2,2nr | head -n 10 | cut -f1,2,6

awk -F'\t' '$4=="G"' b20.report | sort -t$'\t' -k2,2nr | head -n 10 | cut -f1,2,6

head -n 3 b20.kraken | cut -f1-4
```

> **Comentario:**
> - `head -n 2 b20.report`: si hay lecturas no clasificadas, la primera línea (rango `U`, *unclassified*) indica qué porcentaje y cuántas lecturas quedaron sin clasificar; la siguiente (rango `R`, *root*) corresponde a las lecturas clasificadas.
> - Las columnas del reporte (separadas por tabulaciones) son: (1) % de lecturas del taxón y sus descendientes, (2) n.º de lecturas del taxón y sus descendientes, (3) n.º de lecturas asignadas directamente al taxón, (4) rango taxonómico, (5) código de taxonomía y (6) nombre.
> - `awk -F'\t' '$4=="S"' ...`: selecciona las filas de rango **especie** (`S`); con `"G"` se seleccionan los **géneros**. `sort -t$'\t' -k2,2nr` las ordena por número de lecturas (de mayor a menor), `head -n 10` conserva las 10 primeras y `cut -f1,2,6` muestra el porcentaje, el número de lecturas y el nombre.
> - `head -n 3 b20.kraken | cut -f1-4`: muestra la clasificación de las tres primeras lecturas.

> **Punto de control:** Responda con sus datos: (1) ¿Qué porcentaje de lecturas fue clasificado y cuál quedó sin clasificar? (2) ¿El taxón con más lecturas corresponde al organismo esperado de la muestra? (3) ¿Qué otros taxones aparecen (por ejemplo, humano u otras bacterias) y con qué porcentaje? Tenga en cuenta que una lectura **no clasificada no es necesariamente un contaminante**: puede pertenecer a un organismo que no está en la base de datos, y la exactitud de las lecturas de Nanopore puede reducir la fracción que Kraken2 logra clasificar.

## 8. Análisis de calidad, limpieza y contaminación de los datos de secuenciación Nanopore generados en el curso

> - La bitácora se centra **únicamente en los datos de Nanopore** (secciones 4 a 7). Los análisis de Illumina (secciones 2 y 3) forman parte de la práctica, pero no se incluyen en la bitácora.
> - Realizar todo el proceso de basecalling, visualización de calidad, limpieza y análisis de contaminación del FASTQ de su respectivo barcode (secciones 4 a 7), cambiando `b15` y `b20` por el código de su barcode en los nombres de archivos.
> - Localización de los archivos FASTQ:

```bash
tree /data/2026_1/genomics/
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
    └── nanopore/         # Resultados de Porechop, YACRD, NanoFilt y Kraken2
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
9. **Resultados:** número total y porcentaje (%) de lecturas y de bases que se perdieron en el proceso de limpieza (archivo `b15_perdidas.txt`, adaptado a su barcode)
10. **Resultados:** análisis de contaminación con Kraken2 (porcentaje de lecturas clasificadas y no clasificadas, tabla con los 10 taxones con más lecturas e interpretación)
11. **Discusión:** responder las preguntas siguientes.

### Preguntas para la discusión:

1. ¿Qué importancia tiene la limpieza de lecturas de Nanopore para los análisis posteriores (por ejemplo, el ensamblaje de genomas)?
2. ¿Qué etapa de la limpieza eliminó más lecturas y más bases? ¿Es una pérdida esperable? Justifique con sus datos.
3. Dorado ya recorta adaptadores y barcodes durante el basecalling. ¿Qué aportó Porechop en sus datos? ¿Qué desventajas tiene seguir usando una herramienta sin mantenimiento?
4. ¿Qué factores hacen que YACRD funcione mal en datos con baja cobertura? ¿Cómo lo comprobó en sus resultados?
5. Si aumentara el umbral de calidad de NanoFilt a `-q 12` o `-q 15`, o el de longitud a `--length 2000`, ¿cómo cambiaría el número de lecturas y de bases? Explique el compromiso entre calidad y cantidad de datos.
6. ¿Qué organismos identificó Kraken2 en su FASTQ? ¿Corresponden al organismo esperado de la muestra? ¿Qué implican el porcentaje de lecturas no clasificadas y la presencia de posibles contaminantes para un ensamblaje posterior?
