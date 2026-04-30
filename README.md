# hpc-containers 🧪

A small utility repo for building Singularity `.sif` container images via GitHub Actions — no local build environment required.

## How it works

```
Push .def file → GitHub Actions builds .sif → Download artifact → Upload to HPC(or wget on hpc)
```

Singularity images are built automatically in a free GitHub-hosted Linux VM. You never need root access locally, and nothing heavy is stored on your machine.

## Repo structure

```
hpc-containers/
├── .github/
│   └── workflows/
│       └── build-sif.yml   # GitHub Actions workflow
├── envs/
│   ├── aiart.def           # AiArtFinalProject environment
│   └── ...                 # add more .def files here
├── outputs/                # generated .sif files (gitignored)
└── README.md
```

## Adding a new environment

1. Write a `.def` file and place it in `envs/`
2. Push to GitHub
3. Go to **Actions** tab → wait for build to finish (~20 min)
4. Download the sif from released url

## Uploading to HPC (NCHC)

```bash
wget <sif_url>
```

with sftp:
```bash
sftp <your_username>@140.110.148.5
sftp> put aiart.sif
sftp> exit
```

Or with rsync (resumable):
```bash
rsync -avz aiart.sif <your_username>@nano5.nchc.org.tw:/home/user/
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