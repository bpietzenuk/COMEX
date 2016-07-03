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

1. Short read sequences mapping file .bam is a binary file which has to be converted into .sam file format for further work

   COMMAND LINE: Ex- samtools view -h filename.bam > filename.sam

   Converted .sam-file will be used here for further processing. It is required to get cordinates for reads mapping.

2. For further processing of the sam-file ends need to be generated. Output file will be generated within the script and has to be named within the script before running.
