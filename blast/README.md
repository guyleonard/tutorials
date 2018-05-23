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
 * the NCBI Taxonomy DB files from [here](ftp://ftp.ncbi.nlm.nih.gov/blast/db/taxdb.tar.gz)

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
$ makeblastdb -in saccharomyces_cerevisiae.fas -dbtype prot -parse_seqids -taxid 4932
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
$ blastdb_aliastool -title eukaryotes -out eukaryotes -dbtype prot -dblist "arabidopsis_thaliana.fas cyanidioschyzon_merolae.fas entamoeba_histolytica.fas homo_sapiens.fas saccharomyces_cerevisiae.fas trypanosoma_brucei.fas"

Created protein BLAST (alias) database eukaryotes with 190760 sequences
```
This will have created a file `eukaryotes.pal` (a Protein Alias File) for blast use. Inside, it is a simple text file, you will see:

```
#
# Alias file created 05/23/2018 16:20:07
#
TITLE eukaryotes
DBLIST "arabidopsis_thaliana.fas" "cyanidioschyzon_merolae.fas" "entamoeba_histolytica.fas" "homo_sapiens.fas" "saccharomyces_cerevisiae.fas" "trypanosoma_brucei.fas" 
NSEQ 190760
LENGTH 109875585
```
this describes, which files are included in this 'database' and the total number of sequences and total length in base pairs of all those sequences. You will need to keep all of the fasta files, blast db files and .pal files together.

## Taxonomy
You will need to acquire the blast taxonomy db from NCBI and uncompress the file and store them somewhere safe, to do this you can type the following:
```
$ wget ftp://ftp.ncbi.nlm.nih.gov/blast/db/taxdb.tar.gz

$ tar zxvf taxdb.tar.gz

$ mkdir taxonomy
$ cp taxdb.btd taxdb.bti taxonomy
```

After downloading and uncompressing the taxdb.tar.gz from NCBI you will need to let blast know where to look for these files. We can do this with an 'environment variable' much like PATH but in this case called BLASTDB.

```
$ export BLASTDB=`pwd`/taxonomy
```
Now we are set up to blast some sequence data!

## Part Two: Basic BLAST Searches

Now we will explore the `blastp` program and its options.

### blastp
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

#### \-db
This is where you include the database(s) you want to search. If you are only searching one fasta file then you specifiy the full name with extension e.g. `you_file.fasta` but if you are going to use a `.pal` file you do not need to include the extension. The error message is not helpful in this pequliarity.
#### \-query
This is a file that contains either one more more fasta format sequences that you wish to search for in the databases. They need to be in fasta format but do not have to be formatted with the blast tools above.
#### \-out
The name and location of the output file which will contain the results of the BLAST+ search
#### \-evalue
The evalue cut-off you wish to filter your results by, the default is 10 which is very broad!
#### \-outfmt
Specify the type of output you want, the default is '0' and called pairwise (it will be a text version similar to the blast webpage results you are used to) but it is not particularly useful for bioinformatics applications (human readable formats rarely are. We will look at '0' but change the output to '6' or tabular which is much easier to parse quickly.
#### \-max_target_seqs
This is roughly the number of hits you want to have included in your blast output. The default is '500' and so may end up including many paralogues (or erroneous hits if you have a broad e-value). We will limit this to 10 for ease.
#### \-num_threads
This is roughly equivalent to the number of 'cores' your CPU has. It will speed the blast search up if you are able to use more than 1 core. Of course if you are running multiple concurrent blasts, you do not want to specify all the cores on your CPU as they will then have to compete.

We will first run the blast command with the default output options so you can see what it looks like.

```
$ blastp -db arabidopsis_thaliana.fas -query query_one.fas -out query_one_vs_arabidopsis_1e-10.out -evalue 1e-10 -outfmt 0 -max_target_seqs 10 -num_threads 2

Warning: [blastp] The parameter -max_target_seqs is ignored for output formats, 0,1,2,3. Use -num_descriptions and -num_alignments to control output
```
In this case you can ignore the warning. Looking at the file you can see a lot of information in the traditional webpage view of your results. It shows you the alignments and other erroneous information. However, it is not a good format to easily parse with command-line tools. For that we can use the tabulated output option.

```
$ blastp -db homo_sapiens.fas -query query_one.fas -out query_one_vs_homo_1e-10.tab -evalue 1e-10 -outfmt 6 -max_target_seqs 10 -num_threads 2
```

Here we can see the 'top 10' (in this case top 6) hits to our query sequence.
```
$ cat query_one_vs_homo_1e-10.tab

AAH03584.2	NP_000782.1	100.00	187	0	0	1	187	1	187	2e-137	387
AAH03584.2	XP_011510839.1	93.44	183	12	0	5	187	5	187	1e-123	352
AAH03584.2	NP_789785.1	93.44	183	12	0	5	187	5	187	1e-123	352
AAH03584.2	NP_001182572.1	93.44	183	12	0	5	187	5	187	1e-123	352
AAH03584.2	NP_001277283.1	100.00	135	0	0	53	187	1	135	4e-95	278
AAH03584.2	NP_001277286.1	100.00	123	0	0	1	123	1	123	3e-86	255
```

Looking at the first line we can see our query accession `AAH03584.2` has hit the accession `NP_000782.1` in the database. The columns are ordered thus: query id and subject id followed by the percent identity, alignment length      number of mismatches, gap openings, query start position and query end position, the subject start and subject end and finally the e_value and bit_score.

Now let's try the search again but against our 'eukaryotes' database.

```
$ blastp -db eukaryotes -query query_one.fas -out query_one_vs_eukaryotes_1e-10.tab -evalue 1e-10 -outfmt 6 -max_target_seqs 10 -num_threads 2

$ cat query_one_vs_eukaryotes_1e-10.tab
AAH03584.2	NP_000782.1	100.00	187	0	0	1	187	1	187	3e-137	387
AAH03584.2	XP_011510839.1	93.44	183	12	0	5	187	5	187	2e-123	352
AAH03584.2	NP_789785.1	93.44	183	12	0	5	187	5	187	2e-123	352
AAH03584.2	NP_001182572.1	93.44	183	12	0	5	187	5	187	2e-123	352
AAH03584.2	NP_001277283.1	100.00	135	0	0	53	187	1	135	6e-95	278
AAH03584.2	NP_001277286.1	100.00	123	0	0	1	123	1	123	4e-86	255
AAH03584.2	NP_001328947.1	35.39	178	107	3	8	183	69	240	2e-31	121
AAH03584.2	NP_001328950.1	35.39	178	107	3	8	183	22	193	6e-31	120
AAH03584.2	NP_001328948.1	35.39	178	107	3	8	183	22	193	6e-31	120
AAH03584.2	NP_195183.2	35.39	178	107	3	8	183	69	240	6e-31	121
```
We can see the first top 6 hits from the previous search, but now we have some more hits too! But where did they come from? There are three ways to find out:
 1) search through each fasta file manually - slow but easy if you know 'grep'
 2) search the accession on the NCBI website - slow and requires web browser
 3) get BLAST to tell us, but we need the taxonomy dump files - fast (after set up)

Try the command again but this time editing the output format to include the scientific name of the taxa. Here we have indicated that we want format '6' with the standard output 'std' along with the subject's scientific name 'sscinames'.

```
$ blastp -db eukaryotes -query query_one.fas -out query_one_vs_eukaryotes_1e-10.tab -evalue 1e-10 -outfmt '6 std sscinames' -max_target_seqs 10 -num_threads

$ cat query_one_vs_eukaryotes_1e-10.tab

AAH03584.2	NP_000782.1	100.00	187	0	0	1	187	1	187	3e-137	387	Homo sapiens
AAH03584.2	XP_011510839.1	93.44	183	12	0	5	187	5	187	2e-123	352	Homo sapiens
AAH03584.2	NP_789785.1	93.44	183	12	0	5	187	5	187	2e-123	352	Homo sapiens
AAH03584.2	NP_001182572.1	93.44	183	12	0	5	187	5	187	2e-123	352	Homo sapiens
AAH03584.2	NP_001277283.1	100.00	135	0	0	53	187	1	135	6e-95	278	Homo sapiens
AAH03584.2	NP_001277286.1	100.00	123	0	0	1	123	1	123	4e-86	255	Homo sapiens
AAH03584.2	NP_001328947.1	35.39	178	107	3	8	183	69	240	2e-31	121	Arabidopsis thaliana
AAH03584.2	NP_001328950.1	35.39	178	107	3	8	183	22	193	6e-31	120	Arabidopsis thaliana
AAH03584.2	NP_001328948.1	35.39	178	107	3	8	183	22	193	6e-31	120	Arabidopsis thaliana
AAH03584.2	NP_195183.2	35.39	178	107	3	8	183	69	240	6e-31	121	Arabidopsis thaliana
```
That's much better! Now we can see which accession is from which taxa! Why do you think that the percent ID is low for the Arabidopsis hits? Also now you can experiment with the other output options to your heart's content.

Now let's try with our second query file, if you have had a look in this file already you will notice that it has two sequences - you may end up query 10s to 100s (and maybe even 1000s) of sequences against a database - this will let you see how to interpret the results when there are more than 1 queries.

```
$ blastp -db eukaryotes -query query_two.fas -out query_two_vs_eukaryotes_1e-10.tab -evalue 1e-10 -outfmt '6 std sscinames' -max_target_seqs 10 -num_threads

$ cat query_two_vs_eukaryotes_1e-10.tab

AAH03584.2	NP_000782.1	100.00	187	0	0	1	187	1	187	3e-137	387	Homo sapiens
AAH03584.2	XP_011510839.1	93.44	183	12	0	5	187	5	187	2e-123	352	Homo sapiens
AAH03584.2	NP_789785.1	93.44	183	12	0	5	187	5	187	2e-123	352	Homo sapiens
AAH03584.2	NP_001182572.1	93.44	183	12	0	5	187	5	187	2e-123	352	Homo sapiens
AAH03584.2	NP_001277283.1	100.00	135	0	0	53	187	1	135	6e-95	278	Homo sapiens
AAH03584.2	NP_001277286.1	100.00	123	0	0	1	123	1	123	4e-86	255	Homo sapiens
AAH03584.2	NP_001328947.1	35.39	178	107	3	8	183	69	240	2e-31	121	Arabidopsis thaliana
AAH03584.2	NP_001328950.1	35.39	178	107	3	8	183	22	193	6e-31	120	Arabidopsis thaliana
AAH03584.2	NP_001328948.1	35.39	178	107	3	8	183	22	193	6e-31	120	Arabidopsis thaliana
AAH03584.2	NP_195183.2	35.39	178	107	3	8	183	69	240	6e-31	121	Arabidopsis thaliana
AAX78868.1	XP_011775023.1	99.81	527	1	0	1	527	1	527	0.0	1090	Trypanosoma brucei
AAX78868.1	XP_005539048.1	49.08	544	216	12	25	527	12	535	7e-162	477	Cyanidioschyzon merolae
AAX78868.1	NP_195183.2	46.76	556	239	12	3	527	36	565	7e-161	476	Arabidopsis thaliana
AAX78868.1	NP_001328949.1	47.38	534	228	10	21	527	33	540	2e-160	474	Arabidopsis thaliana
AAX78868.1	NP_001328950.1	47.38	534	228	10	21	527	11	518	2e-160	473	Arabidopsis thaliana
AAX78868.1	NP_001328948.1	47.38	534	228	10	21	527	11	518	2e-160	473	Arabidopsis thaliana
AAX78868.1	NP_179230.1	47.18	532	230	9	21	527	14	519	8e-159	469	Arabidopsis thaliana
AAX78868.1	NP_001324593.1	47.18	532	230	9	21	527	14	519	8e-159	469	Arabidopsis thaliana
AAX78868.1	NP_001062.1	60.35	285	112	1	243	527	30	313	6e-124	372	Homo sapiens
AAX78868.1	NP_001324592.1	45.58	419	177	9	21	414	14	406	6e-109	337	Arabidopsis thaliana
```
You'll notice now we have a lot more output! You can see the original search we did (as query_two.fas has the original query sequence and a new one too) and results from a new search. 

## Part 3: Sequence Retrieval 

