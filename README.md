#snpflip

snpflip finds reverse and ambiguous strand SNPs.

##Main use cases

- **Generate a list of SNPs not on the reference strand:** Many bioinformatics applications require the SNPs to be on the reference strand. By running snpflip on your GWAS data you'll find the SNPs that need to be flipped.

- **Quality controlling your GWAS data and pruning it:** If your GWAS data contains ambiguous SNPs, these might ruin the usefulness of your data for certain purposes (e.g., imputation). By running snpflip you can get the names of the ambiguous SNPs and remove them with Plink.

##Install
`pip install snpflip`

##Usage

```
snpflip

Report reverse and ambiguous strand SNPs.
(Visit github.com/endrebak/snpflip for examples and help.)

Usage:
    snpflip --fasta-genome=FA --bim-file=BIM [--output-prefix=PRE]
    snpflip --help

Arguments:
    -f FA --fasta-genome=FA     fasta genome
    -b BIM --bim-file=BIM       plink bed file (extended variant information)

Options:
    -h --help                   show this message
    -o PRE --output-prefix=PRE  the prefix of the output-files
                                (./snpflip_output/<bim_basename> by default)

Note:
    To enable snpflip to match the chromosomes in the `.bim` and `.fa` files, the
    chromosomes in the `.bim` file must be called `1, 2, ... X, Y, M`
    while the chromosomes in the fasta file must be called the same, or
    `chr1, chr2, ... chrX, chrY, chrM`
```

##Output

The output files are:
- `<prefix>.reverse` - The SNPs that are on the reverse strand.
- `<prefix>.ambiguous` - The SNPs that cannot be assigned to a strand.
- `<prefix>.annotated_bim` - Strand annotated bim table.

The `.reverse` and `.ambiguous` output files can be used as input to Plink. This is convenient if you want to remove the ambiguous strand SNPs or flip the SNPs that are on the reverse strand.

####Example usage with plink:

```snipflip -b snp_data.bim -f genome.fa -op snpflip_output```

```plink --file snp_data --flip snpflip_output.reverse --make-bed```

##Extended example

This example uses the example files in the example_files catalog.

```
$ cat example_files/example.fa
>chr1
ACT
>chr2
CCC
>chrY
ACT

$ cat example_files/example.bim
1 snp1 0 1 A C
1 snp2 0 2 A T
1 snp3 0 3 A G
2 snp4 0 1 A G
2 esv5 0 2 AA G
Y snp6 0 2 A G

$ snpflip -b examples/example.bim -f \
 examples/example.fa -op extended_example

$ head extended_example.annotated_bim
chromosome	snp_name	genetic_distance	position	allele_1	allele_2	reference	reference_rev	strand
1	snp1	0	1	A	C	A	T	forward
1	snp2	0	2	A	T	C	G	ambiguous
1	snp3	0	3	A	G	T	A	reverse
2	snp4	0	1	A	G	C	G	reverse
2	esv5	0	2	AA	G	C	G	reverse
Y	snp6	0	2	A	G	C	G	reverse

$ head extended_example.ambiguous
snp2

$ head extended_example.reverse
snp3
snp4
esv5
snp6
```

##Dependencies

`pandas`, `docopt` and `pyfaidx`. These are automatically installed when using pip.

##FAQ

####My SNP-data is not in Plink .bed+.bim+.fam format. What do I do?

Use [Plink](https://www.cog-genomics.org/plink2/data) to convert your files to the binary plink format. Example:

`plink --vcf yourfile.vcf --make-bed --out your_prefix`

####How do you decide the SNP type and what strand it came from?

Since the reference genome is based on the forward strand, finding an 'A' in the reference genome and the bim-file for said SNP tells you that it is from the forward strand. If the SNP were a 'T' it would have been a reverse strand SNP.

##Issues

See https://github.com/endrebak/snpflip/issues

Please ask support questions by raising an issue.
