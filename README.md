# Rat ChIP-Seq Pipeline
* Raw data is in /ludc/Raw_Data_Archive/Sequencing/Chip_Seq/ChIP_Amaya/Raw_Content/raw_data
## 1.raw data possessing
  *change base call files to fastq files*
* program: bcl2fastq (v2.20.0.422)
* input: input_dir (contains $bgzf.bci $bgzf); sample_sheet.csv (so you can specify the sample sheet location and name, if different from the default.)
* output: $fastq.bz
* commands: nohup bcl2fastq --sample-sheet <sample_sheet.csv> -i <Input_dir> -o <Output_dir>
  *merge reads in different lanes*
