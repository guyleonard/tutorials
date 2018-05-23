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

`$ls -lath` will give you a listing of your current directory location.

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

Now we can go ahead and run the `makeblastdb` command on our four sequence files:

```bash
makeblastdb -in taxa_one.fasta -dbtype prot -parse_seqids -taxid XXX
makeblastdb -in taxa_two.fasta -dbtype prot -parse_seqids -taxid XXX
makeblastdb -in taxa_three.fasta -dbtype prot -parse_seqids -taxid XXX
makeblastdb -in taxa_four.fasta -dbtype prot -parse_seqids -taxid XXX


