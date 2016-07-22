COMparative EXpression of transposable elements (COMEX)
=====

Purpose
---

Analysis of transcript amounts from transposable elements (TEs) using RNA-sequencing data. TEs are usually highly repetitive, which causes frequent multi-mapping of the short reads. In addition, RNA-sequencing reads may map to protein coding domains of closely related TE families, which may lead to false positive candidates for down-steam analyses. COMEX was designed to filter reads mapping across TE families, but retain reads mapping to a single TE family and then estimate TE transcript amounts using either uniquely or also multiply mapping reads.

System requirements
---

1. Pipeline were tested under Mac OS and Linux
2. samtools and bedtools needed
3. spreadsheet program i.e. Microsoft Excel or LibreOffice Spreadsheet

COMEX Pipeline
---

Step 1: Discard multiple-mapping reads that map across TE-families 
---

Within this step the mapped-reads will be checked for multiple mapping across different TE families and subsequently discarded if so. This step is merged into a single pipeline that runs via a shell-script. Pipeline-scripts are in pyhton. Corresponding scripts are provided. All files need to be in the same folder. We recommend using one filder for each library. The script runs from 35 minutes up to 12 hours depending on the number of reads and the technical specifications.

	SCRIPT: "comex2.0.sh"
	
	COMMAN LINE: ./comex2.0
	subsequently you will be asked to name mapping.bam-file, annotation-file and genome.fas (genome.fas needs to be indexed)

EXAMPLE

Tasks of comet.sh
-
1. 	Short read sequences mapping file .bam is a binary file which is to be converted into .sam file format for further processing. It is required to get cordinates for reads mapping.

   SCRIPT: samtools view -h $InputBam >$input'.sam'

2. 	For further processing of the sam-file ends need to be generated. Output file will be generated within the script and has to be named within the script before running.

   SCRIPT: "ToPrint_end1.py"

3. 	Removes the cases of mapping errors. It will remove all three possible mapping errors: 
	- same read id mapping to different position with same start and different end (on same chromosome) accept shorter in length
	- same read id mapping to different position  with different end and same end( same chromosome) accept shorter in length
	- same read id mapping at same position on same chromosome
 
   SCRIPT: "Selectnonrepeated1.py"

4. 	Divides the reads mapped into multiple and uniquel mapping reads. Therefore it cretes two files: MULTIPLE and UNIQUELY mapped reads.

   SCRIPT: "Selectmultiplemapped1.py"

5. 	Removes the reads which are mapping across TE families from multiple mapping file. For this step the annotation file is needed, it is used to determine the location of TEs and their family affilliation.

   SCRIPT: "new_cases1.py"

5. 	Uniquely mapping reads and outfile from previous step (multiple mapping reads that do not map across TE families) are merged for further analysis. 

   SCRIPT: cat $input'_multiplecorrected.sam' $input'_uniquelymapped_reads.sam' >$input'_merged.sam'

6. 	Removes the ends generated in step 2. Otherwise the program will change the .sam file format, and will compromise its use for read counting.
	
   SCRIPT: "Remove_end1.py"

7. 	Outfile from the previous step is converted into bam-file.

   Command Line: samtools view -bT refference_genome.fa outfile_prev_step.sam > outfile.bam

- shell Script ends here

Step 2: Counting mapped reads per TE using qualimap
---

- Read counts for annotation will be generated. Annotation file should be in a gtf format.

   http://qualimap.bioinfo.cipf.es/doc_html/command_line.html

   Command Line: qualimap comp-counts [-algorithm <arg>] -bam <arg> -gtf <arg> [-id <arg>] [-out <arg>] [-protocol <arg>] [-type <arg>]

 -algorithm <arg>   uniquely-mapped-reads(default) or proportional
 -b                 calculate 5' and 3' coverage bias
 -bam <arg>         mapping file in BAM format)
 -gtf <arg>         region file in GTF format
 -id <arg>          attribute of the GTF to be used as feature ID. Regions with
                    the same /ID will be aggregated as part of the same feature.
                    Default: gene_id.
 -out <arg>         path to output file
 -protocol <arg>    forward-stranded,reverse-stranded or non-strand-specific
 -type <arg>        Value of the third column of the GTF considered for
                    counting. Other types will be ignored. Default: exon 

Step 10: Manual calculation of RPKM using spreadsheet
---

Read counts per TE generated in previous step will be placed ("copy-paste") in annotation file to corresponding TE.

   Here:	
   For each experimental point, RPKM and RPKM mean of the replicates has to be calculated manually.
   Formula used: RPKM= ((10^9* read count)/(length* No of mapped reads))

   Outfile should contain following columns:
   readcount_Rep1; RPKM_rep1; readcount_Rep2; RPKM; (...) avegerage_RPKM_all_replicates

Step 11: Setting RPKM threshold for expressed TEs
---
Substep 1:
-

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


Substep 2: 
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

   A tab-delimited table as an infile has to be created containing normalized read_counts from step 10 or 14 (for example see DEseq sample file). Run DESeq-COMEX2.0.R in R
   
   - SCRIPT: DESeq-COMEX2.0.R

   Output from here should be pasted into the outfile from step 14.


Additional script for species-specific information
---

Creates a file (tab-delimited) which includes species specific information like how many TEs are expressed above RPKM threshold, their length RPKM, and how many are expressed below RPKM threshold, there length RPKM.
Infile needs to be defined in the script. Outfile needs to be named in command line

   Script: countFile.py

   Command line: python countFile.py > outfile
