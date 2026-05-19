srf 20240109-r2

TAFFISH wrapper for upstream SRF, Satellite Repeat Finder.

Upstream SRF has no semantic release tag and no runtime --version flag.
This package pins upstream commit:
  e54ca8c8eccf6b1f19428b0f862f2c90575290a0

Commit time:
  UTC:           2024-01-08T21:02:55Z
  Asia/Shanghai: 2024-01-09

Usage:
  taf-srf --help
  taf-srf --version
  taf-srf -- --help
  taf-srf srf [srf options] <count.txt>
  taf-srf -- [srf options] <count.txt>
  taf-srf srfutils.js <command> [arguments]
  taf-srf fmd-occ [arguments]
  taf-srf --compile [args...]

Wrapper options:
  -h, --help       Show this TAFFISH help
  -v, --version    Show TAFFISH package version
  --compile        Print generated shell code instead of running it
  --               Pass following arguments to the default .taf program

Primary examples:
  taf-srf -- --help
  taf-srf srf -p sample count.txt > sample.srf.fa
  taf-srf -p sample -l 100 count.txt > sample.srf.fa

Input contract:
  srf reads a KMC dump-style k-mer count table. Each line should contain one
  k-mer and one positive count, separated by whitespace:

    ACGTTGCAACGTTGCAA 125
    CGTTGCAACGTTGCAAC 123

  Plain text and gzip-compressed count tables are supported through zlib.
  Put input files in a path visible inside the TAFFISH container, such as the
  current project directory or your home directory. Upstream SRF has limited
  diagnostics for missing or unreadable count files.

Helper commands:
  taf-srf srfutils.js
  taf-srf srfutils.js enlong sample.srf.fa > sample.srf.enlong.fa
  taf-srf srfutils.js paf2bed sample.srf.paf > sample.srf.bed
  taf-srf srfutils.js bed2abun sample.srf.bed > sample.srf.abun.txt

Included commands:
  srf          assemble satellite repeat contigs from k-mer counts
  srfutils.js  upstream JavaScript helper for downstream analysis
  fmd-occ      upstream low-level Makefile helper
  k8           JavaScript engine used by srfutils.js
  gzip         compressor for local fixtures and streams

Common srfutils.js commands:
  enlong    elongate circular contigs before mapping
  filter    filter read-to-contig PAF alignments
  paf2bed   extract non-overlapping satellite regions
  bed2cnt   summarize per-chromosome counts
  bed2abun  estimate per-contig abundance
  merge     merge identical sequences

Boundary:
  This focused image does not include KMC, minimap2, TRF, TRF-mod, plotting
  tools, reference genomes, read sets, or satellite-repeat databases. Use
  separate TAFFISH tools or host tools for those workflow stages.

Source metadata inside the image:
  /opt/srf/share/doc/srf/source.txt
