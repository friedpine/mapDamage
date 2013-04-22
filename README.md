Introduction
------------

mapDamage2 is a computational framework written in Python and R, which tracks and quantifies DNA damage patterns 
among ancient DNA sequencing reads generated by Next-Generation Sequencing platforms. 


### Requirements

* Python (version >= 2.6)
* [Git](http://git-scm.com/) 
* [R]([http://www.r-project.org/) (version >= 2.15.1) must be present in your $PATH. Otherwise, only tables will be produced.
* [pysam](http://code.google.com/p/pysam/]) (version == 0.6) python package, an interface for reading/writing SAM/BAM files
* [gsl](http://www.gnu.org/software/gsl/) GNU scientific library (GSL) 
* R libraries:
 - inline
 - gam
 - Rccp
 - RcppGSL
 - ggplot2 (>=0.9.2)

Of note, the R libraries and the GSL library are only required for the statistical estimation of DNA damage 
parameters. seqtk is included in this package for fast computation of reference base composition, it is written by Heng Li
and can be found at [here at github.](https://github.com/lh3/seqtk)


### Installation

mapDamage2 was successfully tested on GNU/Linux and MacOSX environments.

1. Install mapDamage:

Clone the repository from GitHub
> `git clone https://github.com/ginolhac/mapDamage.git mapDamage`
go into the resulting folder
> `cd mapDamage`  
then clone the submodules.
> `git submodule update --init`  
Run the following command if you have administrator rights  
> `sudo python setup.py install`  
otherwise install it locally  
> `python setup.py install --user`  
then, $HOME/.local/bin must be in your PATH.


2. Install pysam 0.6:

Download the tarball archive for [pysam](http://code.google.com/p/pysam/downloads/list).
Untar the downloaded tar file and follow instructions in the pysam-0.6/INSTALL text file.
**It is important installing the specific version pysam 0.6 since we had issues with  
newer versions.** It is recommend to check if the pysam installation seems okay 
before proceeding with the following command
> `python -c "import pysam"` 
If nothing appears then proceed.


3. Install R:

Follow the instructions available at [R project](http://www.r-project.org/)
(with installers/packages available for all major operating systems).

4. Install the R packages:

By running in a R console and selecting a CRAN mirror:
> `install.packages("inline")`  
> `install.packages("gam")`  
> `install.packages("Rcpp")`  
> `install.packages("ggplot2")` 


5. Install the  GSL library [gsl](http://www.gnu.org/software/gsl/):

For Debian-based distros by the following. 
> `sudo apt-get install libgsl0-dev`  
There should be a equivalent package for Mac using fink or macports.


6. Install RcppGSL:

In a R session do the following
> `install.packages("RcppGSL")`  
If the installation of RcppGSL ended with an unsuccessfully then, download the tarball at [RcppGSL](http://cran.r-project.org/web/packages/RcppGSL/index.html) and install it with:  
> `install.packages("RcppGSL_0.2.0.tar.gz")`

Note that if steps 4-6 fail for some reason, then it is possible to utilize mapDamage without 
the statistical function.

---

News in version 2.0
--------------------
- The main new feature is the **approximate bayesian estimation of damage parameters**.
- The --rescale parameter can be optionally used to rescale quality scores of likely damaged positions 
in the reads. A new BAM file is constructed by downscaling quality values for misincorporations likely 
due to ancient DNA damage according to their initial qualities, position in reads and damage patterns.
- The program was rewritten in python to make it simpler and the dependency on bedtools was replaced by the use of pysam.
The memory footprint is now negligible and runtime reduced by _ca._ 25%.
- It is now possible to filter out nucleotides with qualities inferior to a threshold defined by users, 
allowing users to reduce the impact of sequencing errors when counting misincorporations.


Inputs
------

Two files are needed:

- A valid SAM/BAM file with a correct header, as argument to the **-i** option.
- A FASTA file that contains reference sequences used for mapping reads, as argument to the **-r** option. 

References described in the SAM/BAM header and the FASTA file must be coherent, _i.e_,  
the references must have identical names and lengths. Extra sequences present in the FASTA header  
raise a warning but the program will proceed since all needed references are available.

As an alternative, one can run only the plotting, statistic estimations or rescaling
on an already processed dataset. Use a combination of **-d** option followed by 
a valid folder and the `--plot-only`, `--stats-only` or  `--rescale-only` options.


Outputs
-------

When all options are activated, 15 files are produced in the result folder.

- Runtime\_log.txt, log file with a summary of command lines used and timestamps.

For the plotting:

 - a pdf file, Fragmisincorporation\_plot.pdf that displays both fragmentation and misincorporation patterns.
 - a pdf file, Length\_plot.pdf, that displays length distribution of singleton reads per strand and cumulative 
    frequencies of C>T at 5'-end and G>A at 3'-end are also displayed per strand.
 - misincorporation.txt, contains a table with occurrences for each type of mutations and relative positions from the reads ends.
 - 5pCtoT\_freq.txt, contains frequencies of Cytosine to Thymine mutations per position from the 5'-ends.
 - 3pGtoA\_freq.txt, contains frequencies of Guanine to Adenine mutations per position from the 3'-ends.
 - dnacomp.txt, contains a table of the reference genome base composition.
 - lgdistribution.txt, contains a table with read length distributions per strand.
 
For the statistical estimation:

 - Stats\_out\_MCMC\_hist.pdf, MCMC histogram for the damage parameters and log likelihood.
 - Stats\_out\_MCMC\_iter.csv, values for the damage parameters and log likelihood in each MCMC iteration.
 - Stats\_out\_MCMC\_trace.pdf, a MCMC trace plot for the damage parameters and log likelihood. 
 - Stats\_out\_MCMC\_iter\_summ\_stat.csv, summary statistics for the damage parameters MCMC samples.
 - Stats\_out\_post\_pred.pdf, empirical misincorporation frequency and predictive intervals from the fitted model.
 - Stats\_out\_MCMC\_correct\_prob.csv, position specific probability of a C.T and G.A misincorporation is due to damage.
 - Rescaled BAM file, where likely post-mortem damaged bases have downscaled quality scores. 


Examples and datasets
---------------------
A simple command line that would process the entire BAM file with plotting and statistic estimation is:

    mapDamage -i mymap.bam -r myreference.fasta

To run the plotting part with a new scale to fit lower levels of DNA damages, 0.1 as y-limit instead of 0.3 by default:

     mapDamage -d results_mydata -y 0.1 --plot-only

To run the statistic estimations using only the 5'-ends and with verbose output:

     mapDamage -d results_mydata --forward --stats-only -v


The original web page with examples, datasets and result files is there:

http://geogenetics.ku.dk/publications/mapdamage/


Usage
-----

    Usage: mapDamage [options] -i BAMfile -r reference.fasta
    
    Use option -h or --help for help
    
    Options:

       --version             show program's version number and exit
       -h, --help            show this help message and exit

    Input files:
  
        -i FILENAME, --input=FILENAME
                             SAM/BAM file, must contain a valid header, use '-' for reading a BAM from stdin
        -r REF, --reference=REF
                            Reference file in FASTA format
    General options:
        -n DOWNSAMPLE, --downsample=DOWNSAMPLE
                            Downsample to a randomly selected fraction of the reads (if 0 < DOWNSAMPLE < 1), 
                            or a fixed number of randomly selected reads (if DOWNSAMPLE >= 1). By default, 
                            no downsampling is performed.
	--downsample-seed   Seed value to use for downsampling. See documentation for py module 'random' for default behavior.
        -l LENGTH, --length=LENGTH
                            read length, in nucleotides to consider [70]
        -a AROUND, --around=AROUND
                            nucleotides to retrieve before/after reads [10]
        -Q MINQUAL, --min-basequal=MINQUAL
                            minimun base quality Phred score considered, Phred-33 assumed [0]
        -d FOLDER, --folder=FOLDER
                        folder name to store results [results_FILENAME]
        -f, --fasta         Write alignments in a FASTA file
        --plot-only         Run only plotting from a valid result folder
        -q, --quiet         Disable any output to stdout
        -v, --verbose       Display progression information during parsing

    Options for graphics:

        -y YMAX, --ymax=YMAX
                            graphical y-axis limit for nucleotide misincorporation frequencies [0.3]
        -m READPLOT, --readplot=READPLOT
                            read length, in nucleotides, considered for plotting nucleotide 
                            misincorporations [25]
        -b REFPLOT, --refplot=REFPLOT
                            the number of reference nucleotides to consider for ploting base 
                            composition in the region located upstream and downstream of 
                            every read [10]
        -t TITLE, --title=TITLE
                            title used for both graph and filename [plot]

    Options for the statistical estimation:

        --rand=RAND         Number of random starting points for the likelihood optimization [30]
        --burn=BURN         Number of burnin iterations [10000]
        --adjust=ADJUST     Number of adjust proposal variance parameters iterations [10]
        --iter=ITER         Number of final MCMC iterations [50000]
        --forward=FORWARD   Using only the 5' end of the seqs [False]
        --reverse=REVERSE   Using only the 3' end of the seqs [False]
        --fix-disp=FIX_DISP Fix dispersion in the overhangs [True]
        --diff-hangs        The overhangs are different for 5' and 3'  [False]
        --fix-nicks=FIX_NICKS 
                            Fix the nick frequency vector nu else estimate it with GAM [False]
        --jukes-cantor	    Use Jukes Cantor instead of HKY85
        --single_stranded=SINGLE_STRANDED
                            Single stranded protocol [False]
        --seq-length=SEQ_LENGTH
                            How long sequence to use from each side [12]
        --stats-only        Run only statistical estimation from a valid result folder
        --rescale           Rescale the quality scores in the BAM file using the output from
                            the statistical estimation
        --rescale-only 	    Run only rescaling from a valid result folder
        --no-stats          Disabled statistical estimation, active by default

Description of tables
---------------------
### misincorporation table

This file looks like:
 
    # table produced by mapDamage version 2
    # using mapped file hits_sort_mts.bam and NC_012920.fasta as reference file
    # Chr: reference from sam/bam header, End: from which termini of DNA sequences, Std: strand of reads
    Chr	End	Std	Pos	A	C	G	T	Total	G>A	C>T	A>G	T>C	A>C	A>T	C>G	C>A	T>G	T>A	G>C	G>T	A>-	T>-	C>-	G>-	->A	->T	->C	->G	S
    NC_012920.1	3p	+	1	424	579	201	424	1628	48	16	3	5	1	0	2	1	0	2	1	0	0	0	0	0	4	4	1	1	0
    NC_012920.1	3p	+	2	466	535	213	404	1618	30	12	2	1	0	1	0	0	3	3	1	0	0	0	0	0	2	6	5	7	0
    NC_012920.1	3p	+	3	514	463	223	417	1617	35	15	3	1	1	0	0	0	1	0	1	1	0	0	0	2	9	4	4	4	0
    NC_012920.1	3p	+	4	514	483	221	400	1618	30	11	3	4	0	1	1	0	0	1	0	1	1	0	0	2	5	5	8	2	0

The first lines that start by a hash contain information about the options used while processing the data.
Then, the table contains occurrences for each of the 12 mutation type + 4 deletions + 4 insertions + 
soft-clipping, per reference (`Chr` column), per strand (`Std` column) , per end (`End` column) and per position (`Pos` column). 
In Z>X, Z comes from the reference, X is the nucleotide from reads. Eventually, A, C, G, T come from the reference
and Total is the sum of the four nucleotides. Frequencies depicted in Fragmisincorporation\_plot.pdf file are computed per
position as follows

occurrences of mutations / occurrences of the reference nucleotide 

in order to compensate for base composition bias.

In this example, for G>A at the first base from the 3'-ends and for the positive strand:
`48 / 201 = 0.238806`

The Fragmisincorporation plot sums up misincorporations  for all references and strand orientations, finally 
displays the frequencies per position.  

**5p** means reads analyzed from their 5'-ends and are depicted at the bottom left.
  
**3p** means reads analyzed from their 3'-ends and are depicted at the bottom right.

**Remark concerning soft-clipping**:

Soft-clipped bases are bases located at read extremities that are NOT aligned to the reference. 
Those bases are not taken into account and alignments are computed on aligned bases. 
However, the soft-clipped bases are recorded and displayed as an orange line as a diagnostic for users.
Even if, the positions for those bases are different from positions of all misincorporations, they should warn
users that reads should be trimmed if soft-clipped base frequencies is high.


### dnacomp table

Example table below:
 
    # table produced by mapDamage version 0.4.0
    # using mapped file hits_sort_mts.bam and NC_012920.fasta as reference file
    # Chr: reference from sam/bam header, End: from which termini of DNA sequences, Std: strand of reads
    Chr	End	Std	Pos	A	C	G	T	Total
    NC_012920.1	3p	+	-70	113	77	43	181	414
    NC_012920.1	3p	+	-69	178	140	97	233	648
    NC_012920.1	3p	+	-68	230	145	98	219	692
    NC_012920.1	3p	+	-67	217	157	76	272	722
    NC_012920.1	3p	+	-66	224	188	88	239	739
    NC_012920.1	3p	+	-65	244	165	104	262	775
    NC_012920.1	3p	+	-64	223	185	102	288	798

The first lines that start by a hash contain information about the options used while processing the data.
Then, the table contains occurrences for each nucleotides, per reference (`Chr` column), 
per strand (`Std` column) , per end (`End` column) and per position (`Pos` column).

At a given position, base composition is recorded either for the reference or for reads depending 
on the position and direction. The following table details which sequences and coordinates are 
used by default:

| Positions       |      5p              |          3p          |
|:---------------:|:--------------------:|:--------------------:|
| negative values | reference, -10 to -1 | reads, -25 to -1     |
| positive values | reads, +1 to +25     | reference, +1 to +10 |

 


Citation
--------
If you use this program, please cite the following publication:  
Jónsson H, Ginolhac A, Schubert M, Johnson P, Orlando L.
mapDamage2.0: fast approximate Bayesian estimates of ancient DNA damage parameters.
_Bioinformatics_ 2013. _in press_


The original mapDamage1 is described in the following article:  
Ginolhac A, Rasmussen M, Gilbert MT, Willerslev E, Orlando L.
mapDamage: testing for damage patterns in ancient DNA sequences. _Bioinformatics_ 2011 **27**(15):2153-5
http://bioinformatics.oxfordjournals.org/content/27/15/2153


FAQ
-----------
Q: I got this error: could not find function "ggtitle"

A: update your ggplot2 package to at least version 0.9.2 by running
update.packages("ggplot2") and selecting a CRAN mirror

Q: How to clone the development version from github?

A: Use the following commands
> `git clone https://github.com/ginolhac/mapDamage.git mapDamage`  
> `cd mapDamage` 
> `git submodule update --init`  


Contact
-------
Please report bugs and suggest possible improvements to Aurélien Ginolhac, Mikkel Schubert or Hákon Jónsson by email:
aginolhac at snm.ku.dk, MSchubert at snm.ku.dk or jonsson.hakon at gmail.com.

