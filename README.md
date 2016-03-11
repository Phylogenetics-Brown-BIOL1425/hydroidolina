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

   No! There appear to be highly conserved regions, where the sequence is identical across species, and other sites that are highly polymorphic. Rate of molecular evolution here is synonymous with the degree of polymorphism/sequence divergence in these three genes across species.  

2. Based on eye-balling the alignments, which gene (16S, 18S, or 28S) do you think has 
   the fastest average rate of molecular evolution? The slowest?

   16s appears to have the fastest rate, while 28s seems to have the slowest rate of evolution.

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
  YES, the trees differ greatly from those published! see below:
  * 16s to published
    * no support for Capitata
    * support for Trachylina
    * support for Aplantulata
    * support for Siphonophorae
        * except for S. quadrivalvis and C. ovata
    * support for Leptothecata
    * support for Filifera I
    * support for Filifera II
        * except for H. epigorgia, which moved into Leptothecata
    * Filifera III polyphyletic
        * C. gallensis is within Leptothecata
        * paraphyletic
    * Filifera IV paraphyletic
        * Bougainvilliidae and Oceanside group together
        * Rathkeidae and R. nudus outside
* 18s to published
    * Siphonophorae paraphyletic
    * unclear from graph whether capitata monophyletic or not
    * support for aplantulata
    * support for trachylina
    * no support for Filifera I
    * support for Filifera II
    * support for Filifera III
        * C. gallensis included, unlike in 16s
    * Filifera IV paraphyletic
        * includes Filifera II and Filifera III
    * support for Leptothecata
* 28s to published
    * Capitata paraphyletic
        * the parent group
    * support for Siphonophorae
    * support for Lepto
    * support for trachylina
    * support for aplantula
    * support for Filifera I
    * support for Filifera II
    * support for Filifera III
    * Filifera IV paraphyletic
        * includes Filifera III
* combined to published
    * support for capitata
    * support for siphonophore
    * support for Lepto
    * support for trachylina
    * support for aplantula
    * support for Filifera I
    * support for Filifera II
    * support for Filifera IIV
    * Filifera III paraphyletic
        * parent group
* differences in my generated phylogenies to those published in paper
        * strong support from the combined data phylogeny for the clades outlined in the paper, but none of the phylogenies, not even the most clade-confident, combined phylogeny show Hydroidolina as sister to Trachylina. Trachylina is nested deep within the combined phylogeny behind high-confidence nodes, and Filifera III appears to be sister to the rest of the phylogeny. For that reason, I contest Cartwright’s conclusion that “Hydroidolina” is a monophyletic group. The topology of the phylogeny past these high confidence nodes is poorly supported; bootstrap values at nodes separating clades from one another are very low. This means that the relationships shallower than the the Filifera III-else bipartition are up in the air, but Filifera III is a clear outgroup.
2. How do the trees for each gene differ from each other?
    * Support for clades by gene tree:
        * Trachylina: 16s, 18s, 28s
        * Aplantulata: 16s, 18s, 28s
        * Capitata: 18s (unclear whether polytomy or realllllly small branches), 28s
        * Siphonophorae: 28s
            * 16s largely supports the clade, but S. quadrivalvis and C. ovata are grouped outside with Rathkeidae from Filifera IV
                * low support
            * 18s shows Siphonophorae as paraphyletic and parent to all clades
        * Leopothecata: 18s, 28s
            * 16s Leptothecata contains H. epigorgia and C. gallensis, making it paraphyletic 
        * Filifera I: 16s, 28s
        * Filifera II: 18s, 28s
            * 16s largely supports the clade, but H. epigorgia moved into Leptothecata
        * Filifera III: 18s, 28s
            * C. gallensis is within Leptothecata in 16s
        * Filifera IV: only supported by combined 
        * Hydroidolina: not supported by any gene or combined phylogeny
        * combined tree supports all clades except Hydroidolina and Filifera III
        * bootstrap values are generally low, but a few relationships are well supported with bootstrap values in some trees
            * 16s:
                * Aplantulata sister to Filifera II
            * 28s:
                * Aplantulata as sister to branch containing all clades except Capitata
3. Take a look at the raxml log files. What do these tells you about the 
   different models of molecular evolution for the four analyses?
      * significance of model parameters
        * Tree length:
            * 16s has the highest tree length, indicating that it has undergone the most change throughout its evolutionary history.
            * 18s appears to have the shortest tree length, indicating that it has undergone the least change.
                * This conflicts with my “eyeballing” of the original alignments, but I guess that’s why we have computers…
        * alpha
            * 16s has an alpha about double that of 18s and 28s, meaning that its sites have more homogenous substitution rates. Maybe all sites in 16s are evolving quickly, while in 18s and 28s there are some sites evolving quickly and others more slowly (or even fixed).
        * frequency of C-G vs A-T
            * the equilibrium frequencies if A and T are fairly similar, while the frequencies of C and G are dramatically different. Given that they must pair, I wonder how the equilibrium frequency of G is allowed to so greatly exceed that of C. 