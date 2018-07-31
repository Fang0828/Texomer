Texomer
===========
Author: Fang Wang
Email: fwang9@mdanderson.org or kchen3@mdanderson.org

Draft date: Feb. 12, 2018

Description
===========
Texomer is a tool used in cancer genomic studies to perform allele-specific, tumor-deconvoluted transcriptome-exome integration of the bulk whole exome (WES) and whole transcriptome sequencing (WTS) data obtained from autologous patient tissue sample. 

It reports estimations of tumor purity at both the DNA and RNA level as well as intratumor heterogeneity at the DNA level. Moreover, Texomer yeilds allelic specific copy numbers and expression levels along with a differential allelic cis-regulatory effect (DACRE) score of each variant allele. 

If the input only includes information at the DNA level, it only reports the estimates at the DNA level and focuses on somatic mutations.

System requirements and dependency
==================================
Texomer runs on a x86_64 Linux system. It depends on samtools and bedtools to extract reads covering variant sites from WTS data. 

It also requires R (version >= 3.2)
to run and has dependency on the R packages: 

	bbmle, emdbook, copynumber,TitanCNA, facets, mixtools, ASCAT and Sequenza. 

These R packeages are already included in this release.

Installation
============
Please download and copy the distribution to your specific location. If you are cloning from github, ensure that you have git-lfs installed.

For example, if the downloaded distribuition is Texomer-1.0.tar.gz.
	Type 'tar zxvf Texomer-1.0.tar.gz'

Then, run Texomer.py in the resulting folder.

Usage
=====
```
Options:
  --version             show program's version number and exit
  -h, --help            Show this help message and exit.
  -p RSCRIPT, --Rscript=RSCRIPT
                        the path of DACRE
  -g GERMLINE, --germline=GERMLINE
                        You can input your own germline mutation file if no
                        -v. The file name of germline input file include 8
                        columns: chromosome, position, RefAllele, Altallele,
                        read counts of RefAllele in normal, read counts of
                        Altallele in normal, read counts of RefAllele in tumor
                        and read counts of Altallele in tumor with header
                        seperated by tab.
  -s SOMATIC, --somatic=SOMATIC
                        You can input your own somatic mutation file if no -v.
                        The file name of somatic input file include 8 columns:
                        chromosome, position, RefAllele, Altallele, read
                        counts of RefAllele in normal, read counts of
                        Altallele in normal, read counts of RefAllele in tumor
                        and read counts of Altallele in tumor with header
                        seperated by tab.
  -v VARSCAN, --varscan=VARSCAN
                        the Varscan2 output file based on somatic calling
  -r RNA, --RNA=RNA     bam file from the RNA-seq data
  -i BAI, --bai=BAI     the index file of the RNA-seq bam file. this is optional.
  -e SNVEXPRESS, --snvexpress=SNVEXPRESS
                        The allelic read count of mutation from RNA-seq
                        including 7 columns: chromosome, position, ref, alt,
                        refnum, altnum and type (germline or somatic)
  -t ITER, --iter=ITER  optimal through somatic mutation, 0 corresponding to
                        no optimation and 1 is optimation. The default = 1
  -o OUTPATH, --outpath=OUTPATH
                        the output path. This is optional
```

Input files: 

DNA input files: 

	Two kinds of input files are allowed in Texomer:

	(1) the output generated by Varscan, which includes both germline and somatic variants through -v;

	(2) two seperate files: one contains germline variants through -g and the other contains somatic mutations through -s. 

	The format of these two files are the same. It should include 8 tab-delimited columns: 
		chromosome (start from "chr"), 
		position, 
		reference allele, 
		alternative allele, 
		ref allelic read counts in the normal sample, 
		alt allelic read counts in the normal sample, 
		ref allelic read counts in the tumor sample,
		alt allelic read counts in the tumor sample.

RNA input files:

	Two kinds of input files are allowed in Texomer:

	(1) a RNA-seq bam file. Load samtools and bedtools first if you input bam file.

	(2) allelic read counts of mutation from the RNA-seq bam, which includes 7 tab-delimited cloumns: 
		chromosome (start from "chr"),
		position,
		reference allele, 
		alternative allele, 
		reference allelic read count from the RNA-seq bam,
		alternative allelic read count from the RNA-seq bam,
		type of mutation (germline or somatic).    

Output files: 

	There are Multiple files: 

	If the input includes a RNA file, Texomer would generate output at both the DNA and the RNA levels. 
	Otherwise, it will output only estimation at the DNA level.

	(1) .segment file: 
		position of copy nuymber segments; 
		allele-specific copy number (Dmajor and Dminor), 
		allele-specific expression levels(Rmajor and Rminor), and 
		posterior probability of discordant expression corresponding to each segment.

	(2) .mutation file: 
		position of mutations, 
		reference(ref) and alternative(alt) allele, 
		allelic read count from RNA-seq(refNum and altNum),
		mutation type(germline or somatic), 
		allele specific copy number (altD corresponding to alternative allele and wildD corresponding to reference allele),
		allele specific expression level (altR corresponding to alternative allele and wildR corresponding to reference allele),
		posterior probability of discordant expression and DACRE score for each mutation.

	(3) .summary file: 
		tumor purity at the DNA and/or the RNA level; 
		ploidy and intra-tumor heterogeneity at the DNA level

Run Texomer:
***The python script for easy run of Texomer is in the release directory. You can tune the
parameters as you wish.***

	Python Texomer.py –p Rscript/path –g germline.input –s somatic.input –r RNA.bam <-t iter.optimal.index> <-e mutation.expression> <-o output.path>

About the default parameters
========================
Texomer optimizes estimation of tumor purity and allele specific copy numbers by combining both germline SNPs and somatic SNVs. 

By default, -t is 1, which turns on iterative optimization.

A user can set -t 0 to turn off the iterative optimization.


Example
=====
Try Texomer in the package directory on the example dataset 

	python Texomer.py -g $PWD/example/TCGA-3C-AAAU-01A-11D-A41F-09.snp.germline.input -s $PWD/example/TCGA-3C-AAAU-10A-01D-A41F-09.mutect.vcf -e $PWD/example/TCGA-3C-AAAU.RNA.SNV -p $PWD 
