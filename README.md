## [Human alphaherpesvirus 1 (HSV1) and human alphaherpesvirus 2 (HSV2) genome annotation using VADR v1.5.1](#howto)

## [HSV1 and HSV2 VADR models](#hsvmodel)
 * [Additional configuration on HSV VADR models](#additionalconfig)

## [Example annotation of HSV sequences](#example)
 * [Explanation of options used in example annotation](#exampleoptions)

## [Additional VADR documentation](#docs)

## [References](#reference)

---
## <a name="howto"></a>Human alphaherpesvirus 1 (HSV1) and human alphaherpesvirus 2 (HSV2) genome annotation using VADR v1.5.1:

Steps for using VADR for HSV1 and HSV2 annotation:

1. Download and install the latest version of VADR (v1.5.1), following the
   instructions on this [page](https://github.com/ncbi/vadr/blob/master/documentation/install.md).

   **VADR runs on a single processor by default and it can generate
   a lot of intermediate files. The default maximum number of files a
   single processor can open on Mac is 256 and sometimes VADR exceeds
   that. To solve this issue, open the ~/.zshrc file in a text editor
   and add this line to the .zshrc file `ulimit -n 1024`. Then run this
   command `source ~/.zshrc` in the terminal.**

2. Download the latest HSV1 (NC_001806) and HSV2 (NC_001798) VADR models
   from this github repository.

3. Run the following `v-annotate.pl` command (using HSV1 as an example):  

```
v-annotate.pl --mdir <path-to-HSV1-VADR-model-directory> --mkey NC_001806.vadr -s --glsearch -r --alt_pass dupregin,discontn,indfstrn,indfstrp,lowsimis,lowsimil,lowsim5s,lowsim3s,indf5pst,indf3pst --alt_mnf_yes insertnp --nmiscftrthr 10 -f --keep <fasta-file-to-annotate> <output-directory-to-create>
```

---
## <a name="hsvmodel"></a>HSV1 and HSV2 VADR models

The VADR model library for HSV1 was built based on [`NC_001806`](https://ncbi.nlm.nih.gov/nuccore/NC_001806) and for HSV2 [`NC_001798`](https://ncbi.nlm.nih.gov/nuccore/NC_001798).

The HSV1 VADR model was built using the following `v-build.pl` command:

```
v-build.pl --forcelong --skipbuild -f --keep NC_001806 NC_001806
```

The HSV2 VADR model was built using the following `v-build.pl` command:

```
v-build.pl --forcelong --skipbuild -f --keep NC_001798 NC_001798
```
### <a name="additionalconfig"></a>Additional configuration on HSV VADR models
1. The `indfstrn_exc` at the top of the .minfo file allows indfstrn
(indefinite strand) alerts for inverted repeat regions in the HSV genomes

```
MODEL NC_001806 blastdb:"NC_001806.vadr.protein.fa" length:"152222" indfstrn_exc:"1..9213:+;117161..126373:+;125975..132607:+;145590..152222:+;"
```

```
MODEL NC_001798 blastdb:"NC_001798.vadr.protein.fa" length:"154675" indfstrn_exc:"1..9300:+;118021..127320:+;127067..133708:+;148034..154675:+"
```

2. The genes and CDS in the inverted repeat regions (LAT, RL1, RL2, RS1)
have `misc_not_failure:"1"` in their FEATURE lines in the .minfo file.
The HSV genomes are high in GC% (68% in HSV1 RefSeq NC_001806 and 70%
HSV2 RefSeq NC_001798) and the number of repeat sequences, particularly
in LAT, RL1, RL2, and RS1, that make the sequencing and genome assembly
of HSV genomes difficult. `misc_not_failure:"1"` allows most alerts
to annotate those genes and CDS as misc_feature instead of causing failure.

```
FEATURE NC_001806 type:"gene" coords:"7569..1:-" parent_idx_str:"GBNULL" gene:"LAT" misc_not_failure:"1"
FEATURE NC_001806 type:"gene" coords:"513..1540:+" parent_idx_str:"GBNULL" gene:"RL1" misc_not_failure:"1"
FEATURE NC_001806 type:"CDS" coords:"513..1259:+" parent_idx_str:"GBNULL" gene:"RL1" product:"neurovirulence protein ICP34.5" misc_not_failure:"1"
```

3. Certain CDS have alternative forms/stop codons. To allow this in the VADR model:
* Add extra FEATURE lines for the gene and CDS in the .minfo file like the following:

```
FEATURE NC_001806 type:"gene" coords:"145198..144124:-" parent_idx_str:"GBNULL" gene:"US10" alternative_ftr_set:"virion protein US10(gene)" alternative_ftr_set_subn:"161"
FEATURE NC_001806 type:"gene" coords:"145198..144144:-" parent_idx_str:"GBNULL" gene:"US10" alternative_ftr_set:"virion protein US10(gene)" alternative_ftr_set_subn:"162"
FEATURE NC_001806 type:"CDS" coords:"145100..144162:-" parent_idx_str:"GBNULL" gene:"US10" product:"virion protein US10" alternative_ftr_set:"virion protein US10(cds)"
FEATURE NC_001806 type:"CDS" coords:"145100..144144:-" parent_idx_str:"GBNULL" gene:"US10" product:"virion protein US10" alternative_ftr_set:"virion protein US10(cds)" 
```

The original CDS for virion protein US10 in HSV1 RefSeq NC_001806
is from genomic position 145100 to 144162 and the alternative is from 145100 to 144144. 
Note that the "gene" FEATURE line for the alternative CDS has the same coordinates
as that of its CDS FEATURE line

The "161" in `alternative_ftr_set_subn:"161"` refers to the line number minus 1, i.e.
that particular FEATURE line is line 162 in the .minfo file.

* Add the alternative amino acid sequence to the .vadr.protein.fa blastx database using
this `build-add-to-blast.db.pl` [script](https://github.com/ncbi/vadr/blob/master/miniscripts/build-add-blast-db.pl) from the VADR github repository:

```
perl build-add-to-blast-db.pl <path-to-minfo-file> <path-to-blast-db-dir> NC_001806 MG999843 144762..143806:- 145100..144144:- <name-for-output-directory>
```

In this case, `NC_001806` is the VADR model accession. `MG999843` is the sequence that 
has the alternative CDS at `144762..143806:-` corresponding to the `145100..144144:-`
genomic positions in the VADR model accession.

After successfully running the `build-add-to-blast.db.pl` script, you should see this
in the .vadr.protein.fa file:

```
...snip...
>NC_001806.2/145100..144162:-
MIKRRGNVEIRVYYESVRTLRSRSHLKPSDRQQSPGHRVFPGSPGFRDHPENLGNPEYRE
LPETPGYRVTPGIHDNPGLPGSPGLPGSPGLPGSPGPHAPPANHVRLAGLYSPGKYAPLA
SPDPFSPQHGAYARARVGIHTAVRVPPTGSPTHTHLRQDPGDEPTSDDSGLYPLDARALA
HLVMLPADHRAFFRTVVEVSRMCAANVRDPPPPATGAMLGRHARLVHTQWLRANQETSPL
WPWRTAAINFITTMAPRVQTHRHMHDLLMACAFWCCLTHASTCSYAGLYSTHCLHLFGAF

...snip...
>MG999843.1:144762..143806:-/145100..144144:-
MIKRRGNVEIRVYYESVRTLRSRSHLKPSDHQQSPGHRVFPGSPGFRDHPENLGNPEYRE
LPETPGYRVTPGIHDSPGLPGSPGLHXSPGLPGSHGPHAPPANHVRLAGLYSPGKYAPLA
SPDPFSPQDGAYARARVGIHTAVRVPPTGSPTHTHLRHDPGDEPTSDDSGLYPLDARALA
HLVMLPADHRAFFRTVVDVSRMCAANVRDPPPPATGAMLGRHARLVHTQWLRANQETSPL
WPWRTAAINFITTMAPRVQTHRHMHDLLMACAFWCCLTHASTCSYAGLYSTHCLHLFGAF
GCGAPALTPPLCQDNLYP
```
 
---
## <a name="example">Example VADR output

Below is an example of VADR output when annotating the sequence MG999860 (HSV1).
The VADR log output includes the options used in the `v-annotate.pl` command,
alert codes and their descriptions, and files saved to output directory.

```
# v-annotate.pl :: classify and annotate sequences using a model library
# VADR 1.5.1 (Feb 2023)
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# date:              Fri Sep  1 16:40:00 2023
# $VADRBIOEASELDIR:  /Users/greningerlab/Documents/vadr-install-dir/Bio-Easel
# $VADRBLASTDIR:     /Users/greningerlab/Documents/vadr-install-dir/ncbi-blast/bin
# $VADREASELDIR:     /Users/greningerlab/Documents/vadr-install-dir/infernal/binaries
# $VADRFASTADIR:     /Users/greningerlab/Documents/vadr-install-dir/fasta/bin
# $VADRINFERNALDIR:  /Users/greningerlab/Documents/vadr-install-dir/infernal/binaries
# $VADRMODELDIR:     /Users/greningerlab/Documents/vadr-install-dir/vadr-models-calici
# $VADRSCRIPTSDIR:   /Users/greningerlab/Documents/vadr-install-dir/vadr
#
# sequence file:                                                                               MG999860.fasta
# output directory:                                                                            ../vadr_output_09012023/MG999860_vadr_output_09012023
# force directory overwrite:                                                                   yes [-f]
# leaving intermediate files on disk:                                                          yes [--keep]
# specify that alert codes in <s> do not cause FAILure:                                        dupregin,discontn,indfstrp,lowsimis,lowsimil,indf5pst,indf3pst [--alt_pass]
# alert codes in <s> for 'misc_not_failure' features cause misc_feature-ization, not failure:  insertnp [--alt_mnf_yes]
# .cm, .minfo, blastn .fa files in $VADRMODELDIR start with key <s>, not 'vadr':               NC_001806.vadr [--mkey]
# model files are in directory <s>, not in $VADRMODELDIR:                                      /Users/greningerlab/AS/vadr_hsv/NC_001806 [--mdir]
# nmiscftr/TOO_MANY_MISC_FEATURES reported if <n> or more misc_features:                       10 [--nmiscftrthr]
# align with glsearch from the FASTA package, not to a cm with cmalign:                        yes [--glsearch]
# use top-scoring HSP from blastn to seed the alignment:                                       yes [-s]
# replace stretches of Ns with expected nts, where possible:                                   yes [-r]
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Validating input                                                                        ... done. [    0.1 seconds]
# Preprocessing for N replacement: blastn classification (1 seq)                          ... done. [    1.9 seconds]
# Preprocessing for N replacement: coverage determination from blastn results (1 seq)     ... done. [    0.0 seconds]
# Replacing Ns based on results of blastn-based pre-processing                            ... done. [    0.0 seconds]
# Classifying sequences with blastn (1 seq)                                               ... done. [    2.0 seconds]
# Determining sequence coverage from blastn results (1 seq)                               ... done. [    0.0 seconds]
# Aligning sequences (NC_001806: 2 seqs)                                                  ... done. [  144.7 seconds]
# Joining alignments from glsearch and blastn for model NC_001806 (1 seq)                 ... done. [    0.1 seconds]
# Determining annotation                                                                  ... done. [    2.1 seconds]
# Validating proteins with blastx (NC_001806: 1 seq)                                      ... done. [   26.1 seconds]
# Generating feature table output                                                         ... done. [    0.0 seconds]
# Generating tabular output                                                               ... done. [    0.0 seconds]
#
# Summary of classified sequences:
#
#                                  num   num   num
#idx  model      group  subgroup  seqs  pass  fail
#---  ---------  -----  --------  ----  ----  ----
1     NC_001806  -      -            1     0     1
#---  ---------  -----  --------  ----  ----  ----
-     *all*      -      -            1     0     1
-     *none*     -      -            0     0     0
#---  ---------  -----  --------  ----  ----  ----
#
# Summary of reported alerts:
#
#     alert     causes   short                             per    num   num  long
#idx  code      failure  description                      type  cases  seqs  description
#---  --------  -------  ---------------------------  --------  -----  ----  -----------
1     dupregin  no       DUPLICATE_REGIONS            sequence     10     1  similarity to a model region occurs more than once
2     discontn  no       DISCONTINUOUS_SIMILARITY     sequence      1     1  not all hits are in the same order in the sequence and the homology model
3     lowsimis  no       LOW_SIMILARITY               sequence      2     1  internal region without significant similarity
4     ambgntrp  no       N_RICH_REGION_NOT_REPLACED   sequence      4     1  N-rich region of unexpected length not replaced
5     indf5pst  no       INDEFINITE_ANNOTATION_START   feature      2     1  protein-based alignment does not extend close enough to nucleotide-based alignment 5' endpoint
6     indf3pst  no       INDEFINITE_ANNOTATION_END     feature      2     1  protein-based alignment does not extend close enough to nucleotide-based alignment 3' endpoint
7     indfstrp  no       INDEFINITE_STRAND             feature      1     1  strand mismatch between protein-based and nucleotide-based predictions
8     lowsimic  no       LOW_FEATURE_SIMILARITY        feature      1     1  region overlapping annotated feature that is or matches a CDS lacks significant similarity
9     lowsimil  no       LOW_FEATURE_SIMILARITY        feature      1     1  long region overlapping annotated feature that does not match a CDS lacks significant similarity
10    ambgnt3f  no       AMBIGUITY_AT_FEATURE_END      feature      2     1  final nucleotide of non-CDS feature is an ambiguous nucleotide
#---  --------  -------  ---------------------------  --------  -----  ----  -----------
11    mutendex  yes*     MUTATION_AT_END               feature      1     1  expected stop codon could not be identified, first in-frame stop codon exists 3' of predicted stop position
12    unexleng  yes*     UNEXPECTED_LENGTH             feature      3     1  length of complete coding (CDS or mat_peptide) feature is not a multiple of 3
13    cdsstopn  yes*     CDS_HAS_STOP_CODON            feature      2     1  in-frame stop codon exists 5' of stop position predicted by homology to reference
14    fstukcft  yes*     POSSIBLE_FRAMESHIFT           feature      3     1  possible frameshift in CDS (frame not restored before end)
15    deletinp  yes*     DELETION_OF_NT                feature      1     1  too large of a deletion in protein-based alignment
#---  --------  -------  ---------------------------  --------  -----  ----  -----------
#
# Output printed to screen saved in:                                                        MG999860_vadr_output_09012023.vadr.log
# List of executed commands saved in:                                                       MG999860_vadr_output_09012023.vadr.cmd
# List and description of all output files saved in:                                        MG999860_vadr_output_09012023.vadr.filelist
# copy of input fasta file saved in:                                                        MG999860_vadr_output_09012023.vadr.in.fa
# copy of input fasta file with descriptions removed for blastn saved in:                   MG999860_vadr_output_09012023.vadr.blastn.fa
# esl-seqstat -a output for input fasta file saved in:                                      MG999860_vadr_output_09012023.vadr.seqstat
# Stockholm file for >= 1 possible frameshifts for CDS.42.1 for model NC_001806 saved in:   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.42.1.frameshift.stk
# Stockholm file for >= 1 possible frameshifts for CDS.79.1 for model NC_001806 saved in:   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.79.1.frameshift.stk
# Stockholm file for >= 1 possible frameshifts for CDS.80.1 for model NC_001806 saved in:   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.80.1.frameshift.stk
# Stockholm file for >= 1 possible frameshifts for CDS.81.1 for model NC_001806 saved in:   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.81.1.frameshift.stk
# model NC_001806 feature gene#1 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.gene.1.fa
# model NC_001806 feature gene#2 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.gene.2.fa
# model NC_001806 feature CDS#1 predicted seqs saved in:                                    MG999860_vadr_output_09012023.vadr.NC_001806.CDS.1.fa
# model NC_001806 feature gene#3 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.gene.3.fa
# model NC_001806 feature CDS#2 predicted seqs saved in:                                    MG999860_vadr_output_09012023.vadr.NC_001806.CDS.2.fa
# model NC_001806 feature gene#4 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.gene.4.fa
# model NC_001806 feature CDS#3 predicted seqs saved in:                                    MG999860_vadr_output_09012023.vadr.NC_001806.CDS.3.fa
# model NC_001806 feature gene#5 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.gene.5.fa
# model NC_001806 feature CDS#4 predicted seqs saved in:                                    MG999860_vadr_output_09012023.vadr.NC_001806.CDS.4.fa
# model NC_001806 feature gene#6 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.gene.6.fa
# model NC_001806 feature CDS#5 predicted seqs saved in:                                    MG999860_vadr_output_09012023.vadr.NC_001806.CDS.5.fa
# model NC_001806 feature gene#7 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.gene.7.fa
# model NC_001806 feature gene#8 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.gene.8.fa
# model NC_001806 feature CDS#6 predicted seqs saved in:                                    MG999860_vadr_output_09012023.vadr.NC_001806.CDS.6.fa
# model NC_001806 feature CDS#7 predicted seqs saved in:                                    MG999860_vadr_output_09012023.vadr.NC_001806.CDS.7.fa
# model NC_001806 feature gene#9 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.gene.9.fa
# model NC_001806 feature CDS#8 predicted seqs saved in:                                    MG999860_vadr_output_09012023.vadr.NC_001806.CDS.8.fa
# model NC_001806 feature gene#10 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.10.fa
# model NC_001806 feature CDS#9 predicted seqs saved in:                                    MG999860_vadr_output_09012023.vadr.NC_001806.CDS.9.fa
# model NC_001806 feature gene#11 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.11.fa
# model NC_001806 feature gene#12 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.12.fa
# model NC_001806 feature CDS#10 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.10.fa
# model NC_001806 feature CDS#11 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.11.fa
# model NC_001806 feature gene#13 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.13.fa
# model NC_001806 feature CDS#12 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.12.fa
# model NC_001806 feature gene#14 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.14.fa
# model NC_001806 feature gene#15 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.15.fa
# model NC_001806 feature gene#16 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.16.fa
# model NC_001806 feature gene#17 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.17.fa
# model NC_001806 feature gene#18 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.18.fa
# model NC_001806 feature CDS#13 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.13.fa
# model NC_001806 feature CDS#14 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.14.fa
# model NC_001806 feature CDS#15 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.15.fa
# model NC_001806 feature CDS#16 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.16.fa
# model NC_001806 feature CDS#17 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.17.fa
# model NC_001806 feature gene#19 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.19.fa
# model NC_001806 feature gene#20 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.20.fa
# model NC_001806 feature CDS#18 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.18.fa
# model NC_001806 feature CDS#19 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.19.fa
# model NC_001806 feature gene#21 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.21.fa
# model NC_001806 feature gene#22 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.22.fa
# model NC_001806 feature gene#23 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.23.fa
# model NC_001806 feature CDS#20 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.20.fa
# model NC_001806 feature CDS#21 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.21.fa
# model NC_001806 feature CDS#22 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.22.fa
# model NC_001806 feature gene#24 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.24.fa
# model NC_001806 feature gene#25 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.25.fa
# model NC_001806 feature gene#26 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.26.fa
# model NC_001806 feature CDS#23 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.23.fa
# model NC_001806 feature CDS#24 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.24.fa
# model NC_001806 feature CDS#25 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.25.fa
# model NC_001806 feature gene#27 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.27.fa
# model NC_001806 feature CDS#26 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.26.fa
# model NC_001806 feature gene#28 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.28.fa
# model NC_001806 feature CDS#27 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.27.fa
# model NC_001806 feature gene#29 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.29.fa
# model NC_001806 feature CDS#28 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.28.fa
# model NC_001806 feature gene#30 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.30.fa
# model NC_001806 feature CDS#29 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.29.fa
# model NC_001806 feature gene#31 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.31.fa
# model NC_001806 feature CDS#30 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.30.fa
# model NC_001806 feature gene#32 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.32.fa
# model NC_001806 feature CDS#31 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.31.fa
# model NC_001806 feature gene#33 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.33.fa
# model NC_001806 feature CDS#32 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.32.fa
# model NC_001806 feature gene#34 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.34.fa
# model NC_001806 feature gene#35 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.35.fa
# model NC_001806 feature CDS#33 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.33.fa
# model NC_001806 feature CDS#34 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.34.fa
# model NC_001806 feature gene#36 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.36.fa
# model NC_001806 feature CDS#35 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.35.fa
# model NC_001806 feature gene#37 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.37.fa
# model NC_001806 feature CDS#36 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.36.fa
# model NC_001806 feature gene#38 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.38.fa
# model NC_001806 feature gene#39 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.39.fa
# model NC_001806 feature CDS#37 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.37.fa
# model NC_001806 feature CDS#38 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.38.fa
# model NC_001806 feature gene#40 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.40.fa
# model NC_001806 feature CDS#39 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.39.fa
# model NC_001806 feature gene#41 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.41.fa
# model NC_001806 feature CDS#40 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.40.fa
# model NC_001806 feature gene#42 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.42.fa
# model NC_001806 feature CDS#41 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.41.fa
# model NC_001806 feature gene#43 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.43.fa
# model NC_001806 feature CDS#42 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.42.fa
# model NC_001806 feature gene#44 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.44.fa
# model NC_001806 feature CDS#43 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.43.fa
# model NC_001806 feature gene#45 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.45.fa
# model NC_001806 feature CDS#44 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.44.fa
# model NC_001806 feature gene#46 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.46.fa
# model NC_001806 feature CDS#45 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.45.fa
# model NC_001806 feature gene#47 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.47.fa
# model NC_001806 feature CDS#46 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.46.fa
# model NC_001806 feature gene#48 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.48.fa
# model NC_001806 feature CDS#47 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.47.fa
# model NC_001806 feature gene#49 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.49.fa
# model NC_001806 feature gene#50 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.50.fa
# model NC_001806 feature CDS#48 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.48.fa
# model NC_001806 feature CDS#49 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.49.fa
# model NC_001806 feature gene#51 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.51.fa
# model NC_001806 feature gene#52 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.52.fa
# model NC_001806 feature CDS#50 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.50.fa
# model NC_001806 feature CDS#51 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.51.fa
# model NC_001806 feature gene#53 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.53.fa
# model NC_001806 feature CDS#52 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.52.fa
# model NC_001806 feature gene#54 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.54.fa
# model NC_001806 feature CDS#53 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.53.fa
# model NC_001806 feature gene#55 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.55.fa
# model NC_001806 feature gene#56 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.56.fa
# model NC_001806 feature CDS#54 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.54.fa
# model NC_001806 feature CDS#55 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.55.fa
# model NC_001806 feature gene#57 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.57.fa
# model NC_001806 feature CDS#56 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.56.fa
# model NC_001806 feature gene#58 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.58.fa
# model NC_001806 feature gene#59 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.59.fa
# model NC_001806 feature CDS#57 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.57.fa
# model NC_001806 feature CDS#58 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.58.fa
# model NC_001806 feature gene#60 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.60.fa
# model NC_001806 feature CDS#59 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.59.fa
# model NC_001806 feature gene#61 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.61.fa
# model NC_001806 feature CDS#60 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.60.fa
# model NC_001806 feature gene#62 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.62.fa
# model NC_001806 feature CDS#61 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.61.fa
# model NC_001806 feature gene#63 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.63.fa
# model NC_001806 feature CDS#62 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.62.fa
# model NC_001806 feature gene#64 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.64.fa
# model NC_001806 feature CDS#63 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.63.fa
# model NC_001806 feature gene#65 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.65.fa
# model NC_001806 feature CDS#64 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.64.fa
# model NC_001806 feature gene#66 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.66.fa
# model NC_001806 feature CDS#65 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.65.fa
# model NC_001806 feature gene#67 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.67.fa
# model NC_001806 feature gene#68 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.68.fa
# model NC_001806 feature CDS#66 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.66.fa
# model NC_001806 feature gene#69 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.69.fa
# model NC_001806 feature CDS#67 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.67.fa
# model NC_001806 feature gene#70 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.70.fa
# model NC_001806 feature CDS#68 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.68.fa
# model NC_001806 feature gene#71 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.71.fa
# model NC_001806 feature CDS#69 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.69.fa
# model NC_001806 feature gene#72 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.72.fa
# model NC_001806 feature CDS#70 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.70.fa
# model NC_001806 feature gene#73 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.73.fa
# model NC_001806 feature CDS#71 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.71.fa
# model NC_001806 feature gene#74 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.74.fa
# model NC_001806 feature CDS#72 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.72.fa
# model NC_001806 feature gene#75 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.75.fa
# model NC_001806 feature CDS#73 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.73.fa
# model NC_001806 feature gene#76 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.76.fa
# model NC_001806 feature CDS#74 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.74.fa
# model NC_001806 feature gene#77 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.77.fa
# model NC_001806 feature CDS#75 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.75.fa
# model NC_001806 feature gene#78 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.78.fa
# model NC_001806 feature CDS#76 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.76.fa
# model NC_001806 feature gene#79 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.79.fa
# model NC_001806 feature CDS#77 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.77.fa
# model NC_001806 feature gene#80 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.80.fa
# model NC_001806 feature CDS#78 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.78.fa
# model NC_001806 feature gene#81 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.81.fa
# model NC_001806 feature gene#82 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.82.fa
# model NC_001806 feature gene#83 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.83.fa
# model NC_001806 feature gene#84 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.84.fa
# model NC_001806 feature CDS#79 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.79.fa
# model NC_001806 feature CDS#80 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.80.fa
# model NC_001806 feature CDS#81 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.81.fa
# model NC_001806 feature CDS#82 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.82.fa
# model NC_001806 feature gene#85 predicted seqs saved in:                                  MG999860_vadr_output_09012023.vadr.NC_001806.gene.85.fa
# model NC_001806 feature CDS#83 predicted seqs saved in:                                   MG999860_vadr_output_09012023.vadr.NC_001806.CDS.83.fa
# model NC_001806 replaced sequence alignment (stockholm) saved in:                         MG999860_vadr_output_09012023.vadr.NC_001806.align.rpstk
# model NC_001806 full original sequence alignment (stockholm) saved in:                    MG999860_vadr_output_09012023.vadr.NC_001806.align.stk
# model NC_001806 full replaced sequence alignment (afa) saved in:                          MG999860_vadr_output_09012023.vadr.NC_001806.align.rpafa
# model NC_001806 full original sequence alignment (afa) saved in:                          MG999860_vadr_output_09012023.vadr.NC_001806.align.afa
# 5 column feature table output for passing sequences saved in:                             MG999860_vadr_output_09012023.vadr.pass.tbl
# 5 column feature table output for failing sequences saved in:                             MG999860_vadr_output_09012023.vadr.fail.tbl
# list of passing sequences saved in:                                                       MG999860_vadr_output_09012023.vadr.pass.list
# list of failing sequences saved in:                                                       MG999860_vadr_output_09012023.vadr.fail.list
# list of alerts in the feature tables saved in:                                            MG999860_vadr_output_09012023.vadr.alt.list
# fasta file with passing sequences saved in:                                               MG999860_vadr_output_09012023.vadr.pass.fa
# fasta file with failing sequences saved in:                                               MG999860_vadr_output_09012023.vadr.fail.fa
# per-sequence tabular annotation summary file saved in:                                    MG999860_vadr_output_09012023.vadr.sqa
# per-sequence tabular classification summary file saved in:                                MG999860_vadr_output_09012023.vadr.sqc
# per-feature tabular summary file saved in:                                                MG999860_vadr_output_09012023.vadr.ftr
# per-model-segment tabular summary file saved in:                                          MG999860_vadr_output_09012023.vadr.sgm
# per-model tabular summary file saved in:                                                  MG999860_vadr_output_09012023.vadr.mdl
# per-alert tabular summary file saved in:                                                  MG999860_vadr_output_09012023.vadr.alt
# alert count tabular summary file saved in:                                                MG999860_vadr_output_09012023.vadr.alc
# alignment doctoring tabular summary file saved in:                                        MG999860_vadr_output_09012023.vadr.dcr
# seed alignment summary file (-s) saved in:                                                MG999860_vadr_output_09012023.vadr.sda
# replaced stretches of Ns summary file (-r) saved in:                                      MG999860_vadr_output_09012023.vadr.rpn
#
# All output files created in directory ./../vadr_output_09012023/MG999860_vadr_output_09012023/
#
# Elapsed time:  00:02:57.07
#                hh:mm:ss
# 
[ok]
```

***Detailed information on how to interpret VADR alerts are [here](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md).***

Particularly useful files:\
`.pass.fa`: fasta sequence that passes annotation\
`.pass.tbl`: feature table format of annotation for sequence that passes annotation\
`.fail.fa`: fasta sequence that fails annotation\
`.fail.tbl`: feature table format of annotation for sequence that fails annotation (note that the end of the file has error messages)\
`.alt.list`: list of alert codes that cause annotation failure\
`.alt`: list of all alert codes\
`.align.stk`: stockholm alignment of input sequence to VADR model sequence\


### <a name="exampleoptions"></a>Explanation of options used in example HSV1 annotation command

The options used in the example command are recommended, but other options might
be more appropriate depending on genome assembly methods and quality of the
genomes. A complete list of options for `v-annotate.pl` can be found [here](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#v-annotatepl-command-line-options).

|option|explanation|
|------|-----------|
| `--mdir <path-to-HSV1-VADR-model-directory>` | specify that the model to use are in directory path-to-HSV1-VADR-model-directory |
| `--mkey NC_001806.vadr` | use the model files with prefix `NC_001806.vadr` in the directory from `--mdir` |
|`-s`| turn on the seed acceleration heuristic: use the max length ungapped region from blastn to seed the alignment, required for HSV annotation |
|`--glsearch` | use the glsearch program from the FASTA package for the alignment stage, required for HSV annotation |
|`-r`| turn on the replace-N strategy: replace stretches of Ns with expected nucleotides, where possible |
| `--alt_pass dupregin,discontn,indfstrp,lowsimis,lowsimil,indf5pst,indf3pst` | specify that these alerts, mostly due to inverted repeats and Ns in the sequence, do NOT cause annotation failure |
| `--alt_mnf_yes insertnp` | specify that features with `misc_not_failure:"1"` in .minfo file (LAT, RL1, RL2, RS1) also allow insertnp alert; `misc_not_failure:"1"` allows insertnn alert but not insertnp alert, which are raised together usually |
| `--nmiscftrthr 10` | increase the number of misc_feature annotation allowed in a sequence to 10 due to the use of `misc_not_failure:"1"` in LAT, RL1, RL2, RS2 |
| `-f` | overwrite output directory if it exists |
| `--keep` | keep all intermediate files, useful for debugging |
| `<fasta-file-to-annotate>` | input fasta sequence to be annotate | 
| `<output-directory-to-create>` | output directory |

---

## <a name="docs"> Additional VADR documentation

* [VADR README](https://github.com/ncbi/vadr/blob/master/README.md#top)
* [VADR installation instructions](https://github.com/ncbi/vadr/blob/master/documentation/install.md#top)
  * [Installation using `vadr-install.sh`](https://github.com/ncbi/vadr/blob/master/documentation/install.md#install)
  * [Setting environment variables](https://github.com/ncbi/vadr/blob/master/documentation/install.md#environment)
  * [Verifying successful installation](https://github.com/ncbi/vadr/blob/master/documentation/install.md#tests)
  * [Further information](https://github.com/ncbi/vadr/blob/master/documentation/install.md#further)
* [`v-build.pl` example usage and command-line options](https://github.com/ncbi/vadr/blob/master/documentation/build.md#top)
  * [`v-build.pl` example usage](https://github.com/ncbi/vadr/blob/master/documentation/build.md#exampleusage)
  * [`v-build.pl` command-line options](https://github.com/ncbi/vadr/blob/master/documentation/build.md#options)
  * [Building a VADR model library](https://github.com/ncbi/vadr/blob/master/documentation/build.md#library)
  * [How the VADR 1.0 model library was constructed](https://github.com/ncbi/vadr/blob/master/documentation/build.md#1.0library)
* [`v-annotate.pl` example usage, command-line options and alert information](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#top)
  * [`v-annotate.pl` example usage](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#exampleusage)
  * [`v-annotate.pl` command-line options](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#options)
  * [Basic Information on `v-annotate.pl` alerts](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#alerts)
  * [Additional information on `v-annotate.pl` alerts](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#alerts2)
* [Explanations and examples of `v-annotate.pl` detailed alert and error messages](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#top)
  * [Output fields with detailed alert and error messages](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#files)
  * [Explanation of sequence and model coordinate fields in `.alt` files](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#coords)
  * [`toy50` toy model used in examples of alert messages](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#toy)
  * [Examples of different alert types and corresponding `.alt` output](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#examples)
  * [Posterior probability annotation in VADR output Stockholm alignments](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#pp)
* [VADR output file formats](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#top)
  * [VADR output files created by all VADR scripts](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#generic)
  * [`v-build.pl` output files](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#build)
  * [`v-annotate.pl` output files](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#annotate)
  * [VADR `coords` coordinate string format](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#coords)
  * [VADR sequence naming conventions](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#seqnames)

---

## Reference <a name="reference"></a>
* The recommended citation for using VADR is:
  Alejandro A Schäffer, Eneida L Hatcher, Linda Yankie, Lara Shonkwiler,
  J Rodney Brister, Ilene Karsch-Mizrachi, Eric P Nawrocki; *VADR:
  validation and annotation of virus sequence submissions to
  GenBank.* BMC Bioinformatics 21, 211
  (2020). https://doi.org/10.1186/s12859-020-3537-3
* https://github.com/ncbi/vadr/wiki/Mpox-virus-annotation
