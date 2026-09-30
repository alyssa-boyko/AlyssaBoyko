## Check sample names

```
conda activate bio_env
bcftools query -l body_size.vcf
```

## Compress and index the VCF

```
bgzip body_size.vcf
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
bcftools view -S group2.txt body_size.vcf.gz -Oz -o group2.vcf.gz
```

## Filter each group's VCF to strictly biallelic SNPs first

```
bcftools view -m2 -M2 -v snps group1.vcf.gz -Oz -o group1_biallelic.vcf.gz
bcftools view -m2 -M2 -v snps group2.vcf.gz -Oz -o group2_biallelic.vcf.gz
```

## Calculate allele frequency within each group

```
bcftools +fill-tags group1_biallelic.vcf.gz -Oz -o group1_af.vcf.gz -- -t AF
bcftools +fill-tags group2_biallelic.vcf.gz -Oz -o group2_af.vcf.gz -- -t AF
```

## Pull out just CHROM, POS, and AF

```
bcftools query -f '%CHROM\t%POS\t%INFO/AF\n' group1_af.vcf.gz > group1_af.tsv
bcftools query -f '%CHROM\t%POS\t%INFO/AF\n' group2_af.vcf.gz > group2_af.tsv
```

## Merge, compute the difference, and plot in R

```
R

# This creates names for the columns, c denotes a list
g1 <- read.table("group1_af.tsv", col.names = c("CHROM", "POS", "AF1"))
g2 <- read.table("group2_af.tsv", col.names = c("CHROM", "POS", "AF2"))


# merge on shared sites only
merged <- merge(g1, g2, by = c("CHROM", "POS"))

# drop any sites where AF couldn't be calculated in one group
# (e.g. no called genotypes in that subset)

merged <- na.omit(merged)
merged$AF_diff <- merged$AF1 - merged$AF2

# make a plot of allele frequency differences
pdf('merged.pdf')

# Always list in x,y order and label your plot and axes
plot(merged$POS, merged$AF_diff,
     pch = 19, col = "steelblue",
     xlab = "Position in gene", ylab = "Allele frequency difference (Group1 - Group2)",
     main = "Allele frequency difference along bbc")
abline(h = 0, lty = 2, col = "grey40")

dev.off()
```

## Download your plot

# You should be working in your computer's terminal, not in the class server

scp -r visitor@134.129.113.23:/storehouse/visitor/table_/pigmentation/merged.pdf .

# Type open when you download it
