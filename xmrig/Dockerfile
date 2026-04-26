# syntax=docker/dockerfile:1
FROM alpine:latest AS builder

ARG TARGETARCH

# renovate: datasource=github-releases depName=xmrig/xmrig
ARG XMRIG_VERSION=6.26.0

# Install build dependencies (following official Alpine build guide)
RUN --mount=type=cache,target=/var/cache/apk apk add --no-cache \
  file \
  git \
  make \
  cmake \
  libstdc++ \
  gcc \
  g++ \
  automake \
  libtool \
  autoconf \
  linux-headers \
  wget \
  patch \
  upx

# Clone XMRig repository
WORKDIR /build
RUN git clone --depth 1 -b v${XMRIG_VERSION} --single-branch https://github.com/xmrig/xmrig.git

WORKDIR /build/xmrig

# Copy and apply donateless patch
COPY enable-donateless-mode.patch .
RUN patch -p1 < enable-donateless-mode.patch

# Build static dependencies using official script
WORKDIR /build/xmrig/scripts
RUN ./build_deps.sh

# Create build directory
WORKDIR /build/xmrig/build

# Configure with architecture-specific options
RUN if [ "$TARGETARCH" = "aarch64" ]; then \
  CMAKE_ARCH_OPTS="-DARM_TARGET=8"; \
  else \
  CMAKE_ARCH_OPTS=""; \
  fi && \
  cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DXMRIG_DEPS=scripts/deps \
  -DBUILD_STATIC=ON \
  -DWITH_MSR=ON \
  -DWITH_SECURE_JIT=ON \
  -DWITH_DMI=ON \
  -DWITH_HWLOC=ON \
  -DWITH_TLS=ON \
  -DWITH_HTTP=ON \
  -DWITH_CUDA=OFF \
  -DWITH_OPENCL=ON \
  ${CMAKE_ARCH_OPTS} && \
  make -j $(nproc) && \
  strip -s xmrig && \
  upx --best --lzma xmrig

# Verify static binary
RUN file xmrig && \
  ./xmrig --version && \
  (ldd xmrig 2>&1 | grep -q "not a dynamic executable" && echo "✓ Static binary verified" || \
  echo "⚠ Binary may have some dynamic dependencies")

# Final minimal image
FROM scratch
COPY --from=builder /build/xmrig/build/xmrig /xmrig
WORKDIR /data
ENTRYPOINT ["/xmrig"]
CMD ["--help"]
