COMparative EXpression of transposable elements (COMEX)
=====

Purpose
---

Analysis of transcript amounts from transposable elements (TEs) using RNA-sequencing data. TEs are usually highly repetitive, which causes frequent multi-mapping of the short reads. In addition, RNA-sequencing reads may map to protein coding domains of closely related TE families, which may lead to false positive candidates for down-steam analyses. COMEX was designed to filter reads mapping across TE families, but retain reads mapping to a single TE family and then estimate TE transcript amounts using either uniquely or also multiply mapping reads.

System requirements
---


1. Pipeline were tested under MacOSX and Linux
2. samtools and bedtools needed
3. spreadsheet program i.e. Microsoft Excel or LibreOffice Spreadsheet

COMEX Pipeline
---

Step 1
---

Short read sequences mapping file .bam is a binary file which has to be converted into .sam file format for further work

   COMMAND LINE: Ex- samtools view -h filename.bam > filename.sam

   Converted .sam-file will be used here for further processing. It is required to get cordinates for reads mapping.

Step 2
---

For further processing of the sam-file ends need to be generated. Output file will be generated within the script and has to be named within the script before running.

   SCRIPT: "ToPrint_end.py"

   COMMAND LINE: python toprint_end.py 

Step 3
---

Removes the cases of mapping errors. It will remove all three possible mapping errors: 

1. same read id mapping to different position with same start and different end (on same chromosome) accept shorter in length
2. same read id mapping to different position  with different end and same end( same chromosome) accept shorter in length
3. same read id mapping at same position on same chromosome

   Output file will be generated as name in script, and output name can be changed within the script before running.
 
   SCRIPT: "Selectnonrepeated.py"

   COMMAND LINE: python Selectnonrepeated.py

Step 4
---

Creates two files: MULTIPLE and UNIQUELY mapped reads.

Output files will be generated within the script and need to be named within the script before running. Infile has to be defined previously within the script.

   SCRIPT: "Selectmultiplemapped.py"

   Command Line: python selectmultiplemapped.py

Step 5
---

This step will remove the reads which are mapping across TE families from multiple mapping file.

Reference .gff annotation file needed. The location and the name of infile (multiple mapping reads from previous step)has to be defined previously within the script. Outfile name has to be defined in command line

   SCRIPT: "new_cases.py"

   Command Line: python new_cases.py > filename.sam

Step 6
---

Uniquely mapping reads and outfile from previous step (multiple mapping reads NOT across TE families) have to be merged for further analysis file. 

   Command Line: cat multiple_corrected.sam uniquely-mapped.sam > filename.sam

Step 7
---

Remove the ends generated in step 2. Otherwise the program will change the .sam file format, and will compromise its use for read counting. Infile has to be defined within the script. Outfile has to be defindes in output-file.
	
   SCRIPT: "Remove_end.py"

   Command Line: python remove_end.py > outfile_name.sam

Step 8
---

Outfile from the previous step has to be converted into bam-file again.

   Command Line: samtools view -bT refference_genome.fa outfile_prev_step.sam > outfile.bam

Step 9
---

Read counts for annotation will be generated. Annotation file should be in a gtf format.

   Here we used qualimapV0.8

   http://qualimap.bioinfo.cipf.es/doc_html/command_line.html

   Command Line: qualimap comp-counts [-algorithm <arg>] -bam <arg> -gtf <arg> [-id <arg>] [-out <arg>] [-protocol <arg>] [-type <arg>]
 
   -algorithm   ----uniquely-mapped-reads(default) or proportional

   -b           ----calculate 5' and 3' coverage bias

   -bam         ----mapping file in BAM format)

   -gtf         ----region file in GTF format

   -id          ----attribute of the GTF to be used as feature ID. Regions with the same /ID will be aggregated as part of the same feature.Default: gene_id.

   -out         ----path to output file

   -protocol    ----forward-stranded,reverse-stranded or non-strand-specific

   -type        ----Value of the third column of the GTF considered for counting. Other types will be ignored. Default: exon


Step 10
---

Read counts per TE generated in previous step will be placed (copy-paste) in annotation file to corresponding TE.

   Here:	
   For each experimental point, RPKM and RPKM mean of the replicates has to be calculated manually.
   Formula used: RPKM= ((10^9* read count)/(length* No of mapped reads))

   Outfile should contain following columns:
   readcount_Rep1; RPKM_rep1; readcount_Rep2; RPKM; (...) avegerage_RPKM_all_replicates

Step 11
---

RPKM threshold in COMEX
   Most TEs are not transcribed or show just background transcription which may be difficult to validate in a biological experiment. Therefore, TEs with RPKM below specific threshold will be considered as not-transcribed. After step10 calculations have to be done on the file on the basis of RPKM-threshold.
RPKM-threshold is set for TEs considered as being expressed. RPKM above 0.55 were used as default.
Outfile will have added the following fields(Column):

   AvgRPKM-FOR-expcopy - average RPKM for expressed copies

   NO-exp-copy			- number of expressed copies

   exp-length			- summed length of expressed copies

   total-copy			- total copy number of family TEs

   minRPKM				- minimum RPKM

   maxRPKM				- maximum RPKM

   SumOF-exp-Read_1	- Summarized number of reads from replicate 1 of expressed TEs

   SumOF-exp-read_2	- Summarized number of reads from replicate 1 of expressed TEs

   Location and name of infile has to be defined previously within the script. 

   Outfile name has to be defined in command line


   SCRIPT: "result_calc.py"

   Command_Line: python reasult_calc.py > filename


Step 12
---	

Output from here will include only the families with RPKM ≥ 0.55 and different fields from step 11.

   SCRIPT: "summary.py"

   COMMAND LINE: python summary.py > filename

Step 13
---	

Species-specific outputs from step 12 can be used for comparative TE transcription analysis. Only the families expressed in multiple species will be used for comparison. TE families which are not expressed within the other species will be listen within outfile. Location and name of infile has to be defined within the script. Outfile name has to be defined in command line

   SCRIPT: "match.py"

   Command Line: python match.py > filename

Step 14
---

OPTIONAL for direct inter-specific comparisons of transcript amounts from different TE families: To normalize for TE length and number of copies per TE family, the read count outfile from step 13 has to be opened in a spreadsheet program. Create a new column and use following formula to normalise the read counts to the length of TE per species.

   Normalization for Species A = (sum of exp_length_speciesB/ sum of exp_length_species A)* Read count species A


Step 15
---

Calculation of differential expression using DEseq package in "R" program.

   A tab-delimited table as an infile has to be created containing normalized read_counts from step 10 or 14 (for example see DEseq sample file).

   http://cartwrightlab.wikispaces.com/DESeq

   Command Line in R

   source("http://bioconductor.org/biocLite.R")

   biocLite("DESeq")

   library("DESeq")

   count_table<- read.table("input.txt", header=T,row.names=1)

   head(count_table)

   conds<-factor(c("a","a","b","b"))

   library("DESeq")

   cds<-newCountDataSet(count_table,conds)

   head(counts(cds))

   cds<-estimateSizeFactors(cds)

   sizeFactors(cds)

   cds<-estimateDispersions(cds,fitType="local")

   res<-nbinomTest(cds, "a","b")

   write.csv(res, "outputname.txt")

   Output from here should be pasted into the outfile from step 14.


Additional script for species-specific information
---

Creates a file (tab-delimited) which includes species specific information like how many TEs are expressed above RPKM threshold, their length RPKM, and how many are expressed below RPKM threshold, there length RPKM.
Infile needs to be defined in the script. Outfile needs to be named in command line

   Script: countFile.py

   Command line: python countFile.py > outfile






