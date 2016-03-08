# Hydroidolina Phylogeny Assignment

In this assignment you will reanalyze the data from the following paper:

> Cartwright, P., Evans, N. M., Dunn, C. W., Marques, A. C., Miglietta, M. P., 
Schuchert, P., & Collins, A. G. (2008). Phylogenetics of Hydroidolina 
(Hydrozoa: Cnidaria). Journal of the Marine Biological Association of the UK, 
88(08), 1663-1672. 
[doi:10.1017/S0025315408002257](http://dx.doi.org/10.1017/S0025315408002257)

Take a look at the paper to familiarize yourself with the problem at hand, the 
approach taken, the data used, and the analyses that are presented.

Below are a series of analyses to perform with these data. As with many analyses, most of the hands-on work is data wrangling (getting files organized and in the right format). New ground that we cover here includes:

- Batch download of data from NCBI
- Reformatting sequence names
- Multiple sequence alignment
- Combining data from multiple genes into a single alignment
- Running multiple analyses

## Steps

Some of the steps below build on things you already did in the [siphonophore16s](https://github.com/Phylogenetics-Brown-BIOL1425/siphonophore16s) analysis. Check that repository if you are unsure about details here.


### Fork the repository on github

Browse to the GitHub repository for this exercise. If you are reading this at GitHub, you are already [there](https://github.com/Phylogenetics-Brown-BIOL1425/hydroidolina).

Click the "Fork" button on the top right of the page. This creates your own independent copy of the repository to work with, so we can all make changes without affecting other peoples' analyses. It also takes you to the web page for the new repository. The url will be something like:

    https://github.com/github_username/hydroidolina

where `github_username` is your github username.


### Download and prepare the data

Clone the repository to your laptop.

Table 1 of the paper presents the [NCBI](http://www.ncbi.nlm.nih.gov) accession 
numbers for all the data included in the analyses. I have extracted the data 
from Table 1 and are presented here in three files (I also cleaned up a couple 
of typos in the accession numbers):

    28s.txt
    18s.txt
    16s.txt
    
Use the 
[NCBI Batch Entrez interface](http://www.ncbi.nlm.nih.gov/sites/batchentrez) 
to download a fasta file for the sequences listed in each of these three files. 
From the top of the interface, select the 'Nucleotide' Database and chose a file 
of accession numbers. Click retrieve, and then follow the link that is generated. 
This will take you to a page with the sequence summaries. Click 'Send to', 
choose 'File' as the destination, 'FASTA' as the format, and then click 
'Create File'.

Repeat this for each of the three gene files, rename them `28s.raw.fasta` etc. 

The fasta header rows (which start with `>`) have lots of information. To combine data across genes we will need the header to just have the species name. Use regular expressions (see Haddock and Dunn chapters 2-3), a powerful and flexible search and replace tool, to make these changes with the `sed` command:

    sed -E 's/>.+\| ([a-zA-Z]+) ([a-zA-Z]+)[\. ].+/>\1_\2/g' 28s.raw.fasta > 28s.fasta
    sed -E 's/>.+\| ([a-zA-Z]+) ([a-zA-Z]+)[\. ].+/>\1_\2/g' 18s.raw.fasta > 18s.fasta
    sed -E 's/>.+\| ([a-zA-Z]+) ([a-zA-Z]+)[\. ].+/>\1_\2/g' 16s.raw.fasta > 16s.fasta

The basic format of each of these commands is `sed -E 's/search/replace/g' input_file > output_file` where search this the text it is looking for and replace is the text that it puts in its place.


Then add the fasta files to the repository, commit, and push them:

   git add *.fasta
   git commit -am "added raw fasta files"
   git push


### Generate alignments

In the raw fasta files, homologous sites are not in the same columns because the sequences start in different places, and there have been insertions and deletions in the course of sequence evolution. Aligners attempt to place homologous sites in the same columns by inserting gaps (usually denoted with the `-` character) to create a data matrix. We'll use the aligner `mafft`. You can clone the respository to your oscar `~/scratch` directory and make the alignments there, or [download mafft](http://mafft.cbrc.jp/alignment/software/) and install it on your own computer.

  interact # only needed on oscar, starts an interactive analysis session
  module load mafft # only needed on oscar, loads the mafft module

  mafft 28s.fasta > 28s.aligned.fasta
  mafft 18s.fasta > 18s.aligned.fasta
  mafft 16s.fasta > 16s.aligned.fasta

  git add *aligned.fasta
  git commit -am "added alignments"
  git push

Unfortunately, there are many different alignment formats formats and different programs use different formats `mafft` creates alignments in fasta format, but we'll need them in phylip and nexus format for phylogenetic analyses. We also need to create combined matrices that include data from all genes. There are many scripts available online for these tasks, but we will just use the interactive tool [mesquite](http://mesquiteproject.org/) on your laptop.

For each of the `*.aligned.fasta` files, do the following. These example steps are for `28s.aligned.fasta`:

- Open `28s.aligned.fasta` in mesquite. It will ask you to save the alignment as a nexus file, save it as `28s.nex`.
- Once the file is open, go to the File menu and select Export. Then select "Phylip (DNA/RNA)". In the window that pops up, leave "Interleave matrix" unchecked, change the "Maximum length of taxon names" to 50, and set "End of line character" to "Unix (LF)". Click Export and save as `28s.phy`.
- Take a look at the alignment to see how it did. Make sure that all the sequences are in the same 
direction (if a sequence is in a different direction, it won't line up with the 
others) and that there are no other conspicuous problems. If some sequences 
are in the wrong direction, copy the original fasta file, replace the 
offending sequences with their reverse compliments (you can generate the 
reverse compliment with one of many online tools, such as 
[this one](http://www.bioinformatics.org/sms/rev_comp.html)). Then realign the 
file with mafft and inspect it again.
- Close the file (you can hit "Don't save" since everything is saved already).


You should now have five files for each gene, eg `28s.raw.fasta`, `28s.fasta`, `28s.aligned.fasta`, `28s.nex` and `28s.phy`.

Questions:

1. Based on eye-balling the alignments, do you think that each gene has a 
   consistent rate of molecular evolution along its full length?

   Clearly not, there are regions with much more incongruence between species than others, while some regions are composed of straight uniform lines of base type. This is probably due to functional constraints.

2. Based on eye-balling the alignments, which gene (16S, 18S, or 28S) do you think has 
   the fastest average rate of molecular evolution? The slowest?

   18s also looks very conseved, I'd say it has the slowest rate of molecular evolution.
   28s looks very uniform in many regions, but it has more variability than 18s.
   16s has more overall variation between species, which suggests it is the gene with the highest evolutionary rates.

### Create concatenated alignments

Now you have alignments for three genes. We will combine the data by creating concatenated alignments that include data for all genes. There are a few ways to do this, we'll use the tool [phyutility](https://github.com/blackrim/phyutility). If you don't have java installed already, you will need to install it (if you try the command below but don't have java, it may describe how to install java on your system). Rather than install java on your laptop, you could just run the command on oscar (which has java installed already)

Execute the following line to create the combined analysis:

    java -jar phyutility.jar -concat -in 16s.aligned.fasta 18s.aligned.fasta 28s.aligned.fasta -out combined.nex

This will create a new nexus file that has data from all three genes. Data are combined in the same row for sequences with the same species names.

You still need to export the file as a phylip file. Open `combined.nex` in Mesquite. Open the "File" menu and select "Export". Then select "Phylip (DNA/RNA)". In the window that pops up, leave "Interleave matrix" unchecked, change the "Maximum length of taxon names" to 50, and set "End of line character" to "Unix (LF)". Click Export and save as `combined.phy`.

Now that you have all your alignment files, go ahead and commit them:

    git add *.nex *.phy
    git commit -am "generated nexus and phylip alignment files"
    git push

### Maximum likelihood analyses

The `raxml.sh` file contains the start of the code for running the analyses. It has lines for the combined and 16s analyses, go ahead and add lines for the 18s and 28s analyses by copying and modifying the 16s line. There are two fields that need to be modified: the name if the input file (specified by `-s`) and the base of the name for the output files (specified with `-n`).

Note that the run time is 24 hours, longer than we used in the previous siphonophore16s analyses. That is because we are analyzing 4 matrices, which will take a bit longer. This script uses 8 CPU cores (specified by `#SBATCH --nodes=1`, `#SBATCH --tasks-per-node=8`, and `mpirun -n 8`) for each raxml command, but doesn't launch a command until the commands before it finish.

Once you have updated the `raxml.sh` file, push the changes:

    git commit -am "added 18s and 28s commands to raxml batch file"
    git push

Log in to oscar, `cd` to the `~/scratch/hydroidolina` repository directory (you will need to clone it to scratch if you haven't already), and then pull changes and launch the run:

    git pull
    sbatch raxml.sh 

Once the run is complete (you can check the status with `myq`), add the new files to the git repository, push them to github, pull them to your laptop, and take a look at the results.

Questions:

1. Do the trees differ from those published? If so, how?

Figure 1: On combined matrix data.

Color key:
-Magenta: Trachilina
-Yellow: Siphonophora
-Cyan: Leptothecata
-Olive: Filifera IV
-Red: Filifera III
-Green: Aplanulata
-Blue: Capitata

[](figure1.svg)

The trees as opened in figtree straight out of the ML analysis are unrooted. The published figures show rooted trees, showing Trachilina as sister group to the Hydroilina clade.
When manually rerooted in figtree to fit the aforementioned constraint, and rotated the branches to resemble the published tree, we can identify further incongruences.

The first logical comparison would be the combined matrix tree with the published Figure 1. Both recognize (((Leptothecates and Siphonophores) and (Filifera III and Filifera IV))  and (Aplanulata and Others) and ()) .

16S doesn't fully recognize siphonophores as a clade! Clausophyes ovata and Sulculeolaria quadrivalvis are placed as a sister group to Rhizogeton nudus.

2. How do the trees for each gene differ from each other?

16S doesn't fully recognize siphonophores as a clade! Clausophyes ovata and Sulculeolaria quadrivalvis are placed as a sister group to Rhizogeton nudus.

3. Take a look at the raxml log files. What do these tells you about the 
   different models of molecular evolution for the four analyses?

   Each gene tree was found estimating different GTR-Gamma molecular evolution model parameters:

   28S:
   		alpha: 0.238208
   		Tree length: 8.26401
		rate A <-> C: 0.578063
		rate A <-> G: 2.084487
		rate A <-> T: 0.666076
		rate C <-> G: 0.728393
		rate C <-> T: 4.537034
		rate G <-> T: 1.000000

		freq pi(A): 0.263767
		freq pi(C): 0.207798
		freq pi(G): 0.283709
		freq pi(T): 0.244726

   18S:
   		alpha: 0.201757
   		Tree length: 3.716563
   		rate A <-> C: 1.395773
		rate A <-> G: 2.974301
		rate A <-> T: 1.277093
		rate C <-> G: 0.992031
		rate C <-> T: 6.531239
		rate G <-> T: 1.000000

		freq pi(A): 0.269513
		freq pi(C): 0.194606
		freq pi(G): 0.260255
		freq pi(T): 0.275625

	16S:
		alpha: 0.462131
		Tree length: 28.790199
		rate A <-> C: 1.749918
		rate A <-> G: 6.177202
		rate A <-> T: 3.800691
		rate C <-> G: 0.685706
		rate C <-> T: 11.188630
		rate G <-> T: 1.000000

		freq pi(A): 0.396763
		freq pi(C): 0.124418
		freq pi(G): 0.157227
		freq pi(T): 0.321593

	Combined:
		alpha: 0.253321
		Tree length: 8.219909
		rate A <-> C: 0.806800
		rate A <-> G: 2.608071
		rate A <-> T: 1.911118
		rate C <-> G: 0.900656
		rate C <-> T: 5.287369
		rate G <-> T: 1.000000

		freq pi(A): 0.277945
		freq pi(C): 0.195911
		freq pi(G): 0.264600
		freq pi(T): 0.261544

18S and 28S had similar Pi values. Tree length seems to correlate with our eye-balling ranking of molecular evolution for each gene. C<->T had the largest rates acroos all tree searches. 
G<->T was constant and equal to 1 in all tree searches.