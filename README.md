# hpc-containers 🧪

A small utility repo for building Singularity `.sif` container images via GitHub Actions — no local build environment required.

## How it works

```
Push .def file → GitHub Actions builds .sif → Uploaded to HuggingFace → wget on HPC
```

Singularity images are built automatically in a free GitHub-hosted Linux VM. You never need root access locally, and nothing heavy is stored on your machine.

## Repo structure

```
hpc-containers/
├── .github/
│   └── workflows/
│       └── build-sif.yml   # GitHub Actions workflow
├── defs/
│   ├── aiart.def           # AiArtFinalProject environment
│   └── ...                 # add more .def files here
├── outputs/                # generated .sif files (gitignored)
└── README.md
```

## Setup (one time)

### 1. HuggingFace token
```
huggingface.co → Settings → Access Tokens → New Token → Write
```

### 2. Add token to GitHub repo secrets
```
GitHub repo → Settings → Secrets and variables → Actions → New repository secret
Name: HF_TOKEN
Value: <paste your HuggingFace token>
```

### 3. Create HuggingFace dataset repo
```
huggingface.co → New Dataset → name: hpc-containers → Private
```

## Adding a new environment

1. Write a `.def` file and place it in `defs/`
2. Push to GitHub
3. Go to **Actions** tab → wait for build to finish (~20 min)
4. `.sif` is automatically uploaded to HuggingFace

## Downloading to HPC (NCHC)

```bash
# on the NCHC login node directly:
hf download \
  <your_hf_username>/hpc-containers \
  aiart.sif \
  --repo-type dataset \
  --token <your_hf_token> \
  --local-dir .
```

Or if repo is public, just use wget:
```bash
wget https://huggingface.co/datasets/<your_hf_username>/hpc-containers/resolve/main/aiart.sif
```

## Running on HPC

```bash
# Interactive shell
singularity shell --nv aiart.sif

# Run a script
singularity run --nv aiart.sif python3 train.py

# Execute directly
singularity exec --nv aiart.sif python3 train.py --config config.yaml
```

> `--nv` enables NVIDIA GPU passthrough inside the container.

## .def file structure

```singularity
Bootstrap: docker
From: nvidia/cuda:12.1.1-devel-ubuntu22.04

%post
    # runs once during build
    apt-get update && apt-get install -y ...
    pip install ...

%environment
    # runs every time container starts
    export PATH="/root/.local/bin:$PATH"

%runscript
    # default behaviour when you call singularity run
    exec "$@"
```

## Notes

- `.sif` files are large (5–15 GB) — do not commit them to git (already in `.gitignore`)
- GitHub Actions free tier gives 2,000 minutes/month — more than enough for occasional builds
- NCHC bans `--fakeroot` on login nodes, which is why we build here instead
- HuggingFace dataset repos support files up to 50GB per file