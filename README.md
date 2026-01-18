# eRemote Server

**A custom deployment of RustDesk Server by [Celeratec](https://celeratec.com)**

[![Build eRemote Server](https://github.com/Celeratec/eremote-server/actions/workflows/build-eremote.yml/badge.svg)](https://github.com/Celeratec/eremote-server/actions/workflows/build-eremote.yml)

eRemote Server is Celeratec's self-hosted remote desktop infrastructure, built on the open-source [RustDesk Server](https://github.com/rustdesk/rustdesk-server). It provides secure, private remote access for managed service providers (MSPs) and enterprises.

## What is eRemote?

eRemote is Celeratec's branded remote desktop solution, offering:

- **Self-hosted infrastructure** – Full control over your data and connections
- **Private relay servers** – No third-party dependencies
- **Secure communications** – End-to-end encrypted connections
- **MSP-ready** – Designed for managing multiple client endpoints

## Architecture

eRemote Server consists of two services:

| Service | Port | Description |
|---------|------|-------------|
| `hbbs` | 21115-21116, 21118 | ID/Rendezvous server – handles device registration and connection brokering |
| `hbbr` | 21117, 21119 | Relay server – relays traffic when direct P2P connection isn't possible |

## Quick Start

### Using Docker Compose (Recommended)

```bash
git clone https://github.com/Celeratec/eremote-server.git
cd eremote-server
docker compose up -d
```

### Required Ports

Ensure these ports are open on your firewall:

| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| 21115 | TCP | hbbs | NAT type test |
| 21116 | TCP/UDP | hbbs | ID registration and heartbeat |
| 21117 | TCP | hbbr | Relay |
| 21118 | TCP | hbbs | WebSocket (web client) |
| 21119 | TCP | hbbr | WebSocket relay |

### Get Your Public Key

After starting the server, retrieve your public key for client configuration:

```bash
cat data/id_ed25519.pub
```

## Client Configuration

Configure eRemote/RustDesk clients with:

1. **ID Server**: `your-server-ip:21116`
2. **Relay Server**: `your-server-ip:21117`
3. **Key**: Contents of `data/id_ed25519.pub`

## Building from Source

```bash
cargo build --release
```

This produces three binaries in `target/release/`:

- `hbbs` – eRemote ID/Rendezvous server
- `hbbr` – eRemote Relay server
- `rustdesk-utils` – CLI utilities

## Based on RustDesk

eRemote Server is built on [RustDesk Server](https://github.com/rustdesk/rustdesk-server), an open-source remote desktop server. We maintain this fork to:

- Integrate with Celeratec's CI/CD pipeline
- Deploy to our private AWS ECR registry
- Provide MSP-specific configurations

For the upstream project, visit [rustdesk.com](https://rustdesk.com).

## Documentation

- [eRemote Setup Guide](EREMOTE_SETUP.md) – Detailed MSP deployment guide
- [RustDesk Documentation](https://rustdesk.com/docs/en/self-host/) – Upstream documentation

## License

This project is licensed under the same terms as RustDesk Server. See [LICENSE](LICENSE) for details.

---

**Maintained by [Celeratec](https://celeratec.com)** – IT Solutions for Modern Businesses
