# srf Help

TAFFISH package:

```text
name: srf
command: taf-srf
version: 20240109-r1
image: ghcr.io/taffish/srf:20240109-r1
```

Upstream SRF has no semantic release tag or runtime `--version` flag. This
package pins commit `e54ca8c8eccf6b1f19428b0f862f2c90575290a0`, committed at
`2024-01-08T21:02:55Z` (`2024-01-09` in Asia/Shanghai time).

## Usage

Show this TAFFISH help:

```sh
taf-srf --help
```

Show the TAFFISH package version:

```sh
taf-srf --version
```

Show upstream SRF usage:

```sh
taf-srf -- --help
taf-srf srf
```

Run SRF on a KMC dump-style k-mer count table:

```sh
taf-srf srf -p sample count.txt > sample.srf.fa
taf-srf -p sample count.txt > sample.srf.fa
```

Use input paths that are visible inside the TAFFISH container, such as files in
the current project directory or your home directory.

Run upstream helper commands:

```sh
taf-srf srfutils.js enlong sample.srf.fa > sample.srf.enlong.fa
taf-srf srfutils.js paf2bed sample.srf.paf > sample.srf.bed
taf-srf srfutils.js bed2abun sample.srf.bed > sample.srf.abun.txt
```

The image includes `k8`, so `srfutils.js` can be executed directly.

## Included Commands

```text
srf          assemble satellite repeat contigs from k-mer counts
srfutils.js  upstream JavaScript helper for analysis
fmd-occ      upstream low-level Makefile helper
k8           JavaScript engine used by srfutils.js
gzip         convenience compressor for local fixtures and streams
```

## Boundaries

SRF consumes dumped k-mer counts. The image does not include KMC, minimap2,
TRF, TRF-mod, plotting tools, reference genomes, read sets, or databases. Use
separate TAFFISH tools or host tools for those stages.

Upstream SRF has limited diagnostics for missing or unreadable count files, so
check paths and TAFFISH mounts when an input file cannot be opened.
