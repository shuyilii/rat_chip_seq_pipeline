# Rat ChIP-Seq Pipeline
Raw data is in /ludc/Raw_Data_Archive/Sequencing/Chip_Seq/ChIP_Amaya/Raw_Content/raw_data.
You can always check the pipeline on https://github.com/shuyilii/rat_chip_seq_pipeline
## 1.Raw data possessing
***change base call files to fastq files***
* program(s): bcl2fastq (v2.20.0.422)
* input: input_dir (contains $bgzf.bci $bgzf)  
sample_sheet.csv (so you can specify the sample sheet location and name, if different from the default.)
* output: $fastq.gz
* commands:  
nohup bcl2fastq --sample-sheet <sample_sheet.csv> -i <Input_dir> -o <Output_dir>

***merge reads in different lanes***
* program(s): bash
* input: $L00{1,2,3,4}_R1_fastq.bz $L00{1,2,3,4}_R2_fastq.bz
* output: $R1.fastq $R2.fastq
* commands:  
cat <(find \*_R1\*fastq.gz | sort | paste - - - -) <(find \*R2\*fastq.gz | sort | paste - - - -) | sort | gawk '{ print "cat",$1,$2,$3,$4,">",gensub(/_L001_(R[12])_001/, "_\\1", "g", $1);}'| sh  
ls \*.fastq.gz | while read file; do gunzip $file; done

## 2.Quality control
* program(s): fastqc (v0.11.8); MultiQC(v1.7)
* input: $fastq
* output: QC_result_dir
* commands:  
ls \*.fastq | while read file ; do fastqc -t 6 $file ; echo $file processed ; done  
multiqc .

## 3.Read alignment
* program(s): BWA (v0.7.13); samtools (v1.9)
* input:  
reference genome - Rattus norvegicus (Rat) Emsemble Rnor_6.0 (http://igenomes.illumina.com.s3-website-us-east-1.amazonaws.com/Rattus_norvegicus/Ensembl/Rnor_6.0/Rattus_norvegicus_Ensembl_Rnor_6.0.tar.gz)  
$R1.fastq $R2.fastq
* output:
$unfiltered_bam
* commands:  
bwa mem -t 8 -M ref_genome.fa $R1.fastq $R2.fastq > $sam  
samtools view -h -b -S -o $unfiltered_bam $sam

## 4.Post-alignment filtering
* program(s): samtools (v1.9); Picard - MarkDuplicates(v2.20.5); sambamba(v0.7.0)
* input: $unfiltered_bam
* output: $filtered_bam
* commands:  
***Remove  unmapped, mate unmapped, not primary alignment, reads failing platform, low MAPQ reads***  
MAPQ_THRESH=30  
samtools view -F 1804 -f 2 -q ${MAPQ_THRESH} -b $unfiltered_bam > $tmp_bam  
samtools fixmate -r $tmp_bam $fil_bam  
samtools view -F 1804 -f 2 -b $fil_bam > $fil_temp_bam  
samtools sort -o $sortedPos_bam $fil_temp_bam  
***Mark duplicates***  
java -Xmx4G -jar picard.jar MarkDuplicates I=$sortedPos_bam O=$markdup_bam M=$metrics.txt   VALIDATION_STRINGENCY=LENIENT ASSUME_SORTED=true REMOVE_DUPLICATES=false  
***Remove duplicates, index final position sorted BAM***  
samtools view -F 1804 -f 2 -b $markdup_bam > $unsorted_bam  
sambamba sort -t 2 -o $filtered_bam $unsorted_bam

## 5.Peak calling
* program(s): macs2 (v2.2.4)
* input: $IP_filtered_bam $INPUT_filtered_bam
* output: $bed $xls $narrowPeak
* commands: macs2 callpeak -t $IP_filtered_bam -c $INPUT_filtered_bam -f BAMPE -g 2.3e+9 -p 0.001 -n <IP_name> --outdir <out_dir> 2>$log

## 6.ChIPQC
* program(s): R package - ChIPQC(1.21.0) (installation: BiocManager::install("ChIPQC"))
* input: sample.csv - The sample sheet contains metadata information for our dataset. Each row represents a peak set (which in most cases is every ChIP sample) and several columns of required information, which allows us to easily load the associated data in one single command. NOTE: The column headers have specific names that are expected by ChIPQC.  
$narrowPeak (the path to which is specified in sample.csv)
* output: ChIPQC report (Chip QC report.html)
* commands:  
library(ChIPQC)  
samples <- read.csv(file="meta/sample.csv",header=TRUE,sep=",")  
chipObj <- ChIPQC(samples, annotation = "rn4")  
#####register(SerialParam())  
ChIPQCreport(chipObj, reportName = "Chip QC report", reportFolder = "ChIPQCreport")

## 7.Handling-replicates
* program(s): Bedtools (v2.28.0); bash

## 8.Peak annotation
* program(s): homer - annotatePeaks.pl(v4.10.4); R package - ChIPseeker(v1.20.0); clusterProfiler(v3.12.0)
* commands: annotatePeaks.pl $bed rn6 -gtf /ludc/Reference_Data/Public/Rat/Ensembl_DNA/Rnor_6.0/Annotation/Genes/genes.gtf > control_1IP.txt
## 9.Plot
* program(s): R package - ChIPQC(v1.21.0); ChIPseeker(v1.20.0)
