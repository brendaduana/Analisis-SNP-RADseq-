# Análisis SNPs-RADseq
Este repositorio contiene un pipeline diseñado para el procesamiento y análisis de SNPs obtenidos a partir de datos RADseq (Restriction-site Associated DNA Sequencing).  El objetivo principal es establecer un flujo de trabajo reproducible para la identificación, filtrado y análisis de SNPs en estudios de genómica poblacional.

## Pipeline para el procesamiento de datos de RADseq
 
A continuación se describen los pasos a seguir desde la obtención de datos crudos pair-end de NovaSeq hasta la construcción de la matriz final de SNPs en formato .vcf. 
Este tutorial incluye el acceso al servidor de genética y el trabajo con scripts de R en la computadora personal. 

El objetivo es exponer una guía con los pasos necesarios para que los estudiantes (con conocimeintos básicos de bash) sean capaces de procesar sus datos y garantizar su reproducibilidad.

**Programas empleados en bash**
- FastQC
- Trimmomatic
- ipyrad
- VCFtools
---

### 1. Subir archivos .fastq.gz al servidor
En mi computadora, agrupar todos los datos crudos (R1 y R2) en una carperta denominada _00.RawData_

En la terminal local colocar la siguiente instrucción:

`scp -r -P 1967 cursopcbr/00.RawData * CMeta05@132.248.15.41:/home/CMeta05/00.RawData`


La carpeta 00.RawData debió ser creada previamente en el usuario del servidor

### 1.1 Copiar los archivos de otro usuario del servidor
Crear la carpeta 00.RawData y colocarse en ella
`cp /home/massiel.alfonsoglez/curso/* .`

### 1.2 Visualizar un archivo fastq
`head archivo.fastq`
`zcat archivo.fastq.gz | head -n 20`

### 2. Conectarse al servidor

`ssh usuario@132.248.15.41 -p 1967 -o ServerAliveInterval=60`

Dentro de la carperta _00.RawData_, verificar la cantidad de secuencias subidas
`ls | wc -l`

### 3. Visualizar la calidad de los datos crudos con FastQC
`fastqc *.fastq.gz`

Como resultado se van a generar archivos .html y .zip. 

Para mantener el orden, crear una nueva carpeta denominada _01.FastQC_crudos_ en /home/usuario/
 
Entrar a la carpeta _01.FastQC_crudos_ y mover los .html y los .zip 
`mv /home/usuario/curso/00.RawData/*.html .`
`mv /home/usuario/curso/00.RawData/*.zip .`

Mover los archivos.html del servidor a la computadora local para poder visualizarlos

`scp -r -P 1967 usuario@132.248.15.41:/home/usuario/curso/01.FastQC_crudos/*fastqc.html /home/Usuario/Desktop/NGS/01.FastQC_crudos`

Abrir los .html, visualizar la calidad y definir los parámetros de limpieza.

### 4. Limpieza de datos en [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)

En home/usuario, crear la carpeta 02.trimmomatic donde se guardaràn los resultados de la corrida

# Para cada muestra single-end de manera independiente
`trimmomatic SE muestra.fastq  muestra_trim.fastq   ILLUMINACLIP:TruSeq3-SE.fa:2:30:10`

Colocarse en home/usuario
# Loop para Trimmomatic con single end (SE)
`for i in 00.rawdata/*.fastq; do     filename=$(basename "$i");     trimmomatic SE -phred33         "$i"         "02.trimmomatic/trimm_$filename"         HEADCROP:7                SLIDINGWINDOW:5:15         MINLEN:150; done`


# Para cada muestra pair-end de manera independiente
`java -jar /opt/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 10 -phred33 -trimlog BK01triminfo.txt BK01_S95_L001_R1_001.fastq.gz BK01_S95_L001_R2_001.fastq.gz BK01_R1_trimm.fastq.gz BK01_R1_unpair.fastq BK01_R2_trimm.fastq.gz BK01_R2_unpair.fastq ILLUMINACLIP:/opt/Trimmomatic-0.39/adapters/TruSeq2-PE.fa:2:8:10:8:True HEADCROP:14 SLIDINGWINDOW:5:15 MINLEN:110`


# Loop para Trimmomatic con pair-end (PE). Para más información consulta [aquí](https://hoytpr.github.io/bioinformatics-semester/materials/loop-extra/)

```Javascript
for infile in *1.fastq.gz ;     do 
base=$(basename ${infile} 1.fastq.gz) ;     
java -jar /opt/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 10 -phred33 ${infile} ${base}2.fastq.gz ${base}1_trim.fastq.gz ${base}1un_trim.fastq.gz ${base}2_trim.fastq.gz ${base}2un_trim.fastq.gz ILLUMINACLIP:/opt/Trimmomatic-0.39/adapters/TruSeq2-PE.fa:2:8:10:8:True HEADCROP:14 SLIDINGWINDOW:5:15 MINLEN:110 ;     
done
```

Se van a generar archivos con terminación _trim.fastq.gz_ (son los de interés) y _un_trim.fastq.gz_ (son reads no pareados, no son de interés para los análisis posteriores).

Para mantener el orden, crear una nueva carpeta  denominada _02.Trimmomatic_ en /home/usuario/

Entrar a la carpeta _02.Trimmomatic_ y mover los _trim.fastq.gz_ y los _un_trim.fastq.gz_
`mv /home/usuario/curso/00.RawData/*.trim.fastq.gz .`
`mv /home/usuario/curso/00.RawData/*.un_trim.fastq.gz .`

### 5. Visualizar la calidad de los datos limpios con FastQC

`fastqc *trim.fastq.gz`

Nuevamente, se generan archivos .html y .zip

Para mantener el orden, crear una nueva carpeta denominada _03.FastQC_limpios_ en /home/usuario/
 
Entrar a la carpeta _03.FastQC_limpios_ y mover los .html y los .zip 

`mv /home/usuario/curso/02.Trimmomatic/*.html .`
`mv /home/usuario/curso/02.Trimmomatic/*.zip .`

Mover los archivos.html del servidor a la computadora local para poder visualizarlos

`scp -r -P 1967 usuario@132.248.15.41:/bhome/usuario/curso/03.FastQC_limpios/*.html /home/Usuario/Desktop/NGS/03.FastQC_limpios`

Abrir los .html y verificar si mejoró la calidad. En caso de no mejorar, repetir a partir del paso 4, definiendo nuevos parámetros de limpieza.

Una vez obtenidos los .fastq con la calidad deseada, copiarlos  del servidor a la computadora local para respaldarlos. Estos serán los datos que se emplearán en los análisis posteriores. 

`scp -r -P 1967 usuario@132.248.15.41:/home/usuario/curso/02.Trimmomatic/*trim.fastq.gz /home/Usuario/Desktop/NGS/02.Trimmomatic`

### 6. SNPs _calling_ en ipyrad
Crear la carpeta _04.ipyrad_ en /home/usuario/
Entrar a la carpeta y activar el ambiente conda para correr el programa.

`conda activate ipyrad`

Crear el archivo de parámetros que emplea ipyrad. Colocar un nombre informativo que permita identificar la corrida. En este ejemplo U90M90

`ipyrad -n U90M90`

 Como resultado se obtendrá el archivo params-U90M90.txt. Este debe ser editado con el comando _nano_. Los parámetros deben ser modificados según el criterio del investigador. Se requieren varias corridas para definir el set de parámetros que optimiza el ensamblaje, permitiendo obtener el mayor número de SNPs de alta calidad, con bajo porcentaje de datos faltantes.
 Para más información consulta el [manual de ipyrad](https://ipyrad.readthedocs.io/en/master/6-params.html)
 
**A continuación, algunos parámetros importantes a considerar:**

- [1] [project_dir]: Especificar directorio de los archivos de salida.
- [4] [sorted_fastq_path]: Location of demultiplexed/sorted fastq files. _/ruta/*trim.fastq.gz_
- [5] [assembly_method]: Assembly method (denovo, reference, etc)
- [7] [datatype]: Datatype (see docs): rad, gbs, ddrad, etc. _pair3rad_
- [8] [restriction_overhang] _colocar el sitio de corte de mis enzimas: AATTC, CTAGC, CTAGA (para EcoR1, Xba y NheI)_
- [14] [clust_threshold]: Clustering threshold for de novo assembly. _Colocar el umbral de similitud para la creación de clusters (en este ejemplo 0.90)_
- [21] [min_samples_locus]: Min # samples per locus for output. _Depende del tamaño de muestra y cuanto missing data acaptaré (ej. si tengo 60 muestras y solo aceptaré el 90% de missing data, en esta casilla debo colocar 54)_
- [22] [max_SNPs_locus]: Número máximo de SNPs por locus. _0.1 = 10%_
- [27] [output_formats]: Output formats (see docs). Colocar * para que genere todos los formatos posibles.

Una vez fijados los parámetros de la corrida modificando el params-U90M90, ya se puede correr ipyrad

`nohup ipyrad -p params-U90M90.txt -s 1234567 -c 24 -f --MPI > output.txt &` 

Dependiendo de la catidad de datos, la corrida puede ser lenta. Recomiendo utilizar _nohup_ para ir monitoreándola. 

Una vez culminada la corrida, verificar los resultados guardados en la carpeta _U90M90_outfiles_, archivo _U90M90_stats.txt_. 
El archivo **_U90M90.vcf_** va a contener la matriz de SNPs.

### 7. Filtrado adicional con VCFtools
Crear la carpeta _VCFtools_ en /home/usuario

Las opciones de filtrado en VCFtools pueden ser visualizadas con el comando `man vcftools`
 
 Dependiendo del objetivo del estudio, se pueden hacer varias corridas. En este ejemplo, haremos análisis poblacionales clásicos (estructura y diversidad genética) y análisis de SNPs candidatos a selección. Por lo tanto, se construirán 2 sets de datos.

**set de datos 1, análsis de selección**

Objetivo: eliminar alelos con frecuencia menor al 4%, retener _loci_ bialelélicos con profundidad mayor a 10 y máximo 10% de _missing data_.

`vcftools --vcf U90M90.vcf --maf 0.04 --min-meanDP 10 --min-alleles 2 --max-alleles 2 --max-missing 0.9 --recode --out U90M90_maf0.04_DP10_MD0.9`


**set de datos 2, análisis de estructura y diversidad**

Objetivo: A la salida anterior, aplicarle el filtro de desequilibrio de ligamiento.

`vcftools --vcf U90M90_maf0.04_DP10_MD0.9.vcf --thin 1000 --recode --out U90M90_maf0.04_DP10_DL.vcf`

 
Para ambos sets de datos, visualizar el archivo.log para ver cuantos sitios se mantienen luego del filtrado.

_En este punto, ya están listos los sets de datos a emplear en análisis de estructura, diversidad y selección. Revisar el nombre de las muestras en los vcf resultantes y modificarlos si es requerido._

## Descargar el vcf 
En la terminal de mi compu
`scp -r -P 1967 massiel.alfonsoglez@132.248.15.41:/home/massiel.alfonsoglez/curso/04.ipyrad/U90M90_outfiles/U90M90_maf0.04_DP10_MD0.9.vcf /home/alex-dll/Escritorio`
 
