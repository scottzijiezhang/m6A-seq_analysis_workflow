# m6A-seq_analysis_flow
A pipeline to process m6A-seq data and down stream analysis. 

## Prepare data for analysis
### 1. Download the data from the genomic core. 
You can use [filezilla](https://filezilla-project.org/download.php?type=client) to download the data from che@osrfftp.uchicago.edu (port: 21) and upload it to **youraccount**@128.135.225.178 (port: 22). 
Alternatively, you can ssh log into **youraccount**@128.135.225.178 and go to the directory by `cd /directory of your favorite` where you want to analyze your data. Then use the `sftp ` to directly download the data from sequencing facility server to the lab analysis server. 

### 2. Organize and rename files for processing
In order to be easily processed, rename the files to short names without space and postfix `.IN.fastq.gz` or `.m6A.fastq.gz` for input and IP respectively. 
For example, 
```
Ctl1.IN.fastq.gz
Ctl1.m6A.fastq.gz
...
FTO_KO5.IN.fastq.gz
FTO_KO5.m6A.fastq.gz
```

## Map the data to the mycoplasma genome to check for potential comtamination.
We are goint to use fast and sensitive alignment program **Hisat2** to align the reads to the mycoplasma genome. You can go to official page of [Hisat2](https://ccb.jhu.edu/software/hisat2/index.shtml) to learn more about the software and parameters setting. 
Mapping can be done at command line by excuting the Hisat2 command one by one for each sample. Alternatively, we use a bash file to wrap the Hisat2 command to loop through all our samples to save us some effort to keep excuting hisat2 command. 
You can use any text editor to make the bash file and save it to **something.sh**. For example, in unix command line, use vim to creat and edit `vi run_alignment.sh` and type `i` to insert content. After entering the content, hit `Esc` to quit insertion mode and `Shift` + `:` + `wq` + `Enter` to save and quit. 
An example bash file is shown below. You use this template and repalce the sample names by yours. 
```
INDEX="$HOME/Database/genome/contamination/mycoplasma"
Data="$HOME/path_to/your_favorite_directory"

mySample="sample1 sample2 sample3 sample4 treated1 treated2 treated3 treated4 treated5"
for s in $mySample
do 

hisat2 -x $INDEX -k 7 --un-gz $s.IN.noMyco.fastq.gz --summary-file $s.IN.myco_summary -p 10 -U $Data/$s.IN.fastq.gz > $s.myco.input.sam

hisat2 -x $INDEX -k 7  --un-gz $s.m6A.noMyco.fastq.gz --summary-file $s.m6A.myco_summary -p 10 -U $Data/$s.m6A.fastq.gz > $s.myco.m6A.sam
## remove alignment result files because we don't really need alignment to mycoplasma, we only want to see alignment summary
rm $s.myco.input.sam
rm $s.myco.m6A.sam 

wait
done
```
Then excute the bash file in the command line by `bash run_alignment.sh`. 
The code above will align reads to mycoplasma genome and output alignment summary. You can check the alignment summary to assess the potential contamination by Mycoplasma.
It will also output reads that doesn't align to mycoplasma nameed as *samplename.IN.noMyco.fastq.gz*. This will be unmmaped reads filtering out mycoplasma reads, which will be input for next round of alignment to human/mouse/whatever_organism reference genome.

