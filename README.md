<img src="https://github.com/hugheslab/snpgenie/blob/master/logo1a.jpg?raw=true" alt="SNPGenie logo" width="450" height="175" align="middle">

SNPGenie is a Perl script for estimating evolutionary parameters from next-generation sequencing (NGS) single-nucleotide polymorphism (SNP) variant data. Three different analyses are possible:

1. For an analysis of **POOLED NGS SNP DATA**, run the script **snpgenie.pl**. SNP reports (acceptable in a variety of formats) much each correspond to a single population, with variants called relative to a single reference sequence (one sequence in one FASTA file). SNP reports may summarize either single sequencing runs where the input was DNA pooled from multiple individuals, or may be summary files for multiple individual sequences. Just run the script in a directory containing the necessary [input files](#snpgenie-input), and we take care of the rest! For the earlier version, see <a target="_blank" href="http://ww2.biol.sc.edu/~austin/">Hughes Lab Bioinformatics Resource</a>.
2. **COMING SOON!** For a traditional **BETWEEN-GROUP ANALYSIS** with groups of FASTA sequences such as possible using the MEGA software, run the script **snpgenie\_between\_group.pl**. This script is being released for use with datasets which exceed the processing and memory capabilities of the MEGA software. Just run the script in a directory containing the necessary [between-group input files](#snpgenie-input-between), and we take care of the rest!

**UPDATE FOR VCF INPUT:** given the preponderance of distinct VCF formats in use, it is now necessary to specify the specific format of the VCF SNP report input using the **--vcfformat** argument. See the section on [VCF](#vcf).

## Contents

* [Introduction](#introduction)
* [SNPGenie Input](#snpgenie-input)
	* **Input 1:** [Reference Sequence](#ref-seq)
	* **Input 2:** [Gene Transfer Format](#gtf)
	* **Input 3:** [SNP Report(s)](#SNP-Reports)
		* [CLC](#clc)
		* [Geneious](#geneious)
		* [VCF](#vcf)
	* [A Note on Reverse Complement ('—' Strand) Records](#revcom)
* [Options](#options)
* [How SNPGenie Works](#how-snpgenie-works)
* [Output](#output)
* [Additional Scripts](#additional-scripts)
* [Troubleshooting](#troubleshooting)
* [Citation](#citation)
* [Studies Using SNPGenie](#studies-using-snpgenie)
* [Contact](#contact)
* [References](#references)


## <a name="introduction"></a>Introduction

New applications of next-generation sequencing (NGS) use pooled samples containing DNA from multiple individuals to perform population genetic analyses. SNPGenie is a Perl program which can analyze the single-nucleotide polymorphism (SNP) caller results to calculate evolutionary parameters, such as nucleotide diversity (including its nonsynonymous and synonymous partitions, *π*<sub>N</sub> and *π*<sub>S</sub>) and gene diversity. These calls are typically present in annotation tables and assume that the pooled nucleic acid sample is representative of the population of interest. For example, if one is interested in determining the nucleotide diversity of a virus population within a single host, it might be appropriate to sequence the pooled nucleic acid content of the virus in a blood sample from that host. Comparing *π*<sub>N</sub> and *π*<sub>S</sub> for, say, a gene product, or comparing gene diversity at polymorphic sites of different types, may help to dicepher instances of positive (Darwinian) selection, negative (purifying) selection, and random genetic drift. SNPGenie also includes such features as minimum allele frequency trimming (see [Options](#options)), and can be combined with upstream applications such as maximum-likelihood SNP calling techniques (*e.g.*, see Lynch *et al.* 2014). For additional background, see Nelson & Hughes (2015) in the [References](#references).

## <a name="snpgenie-input"></a>SNPGenie Input

SNPGenie is a command-line interface application written in Perl, with no additional dependencies beyond the standard Perl package. As such, it is limited only by the memory and processing capabilities of the local hardware. As input, it accepts:

1. One [**Reference Sequence**](#ref-seq) file, containing one reference sequence, in **FASTA** format (.fa/.fasta); 
2. One file with CDS information in [**Gene Transfer Format**](#gtf) (.gtf); and 
3. One or more tab-delimited (.txt) [**SNP Reports**](#SNP-Reports), each corresponding to a single pooled-sequencing run (*i.e.*, population) in CLC, VCF, or Geneious format, with variants called relative to the reference sequence. If you want another format included, just ask!

For ease and simplicity, one need only run SNPGenie in a directory containing the necessary input files, and SNPGenie takes care of the rest (see [Options](#options) if you wish for more control). To do this, first download the **snpgenie.pl** script and place it in your system’s PATH, or simply in your working directory. Next, place your SNP report(s), FASTA (.fa/.fasta), and GTF (.gtf) files in your working directory. Open the command line prompt (or Terminal) and navigate to the directory containing these files using the "cd" command in your shell. Finally, simply execute SNPGenie by typing the name of the script and pressing the \<RETURN\> (\<ENTER\>) key. Further details on input are below.

### <a name="ref-seq"></a>Reference Sequence
Only one reference sequence must be provided in a **FASTA** (.fa/.fasta) file (one file containing one sequence). Thus, all SNP coordinates in the SNP reports are called relative to the single reference sequence. This **ONE-SEQUENCE MODE** allows the maximum number of estimations to be performed, and is the only mode of SNPGenie that remains supported. Because of this one-sequence stipulation, a script has been provided to split a multi-sequence FASTA file into its constituent sequences if need be; see [Additional Scripts](#additional-scripts) below.

>N.B. In the past, a **MULTI-SEQUENCE MODE** was activated if two or more FASTA files were present. In this case, each was assumed to refer to a different protein product, as might occur with a segmented viral genome, and each FASTA file name needed to begin with the name of the product followed by an underscore. For example, if "ORF1" was the name of one of the products in the SNP report, its reference FASTA file was named "ORF1_xxx.fasta". The mode can be found in versions of SNPGenie dating prior to 29 January 2016. It is now recommended simply to run SNPGenie separately for each genome segment in such cases.

### <a name="gtf"></a>Gene Transfer Format
The **Gene Transfer Format** (.gtf) file is tab (\t)-delimited, and must include *non-redundant* records for all CDS elements (*i.e.*, open reading frames, or ORFs) present in your SNP report(s), and should also include any ORFs which do not contain any variants (if they exist). Note that SNPGenie expects every coding element to be labeled as type "CDS", and for its product name to follow a "gene\_id" tag. In the case of CLC and Geneious SNP reports, this name must match that present in the SNP report. If a single coding element has multiple segments (*e.g.*, exons) with different coordinates, simply enter one line for each segment, using the same product name. (Although SNPGenie could only handle 2 segments per ORF in the past, there is now no limit.) Finally, for cases with reverse '–' strand features, SNPGenie must be run twice, once for each strand, with that strand's own set of input files (*i.e.*, the '–' strand FASTA, GTF, and SNP report); see [A Note on Reverse Complement ('–' Strand) Records](#revcom) below. For more information about GTF, please visit <a target="_blank" href="http://mblab.wustl.edu/GTF22.html">The Brent Lab</a>. For a conversion script for GFF files, see [Additional Scripts](#additional-scripts). A simple GTF example follows:

	reference.gbk	CLC	CDS	5694	8369	.	+	0	gene_id "ORF1";
	reference.gbk	CLC	CDS	8203	8772	.	+	0	gene_id "ORF2";
	reference.gbk	CLC	CDS	1465	4485	.	+	0	gene_id "ORF3";
	reference.gbk	CLC	CDS	5621	5687	.	+	0	gene_id "ORF4";
	reference.gbk	CLC	CDS	7920	8167	.	+	0	gene_id "ORF4";
	reference.gbk	CLC	CDS	5395	5687	.	+	0	gene_id "ORF5";
	reference.gbk	CLC	CDS	7920	8016	.	+	0	gene_id "ORF5";
	reference.gbk	CLC	CDS	4439	5080	.	+	0	gene_id "ORF6";
	reference.gbk	CLC	CDS	5247	5549	.	+	0	gene_id "ORF7";
	reference.gbk	CLC	CDS	4911	5246	.	+	0	gene_id "ORF8";

### <a name="SNP-Reports"></a>SNP Reports

Each SNP report should contain variant calls for a single pooled-sequencing run (*i.e.*, population), or alternatively a summary of multiple individual sequences, in one of the following formats:

#### <a name="clc"></a>CLC Genomics Workbench
At minimum, the <a target="_blank" href="http://www.clcbio.com/products/clc-genomics-workbench/">CLC Genomics Workbench</a> SNP report must include the following 8 default column selections, with the unaltered CLC column headers: 

* **Reference Position**, which refers to the start site of the polymorphism within the reference FASTA sequence;
* **Type**, which refers to the nature of the record, usually the type of polymorphism, *e.g.*, "SNV” for single-nucleotide variants;
* **Reference**, the reference nucleotide(s) at that site(s);
* **Allele**, the variant nucleotide(s) at that site(s);
* **Count**, the number of reads containing the variant;
* **Coverage**, the total number of sequencing reads at the site(s);
* **Frequency**, the frequency of the variant as a percentage, *e.g.*, “14.6” for 14.60%; and
* **Overlapping annotations**, containing the name of the protein product or open reading frame (ORF), *e.g.*, “CDS: ORF1”.

In addition to the aforementioned columns, the SNP report should ideally be free of thousand separators (,) in the Reference Position, Count, and Coverage columns (default format). The Frequency must remain a percentage (default format). Finally, the user should verify that the reading frame in the CLC output is correct. SNPGenie will produce various errors to indicate when these conditions are not met, *e.g.*, by checking that all products begin with START and end with STOP codons, and checking for premature stop codons. Make sure to check the SNPGenie LOG file!

#### <a name="geneious"></a>Geneious
At minimum, the <a target="_blank" href="http://www.geneious.com/">Geneious</a> SNP report must include the following default column selections, with the unaltered Geneious column headers:

* **Minimum** and **Maximum**, which refer to the start and end sites of the polymorphism within the reference FASTA sequence, and will hold the same value for SNP records;
* **CDS Position**, with the coordinate of the site relative to the start cite of the relevant CDS annotation;
* **Type**, which refers to the nature of the record entry, *e.g.*, “Polymorphism”;
* **Polymorphism Type**, which gives the type of polymorphism;
* **product**, containing the name of the protein product or open reading frame, e.g., ORF1; 
* **Change**, which contains the reference and variant nucleotides, *e.g.*, "A -> G", and are always populated for SNP records;
* **Coverage**, containing the number of sequencing reads that include the site; and
* **Variant Frequency**, which contains the frequency of the nucleotide variant as a percentage, *e.g.*, 14.60%.

As with CLC, the Geneious SNP report should ideally be free of extraneous characters such as thousand separators (,), but SNPGenie will do its best to adapt if they are present. Again, the Variant Frequency must remain a percentage (default format). Again, the user should verify that the reading frame in the Geneious output is correct. SNPGenie will produce various errors to indicate when these conditions are not met, *e.g.*, by checking that all products begin with START and end with STOP codons, and checking for premature stop codons. Make sure to check the SNPGenie LOG file!

#### <a name="vcf"></a>Variant Call Format (VCF)
At minimum, the <a target="_blank" href="https://github.com/samtools/hts-specs">VCF</a> SNP report must include (and at present does so by definition) the following columns, with the unaltered VCF column headers:

* **CHROM**, the name of the reference genome;
* **POS**, which refers to the start site of the polymorphism within the reference FASTA sequence;
* **REF**, the reference nucleotide(s) at that site(s);
* **ALT**, the variant nucleotide(s) at that site(s), with multiple variants at the same site separated by commas (,);
* **QUAL**, the Phred quality score for the variant;
* **FILTER**, the filter status, based on such metrics as minimum frequencies and minimum quality scores;
* **INFO**, information regarding read depth and/or allele frequencies; 
* **FORMAT**, sometimes an alternative to the **INFO** column for read depth and/or allele frequency data; 
* **\<SAMPLE\>**, a column with variable names depending on user specification, and another occasional alternative to the **INFO** column for read depth and/or allele frequency data.

**SNPGenie now requires** its users to specify exactly which VCF format is being submitted using the **--vcfformat** argument. New formats are being added on a case-by-case basis; users should [contact the author](#contact) to have new formats incorporated. Current formats are:

1. **FORMAT 1: --vcfformat=1**. Multiple individual genomes have been sequenced separated, following SNP summarization in the VCF file. SNPGenie will require the following in the **INFO** column:
	* **NS**, the number of samples (*i.e.*, individual sequencing experiments) being summarized (*e.g.*, "NS=30"); 
	* **AF**, the fractional allele frequency(-ies) for the variant alleles in the same order as listed in the ALT column (*e.g.*, "AF=0.200"). If multiple variants exist at the time, their frequencies must be separated by commans (*e.g.*, "AF=0.200,0.087").

2. **FORMAT 2:  --vcfformat=2**. Variants have been called from a pooled (deep) sequencing sample containing genomes from multiple individuals. SNPGenie will require the following in the **INFO** column:
	* **DP**, the coverage or total read depth (*e.g.*, "DP=4249");
	* **AF**, the fractional allele frequency(-ies) for the variant alleles in the same order as listed in the ALT column (*e.g.*, "AF=0.01247"; "AF=0.01247,0.08956" for two variants; etc.).

3. **FORMAT 3: --vcfformat=3**. Like format 2, variants have been called from a pooled (deep) sequencing sample containing genomes from multiple individuals. However, SNPGenie will instead require the following in the **INFO** column:
	* **DP4**, containing the number of reference and variant reads on the forward and reverse strands in the format *DP4=\<num. fw ref reads\>,\<num. rev ref reads\>,\<num. fw var reads\>,\<num. rev var reads\>* (*e.g.*, "DP4=11,9,219,38").
	* **N.B.**: if multiple single nucleotide variants exist at the same site, this format does not specify the number of reads for each variant. SNPGenie thus approximates by dividing the number of non-reference reads evenly amongst the variant alleles.

4. **FORMAT 4:  --vcfformat=4**. Like formats 2 and 3, variants have been called from a pooled (deep) sequencing sample containing genomes from multiple individuals. However, SNPGenie will instead require data in the **FORMAT** and **\<SAMPLE\>** columns, with the data tag in the former and the data value in the latter, and with order preserved between the two columns:
	* For reference allele depth, include the **AD** tag in the **FORMAT** column and the read depth for each allele in the **\<SAMPLE\>** column, with values for variant allele(s) in the same order as listed in the **ALT** column (*e.g.*, "AD" in the FORMAT column and "75,77" in the SAMPLE column);
	* For coverage (total read depth), include **DP** in the **FORMAT** column and the total read depth in the **\<SAMPLE\>** column (*e.g.*, "DP" in the FORMAT column and "152" in the SAMPLE column).



As usual, you will want to make sure to maintain the VCF file's features, such as TAB(\t)-delimited columns. Unlike some other formats, the allele frequency in VCF is a decimal.

### <a name="revcom"></a>A Note on Reverse Complement ('–' Strand) Records
Many large genomes have coding products on both strands. In this case, SNPGenie must be run twice: once for the '+' strand, and once for the '—' strand. This requires FASTA, GTF, and SNP report input for the '–' strand. Check out **snpgenie-vcf2revcom.pl**, described in the [Additional Scripts](#additional-scripts) below, which creates these files for you from your original data. Note that, regardless of the original SNP report format, the reverse complement SNP report is in a CLC-like format that SNPGenie will recognize. For both runs, the GTF should include all products for both strands, with the products on the strand being analyzed classified as '+' and having coordinates defined with reference to the beginning of that FASTA sequence. Also note that a GTF file containing *only* '—' strand records will not run; SNPGenie does calculations only for the products on the current + strand, using the '—' strand products only to determine the number of overlapping reading frames.

## <a name="options"></a>Options

In case you want to alter the way SNPGenie works, the following options (implemented using Perl's Getopt::Long module) may be used:

* **--minfreq**: optional floating point parameter specifying the minimum allele (SNP) frequency to include. Enter as a proportion/decimal (*e.g.*, 0.01), **not** as a percentage (*e.g.*, 1.0%). Default: 0.
* **--snpreport**: optional string parameter specifying the (one) SNP report to analyze. Default: auto-detect .txt and .csv file(s).
* **--vcfformat**: optional integer parameter for use with VCF SNP reports, specifying the exact format of VCF input. See [VCF](#vcf).
* **--fastafile**: optional string parameter specifying the (one) reference sequence. Default: auto-detect .fa and/or .fasta file(s).
* **--gtffile**: optional string parameter specifying the one file with CDS annotations. Default: auto-detect the .gtf file.
* **--sepfiles**: optional Boolean (flag) parameter specifying whether to product separate results (codon) files for each SNP report (all results already included together in the codon_results.txt file). Simply include in the command line to activate. Default: not included.
* **--slidingwindow**: optional integer parameter specifying the length of the sliding (codon) window used in the analysis. Default: 9 codons.
* **--ratiomode**: optional Boolean (flag) parameter specifying whether to include *π* values for each codon in the codon_results.txt file(s). This is usually inadvisable, as *π* values (especially *π*<sub>S</sub>) are subject to great stochastic error. Simply include in the command line to activate. Default: not included.
* **--sitebasedmode**: optional Boolean (flag) parameter specifying whether to include π values derived using a site-based (reference codon context only) approach in the codon_results.txt file(s). This is usually inadvisable, as *π* values will not reflect the true population pairwise comparisons. Simply include in the command line to activate. Default: not included.

For example, if you wanted to turn on the **sepfiles** option, specify a minimum allele frequency of 1%, and specify your input files, you could enter the command:

	snpgenie.pl --sepfiles --minfreq=0.01 --snpreport=mySNPreport.txt --fastafile=myFASTA.fa --gtffile=myGTF.gtf

## <a name="how-snpgenie-works"></a>How SNPGenie Works

Given the appropriate files, SNPGenie calculates gene and nucleotide diversities for different types of sites in a protein-coding sequence. Nucleotide diversity may be defined as the average number of nucleotide variants per nucleotide site for all pairwise comparisons. To distinguish between nonsynonymous and synonymous differences and sites, it is necessary to consider the codon context of each nucleotide in a sequence. This is why the user must submit the starting and ending sites of the coding regions in the .gtf file, along with the reference FASTA sequence file, so that the numbers of nonsynonymous and synonymous sites for each codon may be accurately estimated by a derivation of the Nei-Gojobori (1986) method. SNPGenie first splits the coding sequence into codons, each of which contains 3 sites. The software then determines the number of these sites which are nonsynonymous and synonymous by testing all polymorphisms present at each site of every codon in the sequence. Because different nucleotide variants at the same site may lead to both nonsynonymous and synonymous polymorphisms, fractional sites occur frequently (*e.g.*, only 2 of 3 possible nucleotide substitutions at the third position of AGA cause an amino acid change; thus, that site is considered 2/3 nonsynonymous and 1/3 synonymous). Next, the SNP report is consulted for the presence of variants to produce a revised estimate. Variants are incorporated through averaging weighted by their frequency. Although it is relatively rare, high levels of sequence variation may alter the number of nonsynonymous and synonymous sites in a particular codon, contributing to an altered picture of natural selection.

Next, SNPGenie calculates the number of nucleotide differences for each codon in each ORF specified in the .gtf file. Calculating nucleotide diversity codon-by-codon enables sliding-window analyses that may help to pinpoint important nucleotide regions subject to varying forms of natural selection. SNPGenie determines the average number of pairwise differences as follows: for every variant in the SNP Report, the number of variants is calculated as the product of the variant’s relative frequency and the coverage at that site. For each variant nucleotide (up to 3 non-reference nucleotides), the number of variants is stored, and their sum is subtracted from the coverage to yield the reference’s absolute frequency. Next, for each pairwise nucleotide comparison at the site, it is determined whether the comparison represents a nonsynonymous or synonymous change. If the former, the product of their absolute frequencies contributes to the number of nonsynonymous pairwise differences; if the latter, it contributes to the number of synonymous pairwise differences. When comparing codons with more than one nucleotide difference, all possible mutational pathways are considered, per the method of Nei & Gojobori (1986). The sum of pairwise differences is divided by the total number of pairwise comparisons at the codon (<sub>*n*</sub>C<sub>2</sub>, where *n* = coverage) to yield the mean number of differences per site of each type. This is calculated separately for nonsynonymous and synonymous comparisons. For further background, see Nelson & Hughes (2015).

## <a name="output"></a>Output

SNPGenie creates a new folder called SNPGenie_Results within the working directory. This contains the following TAB-delimited results files:

1. **SNPGenie_parameters.txt**, containing the input parameters and file names.

2. **SNPGenie_LOG.txt**, documenting any peculiarities or errors encountered. Warnings are also printed to the Terminal (shell) window.

3. **site_results.txt**, providing results for all polymorphic sites. Note that, if the population is genetically homogenous at a site, even if it differs from the reference or ancestral sequence, it will not be considered polymorphic. Also keep in mind that columns are sorted by product first, then site number, with noncoding sites at the end of the file. Columns are:
	* *file*. The SNP report analyzed.
	* *product*. The CDS annotation to which the site belongs; "noncoding" if none on this strand.
	* *site*. The site coordinate of the nucleotide in the reference sequence.
	* *ref_nt*. The identity of the nucleotide in the reference sequence.
	* *maj_nt*. The most common nucleotide in the population at this site.
	* *position_in_codon*. If present in a CDS annotation, the position of this site within its codon (1, 2, or 3).
	* *overlapping_ORFs*. The number of CDS annotations overlapping this site. For example, if the site is part of only one open reading frame, the value will be 0. If the site is part of two open reading frames, the value will be 1.
	* *codon_start_site*. The site coordinate of the relevant codon's first nucleotide in the reference sequence.
	* *codon*. The identity of the relevant codon.
	* *pi*. Nucleotide diversity at this site.
	* *gdiv*. Gene diversity at this site.
	* *class_vs_ref*. This site's classification, as compared to the reference sequence. For example, if the site contains only one SNP, and that SNP is synonymous, the site will be classified as Synonymous. Nonsynonymous or Synonymous.
	* *class*. This site's classification as compared to all sequences present in the population. For example, if the population contains both A and G residues at the third site of a GAA (reference) codon, then the site will be Synonymous, because both GAA and GAG encode Glu. On the other hand, if the population also contains a C at this site, the site will be Ambiguous, because GAC encodes Asp, meaning both nonsynonymous and synonymous polymorphisms exist at the site. Nonsynonymous, Synonymous, or Ambiguous.
	* *coverage*. The NGS read depth at the site.
	* *A*. The number of reads containing an A (adenine) nucleotide at this site. N.B.: may be fractional if the coverage and variant frequency given in the SNP report do not imply a whole number.
	* *C*. For C (cytosine), as for A.
	* *G*. For G (guanine), as for A.
	* *T*. For T (thymine), as for A.

4. **codon_results.txt**, providing results for all polymorphic sites. Columns are:
	* *file*. The SNP report analyzed.
	* *product*. The CDS annotation to which the site belongs; "noncoding" if none.
	* *site*. The site coordinate of the nucleotide in the reference sequence.
	* *codon*. The identity of the relevant codon.
	* *num_overlap_ORF_nts*. The number of nucleotides in this codon (up to 3) which overlap other ORFs (in addition to the current "product" annotation).
	* *mean_nonsyn_diffs*. The mean number of pairwise nucleotide comparisons in this codon which are nonsynonymous (i.e., amino acid-altering) in the pooled sequence sample. The numerator of *π*<sub>N</sub>.
	* *mean_syn_diffs*. The mean number of pairwise nucleotide comparisons in this codon which are synonymous (i.e., amino acid-conserving) in the pooled sequence sample. The numerator of *π*<sub>S</sub>.
	* *nonsyn_sites*. The mean number of sites in this codon which are nonsynonymous, given all sequences in the pooled sample. The denominator of *π*<sub>N</sub> and mean *d*<sub>N</sub> versus the reference.
	* *syn_sites*. The mean number of sites in this codon which are synonymous, given all sequences in the pooled sample. The denominator of *π*<sub>S</sub> mean *d*<sub>S</sub> versus the reference.
	* *nonsyn_sites_ref*. The number of sites in this codon which are nonsynonymous in the reference sequence.
	* *syn_sites_ref*. The number of sites in this codon which are synonymous in the reference sequence.
	* *mean_nonsyn_diffs_vs_ref*. This codon's mean number of nonsynonymous nucleotide differences from the reference sequence in the pooled sequence sample. The numerator of mean *d*<sub>N</sub> versus the reference.
	* *mean_syn_diffs_vs_ref*. This codon's mean number of synonymous nucleotide differences from the reference sequence in the pooled sequence sample. The numerator of mean *d*<sub>S</sub> versus the reference.
	* *mean_gdiv*. Mean gene diversity (observed heterozygosity) for this codon's nucleotide sites.
	* *mean_nonsyn_gdiv*. Mean gene diversity for this codon's nonsynonymous polymorphic sites.
	* *mean_syn_gdiv*. Mean gene diversity for this codon's synonymous polymorphic sites.

5. **\<SNP report name(s)\>_results.txt**, containing the information present in the codon_results.txt file, but separated by SNP report.

6. **product_results.txt**, providing results for all CDS elements present in the GTF file for the '+' strand. Columns are:
	* *file*. The SNP report analyzed.
	* *product*. The CDS annotation to which the site belongs; "noncoding" if none.
	* *mean_nonsyn_diffs*. The sum over all codons in this product of the mean number of pairwise nucleotide comparisons which are nonsynonymous (i.e., amino acid-altering) in the pooled sequence sample. The numerator of *π*<sub>N</sub>.
	* *mean_syn_diffs*. The sum over all codons in this product of the mean number of pairwise nucleotide comparisons which are synonymous (i.e., amino acid-conserving) in the pooled sequence sample. The numerator of *π*<sub>S</sub>.
	* *mean_nonsyn_diffs_vs_ref*. The sum over all codons in this product of the mean number of nonsynonymous nucleotide differences from the reference sequence in the pooled sequence sample. The numerator of mean *π*<sub>N</sub> versus the reference.
	* *mean_syn_diffs_vs_ref*. The sum over all codons in this product of the mean number of synonymous nucleotide differences from the reference sequence in the pooled sequence sample. The numerator of mean *π*<sub>S</sub> versus the reference.
	* *nonsyn_sites*. The mean number of sites in this product which are nonsynonymous, given all sequences in the pooled sample. The denominator of *π*<sub>N</sub> and mean *d*<sub>N</sub> versus the reference.
	* *syn_sites*. The mean number of sites in this product which are synonymous, given all sequences in the pooled sample. The denominator of *π*<sub>S</sub> and mean *d*<sub>S</sub> versus the reference.
	* *piN*. (*π*<sub>N</sub>.) The mean number of pairwise nonsynonymous differences per nonsynonymous site in this product.
	* *piS*. (*π*<sub>S</sub>.) The mean number of pairwise synonymous differences per synonymous site in this product.
	* *mean_dN_vs_ref*. The mean number of nonsynonymous differences from the reference per nonsynonymous site in this product.
	* *mean_dS_vs_ref*. The mean number of synonymous differences from the reference per synonymous site in this product.
	* *mean_gdiv_polymorphic*. Mean gene diversity (observed heterozygosity) at all polymorphic nucleotide sites in this product.
	* *mean_gdiv_nonsyn*. Mean gene diversity at all nonsynonymous polymorphic nucleotide sites in this product.
	* *mean_gdiv_syn*. Mean gene diversity at all synonymous polymorphic nucleotide sites in this product.

7. **population_summary.txt**, providing summary results for each population's sample (SNP report) with respect to the '+' strand. Columns are:
	* *file*. The SNP report analyzed.
	* *sites*. Total number of sites in the reference genome.
	* *sites_coding*. Total number of sites in the reference genome which code for a protein product on the analyzed '+' strand, given the CDS annotations in the GTF file.
	* *sites_noncoding*. Total number of sites in the reference genome which do not code for a protein product, given the CDS annotations in the GTF file.
	* *pi*. Mean number of pairwise differences per site in the pooled sample across the whole genome.
	* *pi_coding*. Mean number of pairwise differences per site in the pooled sample across all coding sites in the genome.
	* *pi_noncoding*. Mean number of pairwise differences per site in the pooled sample across all noncoding sites in the genome.
	* *nonsyn_sites*. The mean number of sites in the genome which are nonsynonymous, given all sequences in the pooled sample. The denominator of *π*<sub>N</sub> and mean *d*<sub>N</sub> versus the reference.
	* *syn_sites*. The mean number of sites in the genome which are synonymous, given all sequences in the pooled sample. The denominator of *π*<sub>S</sub> and mean *d*<sub>S</sub> versus the reference.
	* *piN*. The mean number of pairwise nonsynonymous differences per nonsynonymous site across the genome of the pooled sample.
	* *piS*. The mean number of pairwise synonymous differences per synonymous site across the genome of the pooled sample.
	* *mean_dN_vs_ref*. The mean number of nonsynonymous differences from the reference per nonsynonymous site across the genome of the pooled sample.
	* *mean_dS_vs_ref*. The mean number of synonymous differences from the reference per synonymous site across the genome of the pooled sample.
	* *mean_gdiv_polymorphic*. Mean gene diversity (observed heterozygosity) at all polymorphic nucleotide sites in the genome of the pooled sample.
	* *mean_gdiv_nonsyn*. Mean gene diversity at all nonsynonymous polymorphic nucleotide sites in the genome of the pooled sample.
	* *mean_gdiv_syn*. Mean gene diversity at all synonymous polymorphic nucleotide sites in the genome of the pooled sample.
	* *mean_gdiv*. Mean gene diversity at all nucleotide sites in the genome of the pooled sample.
	* *sites_polymorphic*. The number of sites in the genome of the pooled sample which are polymorphic.
	* *mean_gdiv_coding_poly*. Mean gene diversity at all polymorphic nucleotide sites in the genome of the pooled sample which code for a protein product, given the CDS annotations in the GTF file.
	* *sites_coding_poly*. The number of sites in the genome of the pooled sample which are polymorphic and code for a protein product, given the CDS annotations in the GTF file.
	* *mean_gdiv_noncoding_poly*. Mean gene diversity at all polymorphic nucleotide sites in the genome of the pooled sample which do not code for a protein product, given the CDS annotations in the GTF file.
	* *sites_noncoding_poly*. The number of sites in the genome of the pooled sample which are polymorphic and do not code for a protein product, given the CDS annotations in the GTF file.

8. **sliding_window_length<Length>_results.txt**, containing codon-based results over a sliding window, with a default length of 9 codons.

## <a name="snpgenie-input-between"></a>SNPGenie Between-Group

**COMING SOON!** The script **snpgenie\_between\_group.pl** can be used to calculate mean dN and dS between two or more groups of sequences in FASTA format. Users who have access to a computer cluster may wish to use this parallelized version of SNPGenie for datasets which exceed the processing and memory capabilities of the MEGA software. 

## <a name="snpgenie-input-within"></a>SNPGenie Within-Group

**COMING SOON!** The script **snpgenie\_within\_group.pl** will be used to calculate mean dN and dS within a group of sequences in FASTA format. Like the between-group script, this will be able to perform analyses on sequence data that outsize what can be handled by MEGA and other software platforms. Note that within-group mean dN and dS are equivalent to πN and πS when groups are samples from a single population.

## <a name="additional-scripts"></a>Additional Scripts

Some additional scripts are included to automate some common tasks when preparing SNPGenie input. These currently are:

* **snpgenie-gb2gtf.pl**. At the command line, provide this script with one argument: a GenBank (.gbk) file. It will extract the coding element (CDS) annotations to produce a Gene Transfer Format (.gtf) file ready for SNPGenie. Not working? Let us know, and we'll improve it! Here's an example:

        snpgenie-gb2gtf.pl my_genbank_file.gbk
        
* **snpgenie-gff2gtf.pl**. At the command line, provide this script with one argument: a General Feature Format (.gff) file. It will extract the coding element annotations to produce a Gene Transfer Format (.gtf) file ready for SNPGenie, with "gene_id" annotations identified using the GFF "ID" tag. Not working, or need a different tag? Let us know, and we'll improve it! Here's an example:

        snpgenie-gbk2gtf.pl my_gff_file.gff

* **snpgenie-split_fasta.pl**. At the command line, provide this script with one argument: a FASTA (.fa or .fasta) file containing multiple sequences. This script will create multiple files in the working directory, each containing one of the sequences. Here's an example:

        snpgenie-split_fasta.pl my_multi_fasta_file.fasta

* **snpgenie-vcf2revcom.pl**. This script automates the creation of the reverse complement input files. At the command line, provide this script with three arguments, in the following order: 
	1. A '+' strand FASTA (.fa or .fasta) file containing the reference sequence against which SNPs were called;
	2. A '+' strand GTF file containing both '+' and '–' strand products from the '+' strand point of view; and 
	3. A '+' strand SNP report in VCF format.

	This script will then create a '-' strand (reverse complement) version of each file in the working directory, with "_revcom" concatenated to the original file name. Here's an example:

        snpgenie-vcf2revcom.pl my_snp_report.vcf my_reference_sequence.fasta my_cds_file.gtf

## <a name="troubleshooting"></a>Troubleshooting

* Using **Windows**? SNPGenie was written for Unix systems (including Mac), which have Perl installed by default. Windows doesn't, but getting Perl installed is as simple as following these <a target="_blank" href="http://learn.perl.org/installing/windows.html">three-minute download instructions</a>, and you'll be good to go! Just open the Windows Command Prompt, and remember to type "perl" first when you run SNPGenie, *i.e.*, type "perl snpgenie.pl".
* SNPGenie isn't executing? Try preceding the whole command line with "perl" to make sure SNPGenie is being treated as a script. For example:

        perl snpgenie.pl --sepfiles --minfreq=0.01 --snpreport=mySNPreport.txt --fastafile=myFASTA.fa --gtffile=myGTF.gtf
    
* SNPGenie still isn't executing? You might also try making the script executable at the command line, as follows:

        chmod +x snpgenie.pl	

* Are (end-of-line) newline characters in Unix LF (\n) format? Although SNPGenie was also designed to accept Windows CRLF (\r\n) or Mac CR (\r) formats, these can sometimes introduce problems causing SNPGenie to crash or return all 0 values. Trying changing the newline character to Unix LF using a free program such a <a target="_blank" href="http://www.barebones.com/products/textwrangler/">TextWrangler</a>.

* Are the FASTA files and/or CLC files tab (\t)-delimited?

* Are the Geneious files comma-separated?

* Do the SNP reports contain all necessary columns? (See sections on Input above.)

* Does the Frequency (CLC) or Variant Frequency (Geneious) SNP report column contain a percentage, not a decimal (e.g., 11.0% rather than 0.11)?

* Was the SNP calling frame correct (i.e., do the codons for a product in the SNP report begin with ATG and end with TAA, TAG, or TGA)?

* Are the product coordinates in the gtf file correct? (You might use a free program such as <a target="_blank" href="http://www.megasoftware.net/">MEGA</a> to check that the CDS coordinates begin with ATG and end with TAA, TAG, or TGA.)

## <a name="citation"></a>Citation

When using this software, please refer to and cite:

>Nelson CW, Moncla LH, Hughes AL (2015) <a target="_blank" href="http://bioinformatics.oxfordjournals.org/content/31/22/3709.long">SNPGenie: estimating evolutionary parameters to detect natural selection using pooled next-generation sequencing data</a>. *Bioinformatics* **31**(22):3709-11, doi: 10.1093/bioinformatics/btv449.

## <a name="studies-using-snpgenie"></a>Studies Using SNPGenie

* Bailey AL, Lauck M, Ghai RR, Nelson CW, Heimbruch K, Hughes AL, Goldberg TL, Kuhn JH, Jasinska AJ, Freimer NB, Apetrei C, O’Connor DH (2016) <a target="_blank" href="http://jvi.asm.org/content/90/15/6724.abstract">Arteriviruses, pegiviruses, and lentiviruses are common among wild African monkeys</a>. *Journal of Virology* **90**(15):6724-6737.
* Moncla LH, Zhong G, Nelson CW, Dinis JM, Mutschler J, Hughes AL, Watanabe T, Kawaoka Y, Friedrich TC (2016) <a target="_blank" href="http://www.cell.com/cell-host-microbe/abstract/S1931-3128(16)30010-5">Selective bottlenecks shape evolutionary pathways taken during mammalian adaptation of a 1918-like avian influenza virus</a>. *Cell Host & Microbe* **19**(2):169-80.
* Gellerup D, Balgeman A, Nelson CW, Ericsen A, Scarlotta M, Hughes AL, O’Connor S (2015) <a target="_blank" href="http://jvi.asm.org/content/early/2015/10/16/JVI.02587-15.abstract">Conditional immune escape during chronic SIV infection</a>. *Journal of Virology* **90**(1):545-52.
* Nelson CW, Hughes AL (2015) <a target="_blank" href="http://www.sciencedirect.com/science/article/pii/S1567134814004468">Within-host nucleotide diversity of virus populations: Insights from next-generation sequencing</a>. *Infection, Genetics and Evolution* **30**:1-7.
* Bailey AL, Lauck A, Weiler A, Sibley SD, Dinis JM, Bergman Z, Nelson CW, Correll M, Gleicher M, Hyeroba D, Tumukunde A, Weny G, Chapman C, Kuhn JH, Hughes AL, Friedrich TC, Goldberg TL, O’Connor DH (2014) <a target="_blank" href="http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0090714">High genetic diversity and adaptive potential of two simian hemorrhagic fever viruses in a wild primate population</a>. *PLoS ONE* **9**(3):e90714.
* Wilker PR, Dinis JM, Starrett G, Imai M, Hatta M, Nelson CW, O’Connor DH, Hughes AL, Neumann G, Kawaoka Y, Friedrich TC (2013) <a target="_blank" href="http://www.nature.com/ncomms/2013/131023/ncomms3636/abs/ncomms3636.html?message-global=remove">Selection on haemagglutinin imposes a bottleneck during mammalian transmission of reassortant H5N1 influenza viruses</a>. *Nature Communications* **4**:2636.

## <a name="contact"></a>Contact
Please note that, given the unexpected and unfortunate passing of our friend, mentor, and colleague Dr. Austin L. Hughes, all correspondance should be addressed to Chase W. Nelson at: nelsoncw <**AT**> email <**DOT**> sc <**DOT**> edu.

## <a name="references"></a>References

* Knapp EW, Irausquin SJ, Friedman R, Hughes AL (2011) <a target="_blank" href="http://link.springer.com/article/10.1007%2Fs12686-010-9372-5">PolyAna: analyzing synonymous and nonsynonymous polymorphic sites</a>. *Conserv Genet Resour* **3**:429-431.
* Lynch M, Bost D, Wilson S, Maruki T, Harrison S (2014) <a target="_blank" href="http://gbe.oxfordjournals.org/content/early/2014/04/30/gbe.evu085">Population-genetic inference from pooled-sequencing data</a>. *Genome Biol. Evol.* **6**(5):1210-1218.
* Nei M, Gojobori T (1986) <a target="_blank" href="http://mbe.oxfordjournals.org/content/3/5/418.short?rss=1&ssource=mfc">Simple methods for estimating the numbers of synonymous and nonsynonymous nucleotide substitutions</a>. *Mol Biol Evol* **3**:418-26.
* Nelson CW, Hughes AL (2015) <a target="_blank" href="http://www.sciencedirect.com/science/article/pii/S1567134814004468">Within-host nucleotide diversity of virus populations: Insights from next-generation sequencing</a>. *Infection, Genetics and Evolution* **30**:1-7.

Image Copyright 2015 Elizabeth Ogle
