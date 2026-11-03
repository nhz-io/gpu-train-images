# Pipeline checks for local-ai gpurent: everything a run does on the box except training (copy in, decrypt, a job
# that writes small files and a manifest, encrypt, stop, copy out). No torch, no model: tens of MB instead of 4.4 GB,
# so a new host pulls it in seconds. The host brings the NVIDIA driver; nvidia-smi comes with it.
FROM ubuntu:24.04@sha256:fe81560990fae86dc4743202b2cb4d3df98d0bcf949173cecef15ce1127ef852
RUN apt-get update && apt-get install -y --no-install-recommends python3 openssl ca-certificates openssh-server procps \
 && rm -rf /var/lib/apt/lists/* /var/cache/apt/* /usr/share/doc/* /usr/share/man/*
