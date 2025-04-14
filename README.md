# Circos-plot
#### Sean Chien
This tutorial is to create a circos plot with two genomes (reference / de novo) and add some features (ex: read depth) to inner rings \
Here is the example \
<img src="https://github.com/user-attachments/assets/4d534f6c-6b25-4a8f-bd0a-668e0f51324f" alt="circos" width="600" height="600">
## Karyotype data
The format for this data is \
chr - ID LABEL START END COLOR

##### checking contig size and determine cutoff
```bash
seqkit sort -l genome.fa | awk '/^>/ {if (seqlen){print seqlen}; print ;seqlen=0;next; } { seqlen += length($0)}END{print seqlen}' 
```

##### remove small scaffolds and rename to chromosome_#
```bash
seqkit seq -m 10000000 genome.fa > genome.fa.final
```

###### reorder the contig by size
```bash
seqkit sort --by-length --reverse contigs.fasta > genome.fa.final_sort
```

###### generating karyotype files
```bash
bash karyotypeify.sh -g genome.fa.final_sort -k genome_karyotype.txt
```
###### change the color to contig names so later on we can change name base on the contigs
```bash
awk '{ if ($7 == "black") { $7 = $3 } print }' genome_karyotype.txt
```

## Alignment data
We are using minimap2 here. Please note that minimap2 also maps repeats (soft-masked genome)
If you use soft-maksed genome, when you make your circos plot you will see all scaffolds link to all scaffolds.
This is because the repeats are all over the genomes.

###### Hard-masking 
Here we can turn lowercases into N if you are using soft-masked genomes. 
If you are using hard-maksed genomes. You do not need to do this part. 
```bash
awk '/^>/ {print; next} {gsub(/[a-z]/, "N"); print}' genome.fa.final_sort > genome.fa.final_sort_hard
```

###### Mapping wiht Minimap2
```bash
minimap2 genome1.fa.final_sort_hard genome2.fa.final_sort > genome1_genome2_minimap2.paf &
```

###### generating bundles and links
```bash
awk '{print $6,$8,$9,$1,$3,$4}' genome1_genome2_minimap2.paf > genome1_genome2.tsv
```
###### generating colors for links 
I create a file that first column is contig name and second column is coresponse color 

```bash
awk 'NR==FNR{colors[$1]=$2; next} $1 in colors{$7="color="colors[$1]}1' contig_color.txt genome1_genome2.tsv > genome1_genom2.tsv_color
```

Adjust radius of links  
The reason I do this becasue I only want to show read depth in one side (the genome I asssemble, not the left side the genome I download from NCBI)


```bash


```

## Read-depth data

I map HIFi reads back the genome assembly and normalize the read depth. 
For heterogamete sex (male here), male should have half coverage in sex chromosomes (X&Y)

First we map reads back to the genome and calcualte the read depth in all positions. 




```python
python3 cal_wind_avg.py loci_depth.txt 1000000 > loci_win_depth.txt
```




