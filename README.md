# Rat ChIP-Seq Pipeline  
Raw data is in /ludc/Raw_Data_Archive/Sequencing/Chip_Seq/ChIP_Amaya/Raw_Content/raw_data
## 1.Raw data possessing
*change base call files to fastq files*
* program: bcl2fastq (v2.20.0.422)
* input: input_dir (contains $bgzf.bci $bgzf)  
sample_sheet.csv (so you can specify the sample sheet location and name, if different from the default.)
* output: $fastq.gz
* commands:  
nohup bcl2fastq --sample-sheet <sample_sheet.csv> -i <Input_dir> -o <Output_dir>

*merge reads in different lanes*
* program: bash
* input: L00{1,2,3,4}_R1_fastq.bz L00{1,2,3,4}_R2_fastq.bz
* output: R1.fastq R2.fastq
* commands:  
cat <(find *_R1*fastq.gz | sort | paste - - - -) <(find *R2*fastq.gz | sort | paste - - - -) | sort | gawk '{ print "cat",$1,$2,$3,$4,">",gensub(/_L001_(R[12])_001/, "_\\1", "g", $1);}'| sh

ls *.fastq.gz | while read file; do gunzip $file; done

## 2.Quality control
