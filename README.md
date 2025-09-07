[![pre-commit](https://github.com/ritual-net/infernet-node/actions/workflows/pre-commit.yaml/badge.svg)](https://github.com/ritual-net/infernet-node/actions/workflows/pre-commit.yaml)

# Infernet Node

The Infernet Node is the off-chain counterpart to the [Infernet SDK](https://github.com/ritual-net/infernet-sdk) from [Ritual](https://ritual.net), responsible for servicing compute workloads and delivering responses to on-chain smart contracts.

Developers can flexibly configure an Infernet Node for both on- and off-chain compute consumption, with extensible and robust parameterization at a per-container level.

> [!IMPORTANT]  
> Infernet Node architecture, quick-start guides, and in-depth documentation can be found on the [Ritual documentation website](https://docs.ritual.net/infernet/node/introduction)

> [!WARNING]  
> This software is being provided as is. No guarantee, representation or warranty is being made, express or implied, as to the safety or correctness of the software.

---

## Configuration

The Infernet Node operates according to a set of runtime configurations. Most of these configurations have sane defaults and do not need modification. See [config.sample.json](./config.sample.json) for an example configuration.

For a full list of available configurations, check out our [Node Configuration](https://docs.ritual.net/infernet/node/configuration/v1_3_0) docs.

### Example Configuration (Annotated)

Below is an example configuration showing key fields. Do **not** include real private keys — use environment variables instead.

```json
{
  "rpc_url": "https://YOUR_RPC_PROVIDER.example",
  "chain_id": 1,
  "registry_contract": "0x0000000000000000000000000000000000000000",
  "wallet_private_key_env": "INFERNET_NODE_WALLET_PRIVATE_KEY",
  "models_path": "/models",
  "logging": {
    "level": "info",
    "file": null
  },
  "node": {
    "bind_address": "0.0.0.0",
    "port": 8080
  },
  "docker": {
    "use_docker_socket": true,
    "docker_socket_path": "/var/run/docker.sock"
  }
}
```

## Field explanations:

- rpc_url — Ethereum-compatible RPC endpoint (Infura, Alchemy, or your node).

- chain_id — Numeric chain ID (1 for mainnet, 5 for Goerli, etc.).

- registry_contract — Ritual registry contract address for the network.

- wallet_private_key_env — Name of environment variable holding private key.

- models_path — Path to ML models used by the node.

- logging.level — info, debug, or error.

- node.bind_address/node.port — HTTP server address and port.

- docker.use_docker_socket — true uses Docker socket (set false in containerd or alternative environments; see Issue #28).

## Deployment

### Locally via Docker
```bash
 # Set tag
tag="1.4.0"

# Build image from source
docker build -t ritualnetwork/infernet-node:$tag .

# Configure node
cd deploy
cp ../config.sample.json config.json
# FILL IN config.json #

# Run node and dependencies
docker compose up -d
```
## Locally via Docker (GPU-enabled)
The GPU-enabled version of the image comes pre-installed with the NVIDIA CUDA Toolkit
. Using this image on your GPU-enabled machine enables the node to interact with the attached accelerators for diagnostic and purposes, such as heartbeat checks and utilization reports.
```bash
# Set tag
tag="1.4.0"

# Build GPU-enabled image from source
docker build -f Dockerfile-gpu -t ritualnetwork/infernet-node:$tag-gpu .

# Configure node
cd deploy
cp ../config.sample.json config.json
# FILL IN config.json #

# Run node and dependencies
docker compose -f docker-compose-gpu.yaml up -d
```

## Locally via source
```bash
# Create and source new python venv
python3.11 -m venv env
source ./env/bin/activate
pip install -r requirements.txt

# Install dependencies
make install

# Configure node
cp config.sample.json config.json
# FILL IN config.json #

# Run node
make run
```
## Remotely via AWS / GCP

Follow README instructions in the infernet-deploy
 repository.

## Troubleshooting
### 1. JSON Configuration Errors

Symptom: Node fails to start, crashes immediately, or shows a JSON parsing error.
**Fix:**
```bash
python -m json.tool config.json
# or
jq . config.json
```

- Ensure required fields are present (rpc_url, wallet_private_key_env, registry_contract, etc.).

- Use environment variables for secrets, never hardcode private keys.

### 2. RPC / Contract Call Failures

**Symptom**: Runtime error like:
```bash
eth_abi.exceptions.InsufficientDataBytes: Tried to read 32 bytes, only got 0 bytes.
```

**Cause:** Node could not retrieve data from the chain (empty/invalid contract response).
**Fix:**

Check that rpc_url points to a valid endpoint.

Verify the contract address and ABI match your network.

Ensure chain_id matches your network.
Reference: **Issue #27**

### 3. Docker-Related Errors

**Symptom:** Containers fail to build or start.
**Fix:**

Docker Engine 24+ and Docker Compose v2+ recommended.

Run docker system prune if image conflicts occur.

For GPU builds, ensure drivers match Docker image CUDA version.

### 4. GPU / CUDA Issues

**Symptom:** Node cannot detect accelerators or build fails.
**Fix:**

Verify NVIDIA driver (nvidia-smi) and CUDA match.

Restart Docker if needed:
sudo systemctl restart docker

### 5. Kubernetes / containerd Environments

**Symptom:** Node expects /var/run/docker.sock and fails in containerd-only clusters.
**Cause:** Docker socket is required for current builds.
Workarounds: Use Docker runtime or follow cloud deployment guides. See **Issue #28**
.

## Publishing a Docker image
```bash
# Set tag
tag="1.4.0"

# Build for local platform
make build

# Multi-platform build and push to repo
make build-multiplatform
```
## **License**

BSD 3-clause Clear
