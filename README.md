# CompMap
Introduction:

CompMap is a reference-based compression program to speed up read mapping 
to related reference sequences. CompMap is designed to eliminate repeat 
subsequences based on reference-base compression in the input curating 
database, which could contain complete genomes, plasmids, and/or other sequences. 
Particularly, one or multiple sequences in the database are selected to form 
a reference to which the other sequences are aligned. The repeat segments in other 
sequences are then eliminated by recoding only their aligned positions and lengths 
at very little space cost. The non-aligned segments together with the reference are 
considered as the non-redundant representation of the database. NGS short reads are 
then mapped to the representation sequences rather than the whole database. 
The corresponding mapped positions of the short reads to the original database 
could be readily recovered according to the representation sequences and the alignment results.

If you use CompMap, please cite:
Z. Zhu, L. Li, Y. Zhang, Y. Yang, and X. Yang (2014) 
CompMap: a reference-based compression program to speed up read mapping to related reference sequences, 
Bioinformatics, doi: 10.1093/bioinformatics/btu656

Installation:

CompMAP is developed in C and run on 32/64 bit Linux OS. 
A minimum memory of 1GB is suggested for good user experience. 
To install the program, please download and decompress the source code 
and then compile it in a GNU environment equipped with GCC compiler using the following commands: 

make
make clean

The executable file namely ‘compMAP’ will be generated in the same diretory of the source code. 
Make sure a short read mapping tool e.g., BWA or Bowtie 2 has been installed and configured correctly before running CompMap

Command and Options:

A simple example is provided as follows to illustrate the use of CompMap. To map the short reads in file ‘reads.fq’ to the related sequences (in fasta format) stored in the directory ‘db_dir’, the ‘compMAP comp’ command should be first run to compress the sequences and obtain the reusable non-redundant representation of the sequences say ‘ref.fa’:

compMAP comp db_dir ref.fa
Here, the executable compMAP is assumed to be located in the same directory as ‘db_dir’. If not, the exact directory of compMAP should be specified. Afterward, a short read mapping tool like BWA or Bowtie 2 should be called to map the short reads ‘reads.fq’ to ‘ref.fa’:

bwa index ref.fa
bwa mem –a ref.fa reads.fq > aln.sam
or

bowtie2-build ref.fa bt2.index
bowtie2-align –a -x bt2.index –U reads.fq –S aln.sam

The mapping results are output to ‘aln.sam’ in standard SAM format. The compatibility of BWA and Bowtie 2 to CompMap has been tested by the authors. Nevertheless, CompMap can also work together with other short read mapping tools if only they can support input reference sequences in standard FASTA format and output results in standard SAM format.

Finally, the ‘compMAP map’ command should be executed to recover the mapped positions in the original sequences based on ‘aln.sam’ and ‘ref.fa’. 
compMAP map aln.sam ref.fa
In the final output SAM file (SAM Format Specification), when junction alignment is detected, the field POS is set to 0, FLAG is set randomly, and the RNAME is set to ‘junction’. Note that because approximate matching is adopted in the identification of repeats, the final mapping results could have small deviation in POS and CIAGR fields.



COMMANDS AND OPTIONS
comp	compMAP comp <db_dir> [option] <ref.fa>
 	compress the database sequences in ‘db_dir’ directory and output in FASTA format.
 	OPTIONS
 	
-L INT	Set the minimum length of valid repeat sequences to identify in database sequences. (Default: 1000)
-e FLO	Set the mismatch tolerance rate in local alignment. (Default: 0.05)
-N INT	Set the size of prospecting window in local alignment. When a mismatch is detected, the program looks beyond for N more bases. If there are more than N/2 mismatches within these bases, then the matching goes the other direction or terminates. (Default: 10)
-k INT	Set the length of kmers used in local alignment. Here the definition of k is slightly defferent from the paper by excluding the length of the predefined prefixes. For example, if k=8 and prefix=‘CG’, the kmers actually are 10 bp.(Default: 8)
-P <s1, …, s10>	Set the kmer prefixes. The max number of prefixes is 10. For example, ‘compMAP comp db_dir –P CG,AC,TA’ assign three prefixes namely ‘CG’,‘AC’, and ‘TA’. (Default: ‘CG’).
-R INT <f1, f2, …>[<@textfile>]	Specify the reference sequences for database compression. 0: randomly select a sequence as reference; 1: select the longest sequence as reference; 2: specify the file names of reference sequences. (Default: 1).
For example:
‘compMAP comp db_dir –R 2 f1.fasta,f2.fasta ref.fa’ specifies ‘f1.fasta’ and ‘f2.fasta’ as the reference for compression.
‘compMAP comp db_dir –R 2 @textfile ref.fa’ specifies references in a text file ‘textfile’ which could contain the following contents: 
f1.fasta
f2.fasta
f3.fasta
…
 	
Examples of uisng ‘compMAP comp’

‘compMAP comp db_dir ref.fa’
‘compMAP comp db_dir –R 2 f1.fasta ref.fa’
‘compMAP comp db_dir –L 200 -e 0.03 –N 16 –k 10 –P CG, AT –R 2 @text ref.fa’
Command ‘compMAP comp’ generates three files namely ‘ref.fa’, ‘ref.fa.mat’, and ‘ref.fa.log’. ‘ref.fa’ contains the non-redundant representative sequences of the database. ‘ref.fa.mat’ and ‘ref.fa.log’ store the positions of all valid repeat segments in all non-reference sequences.


map	compMAP map [ option ] <aln.sam> <ref.fa>
 	Recover the mapped positions in the original sequences based on ‘aln.sam’ and ‘ref.fa’. ‘aln.sam’ is the output of the alignment tool such as BWA and Bowtie 2; ‘ref.fa’ is the output of ‘compMAP comp...’ Make sure ‘ref.fa.mat’ and ‘ref.fa.log’ are located in the same directory as ‘ref.fa’.
 	OPTIONS
 	
-F	Used to speed up the running of the program if fixed-length short reads are input.
-D	Display the information of the mapped positions including the number of successful, failed and junction alignments.
 	
Examples of using ‘compMAP map’

‘compMAP map –D aln.sam ref.fa’
‘compMAP map –F aln.sam ref.fa’


tools :
SeqCmp
to find a good reference(s)

input './SeqCmp'  for help
