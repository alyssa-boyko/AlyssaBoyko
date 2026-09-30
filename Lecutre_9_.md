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
