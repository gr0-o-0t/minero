# syntax=docker/dockerfile:1
FROM alpine:latest AS builder

# Set architecture argument (x86_64, aarch64, or riscv64)
ARG TARGETARCH=x86_64
ARG P2POOL_VERSION=4.13

# Set compiler flags based on architecture
ARG CFLAGS_ARCH=""
RUN if [ "$TARGETARCH" = "x86_64" ]; then \
  export CFLAGS_ARCH="-ffunction-sections"; \
  elif [ "$TARGETARCH" = "aarch64" ]; then \
  export CFLAGS_ARCH="-ffunction-sections -mfix-cortex-a53-835769 -mfix-cortex-a53-843419"; \
  elif [ "$TARGETARCH" = "riscv64" ]; then \
  export CFLAGS_ARCH="-ffunction-sections"; \
  fi

# Install build dependencies
RUN --mount=type=cache,target=/var/cache/apk apk add --no-cache \
  file \
  git \
  cmake \
  gcc \
  g++ \
  make \
  linux-headers \
  xz \
  upx

# Clone p2pool repository with submodules
WORKDIR /build
RUN git clone --depth 1 --single-branch \
  -b v${P2POOL_VERSION} --recurse-submodules --shallow-submodules \
  https://github.com/SChernykh/p2pool.git /build/p2pool

# Build libcurl
WORKDIR /build/p2pool/external/src/curl
RUN cmake . \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_C_FLAGS="-Os -flto=auto ${CFLAGS_ARCH}" \
  -DBUILD_CURL_EXE=OFF \
  -DBUILD_SHARED_LIBS=OFF \
  -DCURL_DISABLE_INSTALL=ON \
  -DCURL_ENABLE_EXPORT_TARGET=OFF \
  -DCURL_DISABLE_HEADERS_API=ON \
  -DCURL_DISABLE_BINDLOCAL=ON \
  -DBUILD_LIBCURL_DOCS=OFF \
  -DBUILD_MISC_DOCS=OFF \
  -DENABLE_CURL_MANUAL=OFF \
  -DCURL_ZLIB=OFF \
  -DCURL_BROTLI=OFF \
  -DCURL_ZSTD=OFF \
  -DCURL_DISABLE_ALTSVC=ON \
  -DCURL_DISABLE_COOKIES=ON \
  -DCURL_DISABLE_DOH=ON \
  -DCURL_DISABLE_GETOPTIONS=ON \
  -DCURL_DISABLE_HSTS=ON \
  -DCURL_DISABLE_LIBCURL_OPTION=ON \
  -DCURL_DISABLE_MIME=ON \
  -DCURL_DISABLE_NETRC=ON \
  -DCURL_DISABLE_NTLM=ON \
  -DCURL_DISABLE_PARSEDATE=ON \
  -DCURL_DISABLE_PROGRESS_METER=ON \
  -DCURL_DISABLE_SHUFFLE_DNS=ON \
  -DCURL_DISABLE_SOCKETPAIR=ON \
  -DCURL_DISABLE_VERBOSE_STRINGS=ON \
  -DCURL_DISABLE_WEBSOCKETS=ON \
  -DHTTP_ONLY=ON \
  -DCURL_ENABLE_SSL=OFF \
  -DUSE_LIBIDN2=OFF \
  -DCURL_USE_LIBPSL=OFF \
  -DCURL_USE_LIBSSH2=OFF \
  -DENABLE_UNIX_SOCKETS=OFF \
  -DCURL_DISABLE_TESTS=ON \
  -DUSE_NGHTTP2=OFF \
  -DBUILD_EXAMPLES=OFF \
  -DP2POOL_BORINGSSL=ON \
  -DCURL_DISABLE_SRP=ON \
  -DCURL_DISABLE_AWS=ON \
  -DCURL_DISABLE_BASIC_AUTH=ON \
  -DCURL_DISABLE_BEARER_AUTH=ON \
  -DCURL_DISABLE_KERBEROS_AUTH=ON \
  -DCURL_DISABLE_NEGOTIATE_AUTH=ON \
  -DOPENSSL_INCLUDE_DIR=../grpc/third_party/boringssl-with-bazel/include && \
  make -j$(nproc)

# Build libuv
WORKDIR /build/p2pool/external/src/libuv
RUN mkdir build && cd build && \
  cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_C_FLAGS="-Os -flto=auto ${CFLAGS_ARCH}" \
  -DBUILD_TESTING=OFF \
  -DLIBUV_BUILD_SHARED=OFF && \
  make -j$(nproc)

# Build libzmq
WORKDIR /build/p2pool/external/src/libzmq
RUN mkdir build && cd build && \
  cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_POLICY_VERSION_MINIMUM="3.5" \
  -DCMAKE_C_FLAGS="-Os -flto=auto ${CFLAGS_ARCH}" \
  -DCMAKE_CXX_FLAGS="-Os -flto=auto ${CFLAGS_ARCH}" \
  -DWITH_LIBSODIUM=OFF \
  -DWITH_LIBBSD=OFF \
  -DBUILD_TESTS=OFF \
  -DWITH_DOCS=OFF \
  -DENABLE_DRAFTS=OFF \
  -DBUILD_SHARED=OFF && \
  make -j$(nproc)

# Build p2pool
WORKDIR /build/p2pool
RUN mkdir build && cd build && \
  cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_POLICY_VERSION_MINIMUM="3.5" \
  -DCMAKE_C_FLAGS="${CFLAGS_ARCH} -Wl,-s -Wl,--gc-sections" \
  -DCMAKE_CXX_FLAGS="${CFLAGS_ARCH} -Wl,-s -Wl,--gc-sections" \
  -DWITH_GRPC=OFF \
  -DSTATIC_BINARY=ON \
  -DARCH_ID=${TARGETARCH} && \
  make -j$(nproc) p2pool && \
  strip -s p2pool && \
  upx --best --lzma p2pool

# Verify static binary
RUN file build/p2pool && \
  build/p2pool --version && \
  (ldd build/p2pool 2>&1 | grep -q "not a dynamic executable" && echo "✓ Static binary verified" || \
  echo "⚠ Binary may have some dynamic dependencies")

# Run tests
WORKDIR /build/p2pool/build
RUN ./p2pool --test && \
  make -j$(nproc) randomx-tests && \
  external/src/RandomX/randomx-tests

# Output binary location
FROM scratch
COPY --chown=1000:1000 --from=builder /build/p2pool/build/p2pool /p2pool
USER 1000
WORKDIR /data
ENTRYPOINT ["/p2pool"]
CMD ["--help"]
