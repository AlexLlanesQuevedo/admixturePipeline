[![DOI](https://zenodo.org/badge/96546673.svg)](https://zenodo.org/badge/latestdoi/96546673)

# Admixture Pipeline: A Method for Parsing and Filtering VCF Files for Admixture Analysis
A pipeline that accepts a VCF file to run through Admixture

## Citing Admixture Pipeline
A manuscript will be prepared describing this method. For now, cite this github repository.

S.M. Mussmann, T.K. Chafin 2019. Admixture Pipeline: A Method for Parsing and Filtering VCF Files for Admixture Analysis. DOI: 10.5281/zenodo.3270852

## Installation & Setup for admixturePipeline.py:

This pipeline was written to be run on Unix based operating systems, such as the various Linux distributions and Mac OS X.  To get started, clone this project to the desired location on your computer.  

This pipeline has three dependencies that must be installed:
* **PLINK 1.9 beta 4.5 or newer** (https://www.cog-genomics.org/plink2)
* **VCFtools** (https://vcftools.github.io/index.html)
* **Admixture** (http://software.genetics.ucla.edu/admixture/download.html)

It is advised that you install the latest version of each program manually. For example, admixturePipeline.py utilizes options in PLINK and VCFtools that are not present in the versions curated within the standard Ubuntu repositories. Each program should be added to your $PATH as the lowercase version of its name (i.e., plink, vcftools, admixture).

You may also have to modify the first line of the admixturePipeline.py file, which by default reads:
```
#!/usr/bin/env Python
```

To find the location of your Python installation, you can type the following at the bash command prompt:
```
which python
```
Then modify the first line of admixturePipeline.py to reflect the location of your Python installation.

## Running admixturePipeline.py:

You can run the program to print help options with the following command:

```
./admixturePipeline.py -h
```

List of current required options:
* **-m / --popmap:** Specify a tab-delimited population map (sample --> population).  This will be converted to a population list that can be input into a pipeline such as CLUMPAK (http://clumpak.tau.ac.il/) for visualization of data
* **-v / --vcf:** Specify a VCF file for input.

Optional arguments:
* **-n / --np:** Specify the number of processors.  Currently the only multithreaded program is Admixture.

Admixture optional arguments:
* **-k / --minK:** Specify the minimum K value to be tested (default = 1).
* **-K / --maxK:** Specify the maximum K value to be tested (default = 20).
* **-c / --cv:** Specify the cross-validation number for the admixture program.  See the admixture program manual for more information (default = 20)
* **-R / --rep:** Specify the number of replicates for each K value (default = 20)

VCFtools optional arguments:
* **-a / --maf:** Enter a minimum frequency for the minor allele frequency filter. (default = off, specify a value between 0.0 and 1.0 to turn it on).
* **-b / --bi:** Turns biallelic filter on/off. (default = off, turn on to recover only biallelic SNPs)  
* **-r / --remove:** Provide a blacklist of individuals that will be filtered out by VCFtools. This is a textfile with each name on its own line. Names of individuals must match those in the .vcf file exactly. 
* **-t / --thin:** Filter loci by thinning out any loci falling within the specified proximity to one another, measured in basepairs.  (default = off, specify an integer greater than 0 to turn it on).
* **-C / --indcov:** Filter samples based on maximum allowable missing data. Feature added by tkchafin. (default = 0.9, input = float). 
* **-S / --snpcov:** Filter SNPs based on proportion of allowable missing data. Feature added by tkchafin. (default = 0.1; defined to be between 0 and 1, where 0 allows sites that are completely missing and 1 indicates no missing data allowed; input = float).

## Example:

The following command will run the program from K values 1 through 10, conducting 10 repetitions at each K value.  Admixture will use all 16 processors available on the hypothetical machine, VCFtools will filter SNPs at an interval of 100bp, and the minor allele frequency filter in VCFtools will drop any loci with a minor allele frequency less than 0.05:

```
admixturePipeline.py -m popmap.txt -v input.vcf -k 1 -K 10 -n 16 -t 100 -a 0.05
```

## Outputs:

For the example line of code above, the following outputs will be produced:
* **input.ped**, **input.map**: output of plink
* **results.zip**: a compressed file that can be input into a pipeline such as CLUMPAK
* **loglik.txt**: a file containing the log likelihood values of each iteration of each K value.
* **input.{k}\_{r}.P** and **input.{k}\_{r}.Q**: Admixture output files for each iteration{r} of each K{k} value
* **input\_pops.txt**: a list of population data that can be input into a pipeline such as CLUMPAK
* **input.recode.strct_in**: a structure-formatted file of filtered SNPs

Once you have finished running this stage of the pipeline, you can submit the two above files designated as CLUMPAK inputs to the online resource CLUMPAK (http://clumpak.tau.ac.il/). Once that analysis finishes, you can continue on with the pipeline using distructRerun.py

# distructRerun.py

This code was written to help streamline the process of re-running distruct on the major clusters that are found by CLUMPAK .  This code was written with the intention of operating on CLUMPAK analysis of ADMIXTURE data, however an option has been added that will allow you to run this section of the pipeline on CLUMPAK analysis of STRUCTURE data.

## Installation & Setup for distructRerun.py:

distructRerun.py has a single dependency that should be installed:
* **distruct** (https://rosenberglab.stanford.edu/distructDownload.html)

It is advised that you install distruct manually. It should be added to your $PATH as the lowercase version of its name (i.e., distruct).

## Usage:

Download your results from the CLUMPAK server.  This should give you a zipped folder of your results, named something like "1516030453.zip".  First, unzip this folder:
```
unzip 1516030453.zip
```
This should produce a folder in your current directory named 1516030453.  Now, run distructRerun.py on your folder.  Assuming that you have installed distruct-rerun.py somewhere in your path, the command will be something like the below command.  In this example, -a is used to provide the path to the directory of results produced by admixturePipeline.py, -d is used to give the name of the directory that the program will use as input, -k (lower case) specifies the lowest clustering value that you tested in Admixture, and -K (upper case) specifies the highest clustering value you tested.

```
distructRerun.py -a example_admixturePipeline_result/ -d 1516030453/ -k 1 -K 12
```
This should have produced a file named MajorClusterRuns.txt in the directory from which you executed distructRerun.py.  This file contains all of the names of the .stdout files produced by my admixturePipeline repository that correspond to each of the major clusters recovered by CLUMPAK. You should also have a file named cv_file.txt that contains all of the CV values for your major cluster runs.  Finally, distruct will return a postscript file (.ps) for the major cluster of each K value that you evaluated. The rest of the processing can be accomplished through my admixture_cv_sum repository.

List of current options:
* **-a / --ad:** Specify the directory containing the output of your admixturePipeline.py run (required).
* **-d / --directory:** Specify the directory containing the output of your CLUMPAK run (required).
* **-k / --minK:** Specify the minimum K value to be tested (required).
* **-K / --maxK:** Specify the maximum K value to be tested (required).
* **-m / --mc:** Provide the name of the output file that will hold the names of runs corresponding to the major clusters (optional; default = MajorClusterRuns.txt)
* **-w / --width:** Provide the width of each individual bar in the distruct output (optional; default = 4).

## Outputs:

The following outputs will be produced in the directory where distructRerun.py was executed:
* **MajorClusterRuns.txt**: contains all of the names of the .stdout files produced by admixturePipeline.py that correspond to each of the major clusters recovered by CLUMPAK.
* **cv_file.txt**: CV values for all of the major clusters
