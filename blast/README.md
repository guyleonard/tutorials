# A Blast Tutorial

In this exercise you will learn how to:
 * create a blast database
 * create a blast alias database
 * use blastp to:
   * search a pre-built database for a query sequence
   * interpret the blast output and formats
   * retrieve the sequence results for top hits
 * other?

# Prerequisites

You will need:

 * the BLAST+ executables available from [here](https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=Download)
 * an example query sequence (query.fasta)
 * a set of fasta format predicted proteins from several taxa (taxa_one.fasta, taxa_two.fasta, taxa_three.fasta, taxa_four.fasta)
 * the NCBI Taxonomy Dump files from [here](ftp://ftp.ncbi.nih.gov/pub/taxonomy/taxdump.tar.gz)

You will need to clone this directory to your local hardrive/user space and make sure the blast binaries/executables are accessible in your command line. If you are using the "roger" servers (ramjet, romanoff, roadrunner) they should all be available at the command line or in modules.

When you see a $ sign and a program name after it, you should go ahead and run that command in your terminal (try not to copy and paste!). For example:

`$ ls -lath` will give you a listing of your current directory location.

# Part One: Creating a BLAST Database

## makeblastdb

The program `makeblastdb` (formerly `formatdb`) is used to 'format' your fasta formatted sequence file in to something easily readable for the blast+ programs. It has several options, go ahead and look at them now.

```bash
$ makeblastdb -h

USAGE
  makeblastdb [-h] [-help] [-in input_file] [-input_type type]
    -dbtype molecule_type [-title database_title] [-parse_seqids]
    [-hash_index] [-mask_data mask_data_files] [-mask_id mask_algo_ids]
    [-mask_desc mask_algo_descriptions] [-gi_mask]
    [-gi_mask_name gi_based_mask_names] [-out database_name]
    [-max_file_sz number_of_bytes] [-logfile File_Name] [-taxid TaxID]
    [-taxid_map TaxIDMapFile] [-version]

DESCRIPTION
   Application to create BLAST databases, version 2.2.31+
```

For this tutorial we will only look at the '-in' '-dbtype' 'parse_seqids' and '-taxid' options.

### \-in
This is your input file, it will be the fasta formatted file of your sequences that you wish to search.
### \-dbtype
Very simply: either 'nucl' for nucleotide or 'prot' for protein sequences.
### \-parse_seqids
Each sequence in a fasta format file has a 'header' containing information such as accession numbers. NCBI has it's own strict format, and for blast to process this you must turn this option on. You can leave it out for files that you have created yourself that don't follow NCBI's or a standard format.
### \-taxid
This associates your set of sequences to the NCBI Taxa ID number of your organism - it is useful in downstream analyses to recover taxonomic information automatically. You do not have to include this option however.

Now we can go ahead and run the `makeblastdb` command on our four sequence files (I have only included one example of the output message that you will receive):

```bash
$ makeblastdb -in arabidopsis_thaliana.fas -dbtype prot -parse_seqids -taxid 3702
$ makeblastdb -in cyanidioschyzon_merolae.fas -dbtype prot -parse_seqids -taxid 45157
$ makeblastdb -in entamoeba_histolytica.fas -dbtype prot -parse_seqids -taxid 5759
$ makeblastdb -in homo_sapiens.fas -dbtype prot -parse_seqids -taxid 9606
$ makeblastdb -in trypanosoma_brucei.fas -dbtype prot -parse_seqids -taxid 5691

Building a new DB, current time: 05/23/2018 14:29:34
New DB name:   /home/cs02gl/Dropbox/git/tutorials/blast/trypanosoma_brucei.fas
New DB title:  trypanosoma_brucei.fas
Sequence type: Protein
Keep Linkouts: T
Keep MBits: T
Maximum file size: 1000000000B
Adding sequences from FASTA; added 9822 sequences in 0.574554 seconds.
```
You will also notice that this command has created several new files with similar names to your original input files:

```bash
arabidopsis_thaliana.fas.phr
arabidopsis_thaliana.fas.pin
arabidopsis_thaliana.fas.pog
arabidopsis_thaliana.fas.psd
arabidopsis_thaliana.fas.psi
arabidopsis_thaliana.fas.psq
```
You don't need to know what each of these do, but they are needed by blast to search the database, you don't have to specify them in the blast command, it will automatically look for them wherever you fasta file happens to be. You will need to re-run the above commands if you make any changes to the fasta files.

You can now go ahead and run individual blast queries against these database, skip to Part Two or continue on to group all of these databases in to one database.

## blastdb_aliastool
Often you will want to blast several taxa at one time but keep their sequences separate from one another. This tool allows you to do that by creating an 'alias' file that tells blast to use several databases at once. It does a whole bunch of other things too, but they're for another day. It has a similar set of input option as above, but we'll just jumpt straight in to the command.

```bash
$ blastdb_aliastool -title eukaryotes -out eukaryotes -dbtype prot -dblist "arabidopsis_thaliana.fas cyanidioschyzon_merolae.fas entamoeba_histolytica.fas homo_sapiens.fas trypanosoma_brucei.fas"

Created protein BLAST (alias) database eukaryotes with 184758 sequences
```
This will have created a file `eukaryotes.pal` (a Protein Alias File) for blast use. Inside, it is a simple text file, you will see:

```
#
# Alias file created 05/23/2018 14:45:50
#
TITLE eukaryotes
DBLIST "arabidopsis_thaliana.fas" "cyanidioschyzon_merolae.fas" "entamoeba_histolytica.fas" "homo_sapiens.fas" "trypanosoma_brucei.fas"
NSEQ 184758
LENGTH 106944079
```
this describes, which files are included in this 'database' and the total number of sequences and total length in base pairs of all those sequences. You will need to keep all of the fasta files, blast db files and .pal files together.

## Part Two: Basic BLAST Searches

Now we will explore the `blastp` program and its options.

```
$ blastp -h

USAGE
  blastp [-h] [-help] [-import_search_strategy filename]
    [-export_search_strategy filename] [-task task_name] [-db database_name]
    [-dbsize num_letters] [-gilist filename] [-seqidlist filename]
    [-negative_gilist filename] [-entrez_query entrez_query]
    [-db_soft_mask filtering_algorithm] [-db_hard_mask filtering_algorithm]
    [-subject subject_input_file] [-subject_loc range] [-query input_file]
    [-out output_file] [-evalue evalue] [-word_size int_value]
    [-gapopen open_penalty] [-gapextend extend_penalty]
    [-qcov_hsp_perc float_value] [-max_hsps int_value]
    [-xdrop_ungap float_value] [-xdrop_gap float_value]
    [-xdrop_gap_final float_value] [-searchsp int_value]
    [-sum_stats bool_value] [-seg SEG_options] [-soft_masking soft_masking]
    [-matrix matrix_name] [-threshold float_value] [-culling_limit int_value]
    [-best_hit_overhang float_value] [-best_hit_score_edge float_value]
    [-window_size int_value] [-lcase_masking] [-query_loc range]
    [-parse_deflines] [-outfmt format] [-show_gis]
    [-num_descriptions int_value] [-num_alignments int_value]
    [-line_length line_length] [-html] [-max_target_seqs num_sequences]
    [-num_threads int_value] [-ungapped] [-remote] [-comp_based_stats compo]
    [-use_sw_tback] [-version]

DESCRIPTION
   Protein-Protein BLAST 2.2.31+

Use '-help' to print detailed descriptions of command line arguments
```

Many of these you will not use or will only need in specific circumstances.

### \-db
This is where you include the database(s) you want to search. If you are only searching one fasta file then you specifiy the full name with extension e.g. `you_file.fasta` but if you are going to use a `.pal` file you do not need to include the extension. The error message is not helpful in this pequliarity.
### \-query
This is a file that contains either one more more fasta format sequences that you wish to search for in the databases. They need to be in fasta format but do not have to be formatted with the blast tools above.
### \-out
The name and location of the output file which will contain the results of the BLAST+ search
### \-evalue
The evalue cut-off you wish to filter your results by, the default is 10 which is very broad!
### \-outfmt
Specify the type of output you want, the default is '0' and called pairwise (it will be a text version similar to the blast webpage results you are used to) but it is not particularly useful for bioinformatics applications (human readable formats rarely are. We will look at '0' but change the output to '6' or tabular which is much easier to parse quickly.
### \-max_target_seqs
This is roughly the number of hits you want to have included in your blast output. The default is '500' and so may end up including many paralogues (or erroneous hits if you have a broad e-value). We will limit this to 10 for ease.
### \-num_threads
This is roughly equivalent to the number of 'cores' your CPU has. It will speed the blast search up if you are able to use more than 1 core. Of course if you are running multiple concurrent blasts, you do not want to specify all the cores on your CPU as they will then have to compete.

```
$ blastp -db arabidopsis_thaliana.fas -query query_one.fas -out query_one_vs_arabidopsis_1e-10.out -evalue 1e-10 -outfmt 6 -max_target_seqs 10 -num_threads 2

$ cat query_one_vs_arabidopsis_1e-10.out

AAH03584.2	NP_001328947.1	35.39	178	107	3	8	183	69	240	4e-32	121
AAH03584.2	NP_001328950.1	35.39	178	107	3	8	183	22	193	1e-31	120
AAH03584.2	NP_001328948.1	35.39	178	107	3	8	183	22	193	1e-31	120
AAH03584.2	NP_195183.2	35.39	178	107	3	8	183	69	240	1e-31	121
AAH03584.2	NP_001328949.1	35.39	178	107	3	8	183	44	215	1e-31	120
AAH03584.2	NP_001324592.1	35.56	180	108	3	8	185	25	198	2e-30	116
AAH03584.2	NP_179230.1	35.56	180	108	3	8	185	25	198	3e-30	116
AAH03584.2	NP_001324593.1	35.56	180	108	3	8	185	25	198	3e-30	116
AAH03584.2	NP_001324557.1	30.69	189	109	5	4	183	23	198	1e-22	95.5
AAH03584.2	NP_179750.1	30.69	189	109	5	4	183	23	198	2e-22	95.1
```
Here we can see the top 10 hits to our query sequence.





