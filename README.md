# Minero

Containerized crypto mining tools built directly from source with optimizations for performance and security. Built from source, static binary, optimized for multiple architectures (x86_64, aarch64), UPX compression for smaller size.

## Services

### p2pool

- **Monero P2Pool**: Decentralized mining pool for Monero.
- **Image**: `ghcr.io/gr0-o-0t/minero-p2pool:latest`

### xmrig

- **Monero CPU Miner**: High-performance CPU miner for Monero.
- **Image**: `ghcr.io/gr0-o-0t/minero-xmrig:latest`

## Image Build Details

Images are built using multi-stage Docker builds for minimal final images (`FROM scratch`). Key optimizations include:

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

> I have used the `node3.monerodevs.org` as the monero node. You can use your own.
> Or you can use a public node. Make sure you can trust the node and it supports p2pool (zmq port open).
> I have included 3 p2pool enabled monero nodes I trust [here](./p2pool/p2pool-hosts.txt).
> By default p2pool is setup to mine on the mini sidechain.
> You can change this by either removing the `--mini` (port 37889) flag to mine on the main chain.
> Or you can change it to `--nano` (port 37890) to mine on the nano sidechain.
> Be sure to expose the right ports in the compose file and open the ports in your firewall.

4. **Start the services**:

   ```bash
   docker compose up -d
   ```

This will pull the latest images from GHCR and start p2pool and xmrig in the background. The setup includes:

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
docker pull ghcr.io/gr0-o-0t/minero-p2pool:latest
docker pull ghcr.io/gr0-o-0t/minero-xmrig:latest

# Run p2pool
docker run -d --name p2pool ghcr.io/gr0-o-0t/minero-p2pool:latest --mini --loglevel 3 --light-mode --host node3.monerodevs.org --rpc-port 18089 --zmq-port 18084 --wallet YOUR_WALLET_ADDRESS

# Run xmrig
docker run -d --name xmrig --link p2pool ghcr.io/gr0-o-0t/minero-xmrig:latest --randomx-1gb-pages --donate-level 0 -o localhost:3333
```

## Building Locally

To build images locally (not recommended for production, as pre-built images are optimized):

```bash
# Build p2pool
docker build -t p2pool p2pool/

# Build xmrig
docker build -t xmrig xmrig/
```

## Contributing

Contributions welcome! Images are built via GitHub Actions on tag pushes (e.g., `p2pool-4.13`).
