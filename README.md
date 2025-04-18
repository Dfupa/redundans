![Latest Version](https://img.shields.io/github/v/tag/gabaldonlab/redundans?label=Latest%20Version)
[![BioConda Install](https://img.shields.io/conda/dn/bioconda/redundans.svg?style=flag&label=BioConda%20install)](https://anaconda.org/bioconda/redundans/)
[![GitHub Clones](https://img.shields.io/badge/dynamic/json?color=success&label=Clone&query=count&url=https://gist.githubusercontent.com/Dfupa/0fc9a42bb90e0b6c38767174bce725db/raw/clone.json&logo=github)](https://github.com/MShawon/github-clone-count-badge)
![Docker Pulls](https://img.shields.io/docker/pulls/cgenomics/redundans)
[![run with docker](https://img.shields.io/badge/run%20with-docker-0db7ed?labelColor=000000&logo=docker)](https://hub.docker.com/repository/docker/cgenomics/redundans)
[![run with singularity](https://img.shields.io/badge/run%20with-singularity-1d355c.svg?labelColor=000000)](https://cloud.sylabs.io/library/cgenomics/redundans/redundans)
### Table of Contents
- **[Redundans](#redundans)**  
  - **[Prerequisites](#prerequisites)**  
    - **[Official conda package](#official-conda-package)**
    - **[UNIX installer](#unix-installer)**    
    - **[Docker image](#docker-image)** 
  - **[Running the pipeline](#running-the-pipeline)**  
    - **[Parameters](#parameters)**  
    - **[Test run](#test-run)**  
  - **[Support](#support)**
  - **[Citation](#citation)**  

# Redundans
  
Redundans pipeline assists **an assembly of heterozygous genomes**.  
Program takes [as input](#parameters) **assembled contigs**, **sequencing libraries** and/or **reference sequence** and returns **scaffolded homozygous genome assembly**. Final assembly should be **less fragmented** and with total **size smaller** than the input contigs. In addition, Redundans will automatically **close the gaps** resulting from genome assembly or scaffolding. 

<img align="right" src="/docs/redundans_flowchart.png">

The pipeline consists of several steps (modules):  
1. **de novo contig assembly** (optional if no contigs are given)
2. **redundancy reduction**: detection and selective removal of redundant contigs from an initial *de novo* assembly 
3. **scaffolding**: joining of genome fragments using paired-end reads, mate-pairs, long reads and/or reference chromosomes 
4. **gap closing**: filling the gaps after scaffolding using paired-end and/or mate-pair reads 

Redundans is: 
- **fast** & **lightweight**, multi-core support and memory-optimised, 
so it can be run even on the laptop for small-to-medium size genomes
- **flexible** toward many sequencing technologies (Illumina, 454, Sanger, PacBio & Nanopore) and library types (paired-end, mate pairs, fosmids, long reads)
- **modular**: every step can be omitted or replaced by other tools
- **reliable**: it has been already used to improve genome assemblies varying in size (several Mb to several Gb) and complexity (fungal, animal & plants)

For more information have a look at the [documentation](/docs), [poster](/docs/poster.pdf), [publication](http://nar.oxfordjournals.org/content/44/12/e113), [test dataset](/test) or [manual](http://bit.ly/redundans_manual). 

## Prerequisites
Redundans uses several programs (all except the interpreters and its submodules are provided within this repository):

| Resource | Type | Version |
| :--- | :--- | :--- |
| [Python](https://www.python.org/downloads) | Language interpreter | <3.11, ≥ 3.8 |
| [Platanus](http://platanus.bio.titech.ac.jp/?page_id=14) | Genome assembler | v1.2.4 |
| [Miniasm](https://github.com/lh3/miniasm) | Genome assembler | ≥ v0.3 (r179) |
| [Minimap2](https://github.com/lh3/minimap2) | Sequence aligner | ≥ v2.2.4 (r1122) |
| [LAST](http://last.cbrc.jp/) | Sequence aligner | ≥ v800 |
| [BWA](http://bio-bwa.sourceforge.net/) | Sequence aligner | ≥ v0.7.12 |
| [SNAP aligner](https://github.com/amplab/snap) | Sequence aligner | v2.0.1 |
| [SSPACE3](http://www.baseclear.com/genomics/bioinformatics/basetools/SSPACE) | Scaffolding software | v3.0 |
| [GapCloser](http://sourceforge.net/projects/soapdenovo2/files/GapCloser/) | Gapclosing software | v1.12 |
| [GFAstats](https://github.com/vgl-hub/gfastats) | Stats software | ≥ v1.3.6 |
| [Meryl](https://github.com/marbl/meryl) | K-mer counter software | ≥ v1.3 |
| [Merqury](https://github.com/marbl/merqury) | Assembly evaluation software | v1.3 |
| [k8](https://github.com/attractivechaos/k8/) | Javascript shell based on V8 | v0.2.4 |
| [R](https://cran.r-project.org/) | Language interpreter | ≥ 3.6 |
| [ggplot2](https://ggplot2.tidyverse.org)| R package | ≥ 3.3.2 |
| [scales](https://cran.r-project.org/web/packages/scales/) | R package | ≥ 3.3.2 |
| [argparser](https://cran.r-project.org/web/packages/argparser/) | R package | ≥ 3.6 |

#### WARNING: Some of the third-party requirements are provided precompiled in x86_64 and not readily available for other architectures.

On most Linux distros, the installation should be as easy as:
```
git clone --recursive https://github.com/Gabaldonlab/redundans/
cd redundans && bin/.compile.sh
```

If it fails, make sure you have below dependencies installed: 
- Perl [SSPACE3]
- make, gcc & g++ [BWA, GFAstats, Miniasm & LAST] ie. `sudo apt-get install make gcc g++`
- [zlib including zlib.h headers](http://zlib.net/) [BWA] ie. `sudo apt-get install zlib1g-dev`
- [R  ≥ 3.6](https://cran.r-project.org/) and additional packages [ggplot2, scales, argparser] for plotting the Merqury results.
- optionally for additional plotting `numpy` and `matplotlib` ie. `sudo -H pip install -U matplotlib numpy`

For user convenience, we provide [UNIX installer](#unix-installer) and [Docker image](#docker-image), that can be used instead of manually installation.  

## Official conda package
If you are familiar with conda, this will be by far the easiest way of installing redundans: 
```bash
# create new Python3 >=3.8,<3.11 environment
conda create -n redundans python=3.10
# activate it
conda activate redundans
# and install redundans
conda install -c bioconda redundans 
```


## UNIX installer
UNIX installer will automatically fetch, compile and configure Redundans together with all dependencies.
It should work on all modern Linux systems, given Python >= 3, commonly used programmes (ie. wget, make, curl, git, perl, gcc, g++, ldconfig) and libraries (zlib including zlib.h) are installed. 
```bash
source <(curl -Ls https://github.com/Gabaldonlab/redundans/raw/master/INSTALL.sh)
```

### Docker image
First, you  need to install [docker](https://www.docker.com/): `wget -qO- https://get.docker.com/ | sh`  
Then, you can run the test example by executing: 
```bash
#Pull the image directly from dockerhub
docker pull cgenomics/redundans:latest

# process the data inside the image - all data will be lost at the end
docker run -it -w /root/src/redundans cgenomics/redundans:latest ./redundans.py -v -i test/{600,5000}_{1,2}.fq.gz -f test/contigs.fa -o test/run1

# if you wish to process local files, you need to mount the volume with -v
## make sure you are in redundans repo directory (containing test/ directory)
docker run -v `pwd`/test:/test:rw -it cgenomics/redundans:latest /root/src/redundans/redundans.py -v -i test/*.fq.gz -f test/contigs.fa -o test/run1
```
### Singularity image
Redundans is also supported by singularity. First install [singularity](https://docs.sylabs.io/guides/3.1/user-guide/quick_start.html#quick-installation-steps).

You can either use our singularity repository to build the image or to build the image out of the docker image. Then run the first example:
```
#Pull from the singularity repo
singularity pull --arch amd64 library://cgenomics/redundans/redundans:2.0

#Build the image based on the docker repo
singularity build redundans.sif docker://cgenomics/redundans

#Use exec instead of run to account for shell-based wildcarsds * and ?
singularity exec redundans.sif bash -c "/root/src/redundans/redundans.py -v -i /root/src/redundans/test/*_?.fq.gz -f /root/src/redundans/test/contigs.fa -o /tmp/run1"
```

## Running the pipeline
Redundans input consists of any combination of:
- **assembled contigs** (FastA)
- **paired-end and/or mate pairs reads** (FastQ*)
- **long reads** (FastQ/FastA*) - both PacBio and Nanopore are supported for the scaffolding
- and/or **reference chromosomes/contigs** (FastA). 
* gzipped files are also accepted.

Redundans will return **homozygous genome assembly** in `scaffolds.filled.fa` (FastA). It will also report the heterozygous contigs that were not discarded during the reduction step.
In addition, the program reports [statistics for every pipeline step](/test#summary-statistics), including number of contigs that were removed, GC content, N50, N90 and size of gap regions. 

### Parameters
For the user convenience, Redundans is equipped with a wrapper that **automatically estimates run parameters** and executes all steps/modules.
You should specify some sequencing libraries (FastA/FastQ) or reference sequence (FastA) in order to perform scaffolding. 
If you don't specify `-f` **contigs** (FastA), Redundans will assemble contigs *de novo*, but you'll have to provide **paired-end and/or mate pairs reads** (FastQ).
Most of the pipeline parameters can be adjusted manually (default values are given in square brackets []):  
**HINT**: If you run fails, you may try to resume it, by adding `--resume` parameter. 
- General options:
```
  -h, --help            show this help message and exit
  -v, --verbose         verbose
  --version             show program's version number and exit
  -i FASTQ, --fastq FASTQ
                        FASTQ PE / MP files
  -f FASTA, --fasta FASTA
                        FASTA file with contigs / scaffolds
  -o OUTDIR, --outdir OUTDIR
                        output directory [redundans]
  -t THREADS, --threads THREADS
                        no. of threads to run [4]
  --resume              resume previous run
  --log LOG             output log to [stderr]
  --nocleaning
```
De novo assembly options:
```
  -m MEM, --mem MEM     max memory to allocate (in GB) for the Platanus assembler [2]
  --tmp TMP             tmp directory [/tmp]
```
- Reduction options:
```
  --identity IDENTITY   min. identity [0.51]
  --overlap OVERLAP     min. overlap  [0.80]
  --minLength MINLENGTH
                        min. contig length [200]
  --minimap2reduce      Use minimap2 for the initial and final Reduction step. Recommended for input assembled contigs from long reads or larger contigs using --preset[asm5] by default. By default LASTal is used for Reduction.
  -x INDEX, --index INDEX
                        Minimap2 parameter -i used to load at most INDEX target bases into RAM for indexing [4G]. It has to be provided as a string INDEX ending with k/K/m/M/g/G.
  --noreduction         Skip reduction
```
- Short-read scaffolding options:
```
  -j JOINS, --joins JOINS
                        min pairs to join contigs [5]
  -a LINKRATIO, --linkratio LINKRATIO
                        max link ratio between two best contig pairs [0.7]
  --limit LIMIT         align subset of reads [0.2]
  -q MAPQ, --mapq MAPQ  min mapping quality [10]
  --iters ITERS         iterations per library [2]
  --noscaffolding       Skip short-read scaffolding
  -b, --usebwa          use bwa mem for alignment [use snap-aligner]
```
- Long-read scaffolding options:
```
  -l LONGREADS, --longreads LONGREADS
                        FastQ/FastA files with long reads
  -s, --populateScaffolds
                        Run populateScaffolds mode for long read scaffolding, else generate a dirty assembly for reference-based scaffolding. Not recommended for highly repetitive genomes. Default False.
  --minimap2scaffold         Use Minimap2 for aligning long reads. Preset usage dependant on file name convention (case insensitive): ont, nanopore, pb, pacbio, hifi, hi_fi, hi-fi. ie: s324_nanopore.fq.gz. Else it uses LASTal.
```
- Reference-based scaffolding options:
```
  -r REFERENCE, --reference REFERENCE
                        reference FastA file
  --norearrangements    high identity mode (rearrangements not allowed)
  -p PRESET, --preset PRESET
                        Preset option for Minimap2-based Reduction and/or Reference-based scaffolding. Possible options: asm5 (5 percent sequence divergence), asm10 (10 percent sequence divergence) and asm20(20 percent sequence divergence). Default [asm5]
```
- Gap closing options:
```
  --nogapclosing                        
```
- Meryl and Merqury options:
```
  --runmerqury           Run meryldb and merqury for assembly kmer multiplicity stats. [False] by default.
  -k KMER, --kmer KMER  K-mer size for meryl [21]
```

Redundans is **extremely flexible**. All steps of the pipeline can be ommited using: `--noreduction`, `--noscaffolding`, `--nogapclosing` and/or `--runmerqury` parameters. 

### Test run
To run the test example, execute: 
```bash
./redundans.py -v -i test/*_?.fq.gz -f test/contigs.fa -o test/run1

#Test it using minimap2 for the reduction step, increasing performance for large genomes
./redundans.py -v -i test/*_?.fq.gz -f test/contigs.fa --minimap2reduce -o test/run2

# if your run failed for any reason, you can try to resume it
rm test/run1/_sspace.2.1.filled.fa
./redundans.py -v -i test/*_?.fq.gz -f test/contigs.fa -o test/run1 --resume

# if you have no contigs assembled, just run without `-f`
./redundans.py -v -i test/*_?.fq.gz -o test/run.denovo
```

Note, the **order of libraries (`-i/--input`) is not important**, as long as `read1` and `read2` from each library are given one after another 
i.e. `-i 600_1.fq.gz 600_2.fq.gz 5000_1.fq.gz 5000_2.fq.gz` would be interpreted the same as `-i 5000_1.fq.gz 5000_2.fq.gz 600_1.fq.gz 600_2.fq.gz`.

You can play with **any combination of inputs** ie. paired-end, mate pairs, long reads and / or reference-based scaffolding as well as selecting minimap2 for each step or default to LASTal, for example:
```bash
# reduction, scaffolding with paired-end, mate pairs and long reads used to generate a miniasm assembly to do reference-based scaffolding, and gap closing with paired-end and mate pairs using as an aligner minimap2
./redundans.py -v -i test/*_?.fq.gz -l test/nanopore.fa.gz -f test/contigs.fa -o test/run_short_long_ref --minimap2scaffold

# reduction, scaffolding with paired-end, mate pairs and long reads, and gap closing with paired-end and mate pairs using populateScaffolds method using as aligner minimap2
./redundans.py -v -i test/*_?.fq.gz -l test/pacbio.fq.gz test/nanopore.fa.gz -f test/contigs.fa -o test/run_short_long_populatescaffold --minimap2scaffold --populateScaffolds

# scaffolding and gap closing with paired-end and mate pairs (no reduction)
./redundans.py -v -i test/*_?.fq.gz -f test/contigs.fa -o test/run_short-scaffolding-closing --noreduction

# reduction, reference-based scaffolding and gap closing with paired-end reads (--noscaffolding disables only short-read scaffolding)
./redundans.py -v -i test/600_?.fq.gz -r test/ref.fa -f test/contigs.fa -o test/run_ref_pe-closing --noscaffolding
```

For more details have a look in [test directory](/test). 

## Support 
If you have any issues or doubts check [documentation](/docs) and [FAQ (Frequently Asked Questions)](/docs#faq). 
You may want also to sign to [our forum](https://groups.google.com/d/forum/redundans).

## Citation
Leszek P. Pryszcz and Toni Gabaldón (2016) Redundans: an assembly pipeline for highly heterozygous genomes. NAR. [doi: 10.1093/nar/gkw294](http://nar.oxfordjournals.org/content/44/12/e113)
