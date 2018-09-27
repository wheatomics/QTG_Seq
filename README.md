
1, ...

2, Prerequisite of external softwares

Need to install Perl Statistics module, bwa in your local environment correctly and set the running path of external programs at the begining of the pipeline

3, Usage

usage:./QTG_Seq.sh [VCF] [GFF] [Cov Threshold] [Win Size] \<EuclideanDist\>

QTG-Seq provides several different statistics for analysis as follows:

	EuclideanDist (default) - Euclidean Distance
	
	SNPindex - Delta SNPindex
	
	Pvalue - Delta P(Chi-Sqrt)
	
	ED4 

4, Input

The main input file is the VCF file, which contains genomic variants for both low pool and high pool. For the genomic variant calling, we'd love to recommendate using GATK using the guided bioinformatic pipeline as follows:

<B>#mapping </B>

<I>
bwa mem -t 8 -M -P Referencegenome.fa High_Forward.fastq High_Reverse.fastq >bsa_H.sam

bwa mem -t 8 -M -P Referencegenome.fa Low_Forward.fastq Low_Reverse.fastq >bsa_L.sam
</I>

<B>#pretreatment for GATK SNP calling for hight pool</B>

<I>
java -jar ${EBROOTPICARD}/picard.jar CleanSam INPUT=bsa_H.sam OUTPUT=bsa_H_cleaned.sam

java -jar ${EBROOTPICARD}/picard.jar FixMateInformation INPUT=bsa_H_cleaned.sam OUTPUT=bsa_H_cleaned_fixed.sam SO=coordinate

java -jar ${EBROOTPICARD}/picard.jar AddOrReplaceReadGroups INPUT=bsa_H_cleaned_fixed.sam OUTPUT=bsa_H_cleaned_fixed_group.bam LB=bsa_H SO=coordinate RGPL=illumina PU=barcode SM=bsa_H

samtools index bsa_H_cleaned_fixed_group.bam

java -jar ${EBROOTPICARD}/picard.jar MarkDuplicatesWithMateCigar INPUT=bsa_H_cleaned_fixed_group.bam OUTPUT=bsa_H_cleaned_fixed_group_DEDUP.bam M=bsa_H_cleaned_fixed_group_DEDUP.mx AS=true REMOVE_DUPLICATES=true MINIMUM_DISTANCE=500

samtools index bsa_H_cleaned_fixed_group_DEDUP.bam
</I>

<B>#pretreatment for GATK SNP calling for low pool</B>

<I>
Similar to high pool
</I>


<B>#genomic variant calling</B>

<i>java -Xmx64g -jar $EBROOTGATK/GenomeAnalysisTK.jar -T HaplotypeCaller -R Referencegenome.fa -nct 8 -I bsa_H_cleaned_fixed_group_DEDUP.bam -I bsa_L_cleaned_fixed_group_DEDUP.bam -o bsa_H_L_snps_indels.vcf</i>



5, Output

Several output files will be generated by QTG_Seq. Notably, QTG_Seq_plots.pdf illustrates the genome-wide scan of QTG, while QTG_regions.txt details the QTG regions. QTG_Seq_R_summary_*.csv contains all the statistic information across the whole genome, which could be re-plotted in a more sophisticated manner at the user's will.
