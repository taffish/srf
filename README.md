# srf

TAFFISH wrapper for [SRF](https://github.com/lh3/srf), Heng Li's Satellite
Repeat Finder for assembling satellite repeat units from high-occurrence k-mer
count tables.

Upstream SRF has no semantic release tag or runtime version flag. This app
therefore packages the upstream HEAD commit
`e54ca8c8eccf6b1f19428b0f862f2c90575290a0` as `srf 20240109-r1`. The version
is the commit date in Asia/Shanghai time; the same commit is
`2024-01-08T21:02:55Z` in UTC.

Package metadata:

```text
name: srf
command: taf-srf
version: 20240109-r1
kind: tool
image: ghcr.io/taffish/srf:20240109-r1
upstream commit: e54ca8c8eccf6b1f19428b0f862f2c90575290a0
```

## Quick Start

Show the TAFFISH package version:

```sh
taf-srf --version
```

Show upstream SRF usage:

```sh
taf-srf -- --help
taf-srf srf
```

Assemble satellite repeat contigs from a KMC dump-style k-mer count table:

```sh
taf-srf srf -p sample count.txt > sample.srf.fa
```

Because `srf` is the default upstream command, option-leading SRF calls can
also use the short form:

```sh
taf-srf -p sample -l 100 count.txt > sample.srf.fa
```

Run helper commands through command mode:

```sh
taf-srf srfutils.js enlong sample.srf.fa > sample.srf.enlong.fa
taf-srf srfutils.js paf2bed sample.srf.paf > sample.srf.bed
taf-srf srfutils.js bed2abun sample.srf.bed > sample.srf.abun.txt
```

## Input Contract

The primary `srf` executable reads a plain text or gzip-compressed k-mer count
table. Each line should contain one k-mer and one positive count:

```text
ACGTTGCAACGTTGCAA 125
CGTTGCAACGTTGCAAC 123
```

Upstream usually expects this table to come from `kmc_dump` after high-copy
k-mers have been selected with KMC. The exact `-k`, `-ci`, and `-cs` choices
depend on the input data and coverage; SRF itself consumes the dumped table,
not the original reads or contigs.

Pass input files from the current project directory or another path mounted by
TAFFISH, such as your home directory. Upstream SRF does not always produce a
friendly diagnostic when a count file is missing or not visible inside the
container.

## Helper Tools

The image contains:

```text
srf
srfutils.js
fmd-occ
k8
gzip
```

`srfutils.js` is upstream's JavaScript helper for downstream analysis. It is
installed as an executable script, and the image includes the `k8` JavaScript
engine needed by its shebang. Common helper commands are:

```text
enlong    elongate circular contigs before mapping
filter    filter read-to-contig PAF alignments
paf2bed   extract non-overlapping satellite regions
bed2cnt   summarize per-chromosome counts
bed2abun  estimate per-contig abundance
merge     merge identical sequences
```

The Makefile also defines `fmd-occ`; it is installed for completeness as an
upstream low-level helper, but the public README workflow centers on `srf` and
`srfutils.js`.

## Common Workflow

SRF does not perform k-mer counting by itself. A typical upstream workflow is:

```sh
kmc -fq -k151 -t16 -ci100 -cs100000 reads.fq.gz count.kmc tmp_dir
kmc_dump count.kmc count.txt
taf-srf srf -p sample count.txt > sample.srf.fa
```

For abundance analysis, map the elongated SRF contigs back to reads or an
assembly with an aligner such as minimap2, then process the PAF:

```sh
taf-srf srfutils.js enlong sample.srf.fa > sample.srf.enlong.fa
minimap2 -c -N1000000 -f1000 -r100,100 sample.srf.enlong.fa reads.fa > sample.srf.paf
taf-srf srfutils.js paf2bed sample.srf.paf > sample.srf.bed
taf-srf srfutils.js bed2abun sample.srf.bed > sample.srf.abun.txt
```

## Container Contents

The Dockerfile builds SRF from the pinned upstream commit on `debian:12-slim`.
It compiles the C sources with GCC, GNU Make, and zlib headers, installs the
compiled `srf` and `fmd-occ` binaries, installs `srfutils.js`, and adds
Bioconda-packaged `k8` 1.2 binaries pinned by SHA256 for both supported
architectures.

Runtime dependencies are intentionally small:

```text
bash
gzip
libgcc-s1
zlib1g
```

Source metadata is installed at:

```text
/opt/srf/share/doc/srf/source.txt
```

The image is built and validated for:

```text
linux/amd64
linux/arm64
```

## Boundaries

This app packages SRF and its upstream helper script. It does not bundle KMC,
minimap2, TRF, TRF-mod, read mappers, plotting tools, reference genomes, read
sets, or satellite-repeat databases. Use separate TAFFISH tools or host tools
for those workflow stages.

Upstream has no `--version` command. Version reproducibility is provided by
the TAFFISH package version and by the pinned commit/source SHA256 recorded in
the image.

Input path handling follows normal TAFFISH container mounts. Files outside the
mounted working tree or home directory need to be copied or bound into a
mounted location before they are passed to `srf`.

## Build Notes

The Dockerfile downloads:

```text
https://github.com/lh3/srf/archive/e54ca8c8eccf6b1f19428b0f862f2c90575290a0.tar.gz
sha256: 8a9e6851ff02c119eade2b1a8f1758ba0fda4780cd8547630da204abf84ba6c4
```

The included `k8` runtime is the same Bioconda `k8` 1.2 package family used by
the minimap2 app in this hub, with architecture-specific SHA256 pins.

## Smoke Coverage

The package smoke checks are self-contained and offline. They verify:

```text
srf, srfutils.js, fmd-occ, k8, gzip, ldd, and sh command existence
pinned source metadata for version 20240109 and commit e54ca8c8...
upstream usage text for srf, srfutils.js, and fmd-occ
zlib linkage for srf and k8
a tiny k-mer count table producing an SRF FASTA record
gzip-compressed k-mer count input
srfutils.js enlong, merge, and bed2abun helper paths
```

These checks validate packaging, command surface, and minimal local execution;
they do not replace scientific validation of SRF parameters on real sequencing
data.

## Upstream

- Project: SRF, Satellite Repeat Finder
- Source: <https://github.com/lh3/srf>
- Pinned commit:
  <https://github.com/lh3/srf/commit/e54ca8c8eccf6b1f19428b0f862f2c90575290a0>
- Upstream license: MIT
- Citation: Zhang Y, Chu J, Cheng H and Li H (2023), arXiv:2304.09729

## Maintainer Notes

Useful checks before publishing:

```sh
taf check
taf compile -- -p sample count.txt
taf publish --release --dry-run
docker build -t ghcr.io/taffish/srf:20240109-r1 -f docker/Dockerfile .
docker run --rm ghcr.io/taffish/srf:20240109-r1 srf
docker run --rm ghcr.io/taffish/srf:20240109-r1 srfutils.js
```
