## <a name="start"></a>Getting Started
```sh
git clone https://github.com/ruanjue/wtdbg2
cd wtdbg2 && make
# assemble long reads
./wtdbg2 -t 16 -i reads.fa.gz -fo prefix -L 5000
# derive consensus
./wtpoa-cns -t 16 -i prefix.ctg.lay -fo prefix.ctg.lay.fa
```

## <a name="intro"></a>Introduction

Wtdbg2 is a *de novo* sequence assembler for long noisy reads produced by
PacBio or Oxford Nanopore Technologies (ONT). It assembles raw reads without
error correction and then builds the consensus from intermediate assembly
output. Wtdbg2 is able to assemble the human and even the 32Gb
[Axolotl][Axolotl] genome at a speed tens of times faster than [CANU][canu] and
[FALCON][falcon] while producing contigs of comparable base accuracy.

During assembly, wtdbg2 chops reads into 1024bp segments, merges similar
segments into a vertex and connects vertices based on the segment adjacency on
reads. The resulting graph is called fuzzy Bruijn graph (FBG). It is akin to De
Bruijn graph but permits mismatches/gaps and keeps read paths when collapsing
k-mers. The use of FBG distinguishes wtdbg2 from the majority of long-read
assemblers.

## <a name="install"></a>Installation

Wtdbg2 only works on 64-bit Linux. To compile, please type `make` in the source
code directory. You can then copy `wtdbg2` and `wtpoa-cns` to your `PATH`.

Wtdbg2 also comes with an approxmimate read mapper `kbm`, a faster but less
accurate consesus tool `wtdbg-cns` and many auxiliary scripts in the `scripts`
directory.

## <a name="use"></a>Usage

Wtdbg2 has two key components: an assembler **wtdg2** and a consenser
**wtpoa-cns**. Executable **wtdbg2** assembles raw reads and generates the
contig layout and edge sequences in a file "*prefix*.ctg.lay". Executable
**wtpoa-cns** takes this file as input and produces the final consensus in
FASTA. A typical workflow looks like this:
```sh
./wtdbg2 -t 16 -i reads.fa.gz -fo prefix
./wtpoa-cns -t 16 -i prefix.ctg.lay -fo prefix.ctg.lay.fa
```
where `-t` specifies the number of CPU cores (`-t 0` to use all processors).
When the default doesn't work well, you may need to apply more options briefly
explained as follows.

Wtdbg2 combines normal k-mers and homopolymer-compressed (HPC) k-mers to find
read overlaps. Option `-k` specifies the length of normal k-mers, while `-p`
specifies the length of HPC k-mers. By default, wtdbg2 samples a fourth of all
k-mers by their hashcodes. For data of relatively low coverage, you may
increase this sampling rate by reducing `-S`. This will greatly increase the
peak memory as a cost, though. Option `-e`, which defaults to 3, specifies the
minimum read coverage of an edge in the assembly graph. You may adjust this
option according to the overall sequencing depth, too. For PacBio data,
`-L5000` often leads to better assemblies emperically, so is recommended.
Please run `wtdbg2 --help` for a complete list of available options or consult
[README-ori.md](README-ori.md) for more help.

The following table shows various command lines and their resource usage for
the assembly step:

|Dataset                 |Genome|Coverage|Asm options   |CPU asm|CPU cns|Real  |Peak RAM|
|:-----------------------|-----:|-------:|:-------------|------:|------:|-----:|-------:|
|[E. coli][pbcr]         |4.6Mb |PB x20  |-L5000        |    39s| 10m34s|   29s|   1.14G|
|[C. elegans][ce]        |100Mb |PB x80  |-L5000 -e4    |  1h00m|  5h06m|16m16s|    9.5G|
|[Human CHM1][chm1]      |3Gb   |PB x60  |-L5000 -e4    |378.5  |||  252.7G|
|[Human NA12878][na12878]|3Gb   |ONT x30 |-S2 -e2       |197.4  |||  244.9G|
|[Axolotl][axosra]       |32Gb  |PB x32  |-L5000 -AS2   |3189.7 ||| 1593.6G|

The E. coli and C. elegans datasets were assembled with 32 threads on a server
with Xeon E5-2683 CPUs at 2GHz. The rest were assembled on a different machine.

## Limitations

* Wtdbg2 doesn't work with reads longer than 0x3FFFF (~256kb). Longer reads
  will be split into multiple parts.

* Wtdbg2 only works with up to 0x3FFFFFF (~64 million) reads. If you have more
  reads, please filter short or low-quality reads first.

## Getting Help

Please use the [GitHub's Issues page][issue] if you have questions. You may
also directly contact Jue Ruan at ruanjue@gmail.com.

[miniasm]: https://github.com/lh3/miniasm
[canu]: https://github.com/marbl/canu
[falcon]: https://github.com/PacificBiosciences/FALCON
[Axolotl]: https://www.nature.com/articles/nature25458
[chm1]: https://www.ncbi.nlm.nih.gov/bioproject/?term=PRJNA246220
[na12878]: https://github.com/nanopore-wgs-consortium/NA12878
[pbcr]: http://www.cbcb.umd.edu/software/PBcR/data/selfSampleData.tar.gz
[ce]: https://github.com/PacificBiosciences/DevNet/wiki/C.-elegans-data-set
[axosra]: https://www.ncbi.nlm.nih.gov/bioproject/?term=PRJNA378970
[issue]: https://github.com/ruanjue/wtdbg2/issues
