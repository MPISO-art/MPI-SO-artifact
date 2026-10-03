# MPI-SO Artifact

This repository provides the instructions for obtaining and running the
prebuilt MPI-SO artifact container. The implementation and experiment
environment are distributed through Docker Hub rather than through this Git
repository.

## Image

```text
mpisoartifact/mpiso-artifact:submission-v1
```

Docker Hub:
https://hub.docker.com/r/mpisoartifact/mpiso-artifact

The immutable image digest will be recorded here after the final binary image
is published.

## Pull

```bash
docker pull mpisoartifact/mpiso-artifact:submission-v1
```

## Inspect Published Results

```bash
docker run --rm \
  mpisoartifact/mpiso-artifact:submission-v1 reference
```

This verifies and summarizes the RQ1 and RQ2 reference tables included in the
container.

## Quick Validation

```bash
mkdir -p results
docker run --rm \
  -v "$PWD/results:/results" \
  mpisoartifact/mpiso-artifact:submission-v1 quick
```

The quick workflow checks the packaged tools, MPI-SO regressions, SimGrid/SMPI,
and a small end-to-end transformation. It is a correctness check rather than a
performance reproduction.

## Full Reproduction

```bash
mkdir -p results
docker run --rm \
  -v "$PWD/results:/results" \
  mpisoartifact/mpiso-artifact:submission-v1 full
```

The full workflow runs five repetitions of the paper configurations and writes
all logs, statuses, and generated RQ1/RQ2 tables under `results/`. It may take
several days depending on the host.

## Requirements

- x86-64 Linux with Docker
- At least 16 CPU threads and 32 GiB RAM for quick validation
- At least 100 GiB of free disk space for the complete workflow

The complete workflow uses an 8-hour symbolic-execution limit, a 2-hour static
analysis limit, and a 6-hour host-side limit for each SMPI run. Congrad
2048/NP64 is launched normally and may reach the host-side timeout.
