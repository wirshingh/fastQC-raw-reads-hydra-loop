# Run fastQC on Hydra in a loop using raw genomic reads

### Job Summary

This job will run fastQC on Hydra in a loop using raw genomics reads. Output files will be copied to a directory named 'fastqc_raw_results'. To run the script see 'To Run this Job' below.

```
#!/bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -pe mthread 6
#$ -q mThC.q
#$ -l mres=24G,h_data=4G,h_vmem=4G
#$ -cwd
#$ -j y
#$ -N fastqc
#$ -o fastqc.log
#
# ----------------Modules------------------------- #
module load bio/fastqc
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS
#

# Create a directory to store FastQC output files
mkdir -p fastqc_raw_results

# Define the directory containing the raw FASTQ files
# Replace what is in quotes with the actual path to the raw sequence data
SAMPLEDIR_RAW="path/to/raw/reads/directory"

# Define the base directory for your project
# This is current directory, where the 'fastqc_raw_results' folder is created
SAMPLEDIR_BASE="path/to/base/project/directory"

# Loop through all R1 FASTQ files in the raw reads directory
# The pattern *_R1_001.fastq.gz matches typical Illumina paired-end read filenames
for GETSAMPLENAME in "${SAMPLEDIR_RAW}"/*_R1_001.fastq.gz; do

    # Extract the sample name by removing the suffix from the filename
    # For example, from "sample1_R1_001.fastq.gz" it extracts "sample1"
    SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_001.fastq.gz)

    echo "Processing sample: $SAMPLENAME"

    # Run FastQC on the R1 read file
    # -o specifies the output directory for the FastQC report
    fastqc "${SAMPLEDIR_RAW}/${SAMPLENAME}_R1_001.fastq.gz" \
        -o "${SAMPLEDIR_BASE}/fastqc_raw_results"

    # Run FastQC on the R2 read file
    # Assumes the R2 file follows the same naming convention as R1
    fastqc "${SAMPLEDIR_RAW}/${SAMPLENAME}_R2_001.fastq.gz" \
        -o "${SAMPLEDIR_BASE}/fastqc_raw_results"
done

#
echo = `date` job $JOB_NAME done

```

### To Run this Job

The raw read files must end in '_R1_001_fastq.gz' (forward) and '_R2_001_fastq.gz' (reverse) for the script to work. Alternatively, the job file can be edited to match the file names of the R1 and R2 reads.

Edit these items in the job file:

1. SAMPLEDIR_RAW="path to raw reads"

After the '=' copy full path to the directory that contains the raw reads.

2. SAMPLEDIR_BASE="path to base project directory"

After the '=' copy the full path to the base directory. This is the directory where the output directory will be copied, and where the job file can be placed.

After the job is edited save it as 'fastqc_loop.job' and run the job on hydra (qsub fastp_loop.job).

