# <img src="https://raw.githubusercontent.com/Tarikul-Islam-Anik/Telegram-Animated-Emojis/main/Objects/Books.webp" alt="Books" width="25" height="25" /> A Cloud-Based Variant Calling Wokrflow <img src="https://raw.githubusercontent.com/Tarikul-Islam-Anik/Telegram-Animated-Emojis/main/Objects/Books.webp" alt="Books" width="25" height="25" />

## Code
---
### 1. BWA (Burrows-Wheeler Aligner) MEM (matching extension) algorithm & Samtools

```
bwa mem -t 8 -M -Y -K 100000000 \
 -R '@RG\tID:NA12878\tPL:ILLUMINA\tPU:NA12878\tSM:NA12878\tLB:NA12878’ \
/home/rocky/Data/hg38/hg38.fa \
<(zcat /home/rocky/Data/SRR1518158/SRR1518158_sub_1.fastq.gz) \
<(zcat /home/rocky/Data/SRR1518158/SRR1518158_sub_2.fastq.gz) \
| samtools view -huS - \
| samtools sort -@ 2 -m 2G -o SRR1518158_sub.bam -O bam -T SRR1518158_sub.tmp
```
