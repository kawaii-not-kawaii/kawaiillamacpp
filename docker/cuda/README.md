# llama.cpp CUDA — GitHub Actions build

GitHub Actions builds an `llama-server` Docker image with CUDA enabled and pushes it to GHCR. The image is sized for a single GPU architecture (default `sm_86` = RTX 30xx) so the binary stays lean.

## First-time setup

1. **Push this repo to GitHub** (any name; the workflow uses `${{ github.repository_owner }}` so it works for whatever account it lives under):

   ```bash
   gh repo create sukei --private --source=. --push   # or via the web UI
   ```

   If you don't have `gh` installed and just want to use the web UI: create an empty repo at github.com/new, then:

   ```bash
   git remote add origin git@github.com:<you>/sukei.git
   git push -u origin master
   ```

2. **GHCR permission**: by default the `GITHUB_TOKEN` can push to `ghcr.io/<owner>/<repo>` style packages. The first successful push will create the package as **private**. Make it **public** at `https://github.com/users/<you>/packages/container/llama-cpp-cuda/settings` if you want pull-without-auth.

## Triggering a build

Workflow runs on:

- **Manual** dispatch (Actions tab → "build-llama-cpp-cuda" → Run workflow). Inputs: `llama_ref` (git ref) and `cuda_arch` (e.g. `86` for 30xx, `75;86;89` for Turing+Ampere+Ada).
- **Weekly cron** (Mondays 06:00 UTC) — rebuilds against latest `master`.
- **On commit** to `docker/llama-cpp-cuda/**` or the workflow file itself.

Build typically takes ~10 min cold, ~3 min with cache hits.

## Pulling on Unraid

```bash
docker pull ghcr.io/<you>/llama-cpp-cuda:latest

docker run -d --name llamacpp-cuda --gpus all \
  -p 9090:8080 \
  -v /mnt/user/appdata/llamacpp/models:/models \
  ghcr.io/<you>/llama-cpp-cuda:latest \
  -m /models/gemma-4-26b-a4b-Q4_K_M.gguf \
  -ngl 999 --cpu-moe -c 8192 -t 12 --parallel 2 \
  --reasoning off --reasoning-budget 0
```

(`--host 0.0.0.0 --port 8080` are baked in via env vars — the Dockerfile sets `LLAMA_ARG_HOST=0.0.0.0` and `LLAMA_ARG_PORT=8080`.)

## Why a custom build?

- **Faster builds** — only the GPU arch you actually use, not every Compute Capability under the sun.
- **Smaller image** — runtime CUDA layer instead of devel; no SYCL/Vulkan/ROCm baggage.
- **Easy version pinning** — set `llama_ref` to a commit hash for reproducibility.
- **Free** on public repos; ~10 min/week of the 2000 free private-repo minutes if private.
