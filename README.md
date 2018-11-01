Texomer
===========
Author: Fang Wang
Email: fwang9@mdanderson.org or kchen3@mdanderson.org

Draft date: Nov. 1, 2018

Description
===========
Texomer is a tool used in cancer genomic studies to perform allele-specific, tumor-deconvoluted transcriptome-exome integration of the bulk whole exome (WES) and whole transcriptome sequencing (WTS) data obtained from autologous patient tissue sample.

It reports estimations of tumor purity at both the DNA and RNA level as well as intratumor heterogeneity at the DNA level. Moreover, Texomer yeilds allelic specific copy numbers and expression levels along with a differential allelic cis-regulatory effect (DACRE) score of each variant allele.

If the input only includes information at the DNA level, it only reports the estimates at the DNA level and focuses on somatic mutations.

System requirements and dependency
==================================
Texomer runs on a x86_64 Linux system. It depends on samtools and bedtools to extract reads covering variant sites from WTS data.

It also requires R (version >= 3.4)
to run and has dependency on the R packages:

	bbmle, emdbook, copynumber,TitanCNA, facets, mixtools, ASCAT and Sequenza.

***These R packeages are already incorporated in this release. Users do not need to install them.***

Installation
============
Please download and copy the distribution to your specific location. If you are cloning from github, ensure that you have git-lfs installed.

For example, if the downloaded distribuition is Texomer-2.0.tar.gz.
	Type 'tar zxvf Texomer-2.0.tar.gz'

Then, run Texomer.py in the resulting folder.

Usage
=====
```
Options:
  --version             show program's version number and exit
  -h, --help            Show this help message and exit.
  -p TEXOMER, --Texomer=TEXOMER
                        the path of Texomer
  -t TUMOR, --Tumor=TUMOR
                        Tumor WES bam file.
  -n NORMAL, --Normal=NORMAL
                        Normal WES bam file.
  -o OUTPATH, --outpath=OUTPATH
                        the output path.
  -r RNA, --RNA=RNA     RNA-seq data bam file       
  -g GERMLINE, --germline=GERMLINE
                        You can input your own germline mutation file instead of
                        -t, -n and -v. When you use -g, -s is required. The file
                        include 8 columns:
                        chromosome, position, RefAllele, Altallele,
                        read counts of RefAllele in normal, read counts of
                        Altallele in normal, read counts of RefAllele in tumor
                        and read counts of Altallele in tumor with header
                        seperated by tab.
  -s SOMATIC, --somatic=SOMATIC
                        You can input your own somatic mutation file instead of
                        -t, -n and -v. When you use -s, -g is required.
                        The file include 8 columns:
                        chromosome, position, RefAllele, Altallele, read
                        counts of RefAllele in normal, read counts of
                        Altallele in normal, read counts of RefAllele in tumor
                        and read counts of Altallele in tumor with header
                        seperated by tab.
  -v VARSCAN, --varscan=VARSCAN
                        the Varscan2 output file based on somatic calling
  -e SNVEXPRESS, --snvexpress=SNVEXPRESS
                        The allelic read count of mutation from RNA-seq
                        including 7 columns: chromosome, position, ref, alt,
                        refnum, altnum and type (germline or somatic)
  -u ITER, --iter=ITER  optimal through somatic mutation, 0 corresponding to
                        no optimation and 1 is optimation. The default = 1

```

Input files:
============

DNA input files:

	Three kinds of input files are allowed in Texomer:

	(1) WES bam file of tumor and normal tissue

	(1) The output generated by Varscan, which includes both germline and somatic variants through -v;

	(2) Two seperate files: one contains germline variants through -g and the other contains somatic mutations through -s.

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

	(1) A RNA-seq bam file.

	(2) Allelic read counts of mutation from the RNA-seq bam, which includes 7 tab-delimited cloumns:
		chromosome (start from "chr"),
		position,
		reference allele,
		alternative allele,
		reference allelic read count from the RNA-seq bam,
		alternative allelic read count from the RNA-seq bam,
		type of mutation (germline or somatic).    

Output files:
=============

There are multiple files:

If the input includes a RNA file, Texomer would generate output at both the DNA and the RNA levels. Otherwise, it will output only estimation at the DNA level.

	(1) output.segment.txt file:
		position of copy nuymber segments;
		allele-specific copy number (Dmajor and Dminor),
		allele-specific expression levels(Rmajor and Rminor), and
		posterior probability of discordant expression corresponding to each segment.

	(2) output.mutation.txt file:
		position of mutations,
		reference(ref) and alternative(alt) allele,
		allelic read count from RNA-seq(refNum and altNum),
		mutation type(germline or somatic),
		allele specific copy number (altD corresponding to alternative allele and wildD corresponding to reference allele),
		allele specific expression level (altR corresponding to alternative allele and wildR corresponding to reference allele),
		posterior probability of discordant expression and DACRE score for each mutation.

	(3) output.summaryres.txt file:
		tumor purity at the DNA and/or the RNA level;
		ploidy and intra-tumor heterogeneity at the DNA level

Run Texomer:
***The python script for easy run of Texomer is in the release directory. You can tune the
parameters as you wish.***

	Python Texomer.py –p path/Texomer –t tumor.bam –n normal.bam –r RNA.bam [...]
[...] contains optional parameters

About the default parameters
========================
Texomer optimizes estimation of tumor purity and allele specific copy numbers by combining both germline SNPs and somatic SNVs.

By default, -u is 1, which turns on iterative optimization.

A user can set -u 0 to turn off the iterative optimization.


Example
=======
Try Texomer in the package directory on the different example datasets

**Example 1: Input defined mutation file**

	python Texomer.py -p ./ -g ./example/germline.input -s ./example/somatic.input.vcf -e ./example/RNA.SNV -o ./res1

*germline.input:*
	chr	pos	ref	alt	refNumN	altNumN	refNumT	altNumT
	chr1	12198	G	C	51	40	26	25
	chr1	12383	G	A	31	22	15	19

*somatic.input:*
	chr	pos	ref	alt	refNumN	altNumN	refNumT	altNumT
	chr1	16757604	G	A	170	0	45	8
	chr1	23083363	G	A	34	0	22	3

*RNA.SNV:*
	chr	pos	ref	alt	refNum	altNum	type
	chr1	12198	G	C	8	0	germline
	chr1	12383	G	A	2	0	germline

**Example 2: Input varscan2 output**

	python Texomer.py -p ./ -v ./example/varscan.snp -r ./example/RNA.bam -o ./res2

**Example 3: Input WES bam file**

	python Texomer.py -p ./ -t ./example/Tumor.bam -n ./example/Normal.bam -r ./example/RNA.bam -o ./res3


*Texomer output:*

output.summaryres.txt

	DNApurity	0.272030490933697
	Heterogeneity	0.362166913670839
	Ploidy	2.43420373296619
	RNApurity	0.333623992547959

output.segment.txt

	chr	start	end	Dmajor	Dminor	Rmajor	Rminor	RTEL	BayesP
	chr1	12383	149854059	3	0	3	0	3	0.00271288011564652
	chr1	149904572	151142541	11	0	11	0	11	0.00976386546342944
	chr1	151242535	153818025	2	0	2	0	2	0.000597295774993456
	chr1	153923949	158458304	3	0	7	1	7.66332510379357	0.999999999972296
	chr1	158458909	164820155	3	0	3	0	3	0.0529930675747959
	chr1	164984095	186977737	1	0	1	0	1	0.000118593403366063
	chr1	187641566	248855548	2	0	2	0	2	0.0104407585632365
	chr2	277314	242175216	2	0	2	0	2	0.00448976931165068

Dmajor: Copy number of major allele

Dminor: Copy number of minor allele

Rmajor: Expressoin level of major allele

Rminor: Expression level of minor allele

RTEL: Total expression level of two alleles

BayesP: Posterior probability that expression level is discordant with copy number

output.mutations.txt

	chr	pos	ref	alt	refNum	altNum	type	altD	altR	wildD	wildR	BayesP	eASEL	AEI	DACRE
	chr1	14464	A	T	161	29	Germline	0	0.869112773207088	3	4.26647728279266	0.165069332450197	0.869112773207088	-2.29543007263941	-2.99135865987574
	chr1	14677	G	A	562	275	Germline	0	0.565025922875357	3	3.23925254542283	0.0153830861293918	0.565025922875357	-2.51927198724942	-2.26822077283764
	chr1	14907	A	G	31	55	Germline	0	1.46755414239337	3	0.0444198889562267	0.000196160197227124	1.46755414239337	5.0460641330148	1.82914010314698
	chr1	14930	A	G	19	74	Germline	0	2.51062821238407	3	0.839923148663372	0.0025752700881877	-0.489371787615929	1.57971916697172	1.26469921401086
	chr1	15118	A	G	22	17	Germline	0	0.662118740553368	3	1.44432611898557	0.000185221478464004	0.662118740553368	-1.12523465975723	-0.376201528722366
	chr1	15211	T	G	14	33	Germline	3	2.90842246811015	0	0.0838658781010807	0.00685173702972997	-0.0915775318898535	5.11600908428328	2.41855074029923

refNum: Read counts of reference allele from RNA-seq data

altNum: Read counts of alernative allele from RNA-seq data

type: the type of mutation Germline or Somatic

altD: copy number of alternative allele

altR: expression level of alternative allele

wildD: copy number of reference allele (wildtype allele)

wildR: expression level of reference allele (wildtype allele)

BayesP: posterior probability that expression level is discordant with copy number

eASEL: difference of alternative allele between expression and copy number (altR-altD)

AEI: difference of expression levels between alternative and reference allele (altR-wildR)

DACRE: DACRE score
