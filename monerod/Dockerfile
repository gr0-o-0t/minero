# syntax=docker/dockerfile:1
FROM alpine:latest

ARG MONERO_VERSION=0.18.4.4

# Upgrade base image
RUN --mount=type=cache,target=/var/cache/apk set -ex && apk --update --no-cache upgrade

# Install all dependencies for static binaries + curl for healthcheck
RUN --mount=type=cache,target=/var/cache/apk set -ex && apk add --update --no-cache \
  curl \
  jq \
  ca-certificates \
  libsodium \
  ncurses-libs \
  pcsc-lite-libs \
  readline \
  tzdata \
  zeromq

# Add user and setup directories for monerod
RUN set -ex && adduser -Ds /bin/ash monero \
  && mkdir -p /home/monero/.bitmonero \
  && chown -R monero:monero /home/monero/.bitmonero

# Install and configure fixuid and switch to MONERO_USER
ARG MONERO_USER="monero"
ARG TARGETARCH
# renovate: datasource=github-releases depName=boxboat/fixuid
ARG FIXUID_VERSION=0.6.0
RUN set -ex && case ${TARGETARCH:-amd64} in \
  "arm64") curl -SsL https://github.com/boxboat/fixuid/releases/download/v${FIXUID_VERSION}/fixuid-${FIXUID_VERSION}-linux-arm64.tar.gz | tar -C /usr/local/bin -xzf - ;; \
  "amd64") curl -SsL https://github.com/boxboat/fixuid/releases/download/v${FIXUID_VERSION}/fixuid-${FIXUID_VERSION}-linux-amd64.tar.gz | tar -C /usr/local/bin -xzf - ;; \
  *) echo "Dockerfile does not support this platform"; exit 1 ;; \
  esac && \
  chown root:root /usr/local/bin/fixuid && \
  chmod 4755 /usr/local/bin/fixuid && \
  mkdir -p /etc/fixuid && \
  printf "user: ${MONERO_USER}\ngroup: ${MONERO_USER}\n" > /etc/fixuid/config.yml
USER "${MONERO_USER}:${MONERO_USER}"

# Switch to home directory and install newly built monerod binary
WORKDIR /home/${MONERO_USER}
COPY --chown=monero:monero --from=ghcr.io/sethforprivacy/simple-monerod:v0.18.4.4 /usr/local/bin/monerod /usr/local/bin/monerod
ADD --chown=monero:monero https://gui.xmr.pm/files/block.txt ./ban_list.txt

EXPOSE 18080 18089

# Copy and enable entrypoint script
COPY --chmod=0755 entrypoint.sh ./entrypoint.sh
ENTRYPOINT [ "./entrypoint.sh" ]
CMD [ "--help" ]
