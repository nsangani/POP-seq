# POP-seq
Protein Occupancy Profile Sequencing using RNA-seq replicates

#Step 1: Quality Check
Fastqc/0.11.5 </br>
fastqc replicate_1.fq.gz </br>
****
cutadapt/intel/1.9.1 </br>
cutadapt -a 3'-adaptor-seqeunce -g 5'-adaptor-sequence -n 10 replicate_1.fq -o output_replicate_1.fq

# Step 2: Alignment
Hisat2/2.1.0 </br>
hisat2 -x reference-genome -U output_replicate_1.fq -S output.sam

# Step 3: Peak Calling
Sequence alignment map (SAM) to Binary alignment map (BAM) using samtools/1.5 </br>
samtools view -hbS ouput.sam > output.bam </br>
****
Sorted BAM </br>
sort output_bam > output_sorted.bam </br>
****
Sorted BAM to Browser Extensible Data (BED) using bedtools/gnu/2.26.0 </br>
bedtools bamtobed -i output_sorted.bam > output.bed </br>
****
BED to Sorted BED (if needed) </br>
Method-1: Bedtools </br>
sortBed -i output.bed > sorted_ouput.bed </br>
Method-2: one-line unix script </br>
LC_ALL=C sort --parallet=24 --buffer-size=2G -k1,1 -k2,2n ouput.bed > soted_output.bed 

# Step 4: Differential Peak Analysis
Method-1: DiffHunter based on Java/1.8.0_131 (pvalue < 0.05) </br>
1. Binding Sites (Indexing)</br>
java -jar diffHunter.jar -i -b sorted.bed -r reference-genome -o output_folder </br>
2. Differential Binding Patterns </br>
java -jar diffHunter.jar -c -r reference-genome -1 input_1_folder -2 input_2_folder -w 10 -s 5 -o results_folder </br>
****
Method-2: Piranha + DiffBind </br>
Peak Calling: Piranha based on gsl/gnu/2.3 </br>
piranha -l -s -p 0.05 sorted_output.bed > piranha_peaks.bed </br>
****
DiffBind: R package (FDR < 0.05) </br>
1. Reading Peaks </br>
NPOP <- dba(sampleSheet="cell_line.1_vs_cell_line.2.csv") </br>
plot(NPOP) </br>
2. Counting reads </br>
NPOP1 <- dba.count(NPOP1) </br>
3. Contrast </br>
NPOP <- dba.contrast(NPOP1, categories - DBA_CONDITION,minMembers=2) </br>
4. Differential Analysis using Edger R package </br>
NPOP <- dba.report(NPOP, th=1, method=DBA_EDGER) </br>
5. Plots - PCA, MA, Boxplots. etc. </br>
dba.plotMA(NPOP,method=DBA_EDGER) </br>


