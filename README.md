# UMI_analysis
### Read 1 = BC, Read 2 = UMI

### seq analysis

/net/fowler/vol1/home/jjsteph/hts/MAC/170807_NS500488_0436_AHMKL3AFXX

### Split 

bcl2fastq/2.19 

###create sample sheet###

bcl2fastq --minimum-trimmed-read-length 0 --mask-short-adapter-reads 0 --no-lane-splitting  


### gunzip files

gunzip *.gz

### trim using:

module load fastx-toolkit/latest

### trimming both reads to size 18bp (if not that size to begin) ###

for sample in $(grep -v "^#" barcodes.grp | cut -f 1); do  fastx_trimmer -l 18 -Q33 -i ${sample}_R1_001.fastq -o ${sample}_R1.trimmed.fastq; fastx_trimmer -l 18 -Q33 -i ${sample}_R2_001.fastq -o ${sample}_R2.trimmed.fastq; done

### at this point R2 in std amplicons should be almost all 18 G’s in a row as they have not been read 

###UMIs:

### gzip yer files

gzip *.fatsq

### mash files together ( 18bp barcdode + 18bp UMI) and uniquify reads based on UMI ###

### for single samples###

paste <( zcat /net/fowler/vol1/home/jjsteph/hts/Temporary/170808_NS500272_0340_AHJTJYAFXX/Data/Intensities/BaseCalls/20170614_LLa_UMI_R1.fq.gz) <( zcat /net/fowler/vol1/home/jjsteph/hts/Temporary/170808_NS500272_0340_AHJTJYAFXX/Data/Intensities/BaseCalls/20170614_LLa_UMI_R2.fq.gz) |awk '{ count+=1; if (count == 2) { print $1$2 }; if (count == 4) { count=0 }}' | sort | uniq -c | awk '{ print "@"2"\n"substr($2,0,18)"\n+\n"substr($2,0,18) }' | gzip -c > 20170614_LLa_UMI.fq.gz


###for multiple samples###

for sample in $(grep -v "^#" barcodes.grp | cut -f 1); do paste <( zcat ${sample}_R1.trimmed.fastq.gz) <( zcat ${sample}_R2.trimmed.fastq.gz) | awk '{ count+=1; if (count == 2) { print $1$2 }; if (count == 4) { count=0 }}' | sort | uniq -c | awk '{ print "@"2"\n"substr($2,0,18)"\n+\n"substr($2,0,18) }' | gzip -c > ${sample}.fastq.gz; done

### download fq.gz files and run through enrich, barcodes only as individual time points at 0

