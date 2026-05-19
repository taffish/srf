ARG SRF_VERSION=20240109
ARG SRF_COMMIT=e54ca8c8eccf6b1f19428b0f862f2c90575290a0
ARG SRF_COMMIT_UTC=2024-01-08T21:02:55Z
ARG SRF_COMMIT_ASIA_SHANGHAI=2024-01-09
ARG SRF_SOURCE_SHA256=8a9e6851ff02c119eade2b1a8f1758ba0fda4780cd8547630da204abf84ba6c4
ARG K8_VERSION=1.2
ARG K8_LINUX_64_SHA256=5cfc413ea59acb842a0f0241382e64dc543712b9529506a50a63ee93466e080e
ARG K8_LINUX_AARCH64_SHA256=e1ef93069d7d2fecd45a4a63213bd0dd2b26cbcad50103ae82c8c1d3fb871ce8
ARG DEBIAN_MIRROR=http://deb.debian.org/debian
ARG DEBIAN_SECURITY_MIRROR=http://deb.debian.org/debian-security

FROM debian:12-slim AS builder

ARG SRF_VERSION
ARG SRF_COMMIT
ARG SRF_COMMIT_UTC
ARG SRF_COMMIT_ASIA_SHANGHAI
ARG SRF_SOURCE_SHA256
ARG K8_VERSION
ARG K8_LINUX_64_SHA256
ARG K8_LINUX_AARCH64_SHA256
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            bzip2 \
            ca-certificates \
            curl \
            gcc \
            make \
            zlib1g-dev; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/build

RUN set -eux; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL \
      "https://github.com/lh3/srf/archive/${SRF_COMMIT}.tar.gz" \
      -o srf.tar.gz; \
    echo "${SRF_SOURCE_SHA256}  srf.tar.gz" | sha256sum -c -; \
    mkdir -p srf; \
    tar -xzf srf.tar.gz -C srf --strip-components=1; \
    cd srf; \
    grep -F "Satellite Repeat Finder" README.md >/dev/null; \
    make clean all fmd-occ; \
    mkdir -p \
      /opt/srf/bin \
      /opt/srf/share/doc/srf \
      /opt/srf/share/licenses/srf \
      /opt/srf/share/licenses/k8; \
    install -m 0755 srf fmd-occ srfutils.js /opt/srf/bin/; \
    install -m 0644 README.md /opt/srf/share/doc/srf/; \
    install -m 0644 LICENSE.txt /opt/srf/share/licenses/srf/LICENSE.txt; \
    printf 'version=%s\ncommit=%s\ncommit_date_utc=%s\ncommit_date_asia_shanghai=%s\nsource_sha256=%s\n' \
      "${SRF_VERSION}" "${SRF_COMMIT}" "${SRF_COMMIT_UTC}" "${SRF_COMMIT_ASIA_SHANGHAI}" "${SRF_SOURCE_SHA256}" \
      > /opt/srf/share/doc/srf/source.txt; \
    arch="$(uname -m)"; \
    case "${arch}" in \
      x86_64|amd64) \
        k8_url="https://conda.anaconda.org/bioconda/linux-64/k8-${K8_VERSION}-h6618dc6_3.tar.bz2"; \
        k8_sha="${K8_LINUX_64_SHA256}" ;; \
      aarch64|arm64) \
        k8_url="https://conda.anaconda.org/bioconda/linux-aarch64/k8-${K8_VERSION}-h7f4e536_3.tar.bz2"; \
        k8_sha="${K8_LINUX_AARCH64_SHA256}" ;; \
      *) echo "Unsupported build architecture: ${arch}" >&2; exit 1 ;; \
    esac; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL "${k8_url}" -o k8.tar.bz2; \
    echo "${k8_sha}  k8.tar.bz2" | sha256sum -c -; \
    mkdir -p k8-pkg; \
    tar -xjf k8.tar.bz2 -C k8-pkg; \
    install -m 0755 k8-pkg/bin/k8 /opt/srf/bin/k8; \
    install -m 0644 k8-pkg/info/licenses/LICENSE.txt /opt/srf/share/licenses/k8/LICENSE.txt; \
    { /opt/srf/bin/srf 2>&1 || true; } | grep -F "Usage: srf [options] <in.txt>" >/dev/null; \
    { /opt/srf/bin/fmd-occ 2>&1 || true; } | grep -F "Usage: fmd-occ [options] <in1.fmd>" >/dev/null; \
    PATH="/opt/srf/bin:${PATH}" /opt/srf/bin/srfutils.js 2>&1 | grep -F "Usage: srfutils.js <command>" >/dev/null

FROM debian:12-slim

LABEL maintainer="taffish@163.com"

ARG SRF_VERSION
ARG SRF_COMMIT
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            bash \
            gzip \
            libgcc-s1 \
            zlib1g; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /opt/srf /opt/srf

RUN set -eux; \
    ln -sf /opt/srf/bin/srf /usr/local/bin/srf; \
    ln -sf /opt/srf/bin/srfutils.js /usr/local/bin/srfutils.js; \
    ln -sf /opt/srf/bin/fmd-occ /usr/local/bin/fmd-occ; \
    ln -sf /opt/srf/bin/k8 /usr/local/bin/k8

ENV PATH=/opt/srf/bin:${PATH}

RUN set -eux; \
    command -v srf >/dev/null; \
    command -v srfutils.js >/dev/null; \
    command -v fmd-occ >/dev/null; \
    command -v k8 >/dev/null; \
    command -v gzip >/dev/null; \
    command -v ldd >/dev/null; \
    test "$(sed -n 's/^version=//p' /opt/srf/share/doc/srf/source.txt)" = "${SRF_VERSION}"; \
    test "$(sed -n 's/^commit=//p' /opt/srf/share/doc/srf/source.txt)" = "${SRF_COMMIT}"; \
    { srf 2>&1 || true; } | grep -F "Usage: srf [options] <in.txt>" >/dev/null; \
    { fmd-occ 2>&1 || true; } | grep -F "Usage: fmd-occ [options] <in1.fmd>" >/dev/null; \
    { srfutils.js 2>&1 || true; } | grep -F "Usage: srfutils.js <command>" >/dev/null; \
    ldd "$(command -v srf)" | grep -F "libz.so" >/dev/null; \
    ldd "$(command -v k8)" | grep -F "libz.so" >/dev/null; \
    mkdir -p /tmp/srf-check; \
    cd /tmp/srf-check; \
    printf 'CCCCC 10\n' > count.txt; \
    srf -p tiny -l 1 count.txt > out.fa 2> out.log; \
    grep -F "[M::main] read 1 distinct k-mers" out.log >/dev/null; \
    grep -F ">tiny#circ1-1 min=10,max=10,avg=10" out.fa >/dev/null; \
    grep -Fx "C" out.fa >/dev/null; \
    gzip -c count.txt > count.txt.gz; \
    srf -p gz -l 1 count.txt.gz > gz.fa; \
    grep -F ">gz#circ1-1" gz.fa >/dev/null; \
    printf '>circ1-4\nACGT\n' > circ.fa; \
    srfutils.js enlong -l 10 circ.fa > enlong.fa; \
    grep -F "ACGTACGTACGT" enlong.fa >/dev/null; \
    printf '>a\nACGT\n>b\nACGT\n' > merge.fa; \
    srfutils.js merge merged merge.fa > merged.fa; \
    grep -F ">merged*0001 a" merged.fa >/dev/null; \
    test "$(grep -c '^>' merged.fa)" -eq 1; \
    printf 'chr1\t0\t20\tcirc1-4\t2.5\t100\t4\t1\n' > hit.bed; \
    srfutils.js bed2abun hit.bed > abun.txt; \
    grep -F "circ1-4" abun.txt >/dev/null; \
    cd /; \
    rm -rf /tmp/srf-check

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=srf
