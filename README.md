# Minero

Containerized crypto mining tools built directly from source with optimizations for performance and security. Built from source, static binary, optimized for multiple architectures (x86_64, aarch64), UPX compression for smaller size.

## Services

### monerod

- **Monero daemon**: The monero daemon as your own node.
- **Image**: `ghcr.io/gr0-o-0t/minero-monerod:latest`

### p2pool

- **Monero P2Pool**: Decentralized mining pool for Monero.
- **Image**: `ghcr.io/gr0-o-0t/minero-p2pool:latest`

### xmrig

- **Monero CPU Miner**: High-performance CPU miner for Monero.
- **Image**: `ghcr.io/gr0-o-0t/minero-xmrig:latest`

## Image Build Details

The monerod build is copied from [simle-monerod-docker](https://github.com/sethforprivacy/simple-monerod-docker) who builds the binaries from source.

Other images are built using multi-stage Docker builds for minimal final images (`FROM scratch`). Key optimizations include:

- **Static binaries**: No runtime dependencies, enhanced security.
- **Architecture-specific optimizations**: Custom CFLAGS for better performance on supported platforms.
- **Compression**: UPX for reduced image size.
- **Caching**: BuildKit caching for faster rebuilds in CI.
- **Security**: Non-root users where applicable, static linking.

Images are automatically built and pushed to GitHub Container Registry (GHCR) via GitHub Actions on tag releases, supporting multi-architecture (amd64/arm64).

## Quick Start with Docker Compose

The repository includes a `compose.yaml` file for easy deployment:

1. **Prerequisites**: Docker and Docker Compose installed.

2. **Clone the repository**:

   ```bash
   git clone https://github.com/gr0-o-0t/minero.git
   cd minero
   ```

3. **Setup**

- Make sure to change your wallet address in the compose.yaml file.
- Make sure to open ports 18080 and 37888 if using firewall.

> The compose file is set up for you to run your own node.
> Or you can use a public node.
> Make sure you can trust the node and it supports p2pool (zmq port open).
> I have included 3 p2pool enabled monero nodes I trust [here](./p2pool/p2pool-hosts.txt).
> By default p2pool is set up to mine on the mini sidechain in this compose file.
> You can change this by either removing the `--mini` (port 37888) flag to mine on the main chain (port 37889).
> Or you can change it to `--nano` (port 37890) to mine on the nano sidechain.
> Be sure to expose the right ports in the compose file and open the ports in your firewall.

4. **Start the services**:

   ```bash
   docker compose up -d
   ```

This will pull the latest images from GHCR and start monerod, p2pool and xmrig in the background. The setup includes:

- monerod running on ports 18080
- p2pool running on ports 18080 and 37888.
- xmrig configured to mine with p2pool.
- Persistent volumes for data.
- Security options like `no-new-privileges` for p2pool.

5. **Monitor**:

   ```bash
   docker compose logs -f
   ```

6. **Stop**:

   ```bash
   docker compose down
   ```

## Manual Docker Run

If you prefer running manually:

```bash
# Pull images
docker pull ghcr.io/gr0-o-0t/minero-monerod:latest
docker pull ghcr.io/gr0-o-0t/minero-p2pool:latest
docker pull ghcr.io/gr0-o-0t/minero-xmrig:latest

# Run p2pool
docker run -d --name monerod ghcr.io/gr0-o-0t/minero-p2pool:latest \
      --rpc-restricted-bind-ip=0.0.0.0 \
      --rpc-restricted-bind-port 18089 \
      --zmq-pub=tcp://0.0.0.0:18084 \
      --confirm-external-bind \
      --restricted-rpc \
      --allow-local-ip \
      --no-igd \
      --fast-block-sync 1 \
      --prep-blocks-threads $(nproc) \
      --sync-pruned-blocks \
      --prune-blockchain \
      --check-updates disabled \
      --disable-dns-checkpoints \
      --enforce-dns-checkpointing \
      --enable-dns-blocklist \
      --public-node \
      --in-peers 32 \
      --out-peers 32 \
      --add-priority-node=p2pmd.xmrvsbeast.com:18080 \
      --add-priority-node=nodes.hashvault.pro:18080 \
      --ban-list=/home/monero/ban_list.txt

# Run p2pool
docker run -d --name p2pool ghcr.io/gr0-o-0t/minero-p2pool:latest --mini --loglevel 3 --light-mode --host node3.monerodevs.org --rpc-port 18089 --zmq-port 18084 --wallet YOUR_WALLET_ADDRESS

# Run xmrig
docker run -d --name xmrig --link p2pool ghcr.io/gr0-o-0t/minero-xmrig:latest --randomx-1gb-pages --donate-level 0 -o localhost:3333
```

## Building Locally

To build images locally (not recommended for production, as pre-built images are optimized):

```bash
# Build monerod
docker build -t monerod monerod/

# Build p2pool
docker build -t p2pool p2pool/

# Build xmrig
docker build -t xmrig xmrig/
```

## Contributing

Contributions welcome! Images are built via GitHub Actions on tag pushes (e.g., `p2pool-4.13`).
