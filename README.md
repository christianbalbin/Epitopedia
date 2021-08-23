# Epitopedia

## Getting started

The quickest way to start using Epitopedia is by downloading the docker container which contains all the dependencies preinstalled

```shell
docker pull cbalbin/epitopedia
```

Epitopedia requires the [PDB in mmCIF](https://www.wwpdb.org/ftp/pdb-ftp-sites) format, EpitopediaDB and EPI-SEQ DB. EpitopediaDB and EPI-SEQ DB can be downloaded [here](https://fiudit-my.sharepoint.com/:u:/g/personal/cbalbin_fiu_edu/EWW8XKxSx09CvWC2mhzp8_sBntrnXX9mju4SbA0_ygUFMA?e=zKcZvU).


To download the entirety of PDB in mmCIF format
```shell
rsync -rlpt -v -z --delete --port=33444 \
rsync.rcsb.org::ftp_data/structures/divided/mmCIF/ ./mmCIF
```

**OR**
<br>

To download the only the PDB files present in EpitopediaDB (EPI-PDB) you can supply the pdb_id_list.txt to rsync
```shell
rsync -rlpt -v -z --delete --port=33444 --include-from=/path/to/pdb_id_list.txt \
rsync.rcsb.org::ftp_data/structures/divided/mmCIF/ ./mmCIF
```

To run Epitopedia provide the paths to the various directories discussed below.

The data directory should contain Epitopedia DB (epitopedia.sqlite3) and EPI-SEQ (EPI-SEQ.fasta*) which can be downloaded [here](https://fiudit-my.sharepoint.com/:u:/g/personal/cbalbin_fiu_edu/EWW8XKxSx09CvWC2mhzp8_sBntrnXX9mju4SbA0_ygUFMA?e=zKcZvU).

The mmcif directory should point to the sharded PDB directory in mmCIF format as downloaded above.

The output directory is where the output files will be written.

Replace the the paths on the left side of the colon with the actual path on your local system. The paths on the right side of the colon are internal and should not be altered.

```shell
docker run --rm -it -p 5000:5000 \
-v /Path/to/Data/Dir/:/app/data \
-v /Path/to/PDBDB/Dir/:/app/mmcif \
-v /Path/to/Output/Dir/:/app/output \
cbalbin/epitopedia run_epitopedia.py 6VXX_A --taxid_filter 11118
```

It is recommended to use a taxid_filter to prevent the input protein from finding itself as a StructBMM. For example, 6VXX is a SARS-CoV-2 protein and using a taxid_filter of 11118 will filter out all Coronaviridae.


Epitopedia will output the following files:



File Name | Description
------------ | -------------
Place holder | Place holder
Place holder | Place holder




Epitopedia can run on multiple input structures to represent a conformational ensemble. To do so, simply provide a list of structures in the format PDBID_CHAINID as shown below.
```shell
run_epitopedia.py 6VXX_A 6VXX_B 6XR8_A 6XR8_B
```

Epitopedia defaults to a span length of 5, surface accesbility cutoff of 20% surface accesbility span legnth of 3, and no taxa filter, but these parameters can be set using the follow flags:

Flag | Description
------------ | -------------
--span | Minimum span length for a hit to progress
--rasa | Cutoff for relative accessible surface area
--rasa_span | Minimum consecutive accesssible residues to consider a hit a SeqBMM
--taxid_filter | taxa filter; example to filter out all Coronaviridae --taxid_filter 11118


## Epitopedia database generation

Epitopedia uses IEDB and PDB to generate EpitopediaDB, which is used in the molecular mimicry search.

Generation of the database takes some time (~12 hours). Thus, the EpitopediaDB is provided above.

To create the EpitopediaDB, download [IEDB](https://www.iedb.org/downloader.php?file_name=doc/iedb_public.sql.gz) and a mmCIF version of PDB

Point the container to the approriate paths for the IEDB, PDB (mmCIF format) and a data directory where the databases will be written.


```shell
docker run --rm -it \
-v /Path/To/iedb_public.sql:/app/iedb \
-v /Path/to/mmCIF/Dir/:/app/mmcif \
-v /Path/to/Data/Dir/:/app/data \
cbalbin/mimicrypipeline generate_database.py
```

## License

This software is released under the MIT License

Software and databases used in Epitopedia may be released under various licenses:

Software:

* [DSSP](https://github.com/cmbi/dssp/blob/master/COPYING)
* [TM-align](https://zhanggroup.org/TM-align/TMalign.f)
* [NCBI BLAST](https://www.ncbi.nlm.nih.gov/IEB/ToolBox/CPP_DOC/lxr/source/scripts/projects/blast/LICENSE)
* [Entrez](https://www.ncbi.nlm.nih.gov/IEB/ToolBox/CPP_DOC/lxr/source/scripts/projects/blast/LICENSE)
* [MMseqs2](https://github.com/soedinglab/MMseqs2/blob/master/LICENSE.md)
* [mysql2sqlite](https://github.com/dumblob/mysql2sqlite/blob/master/LICENSE)
* [sqlite](https://www.sqlite.org/copyright.html)
* [Docker](https://github.com/docker/cli/blob/master/LICENSE)
* [Flask](https://github.com/pallets/flask/blob/main/LICENSE.rst)
* [gemmi](https://github.com/project-gemmi/gemmi/blob/master/LICENSE.txt)
* [rich](https://github.com/willmcgugan/rich/blob/master/LICENSE)
* [biopython](https://github.com/biopython/biopython/blob/master/LICENSE.rst)
* [dataclasses-json](https://github.com/lidatong/dataclasses-json/blob/master/LICENSE)
* [python](https://github.com/python/cpython/blob/main/LICENSE)

Databases:
* [IEDB](http://www.iedb.org/terms_of_use_v3.php)
* [PDB](https://www.rcsb.org/pages/usage-policy)