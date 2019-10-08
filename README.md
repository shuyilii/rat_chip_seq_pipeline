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
$bam
* commands:  
bwa mem -t 8 -M ref_genome.fa $R1.fastq $R2.fastq > $sam  
samtools view -h -b -S -o $bam $sam

## 4.Post-alignment filtering
* program(s): samtools (v1.9); Picard - MarkDuplicates(v2.20.5)
* input:

## 5.Peak calling
* program(s): macs2 (v2.2.4)
* input:

## 6.ChIPQC
* program(s): R package - ChIPQC(1.21.0)

## 7.Handling-replicates
* program(s): Bedtools (v2.28.0); bash

## 8.Peak annotation
* program(s): homer - annotatePeaks.pl(v4.10.4); R package - ChIPseeker(v1.20.0); clusterProfiler(v3.12.0)

## 9.Plot
* program(s): R package - ChIPQC(v1.21.0); ChIPseeker(v1.20.0)
