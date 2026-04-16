# kawaiillamacpp

GitHub Actions builds llama.cpp Docker images and publishes to GHCR.

## Images

| Backend | Path | Workflow | Image |
|---------|------|----------|-------|
| CUDA (NVIDIA) | `docker/cuda/` | `.github/workflows/cuda.yml` | `ghcr.io/kawaii-not-kawaii/llama-cpp-cuda:latest` |

See `docker/cuda/README.md` for usage details.

## How it works

Each backend has a Dockerfile + a workflow that:
- builds on push to its `docker/<backend>/**` path
- runs weekly (Mondays 06:00 UTC) against latest llama.cpp `master`
- can be manually triggered with a custom git ref + GPU arch

Built images are pushed to `ghcr.io/kawaii-not-kawaii/<image-name>` with three tags: `latest`, the ref name, and `<ref>-<YYYYMMDD>`.
