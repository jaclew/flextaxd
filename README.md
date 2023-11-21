# Flexible Taxonomy Databases (FlexTaxD)

Flexible Taxonomy Databases - A cross platform tool for customization and merging of various taxonomic classification sources.

New to FlexTaxD? --> see [wiki](https://github.com/FOI-Bioinformatics/flextaxd/wiki)


Supported input formats in version later than v0.4.2: For details see [Formats-wiki](https://github.com/FOI-Bioinformatics/flextaxd/wiki/2a.-File-formats)
* NCBI
* TSV
* MP-style
    * GTDB
    * QIIME
    * SILVA
    * CanSNPer

Supported database build programs
* kraken2
* ganon
* krakenuniq
* centrifuge

The flextaxd (flextaxd) script allows customization of databases from several sources (See supported source formats above) and supports export functions into NCBI formatted names and nodes.dmp files as well as a standard tab separated file (or a selected separation).
### Create database
* Create your database from source files --taxonomy_file
* and select --taxonomy_type [supported formats]

### Modify Databases
* Modify taxonomy (--mod_file/----mod_database)
    * Modify taxonomy tree from selected node (--parent) Required for modification
    * If the new database contains edges that are obsolete use --replace to remove any existing links
    * Annotate genomes to database using genomeid2taxid
* Clean up tree, if your tree have many non annotated nodes it may be worthwile removing those from the database (for speed) use --clean_database to remove all non annotated nodes.

### Output options
* To export the database into a file use --dump
    * --dbprogram          Select the required output format for downstream programs [supported_programs]
    * --dump_prefix        Change prefix on outputfiles from names,nodes)
    * --dump_sep           Set output separator default (NCBI \t|\t) unless --dump_mini is used (default \t sep)
    * --dump_descriptions  Dump node names to file instead of node ids

All data is kept in a sqlite3 database (.ftd by default) and can be dumped into NCBI formatted names and nodes.dmp (--dump) or tab separated files (--dump_mini, --dump_descriptions). The TSV dump format is similar to the NCBI dump except that it contains a header (parent<tab>child), has parent on the left and only uses tab to separate each column (not \<tab\>|\<tab\>).

# Reqirements
```
Python >=3.6
## additional requirements depending on executed functions
ncbi-genome-download - to download additional genomes from NCBI
## Visualisation
required for visualisation (plain newick format can be printed to stdout without biopython).
* biopython     - required for newick_vis
* matplotlib    - required for tree
* inquirer      - required when multiple parents are present
## Database program requirement if create_database is executed
kraken2
krakenuniq
ganon
centrifuge
```

# Installation
```
## Using conda
Install mamba in your base conda environment. 
```
conda install mamba -n base -c conda-forge
```

We will then use mamba to create a self-contained conda environment with flextaxd:

```
mamba create -c conda-forge -c bioconda -n flextaxd flextaxd d
```

(For user new to conda please see the [conda guide])


## Manual install python - download git release https://github.com/FOI-Bioinformatics/flextaxd and run
python setup.py install
```
# Usage

Read more about flextaxd on Wiki
-> https://github.com/FOI-Bioinformatics/flextaxd/wiki

## Quick tutorial

#### Save the default FlexTaxD into a custom taxonomy database
```
flextaxd --taxonomy_file taxonomy.tsv --database .ftd
```

### Write the database into NCBI formatted nodes and names.dmp
```
flextaxd --dump
```

### Statistics
Print statistics
--stats will print the number of nodes links and the number of annotated genomes.

### Optional parameters
Use the --help option for a complete list of parameters
```
flextaxd --help
```

## Formats
The different formats supported and used by FlexTaxD are described in the wiki
-> https://github.com/FOI-Bioinformatics/flextaxd/wiki/Formats

## Modify your database
The database update function can use either a previously built flextaxd database or directly through a TAB separated text file with headers (parent, child, (level))). Using the --parent parameter, all nodes/edges subsequent to that parent will be added (or can replace an existing node see options) with the links supplied. The parent node must exist in the database/tables and must have the same name (ex "<i>Francisella tularensis</i>"). Using the (--replace) parameter all children in the old database under the given parent will be removed, if you only want to replace for example <i>Francisella tularensis</i> be sure not to choose <i>Francisella</i> as parent.

#### Modify the database and add sub-species specifications (for <i>Francisella tularensis</i>)
```
flextaxd --mod_file custom_tree_file.txt --parent "Francisella tularensis" --genomeid2taxid custom_genome_annotations.txt
```


## One liner version
Create, modify and dump your database (observe modification can only be done simultanously when GTDB is the source, otw the genomeid2taxid is occupied with other nessesary files)
Parent gives the name of the node where the modification should happen --replace will remove all child nodes of the given parent.
```
flextaxd --taxonomy_file taxonomy.tsv --mod_file custom_modification.txt --genomeid2taxid custom_genome_annotations.txt --parent "Francisella tularensis" --dump
```

#####
#    Create a kraken database
#####

Finally there is a quick option to create a kraken2 or a krakenuniq database using your custom taxonomy database by only supplying genome names matching your annotation table given as --genomeid2taxid
Requirements: kraken2 or krakenuniq needs to be installed, For the FlexTaxD standard database, source data from genbank or refseq is required (for genomeid2taxid match)

Note: If your genome names are different, you can create a custom genome2taxid file and import into your database to match the names of your genome fasta/fa please avoid naming custom fasta files fna as this will be used to detect if genome names are formatted using refseq/genbank naming. Note that the refseq/genbank genomefiles needs to be gzipped (and end with .gz) in their stored location.

### Create Kraken DB
First dump your FlexTaxDatabase into names.dmp and nodes.dmp (default) if that was not already done.
```
flextaxd --dump -o NCBI_database
```

Add genomes to a kraken database <my_custom_krakendb> (and create the kraken database using --create_db) -o must be set to where the database names and nodes were dumped (NCBI_database)
```
flextaxd-create --kraken_db path/to/my_custom_krakendb --genomes_path path/to/genomes/folder --create_db --krakenversion kraken2 -o NCBI_database
```

The script will find all custom fasta, fa and refseq/genbank fna files in the given path and then add them to the krakendb, if --create_db parameter is given
the script will execute the kraken-build --build command.



#####
#    Customize the NCBI taxonomy tree
#####

For a more in depth tutorial on how to merge a database onto the NCBI taxonomy see wiki
-> https://github.com/FOI-Bioinformatics/flextaxd/wiki/Walkthrough---merge-NCBI-with-GTDB


## Quick guide
The most common database to start with is the NCBI taxonomy tree, however there are many known caveats to the NCBI tree in particular in the Bacterial kingdom,
FlexTaxD allows modifications of the NCBI taxonomy by replacing nodes with correct structures.

Creating a custom taxonomy database using the NCBI taxonomy tree instead of FlexTaxD as base
```
Required files from NCBI (ftp://ftp.ncbi.nlm.nih.gov/pub/taxonomy):
from taxdmp.zip
    names.dmp
    nodes.dmp
nucl_gb.accession2taxid.gz (ftp://ftp.ncbi.nlm.nih.gov/pub/taxonomy/accession2taxid/nucl_gb.accession2taxid.gz)
```
### Create a custom taxonomy database from the NCBI taxonomy
```
flextaxd
    --taxonomy_file taxonomy/nodes.dmp  ## Path to taxonomy nodes.dmp
    --taxonomy_type NCBI                ## NCBI formatted input
    --genomes_path refseq/bacteria/     ## path to ncbi-genome-download folder with bacteria (or other folder structures)
    --genomeid2taxid taxonomy/nucl_gb.accession2taxid.gz ## accession numbers to taxid annotation
    --database NCBI_taxonomy.db         ## Name of the database where all information will be kept (for future use or reuse of flextaxd)
    -o NCBI_database                    ## Output folder
    --dump                              ## Print out names.dmp and nodes.dmp from flextaxd database
```


### Modify the NCBI database using a previously created flextaxd (example database from another source (CanSNPer.db))
Replace the Francisella node in the NCBI database with the node structure from a CanSNP database containing the Francisella CanSNPer tree.
```
## Step one: create your database containing required modifications
flextaxd --taxonomy_file cansnper_tree.txt --taxonomy_type CanSNPer --genomeid2taxid cansnper_genometotaxid_annotation.txt --database canSNPer_database/CanSNPer.db

## Step two: create your custom taxonomy database from NCBI
flextaxd --database NCBI_taxonomy.db --mod_database canSNPer_database/CanSNPer.db --parent "Francisella"

## Step three: dump your database
flextaxd --dump
```

### Create kraken2 NCBI database (approx 60min with 40 cores with complete bacterial genomes from genbank may 2019 (~9000))  
```
conda activate kraken2
flextaxd-create
    --database NCBI_taxonomy.db         ## flextaxd database file
    -o NCBI_database                    ## output dir (must be the same as where names and nodes.dmp were exported (--dump))
    --kraken_db NCBI_krakendb           ## Name of kraken database, relative path from script, not realtive to --outdir (-o)
    --genomes_path refseq/bacteria/     ## Path to ncbi-genome-download folder with bacteria (or other folder structures) using the same as when
                                            # the database was created is recommended otw genomes may be missed
    --create_db                         ## Request script to create a kraken database (kraken2 or krakenuniq must be installed)
    --krakenversion kraken2             ## Select version of kraken which is installed
    --processes 40                      ## Number of cores to use for adding genomes to the kraken database as well as the number of cores
                                            used to run kraken*-build (\*2,uniq)
    --skip "taxid"                      ## exclude genomes in taxid see details below
```

### Exclude genomes in taxid on database creation
Remove branches, the --skip parameter was implemented for benchmarking purposes as an option to remove branches by taxid, all children of the given taxid will be excluded.

flextaxd-create --skip "taxid,taxid2"

# Citation
https://academic.oup.com/bioinformatics/advance-article-abstract/doi/10.1093/bioinformatics/btab621/6361544

Sundell, D. et al. (2021) ‘FlexTaxD: flexible modification of taxonomy databases for improved sequence classification’, Bioinformatics. Edited by J. Kelso. Bioinformatics. doi: 10.1093/bioinformatics/btab621.
