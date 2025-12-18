# Minecraft Docker Kit

A production-ready Docker Compose setup for running a Minecraft server with Cloudflare Tunnel support. This project provides a secure, scalable, and easy-to-deploy Minecraft server environment with automated CurseForge modpack support.

## Features

- 🚀 **Easy Deployment**: One-command setup with Docker Compose
- 🔒 **Secure by Default**: Built with security best practices (no-new-privileges, capability dropping, AppArmor)
- 🌐 **Cloudflare Tunnel Integration**: Expose your server securely without port forwarding
- 📦 **CurseForge Support**: Automatic modpack installation and updates
- 🔄 **Auto-healing**: Health checks and automatic restart policies
- 📊 **Resource Management**: Configurable CPU and memory limits
- 📝 **Comprehensive Logging**: Structured logs with rotation
- ⚡ **Daily Updates**: Automatic image pulls for latest Minecraft server updates

## Prerequisites

Before you begin, ensure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/) (version 20.10 or later)
- [Docker Compose](https://docs.docker.com/compose/install/) (version 2.0 or later)
- A [Cloudflare account](https://www.cloudflare.com/) (for tunnel setup)
- At least 4GB of available RAM (8GB+ recommended for modpacks)

## Quick Start

### 1. Clone or Download the Repository

```bash
# Clone the repository
git clone https://github.com/barrax63/mc-kit.git
cd mc-kit
```

### 2. Configure Environment Variables

Copy the example environment file and edit it with your settings:

```bash
cp .env.example .env
nano .env  # or use your preferred editor
```

**Required Configuration:**

```env
# Memory settings (adjust based on your server resources)
MEMORY=12G

# RCON settings
RCON_PASSWORD=very-secure-password

# CurseForge settings
CF_PAGE_URL=https://www.curseforge.com/minecraft/modpacks/your-modpack
CF_API_KEY=your-curseforge-api-key
```

See the [Configuration Guide](#configuration-guide) for detailed information on all available options.

### 3. Set Up Cloudflare Tunnel

To expose your Minecraft server without port forwarding:

1. Create a Cloudflare Tunnel:
   - Go to [Cloudflare Zero Trust Dashboard](https://one.dash.cloudflare.com/)
   - Navigate to **Access** → **Tunnels**
   - Click **Create a tunnel**
   - Follow the wizard to create your tunnel

2. Configure the tunnel for Minecraft:
   - **Service Type**: TCP
   - **URL**: `minecraft:25565`
   
3. Copy your tunnel token and add it to `.env`:

```env
CLOUDFLARED_TUNNEL_TOKEN=your-tunnel-token-here
```

### 4. Start the Server

```bash
docker compose up -d
```

### 5. Monitor Startup

Watch the logs to ensure the server starts correctly:

```bash
docker compose logs -f minecraft
```

Wait for the message: `[Server thread/INFO]: Done! For help, type "help"`

## Configuration Guide

### General Options

| Variable | Description | Example |
|----------|-------------|---------|
| `TZ` | Server timezone | `Europe/Berlin`, `America/New_York` |
| `INIT_MEMORY` | Initial heap size for Java | `4G` |
| `MAX_MEMORY` | Maximum heap size for Java | `8G` |

### Server Settings

| Variable | Description | Example |
|----------|-------------|---------|
| `MOTD` | Message of the day shown in server list | `Welcome to My Server!` |
| `MAX_PLAYERS` | Maximum concurrent players | `20` |
| `SEED` | World generator seed (leave empty for random) | `12345678` |
| `LEVEL` | World/level name | `world` |
| `SERVER_NAME` | Server identifier | `my-minecraft-server` |

### Resource Pack

| Variable | Description | Example |
|----------|-------------|---------|
| `RESOURCE_PACK` | URL to resource pack ZIP | `https://example.com/pack.zip` |
| `RESOURCE_PACK_SHA1` | SHA1 hash of the resource pack | `abc123...` |

### RCON (Remote Console)

| Variable | Description | Example |
|----------|-------------|---------|
| `ENABLE_RCON` | Enable RCON support | `true` |
| `RCON_PASSWORD` | RCON password | `strong-password-here` |

### CurseForge

| Variable | Required | Description |
|----------|----------|-------------|
| `CF_PAGE_URL` | Yes | URL of CurseForge modpack page |
| `CF_API_KEY` | Yes | API key from [CurseForge Console](https://console.curseforge.com/) |

**Getting a CurseForge API Key:**

1. Go to [CurseForge Developer Console](https://console.curseforge.com/)
2. Log in with your account
3. Navigate to **API Keys**
4. Generate a new API key
5. Copy the key to your `.env` file

### Cloudflare Tunnel

| Variable | Required | Description |
|----------|----------|-------------|
| `CLOUDFLARED_TUNNEL_TOKEN` | Yes | Tunnel token from Cloudflare Zero Trust |

## Usage

### Starting the Server

```bash
docker compose up -d
```

### Stopping the Server

```bash
docker compose down
```

To also remove volumes (⚠️ **this deletes your world data**):

```bash
docker compose down -v
```

### Viewing Logs

```bash
# All services
docker compose logs -f

# Minecraft server only
docker compose logs -f minecraft

# Cloudflared only
docker compose logs -f cloudflared

# Last 100 lines
docker compose logs --tail=100 minecraft
```

### Accessing the Minecraft Console

```bash
docker compose exec minecraft rcon-cli
```

Or attach to the running container:

```bash
docker compose attach minecraft
```

To detach without stopping the server, press `Ctrl+P` then `Ctrl+Q`.

### Updating the Server

```bash
# Pull latest images
docker compose pull

# Restart with new images
docker compose up -d
```

## Resource Management

### Memory Allocation

The server uses two memory settings:

- **INIT_MEMORY**: Starting heap size (faster startup with higher values)
- **MAX_MEMORY**: Maximum heap size (prevents out-of-memory errors)

**Recommendations:**

| Server Type | INIT_MEMORY | MAX_MEMORY |
|-------------|-------------|------------|
| Vanilla (1-10 players) | 2G | 4G |
| Vanilla (10-20 players) | 4G | 6G |
| Light modpack | 4G | 6G |
| Heavy modpack | 6G | 10G |
| Very heavy modpack | 8G | 16G |

### CPU Allocation

Current settings in `docker-compose.yml`:

- **Limits**: Up to 8 CPU cores
- **Reservations**: Guaranteed 4 CPU cores

Adjust these in `docker-compose.yml` based on your host system:

```yaml
deploy:
  resources:
    limits:
      cpus: '4.0'  # Adjust this
      memory: ${MAX_MEMORY}
    reservations:
      cpus: '2.0'  # Adjust this
      memory: ${INIT_MEMORY}
```

## Network Configuration

### Default Network

The project uses a custom bridge network with subnet `10.254.11.0/24`.

### Port Mapping

By default, no ports are exposed to the host. This is by design for security when using Cloudflare Tunnel.

**To expose ports directly** (not recommended if using Cloudflare Tunnel):

Add to the `minecraft` service in `docker-compose.yml`:

```yaml
ports:
  - "25565:25565"  # Minecraft
  - "25575:25575"  # RCON (if enabled)
```

## Security Considerations

This project implements several security best practices:

### Container Security

- ✅ **no-new-privileges**: Prevents privilege escalation
- ✅ **AppArmor**: Mandatory Access Control
- ✅ **Capability dropping**: All capabilities dropped by default
- ✅ **Read-only filesystem**: Cloudflared runs with read-only root
- ✅ **Minimal capabilities**: Only required capabilities added back (CHOWN, SETGID, SETUID)

### Network Security

- ✅ **No exposed ports**: Use Cloudflare Tunnel instead of port forwarding
- ✅ **Isolated network**: Services run in a dedicated bridge network
- ✅ **DDoS protection**: Cloudflare provides built-in DDoS mitigation

### Best Practices

1. **Never commit your `.env` file** - it contains sensitive tokens
2. **Use strong RCON passwords** if enabling RCON
3. **Keep images updated** - run `docker compose pull` regularly

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server) - Excellent Minecraft server Docker image
- [Cloudflare Tunnel](https://www.cloudflare.com/products/tunnel/) - Secure server exposure
- [CurseForge](https://www.curseforge.com/) - Minecraft modpack hosting

