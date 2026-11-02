# Training image for ModernBERT fine-tunes on rented GPUs (local-ai gpurent, Vast). Minimal system layer: the host
# brings the NVIDIA driver and torch's wheels bring the CUDA runtime, so no CUDA base image and no toolkit.
#   /opt/torch   torch 2.8.0+cu128 and its dependencies (torch.txt, hash-locked)
#   /opt/py      local-ai's training deps (requirements.txt = local-ai eval/router/cuda-requirements.txt);
#                /opt/py/LOCK is that file's sha256, which the job checks
#   /vol/models  ModernBERT-base at a pinned revision, checked against model.sha256
# /opt/py comes first on sys.path, /opt/torch after it (both through .pth files, so `python3 -I` sees them).
ARG BASE=ubuntu:24.04@sha256:fe81560990fae86dc4743202b2cb4d3df98d0bcf949173cecef15ce1127ef852

FROM ${BASE} AS build
RUN apt-get update && apt-get install -y --no-install-recommends python3 python3-pip ca-certificates curl \
 && rm -rf /var/lib/apt/lists/*
COPY torch.txt requirements.txt model.sha256 /src/
RUN python3 -m pip install --quiet --no-cache-dir --break-system-packages --require-hashes --no-deps \
      --target /opt/torch -r /src/torch.txt
RUN python3 -m pip install --quiet --no-cache-dir --break-system-packages --require-hashes --no-deps \
      --target /opt/py -r /src/requirements.txt \
 && sha256sum < /src/requirements.txt | cut -d' ' -f1 > /opt/py/LOCK
ARG MODEL_REPO=answerdotai/ModernBERT-base
ARG MODEL_REV=8949b909ec900327062f0ebf497f51aef5e6f0c8
RUN d=/vol/models/$(echo "$MODEL_REPO" | sed 's|/|--|')@$(echo "$MODEL_REV" | cut -c1-12) && mkdir -p "$d" && cd "$d" \
 && for f in $(cut -d' ' -f3 /src/model.sha256); do \
      curl -fsSL --retry 3 -o "$f" "https://huggingface.co/$MODEL_REPO/resolve/$MODEL_REV/$f"; done \
 && sha256sum -c --quiet /src/model.sha256

FROM ${BASE}
RUN apt-get update && apt-get install -y --no-install-recommends python3 openssl ca-certificates openssh-server procps \
 && rm -rf /var/lib/apt/lists/* /var/cache/apt/* /usr/share/doc/* /usr/share/man/*
COPY --from=build /opt/torch /opt/torch
COPY --from=build /opt/py /opt/py
COPY --from=build /vol/models /vol/models
RUN site=$(python3 -c 'import site; print(site.getsitepackages()[0])') && mkdir -p "$site" \
 && echo "import sys; sys.path.insert(0, '/opt/py')" > "$site/local-ai-vol.pth" \
 && echo /opt/torch > "$site/torch.pth" \
 && python3 -I -c "import torch, transformers; assert transformers.__file__.startswith('/opt/py/'); print(torch.__version__, torch.version.cuda, transformers.__version__)"
