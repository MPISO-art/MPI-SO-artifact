# MPI-SO Artifact

This repository documents the experimental artifact for MPI-SO. The artifact
is distributed as a prebuilt Docker image; this GitHub repository contains only
the documentation and result figures.

## Docker Image

Docker Hub: [mpisoartifact/mpiso-artifact](https://hub.docker.com/r/mpisoartifact/mpiso-artifact)

```text
Image:  mpisoartifact/mpiso-artifact:submission-v1
Digest: sha256:7f409c1cb4351f676dfbf337c46a97f164e5cae70cbf8aa39a01b57624ad36bd
```

Pull the fixed submission image:

```bash
docker pull \
  mpisoartifact/mpiso-artifact@sha256:7f409c1cb4351f676dfbf337c46a97f164e5cae70cbf8aa39a01b57624ad36bd
```

The image contains the prebuilt MPI-SO, SVF, and SimGrid executables, their
runtime libraries, a source-free binary self-check, and the complete reference
RQ1/RQ2 tables. It contains no C/C++ implementation or benchmark source files.

## Validate the Artifact

Verify and summarize the published result tables:

```bash
docker run --rm \
  mpisoartifact/mpiso-artifact:submission-v1 reference
```

Run the binary health check:

```bash
mkdir -p results
docker run --rm \
  -v "$PWD/results:/results" \
  mpisoartifact/mpiso-artifact:submission-v1 quick
```

The quick command verifies the packaged RQ1/RQ2 tables and starts the prebuilt
MPI-SO executable on packaged LLVM bitcode. Its log and status are written to
`results/quick-validation.log` and `results/quick-validation.txt`.

Extract the complete reference tables from the image:

```bash
container=$(docker create mpisoartifact/mpiso-artifact:submission-v1)
docker cp "$container:/opt/mpiso/reference-results" ./reference-results
docker rm "$container"
```

## Evaluation

We evaluate six programs: Congrad, Data Traffic, Poisson, Adept Stencil,
Parallel Heat, and PRACE Heat. Data Traffic contributes separate S and W
configurations. Petal is the primary static-analysis baseline. Original is the
unoptimized program used to calculate end-to-end speedup, while Manual denotes
the original hand-written nonblocking implementation when available.

Each completed runtime configuration is simulated five times with SimGrid/SMPI.
Speedup is the ratio of the Original mean time to the corresponding variant's
mean time. RQ1 values are weighted by the number of executed request instances.

### RQ1: Overlapping Windows

MPI-SO enlarges the effective computation window for all seven program and
analysis configurations. The estimated Petal-to-MPI-SO window speedup ranges
from 1.014x to 1.385x. PRACE Heat has zero-length Petal windows for the measured
calls but nonzero windows under MPI-SO; its instruction ratio is therefore
reported as a newly established window rather than a finite multiplier.

| Program | Measured calls | Dynamic instances | Petal/MPI-SO window time |
| --- | ---: | ---: | ---: |
| Adept Stencil | 8 | 6,000 | 1.050x |
| Congrad | 7 | 142,864 | 1.385x |
| Parallel Heat | 18 | 9,632 | 1.014x |
| Poisson | 9 | 277,856 | 1.151x |
| PRACE Heat | 8 | 9,600 | 1.018x |
| Data Traffic S | 5 | 52 | 1.089x |
| Data Traffic W | 4 | 136 | 1.085x |

![RQ1 instruction windows](fig/rq1-instruction-windows.png)

![RQ1 window times](fig/rq1-window-times.png)

### RQ2: End-to-End Performance

The evaluation contains 32 runtime configurations. Thirty-one complete with
five valid repetitions; Congrad 2048/NP64 reaches the documented host-side SMPI
timeout. MPI-SO improves all 31 completed configurations over Original, with
speedups from 1.017x to 6.430x, and outperforms Petal in 30 of them. Among the
24 configurations with a Manual implementation, MPI-SO is faster in 16.

| Program | Completed configurations | MPI-SO speedup | Faster than Petal | Faster than Manual |
| --- | ---: | ---: | ---: | ---: |
| Adept Stencil | 6 | 1.033x-1.144x | 6/6 | 6/6 |
| Congrad | 5 | 1.070x-1.164x | 5/5 | N/A |
| Data Traffic | 2 | 1.017x-1.081x | 2/2 | N/A |
| Parallel Heat | 6 | 1.027x-1.056x | 5/6 | 6/6 |
| Poisson | 6 | 1.233x-6.430x | 6/6 | 4/6 |
| PRACE Heat | 6 | 1.042x-1.228x | 6/6 | 0/6 |

MPI-SO's advantage over Petal follows from later completion points on explored
paths. End-to-end performance also depends on request-completion overhead and
communication progress. This explains the single Parallel Heat configuration
where Petal is slightly faster. Manual grouped completion is particularly
effective in PRACE Heat, whereas MPI-SO can outperform Manual when completing
individual requests later avoids waiting for unrelated communication.

![RQ2 end-to-end performance, part 1](fig/rq2-end-to-end-1.png)

![RQ2 end-to-end performance, part 2](fig/rq2-end-to-end-2.png)

## Packaged Reference Data

The container includes the complete machine-readable tables used for the
summary above:

- one row per RQ2 runtime configuration, including all five raw measurements;
- program-level and communication-call-level RQ1 results;
- the PSE/PSA strategy coverage audit;
- the documented timeout status.

The `reference` command verifies the expected row counts and validity fields
before printing the program-level RQ1 summary.
