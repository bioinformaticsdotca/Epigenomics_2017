
---
layout: tutorial_page
permalink: /epigenomics_2017_module2_lab
title: Epigenomics Lab 2
header1: Workshop Pages for Students
header2: Epigenomic Data Analysis 2017 Module 2 Lab
image: /site_images/CBW_Epigenome-data_icon.jpg
home: https://bioinformaticsdotca.github.io/high-throughput_biology_2017
---

# Module 2 Lab: ChIP-Seq Alignment, Peak Calling (Enrichment Regions Detection), and Visualisation

By Misha Bilenky

1. Set up some useful variables etc...
```
source /home/partage/epigenomics/chip-seq/setup.sh
```
you can see what variables we defined by
```
less /home/partage/epigenomics/chip-seq/setup.sh
```
2. Create your personal working directory
[for example i create my own working spase:

```
out=/home/partage/epigenomics/chip-seq/H1test
mkdir -p $out
cd $out
```
We will perform alignment of small fastq file (H1 cells, H3K27ac ChIP-seq) to the human reference hg19

Fastq file is located in "/home/partage/epigenomics/chip-seq/H1/data/H3K27ac"
```
less $H1data/H3K27ac/H3K27ac.H1.fastq.gz
```
It is a very small region of genome on chr3
~50K reads.

```
less $H1data/H3K27ac/H3K27ac.H1.fastq.gz
```

3. Human genome is stored in the variable
```
less $hg19/Homo_sapiens.hg19.fa | more
```

4. BWA: alignment.
Lets check that we have bwa installed
```
which bwa
bwa
```

Run first step, "bwa aln"
```
bwa aln $hg19/Homo_sapiens.hg19.fa $H1data/H3K27ac/H3K27ac.H1.fastq.gz > H3K27ac.H1.sai
```

small delay at the start as genome was beeing loaded...
We should see a .sai file
```
ls -l
```
The .sai file is an intermediate file containing the suffix array indexes. Such file is afterwards translated into a SAM file.

NOTE: if we align data from pair-end experiment we need to do alignment of both read1 and read2

bwa aln <genome> read1.fastq > read1.sai
bwa aln <genome> read1.fastq > read2.sai

5. BWA: translation of suffix index file into SAM
```
bwa samse -f H3K27ac.H1.sam $hg19/Homo_sapiens.hg19.fa H3K27ac.H1.sai $H1data/H3K27ac/H3K27ac.H1.fastq.gz
```
See that sam file is there
```
ls -l
less H3K27ac.H1.sam
```

6. Now using "samtools" to manipulate SAM file

Convert into binary BAM file:
```
samtools view -Sb H3K27ac.H1.sam > H3K27ac.H1.bam
```
Position sort:
```
samtools sort H3K27ac.H1.bam H3K27ac.H1.sorted
```
NOTE: if we use option '-n' in "samtools sort" file will be name sorted. Useful for pair-end data; then two reads from the pair are next to each other in the file.

Check:
``` 
ls -lh
```
You see that "BAM" file is ~4-5 times smaller than "SAM"

7. Lets look into alignment file:
File has a header
```
samtools view -H H3K27ac.H1.sorted.bam | more
```
and the aliggned/unuligned reads information
```
samtools view H3K27ac.H1.sorted.bam | head
```
[all fields we discussed]

8. Using BAM file we can obtain useful informtion just using "samtools"
```
samtools view H3K27ac.H1.sorted.bam | wc -l
```
49990
```
samtools view -F 4 H3K27ac.H1.sorted.bam | wc -l
```
31555
```
samtools view -F 4 -q 5 H3K27ac.H1.sorted.bam | wc -l
```
27735
```
samtools view -F 4 -q 5 H3K27ac.H1.sorted.bam | cut -f10 | grep TATA | wc -l
```
2178
```
samtools view -F 4 -q 5 H3K27ac.H1.sorted.bam | cut -f10 | grep TATAA | wc -l
```
718

9. Marking duplicated reads
```
java -jar $picard/MarkDuplicates.jar I=H3K27ac.H1.sorted.bam O=H3K27ac.H1.sorted.dupsMarked.bam M=dups AS=true VALIDATION_STRINGENCY=LENIENT QUIET=true
```

```
samtools view -F 1028 -q 5 H3K27ac.H1.sorted.dupsMarked.bam | wc -l
```
25902

Useful "samtools" option
```
samtools view -X  H3K27ac.H1.sorted.dupsMarked.bam | more
```
shows second field ("SAM flag") in a user readable format.

10. Lets count reads aligned to (+) and (-) strands
```
samtools view -X -F 20 H3K27ac.H1.sorted.dupsMarked.bam | wc -l
```
15995
```
samtools view -X -f16 H3K27ac.H1.sorted.dupsMarked.bam | wc -l
```
15560

Looks ver reasonable: numbers of (+) and (-) reads are very close

11. BAM file statistics:
```
samtools flagstat H3K27ac.H1.sorted.dupsMarked.bam > H3K27ac.H1.sorted.dupsMarked.bam.flagstat
```
check the output
```
more H3K27ac.H1.sorted.dupsMarked.bam.flagstat
```
you should see alignments file details:

49990 + 0 in total (QC-passed reads + QC-failed reads)
1854 + 0 duplicates
31555 + 0 mapped (63.12%:-nan%)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (-nan%:-nan%)
0 + 0 with itself and mate mapped
0 + 0 singletons (-nan%:-nan%)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)













/gs/project/mugqic/bioinformatics.ca/epigenomics/chip-seq/H1/data/H3K27ac/H3K27ac.H1.fastq.gz
``` 

#### Setting up the Variables

```
data=/gs/project/mugqic/bioinformatics.ca/epigenomics/chip-seq/H1/data/
genome=/cvmfs/ref.mugqic/genomes/species/Homo_sapiens.GRCh37/genome/bwa_index/Homo_sapiens.GRCh37.fa
mark=H3K27ac
f=$data/$mark/$mark.H1.fastq.gz
out=~/test/H3K27ac; mkdir -p $out
n=$mark.H1
```

Note: By changing H3K27ac to H3K36me3 you will align for another mark

#### Alignment Step: bwa aln

Try:

```
$BWA aln
```

You will see all the options/parameters and default values

**Important parameters**

Seed length

-l INT    seed length [32]

Number of Mismatches

-k INT    maximum differences in the seed [2]

Try:

```
$BWA aln $genome $f > $out/$n.sai
```

The .sai file is an intermediate file containing the suffix array indexes. Such file is afterwards translated into a SAM file.

**NOTE** if we align data from pair-end experiment we need to do alignment of both read1 and read2


As an example:

```
$BWA aln $genome $f1 > $out/$n1.sai
$BWA aln $genome $f2 > $out/$n2.sai
```

#### Alignment Step: Translation into SAM file

```
$BWA samse -f $out/$n.sam $genome $out/$n.sai $f
```

Resulting is a SAM file with a proper header etc.

To see the file:

```
less $out/$n.sam
```

To quit less, press q.

NOTE in case of pair end data

```
$BWA sampe [options] <genome> <in1.sai> <in2.sai> <in1.fq> <in2.fq> > out.sam
```

#### Alignment Step: Conversion of SAM to BAM and Sorting

Let's define samtools variable:

```
SAMTOOLS=/cvmfs/soft.mugqic/CentOS6/software/samtools/samtools-1.3/bin/samtools
```

And convert the SAM to BAM and sort the BAM

```
$SAMTOOLS view -Sb $out/$n.sam > $out/$n.bam
$SAMTOOLS sort $out/$n.bam > $out/$n.sorted.bam
```

This was *position sorting*; Option *-n* gives *name sorted* BAM file

Now we can view bam file

```
$SAMTOOLS view -h $out/$n.sorted.bam | more
```

Check the size of SAM/BAM files

```
ls -lh $out
```

We can delete intermediate files (sai sam and unsorted bam):

```
rm $out/*.sa*
rm $out/$n.bam 
```

#### Alignment Step: Marking Duplicates with **Picard**

Define location of PICARD jars:

```
PICARD=/cvmfs/soft.mugqic/CentOS6/software/picard/picard-tools-1.123/
```

Mark the duplicates

```
cd $out
java -jar -Xmx20G $PICARD/MarkDuplicates.jar I=$out/$n.sorted.bam O=$out/$n.sorted.dupsMarked.bam M=dups AS=true VALIDATION_STRINGENCY=LENIENT QUIET=true
```

#### Alignment Step: BAM file Statistics and Indexing


```
$SAMTOOLS flagstat $out/$n.sorted.dupsMarked.bam > $out/$n.flagstat
less $out/H3K27ac.H1.flagstat
```

Flagstat contains different BAM file statistics; check the file

```
$SAMTOOLS index $out/$n.sorted.dupsMarked.bam
```

Index $out/$n.sorted.dupsMarked.bam.bai file is generated; it allows samtool access BAM file from a given location

#### Alignment Step: Generating wig File

[**BAM2WIG java tool**](http://www.epigenomes.ca/tools-and-software/index.html)

Define:

```
BAM=$out/H3K27ac.H1.sorted.dupsMarked.bam
SAMTOOLS=/cvmfs/soft.mugqic/CentOS6/software/samtools/samtools-0.1.19/samtools 

BIN=/gs/project/mugqic/bioinformatics.ca/epigenomics/chip-seq/bin/

java -jar -Xmx2G $BIN/BAM2WIG.jar -bamFile $BAM -out $out -q 5 -F 1028 -cs -x 150 -samtools $SAMTOOLS > $out/wig.log

cat $out/wig.log

zcat $out/H3K27ac.H1.sorted.dupsMarked.q5.F1028.SET_150.wig.gz | head
```

**Note**: if wig.log says `Output dir is not defined` try retyping the java command (do not copy and paste it).

#### Alignment Step: Visualization

We would need to add a track header to the file

```
cd $out
gunzip *.wig.gz
H=/gs/project/mugqic/bioinformatics.ca/epigenomics/chip-seq/H1/data/H3K27ac/wig_track_header
cp $H $out/test.wig
less $out/H3K27ac.H1.sorted.dupsMarked.q5.F1028.SET_150.wig >> $out/test.wig
gzip $out/test.wig
zcat $out/test.wig.gz | head
```
Move test.wig to your local computer using WinSCP or scp.

In your browser, open <http://genome.ucsc.edu>

Select genomes and choose hg19.

Click the *Manage custom tracks* button.

*Add custom track* and browse to $OUT/test.wig.gz

![Region of interest](https://github.com/bioinformatics-ca/bioinformatics-ca.github.io/blob/master/2016_workshops/epigenomics/img/img2.png?raw=true)

Browse to the region of interest: chr3:43375889-45912052

### Enriched Regions

We should use full size bam files in order to call enrichments (or at least bam file for a whole chromosome) as tools infer different distributions from the data and we need to have enough statistics

#### Run FindER on H3K27ac (just chr3)

First, set some variables.

```
mark=H3K27ac
dir=/gs/project/mugqic/bioinformatics.ca/epigenomics/chip-seq/H1/
sig=$dir/original/chr3/$mark.H1.bam
inp=$dir/original/chr3/Input.H1.bam
BIN=/gs/project/mugqic/bioinformatics.ca/epigenomics/chip-seq/bin/
out=~/test/FindER; mkdir -p $out
SAMTOOLS=/cvmfs/soft.mugqic/CentOS6/software/samtools/samtools-0.1.19/samtools
```
Now, run the command.

```
java -jar -Xmx10G $BIN/FindER_1_0_0.jar -signalBam $sig -inputBam $inp -SE -xsetI 150 -xsetS 150 -bin 150 -v -out $out -samtools $SAMTOOLS -regions 3 &> $out/$mark.log
```

#### Run MACS2

Setting up Python environment

```
export PATH=/cvmfs/soft.mugqic/CentOS6/software/python/Python-2.7.8/bin:/cvmfs/soft.mugqic/CentOS6/software/MACS2/MACS2-2.1.0.20151222/bin:$PATH
export PYTHONPATH=/cvmfs/soft.mugqic/CentOS6/software/MACS2/MACS2-2.1.0.20151222/lib/python2.7/site-packages:$PYTHONPATH
```

```
out=~/test/macs2; mkdir -p $out

macs2 callpeak -t $sig -c $inp -f BAM -g hs -n $out/$mark.v.Input.chr3 -B -q 0.01 &> $out/$mark.v.Input.chr3.log
```

Running macs in a standard (narrow peak mode) with a q-value threshold 0.01

Broad mode (for H3K36me3, H3K27me3, for example)

For example:

```
macs2 callpeak -t ChIP.bam -c Control.bam --broad -g hs --broad-cutoff 0.1
# where ChIP.bam is your ChIP-Seq bam and Control.bam is your control input bam
```

#### Load MACS2/FindER track to UCSC

Post-process mac2 file

```
cd $out
cp /gs/project/mugqic/bioinformatics.ca/epigenomics/chip-seq/H1/macs2/macs_track macs2.bed
less H3K27ac.v.Input.chr3_peaks.narrowPeak | cut -f1-3 | awk '{print "chr"$0}' >> macs2.bed
```

Move both files to your local computer using WinSCP or scp.

Now load both files into UCSC

macs output file: macs2.bed

FindER output file: H3K27ac.H1.vs.Input.H1.bin_150.FDR_0.05.FindER.bed.gz


You should now see BED tracks.

![BED added](https://github.com/bioinformatics-ca/bioinformatics-ca.github.io/blob/master/2016_workshops/epigenomics/img/img3.png?raw=true)
