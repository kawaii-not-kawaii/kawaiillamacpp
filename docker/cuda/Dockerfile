# llama.cpp server, CUDA build, optimized for Ampere consumer GPUs (RTX 30xx).
#
# Build args:
#   LLAMA_REF — git ref to build (default master)
#   CUDA_ARCH — comma-separated CC list (default 86 = RTX 30xx;
#               use "75;86;89;90" for broader Turing/Ampere/Ada/Hopper)
#
# Targets a 22-23 series CUDA toolkit so that 32-bit subnormals + fp16 perf
# are enabled. Final runtime layer drops the toolkit to keep the image lean.

ARG CUDA_VERSION=12.6.2
ARG UBUNTU_VERSION=22.04

FROM nvidia/cuda:${CUDA_VERSION}-devel-ubuntu${UBUNTU_VERSION} AS build
ARG LLAMA_REF=master
ARG CUDA_ARCH=86

RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
        cmake \
        build-essential \
        libcurl4-openssl-dev \
        ca-certificates \
        pkg-config && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /src
RUN git clone --depth 1 --branch ${LLAMA_REF} https://github.com/ggml-org/llama.cpp .

RUN cmake -B build \
      -DGGML_CUDA=ON \
      -DCMAKE_CUDA_ARCHITECTURES="${CUDA_ARCH}" \
      -DLLAMA_CURL=ON \
      -DBUILD_SHARED_LIBS=OFF && \
    cmake --build build --config Release --target llama-server -j"$(nproc)"

FROM nvidia/cuda:${CUDA_VERSION}-runtime-ubuntu${UBUNTU_VERSION}
RUN apt-get update && apt-get install -y --no-install-recommends \
        libcurl4-openssl-dev \
        libgomp1 \
        ca-certificates && \
    rm -rf /var/lib/apt/lists/*

COPY --from=build /src/build/bin/llama-server /usr/local/bin/llama-server

ENV LLAMA_ARG_HOST=0.0.0.0 \
    LLAMA_ARG_PORT=8080
EXPOSE 8080
ENTRYPOINT ["llama-server"]
