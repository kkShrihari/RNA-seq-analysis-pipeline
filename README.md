#!/bin/bash
SECONDS=0

# Change working directory (if required)
# cd /home/shrihari/RNAseq

# Step 1: Run fastqc
source activate fastqc
fastqc demo.fastq -O demo_fastqc
exit_status=$?
conda deactivate

# Check if fastqc command was successful
if [ $exit_status -eq 0 ]; then
    echo "FastQC completed successfully."
else
    echo "Error: FastQC did not run successfully."
fi

duration=$SECONDS
echo "Time taken for running fastqc is: $SECONDS "

# Step 2: Do trimming
source activate trimmomatic
mkdir -p Trim_data
trimmomatic SE -threads 4 demo.fastq Trim_data/demo_trimmed.fastq TRAILING:10 -phred33
exit_status=$?
conda deactivate

# Check if trimmomatic command was successful
if [ $exit_status -eq 0 ]; then
    echo "Trimming completed successfully."
else
    echo "Error: trimmomatic did not run successfully."
fi

duration=$SECONDS
echo "Time taken for running Trimmomatic is: $duration seconds"

# Step 3: QC of trimmed data
source activate fastqc
mkdir -p Trim_fastqc
fastqc Trim_data/demo_trimmed.fastq -O Trim_fastqc
exit_status=$?
conda deactivate

# Check if fastqc command was successful
if [ $exit_status -eq 0 ]; then
    echo "FastQC for trimmed data completed successfully."
else
    echo "Error: FastQC did not run successfully."
fi

duration=$SECONDS
echo "Time taken for running fastqc of trimmed data is: $SECONDS "


# Step 4: Run alignment using HISAT2 (here bamtools is installed inside hisat2 environment)
source activate hisat2
mkdir -p HISAT2
hisat2 -q --rna-strandness R -x HISAT2/grch38/genome -U demo_trimmed.fastq -S HISAT2/demo_aligned.sam
exit_status=$?
conda deactivate

# Check if samtools command was successful
if [ $exit_status -eq 0 ]; then
    echo "Alignment using HISAT2 completed successfully."
else
    echo "Error: HISAT2 did not run successfully."
fi

# step 5: using samtools
source activate samtools
samtools sort -o HISAT2/demo_aligned.bam HISAT2/demo_aligned.sam
conda deactivate
exit_status=$?

# Check if hisat2 command was successful
if [ $exit_status -eq 0 ]; then
    echo "samtoolsfile conversion completed successfully."
else
    echo "Error: samtools did not run successfully."
fi

duration=$SECONDS
echo "Time taken for file conversion is: $SECONDS "

# step 5: quantification using htseq
source activate htseq
mkdir -p quants
htseq-count -f bam -r pos -s reverse -t exon -i gene_id HISAT2/demo_aligned.bam GRCh38.gtf > quants/demo_counts.txt
conda deactivate
exit_status=$?

# Check if hisat2 command was successful
if [ $exit_status -eq 0 ]; then
    echo "Quantification completed successfully."
else
    echo "Error: htseq did not run successfully."
fi

duration=$SECONDS
echo "Time taken for quantification of data is: $SECONDS minutes "

echo "Total time taken for overall run: $(($duration / 60)) minutes and $(($duration % 60)) seconds elapsed."
