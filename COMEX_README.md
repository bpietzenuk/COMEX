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







