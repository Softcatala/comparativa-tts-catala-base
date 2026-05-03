# comparativa-tts-catala-base

Base Docker image for the [Síntesi en català](https://huggingface.co/spaces/softcatala/comparativa-tts-catala) HuggingFace space.

## Why this exists

The HuggingFace space compares several Catalan TTS engines and depends on a set of heavy, slow-to-build system dependencies:

- **festival** + **festvox-ca-ona-hts** (Catalan female voice, from Debian repos)
- **festvox-ca-pau-hts** (Catalan male voice) — this package was only ever available in the `ppa:zeehio/festcat` Ubuntu PPA, which is unreachable from HuggingFace's build environment. The voice files are installed directly from [FestCat/festcat-voices](https://github.com/FestCat/festcat-voices) instead.
- **espeak-ng** compiled from source from the [`ca-pr` branch](https://github.com/projecte-aina/espeak-ng/tree/ca-pr) of projecte-aina's fork, which adds Catalan support not yet in upstream.
- **VITS** with its compiled `monotonic_align` C extension.

Building all of this from scratch on every HuggingFace deploy took 10–15 minutes and was fragile: any branch deletion, PPA outage, or base image tag move would silently break the space (as happened in May 2026 when `python:3.9` moved from Debian Bullseye to Trixie, removing `apt-key`).

## What this repo does

It builds and publishes `ghcr.io/softcatala/comparativa-tts-catala-base` via GitHub Actions whenever `Dockerfile.base` changes. All upstream dependencies are pinned by commit SHA so the image is reproducible and immune to upstream changes.

The HuggingFace space `Dockerfile` simply does:

```dockerfile
FROM ghcr.io/softcatala/comparativa-tts-catala-base:latest
```

…and then copies the application code on top. Space builds take seconds instead of minutes, and the compiled artefacts are preserved in the base image indefinitely.

## Updating a dependency

1. Update the relevant commit SHA in `Dockerfile.base`
2. Push — GitHub Actions rebuilds and pushes a new `latest` tag to GHCR
3. Trigger a rebuild of the HuggingFace space
