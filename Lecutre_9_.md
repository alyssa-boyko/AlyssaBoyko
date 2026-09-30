## Check sample names

```
conda activate bio_env
```

```
bcftools query -l body_size.vcf
```

## Compress and index the VCF

```
bgzip body_size.vcf
```

```
bcftools index body_size.vcf.gz
```

## Define your two groups by making two plain text files, one sample name per line, matching exactly what bcftools query -l printed

```
# group1.txt
SRR10729165.sorted.bam
SRR10729166.sorted.bam
SRR10729566.sorted.bam
SRR10733526.sorted.bam
```

```
# group2.txt
SRR31835375.sorted.bam
SRR31835473.sorted.bam
SRR31835482.sorted.bam
SRR31835573.sorted.bam
```
## Split the VCF by group

```
bcftools view -S group1.txt body_size.vcf.gz -Oz -o group1.vcf.gz
```

```
bcftools view -S group2.txt body_size.vcf.gz -Oz -o group2.vcf.gz
```
