# <img src="https://raw.githubusercontent.com/Tarikul-Islam-Anik/Telegram-Animated-Emojis/main/Objects/Books.webp" alt="Books" width="25" height="25" /> A Cloud-Based Variant Calling Wokrflow <img src="https://raw.githubusercontent.com/Tarikul-Islam-Anik/Telegram-Animated-Emojis/main/Objects/Books.webp" alt="Books" width="25" height="25" />

## Code
---
### 1. BWA (Burrows-Wheeler Aligner) MEM (matching extension) algorithm & Samtools

```
bwa mem -t 8 -M -Y -K 100000000 \
 -R '@RG\tID:NA12878\tPL:ILLUMINA\tPU:NA12878\tSM:NA12878\tLB:NA12878' \
/home/rocky/Data/hg38/hg38.fa \
<(zcat /home/rocky/Data/SRR1518158/SRR1518158_sub_1.fastq.gz) \
<(zcat /home/rocky/Data/SRR1518158/SRR1518158_sub_2.fastq.gz) \
| samtools view -huS - \
| samtools sort -@ 2 -m 2G -o SRR1518158_sub.bam -O bam -T SRR1518158_sub.tmp
```


### 2. Markduplicates

```
java -jar /home/rocky/Packages/picard.jar MarkDuplicates \
I=SRR1518158_sub.bam \
O=dedup.SRR1518158_sub.bam \
M=markdups_SRR1518158_sub.txt \
ASSUME_SORT_ORDER=queryname \
MAX_RECORDS_IN_RAM=2000000 \
COMPRESSION_LEVEL=1 \
CREATE_INDEX=true \
VALIDATION_STRINGENCY=SILENT \
PROGRAM_RECORD_ID=null \
ADD_PG_TAG_TO_READS=false \
READ_NAME_REGEX=null 
```
```
samtools index dedup.SRR1518158_sub.bam
```

### 3. Base Quality Score Recalibration (BQSR)

```
gatk BaseRecalibrator \
-R /home/rocky/Data/hg38/hg38.fa \
-I dedup.SRR1518158_sub.bam \
--known-sites /home/rocky/Data/hg38/dbsnp_146.hg38.vcf.gz \
--known-sites /home/rocky/Data/hg38/1000G_phase1.snps.high_confidence.hg38.vcf.gz \
--known-sites /home/rocky/Data/hg38/Mills_and_1000G_gold_standard.indels.hg38.vcf.gz \
-O SRR1518158_sub.recal.table
```

```
gatk ApplyBQSR \
-R /home/rocky/Data/hg38/hg38.fa \
-I dedup.SRR1518158_sub.bam \
-O SRR1518158_sub.cram \
-bqsr ./SRR1518158_sub.recal.table 
```

```
samtools index SRR1518158_sub.cram 
```



### 4. HaplotypeCaller

```
gatk HaplotypeCaller \
-R /home/rocky/Data/hg38/hg38.fa \
-I SRR1518158_sub.cram \
-O SRR1518158_sub.vcf.gz \
-L /home/rocky/Data/hg38/hg38.refGene.exon.bed.gz 
```


### 5. Build GenomicDB (replaces CombineGVCFs) & Joint Genotyping 
이번 실습에서는 진행하지 않음
```
gatk GenomicDBImport \
	-V [vcf file_list] \ 
	-L [targets.interval_list] \ 
	--genomicsdb-workspace-path [my_database] \
	--tmp-dir=/path/to/large/tmp
```

```
gatk GenotypeGVCFs \ 
	-R [reference.fa] \ 
	-V gendb://genomicDB \
	-O [cohort.vcf]
```


### 6. Variant Quality Score Recalibration (VQSR) 
```
gatk VariantRecalibrator \
-R /home/rocky/Data/hg38/hg38.fa \
-V SRR1518158_sub.vcf.gz \
-O SRR1518158_sub.raw.recal \
--tranches-file SRR1518158_sub.raw.tranches \
--resource:hapmap,known=false,training=true,truth=true,prior=15.0 /home/rocky/Data/hg38/hapmap_3.3.hg38.vcf.gz \
--resource:mills,known=false,training=true,truth=true,prior=12.0 /home/rocky/Data/hg38/Mills_and_1000G_gold_standard.indels.hg38.vcf.gz \
--resource:dbsnp,known=true,training=false,truth=false,prior=2.0 /home/rocky/Data/hg38/dbsnp_146.hg38.vcf.gz \
-an QD -an MQ -an MQRankSum -an ReadPosRankSum -an FS -an SOR \
-mode BOTH
```

```
gatk ApplyVQSR \
-R /home/rocky/Data/hg38/hg38.fa \
-V SRR1518158_sub.vcf.gz \
--recal-file SRR1518158_sub.raw.recal \
--tranches-file SRR1518158_sub.raw.tranches \
-O SRR1518158_sub.recal.vcf.gz \
-ts-filter-level 99.5 \
-mode BOTH
```


### 7. Annotate variants

```
perl /home/rocky/Packages/exercise1/table_annovar.pl \
--vcfinput SRR1518158_sub.recal.vcf.gz /home/rocky/Packages/exercise1/humandb/ -buildver hg38 \
-out SRR1518158_sub_calls_annot \
-remove \
-protocol refGeneWithVer,gnomad41_exome,clinvar_20240917,intervar_20180118,alphamissense \
-operation gx,f,f,f,f \
-nastring . \
-polish
```
