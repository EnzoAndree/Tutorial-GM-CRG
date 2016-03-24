![banner](https://raw.githubusercontent.com/microgenomics/tutorials/master/img/microgenomics.png)
# A pipeline for the de novo assembly of paired-end Illumina reads using SPAdes
-------------------------
#### This pipeline takes 5 hours in a MacBook Air 1,3 GHz Intel Core i5 4 GB 1600 MHz DDR3 with reads of PUC64 and need ≈5 Gb in your hard drive

This demo relies on four pieces of software, *SRA toolkit*, *PEAR*, *SPAdes* and *Prokka* so please remember to cite them if you end up publishing results obtained with these tools.

## Obtaining data

If you want know how install SRA toolkit visit this [tutorial](https://github.com/microgenomics/tutorials/blob/master/sra.md).

Once you have instaled, you must dowload the fastq for this tutorial. Just type in the terminal:

	fastq-dump --split-files --accession SRR1776954 --outdir demospades

Move to folder demospades, there you have 2 fastq (~ 4,73 GB) from *Mycoplasma mycoides subsp. mycoides* reported in *High quality draft genomes of the Mycoplasma mycoides subsp. mycoides challenge strains Afadé and B237* [PMID: 26516405](http://www.ncbi.nlm.nih.gov/pmc/articles/pmid/26516405/).

![genomes](https://raw.githubusercontent.com/eandree/TutorialDeNovoAssembly/master/img/demospades.png)

## Genome assembly

In the fastq we have the rawdata of the sequencer, in this demo is from a Illumina HiSeq 2000. First of all, we need to analyze the quality of our reads. For this we going to use FastQC, a multiplataform wrote in java. In mac you have to download the file whit extension .DMG from [here](http://www.bioinformatics.babraham.ac.uk/projects/download.html#fastqc).

You have to open the file *fastqc_v0.11.5.dmg* and move to folder *Applications*

![FastQC install](https://raw.githubusercontent.com/eandree/TutorialDeNovoAssembly/master/img/install-fastqc.png)

Open FastQC and in the top menu clik on "File > Open" and find the fastq files, select and open it.

![FastQC open](https://raw.githubusercontent.com/eandree/TutorialDeNovoAssembly/master/img/open-menu.png)

When FastQC finish the analysis, you will see something like this. 

![FastQC open](https://raw.githubusercontent.com/eandree/TutorialDeNovoAssembly/master/img/fastqc.png)

You can click on any submenu and see the stats. Primarily the 2º submenu is very important, this menu give a general view of quality of all your read. In summary, if the check is green, it's ok, else, it is not ok. For more information you can see de [documentation](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/).

Either way, in the next step, we going to filter and merge the reads by quality whit PEAR.

## Filter and merge of the reads

PEAR is a program write in C, so this step will be very fast, first of all we need download and install the program following [this tutorial](http://sco.h-its.org/exelixis/web/software/pear/doc.html#installing) in summary you must type this:

	mkdir $HOME/pear
	git clone https://github.com/xflouris/PEAR.git
	cd pear
	./autogen.sh
	./configure --prefix=$HOME/pear
	make
	make install
	echo 'export PATH=/Users/Enzo/pear/bin:${PATH}' >> .bash_profile
	source .bash_profile

**NOTE 1: You must change the username "Enzo" by the name of your home.**
**NOTE 2: You must to have instaled git in your Mac**
**NOTE 3: If you can't install from the source, in this [link](addlink) i upload the binary.**


Now go to folder demospades, in my case i must type in the terminal this.

	cd ~/demospades

Because the folder demospades is in my home directory. Next, you must type this in the terminal:

	pear -f SRR1776954_1.fastq -r SRR1776954_2.fastq -v 20 -q 20 -u 0 -j 4 -o merged

The output is this:

![FastQC open](https://raw.githubusercontent.com/eandree/TutorialDeNovoAssembly/master/img/fastqc.png)


## Determining the pangenome

Let's put all the .gff files in the same folder (e.g., `./gff`) and run *Roary*
		
		roary -f ./demo -e -n -v ./gff/*.gff

Roary will get all the coding sequences, convert them into protein, and create pre-clusters. Then, using BLASTP and MCL, *Roary* will create clusters, and check for paralogs. Finally, *Roary* will take every isolate and order them by presence/absence of orthologs. The summary output is present in the `summary_statistics.txt` file. In our case, the results are as follows:

Genes| Number
|----|-------|
|Core genes (99% <= strains <= 100%)|	2010|
|Soft core genes (95% <= strains < 99%)| 0|
|Shell genes (15% <= strains < 95%)| 2454|
|Cloud genes (0% <= strains < 15%)|	0|
|Total genes|	4464|

Additionally, *Roary* produces a `gene_presence_absence.csv` file that can be opened in any spreadsheet software to manually explore the results. In this file, you will find information such as gene name and gene annotation, and, of course, whether a gene is present in a genome or not.

We already have a phylogeny that represents the evolutionary history of this six isolates, where they form clades according to their genotype, i.e., type I isolates together, and so on.

![phylogeny](https://raw.githubusercontent.com/microgenomics/tutorials/master/img/core_gene_alignment.tre.png)

*Roary* comes with a python script that allows you to generate a few plots to graphically assess your analysis output. Try issuing the following command:

		python roary_plots.py core_gene_alignment.nwk gene_presence_absence.csv

You should get three files: a pangenome matrix, a frequency plot, and a pie chart. 

![matrix](https://raw.githubusercontent.com/microgenomics/tutorials/master/img/pangenome_matrix.png)
![frequency](https://raw.githubusercontent.com/microgenomics/tutorials/master/img/pangenome_frequency.png)
![pie](https://raw.githubusercontent.com/microgenomics/tutorials/master/img/pangenome_pie.png)

## Pangenome sequence analysis
We have already Genome annotation and Pangenome analysis, but if you wanna know the sequence of a gene in particular in the pangenome you have to search by your own the sequence in the .ffn files. To avoid this inconvenient, Enzo Guerrero-Araya wrote a script in Python3 that make csv files of all loci in the pangenome. The csv's files can be imported to a database like Sqlite3.

Let's put all the .ffn files in the same folder (e.g., `./ffn`) and run [*GeneratorDB.py*](https://raw.githubusercontent.com/jefesin/tutorials/patch-1/GeneratorDB.py) in the same directory where is the `gene_presence_absence.csv` file.
		
	python3 GeneratorDB.py ffn

The script in this version will generate 4 csv files:

Files| Description
|---|---|
|genomas_locus.csv|It contains 2 columns [name of genome, name of locus]|
|pangenoma.csv|It contains all the information of the annotation that Roary reported in the `gene_presence_absence.csv` file|
|pangenoma_locus.csv|It contains 2 columns [name of gene, name of locus]|
|locus_sequence.csv|It contains 2 columns [name of locus, nucleotide sequence]|

Now we have all csv files for make our own database, in terminal type:

	sqlite3 database.sqlite

In the sqlite3 prompt type:
	
	create table genomas_locus (cod text, locus text);
	create table pangenoma (gene text, non_unique_gene_name text, annotation text, no_isolates integer, no_sequences integer, avg_sequences_per_isolate integer, genome_fragment integer, order_within_fragment integer, accessory_fragment integer, accessory_order_with_fragment integer, qc text, min_group_size_nuc integer, max_group_size_nuc integer, avg_group_size_nuc integer);
	create table pangenoma_locus (gene text, locus text);
	create table locus_sequence (locus text, sequence text);
	
	.separator '|'
	.import genomas_locus.csv genomas_locus
	.import pangenoma.csv pangenoma
	.import pangenoma_locus.csv pangenoma_locus
	.import locus_sequence.csv locus_sequence
	
	create index genomas_locus_index on genomas_locus(cod, locus);
	create index pangenoma_index on pangenoma(gene, non_unique_gene_name, annotation, no_isolates, no_sequences, avg_sequences_per_isolate, genome_fragment, order_within_fragment, accessory_fragment, accessory_order_with_fragment, qc, min_group_size_nuc, max_group_size_nuc, avg_group_size_nuc);
	create index pangenoma_locus_index on pangenoma_locus(gene, locus);
	create index locus_sequence_index on locus_sequence(locus, sequence);

Now just we have to join tables with sql sentences like:

	select '>'|| cod || '|' || locus_sequence.locus || '|' || pangenoma.gene || x'0a' || sequence
	from locus_sequence
	inner join pangenoma_locus on locus_sequence.locus = pangenoma_locus.locus
	inner join pangenoma on pangenoma_locus.gene = pangenoma.gene
	inner join genomas_locus on locus_sequence.locus = genomas_locus.locus
	where pangenoma.gene = 'tetC';
	
	>GCA_000008285_01152016|GCA_000008285_02480|tetC
	ATGGAAAAGAAGCGGACTCGGGCAGAAGAATTAGGAATAACTAGAAGAAAAATTTTGGATACAGCACGTGATTTATTTATGGAAAAGGGTTACCGGGCAGTTTCAACAAGAGAAATAGCTAAAATTGCCAACATTACCCAACCGGCACTATATCACCACTTTGAAGATAAAGAATCCCTATATATTGAAGTGGTTCGTGAATTGACGCAAAATATCCAAGTGGAAATGCATCCAATTATGCAAGTGACCAAAGCAAAAGAAGAACAATTACATGATATGTTAATTATGTTAATTGAGGAACATCCAACCAATATTCTATTAATGATTCACGATATTCTTAATGAAATGAAACCAGAAAATCAATTTTTACTTTATAAATTATGGCAAAAAACCTATTTGGAACCATTTCAACTATTTTTTGAGCGTCTAGAAAATGCTGGCGAATTGCGTGATGGTGTCAGTGCTGAGACTGCTGCGAGATACTGTTTGTCCACTATTAGCCCTCTTTTTTCTGGGAAAGGCAGCTTTGCGCAAAAGCAAACGACTACAGAACAAATTGATGAATTAATCAACTTAATGATGTTTGGTATATGTAAAAAAGAGGTATAA
	>GCA_000021185_01152016|GCA_000021185_00131|tetC
	ATGGAAAAGAAGCGGACTCGAGCAGAAGAATTAGGAATAACCAGAAGGAAAATCCTTGATACAGCAAGGGATTTATTTATGGAAAAAGGGTACCGGGCAGTCTCGACAAGAGAAATTGCTAAAATTGCCAAAATTACCCAACCAGCACTTTATCACCATTTTGAAGATAAAGAATCACTTTATATTGAAGTAGTTCGTGAATTGACGCAAAATATTCAAGTGGAAATGCACCCAATTATGCAAACGAGCAAAGCAAAAGAAGAACAACTGCATGATATGTTAATCATGTTAATTGAGGAGCATCCAACCAATATTCTGCTAATGATTCATGATATTCTTAATGAAATGAAGCCAGAAAATCAATTTTTACTTTATAAATTGTGGCAAAAAACCTATTTAGAACCATTTCAAGACTTTTTTGAGCGATTAGAAAATGCTGGCGAATTGCGTGATGGTATCAGTGCTGAGACCGCTGCGAGATACTGTTTATCCACTATTAGCCCGCTTTTTTCAGGGAAAGGTAGCTTTGCGCAAAAGCAAACGACTACAGAACAAATCGATGAATTAATCAACTTAATGATGTTTGGCATATGTAAAAAAGAGGTATAA
	>GCA_000026705_01152016|GCA_000026705_02479|tetC
	ATGGAAAAGAAGCGGACTCGGGCAGAAGAATTAGGAATAACTAGAAGAAAAATTTTGGATACAGCACGTGATTTATTTATGGAAAAGGGTTACCGGGCAGTTTCAACAAGAGAAATAGCTAAAATTGCCAACATTACCCAACCGGCACTATATCACCACTTTGAAGATAAAGAATCCCTATATATTGAAGTGGTTCGTGAATTGACGCAAAATATCCAAGTGGAAATGCATCCAATTATGCAAGTGACCAAAGCAAAAGAAGAACAATTACATGATATGTTAATTATGTTAATTGAGGAACATCCAACCAATATTCTATTAATGATTCACGATATTCTTAATGAAATGAAACCAGAAAATCAATTTTTACTTTATAAATTATGGCAAAAAACCTATTTGGAACCATTTCAACTATTTTTTGAGCGTCTAGAAAATGCTGGCGAATTGCGTGATGGTGTCAGTGCTGAGACTGCTGCGAGATACTGTTTGTCCACTATTAGCCCTCTTTTTTCTGGGAAAGGCAGCTTTGCGCAAAAGCAAACGACTACAGAACAAATTGATGAATTAATCAACTTAATGATGTTTGGTATATGTAAAAAAGAGGTATAA
	>GCA_000168635_01152016|GCA_000168635_02549|tetC
	ATGGAAAAGAAGCGGACTCGAGCAGAAGAATTAGGAATAACTAGAAGAAAAATTTTGGATACAGCACGTGATTTATTTATGGAAAAGGGTTACCGGGCAGTTTCTACAAGAGAAATAGCTAAAATTGCTAACATTACCCAACCGGCACTTTATCATCACTTTGAAGATAAAGAATCCCTATATATTGAAGTGGTTCGTGAATTGACGCAAAATATCCAGGTGGAAATGCATCCAATTATGCAAACGAACAAAGCAAAAGAAGAACAATTACATGATATGTTAATTATGTTAATTGAGGAACATCCCACCAATATTCTATTAATGATTCACGATATTCTTAATGAAATGAAACCAGAGAATCAATTTTTACTTTATAAATTATGGCAAAAAACCTATTTAGAACCATTTCAACAATTTTTTGAGCGTCTAGAAAATGCTGGTGAATTGCGTAATGGTATCAGTGCTGAGACCGCTGCAAGATACTGTTTGTCCACTATTAGCCCTCTTTTTTCAGGGAAAGGTAGCTTTGCGCAAAAGCAAACGACTACAGAACAAATCGATGAATTAATCAACTTAATGATGTTTGGCATATGTAAAAAAGAGGTATAA
	>GCA_000168815_01152016|GCA_000168815_01572|tetC
	ATGGAAAAGAAGCGGACTCGGGCAGAAGAATTAGGAATAACTAGAAGAAAAATTTTGGATACAGCACGTGATTTATTTATGGAAAAGGGTTACCGGGCAGTTTCAACAAGAGAAATAGCTAAAATTGCCAACATTACCCAACCGGCACTATATCACCACTTTGAAGATAAAGAATCCCTATATATTGAAGTGGTTCGTGAATTGACGCAAAATATCCAAGTGGAAATGCATCCAATTATGCAAGTGACCAAAGCAAAAGAAGAACAATTACATGATATGTTAATTATGTTAATTGAGGAACATCCAACCAATATTCTATTAATGATTCACGATATTCTTAATGAAATGAAACCAGAAAATCAATTTTTACTTTATAAATTATGGCAAAAAACCTATTTGGAACCATTTCAACTATTTTTTGAGCGTCTAGAAAATGCTGGCGAATTGCGTGATGGTGTCAGTGCTGAGACTGCTGCGAGATACTGTTTGTCCACTATTAGCCCTCTTTTTTCTGGGAAAGGCAGCTTTGCGCAAAAGCAAACGACTACAGAACAAATTGATGAATTAATCAACTTAATGATGTTTGGTATATGTAAAAAAGAGGTATAA
	>GCA_000196035_01152016|GCA_000196035_02552|tetC
	ATGGAAAAGAAGCGGACTCGAGCAGAAGAATTAGGAATAACTAGAAGAAAAATTTTGGATACAGCACGTGATTTATTTATGGAAAAGGGTTACCGGGCAGTTTCTACAAGAGAAATAGCTAAAATTGCCAACATTACCCAACCGGCACTGTATCATCACTTTGAAGATAAAGAATCCCTATATATTGAAGTGGTTCGTGAATTGACGCAAAATATCCAGGTGGAAATGCATCCAATTATGCAAACGAACAAAGCAAAAGAAGAACAATTACATGATATGTTAATTATGTTAATTGAGGAACATCCCACCAATATTCTATTAATGATTCACGATATTCTTAATGAAATGAAACCAGAGAATCAATTTTTACTTTATAAATTATGGCAAAAAACCTATTTAGAACCATTTCAACAATTTTTTGAGCGTCTAGAAAATGCTGGTGAATTGCGTAATGGTATCAGTGCTGAGACCGCTGCAAGATACTGTTTGTCCACTATTAGCCCTCTTTTTTCAGGGAAAGGTAGCTTTGCGCAAAAGCAAACGACTACAGAACAAATCGATGAATTAATCAACTTAATGATGTTTGGCATATGTAAAAAAGAGGTATAA

And thats its all. we get all sequences in fasta format of tetC gene.
 

## Citation

Seemann T.  
*Prokka: rapid prokaryotic genome annotation*  
**Bioinformatics** 2014 Jul 15;30(14):2068-9.   
[PMID:24642063](http://www.ncbi.nlm.nih.gov/pubmed/24642063)  

Andrew J. Page, Carla A. Cummins, Martin Hunt, Vanessa K. Wong, Sandra Reuter, Matthew T. G. Holden, Maria Fookes, Daniel Falush, Jacqueline A. Keane, Julian Parkhill.   
*Roary: Rapid large-scale prokaryote pan genome analysis*  
**Bioinformatics** 2015 Jul 20. pii: btv421  
[PMID: 26198102](http://www.ncbi.nlm.nih.gov/pubmed/26198102)

