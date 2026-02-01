# Flux Task Management

Flux is a completely open, hackable, unopinionated task management engine with built-in MCP (Model Context Protocol) integration that lets humans, AI agents, and automations work together seamlessly.

## Features

- **Multi-Project Kanban Boards**: Manage epics, tasks, and dependencies across multiple projects
- **AI-Powered**: Built-in MCP integration for LLM-driven task management
- **Task Dependencies**: Visual dependency tracking and blocking status
- **Real-Time Updates**: Server-sent events keep all clients synchronized
- **Webhooks**: Integrate with external services (Slack, GitHub, CI/CD)
- **API-First Design**: Full REST API for automations and integrations
- **Git-Native Sync**: Push/pull tasks via git branches
- **CLI Support**: Complete command-line interface with MCP parity
- **Blob Storage**: Attach files directly to tasks
- **Priority System**: P0/P1/P2 priorities for smart task ordering

## Deployment

This deployment uses persistent storage on the SD card at `/mnt/sdcard/flux` for firmware re-install resilience.

### Building from Source

The Flux repository is configured as a git submodule. To initialize it:

```bash
# First time setup
cd TachyonApps
git submodule update --init --recursive Flux/flux-repo

# To update to latest Flux version
cd Flux/flux-repo
git pull origin main
```

The docker-compose.yml will build the image from the `flux-repo` submodule directory.

### Accessing Flux

After deployment, access the Flux web interface at:
- **Local**: http://localhost:3100
- **Tachyon Device**: http://<device-ip>:3100
- **Domain**: https://flux.internal.keepdream.in (via Caddy reverse proxy)

### Data Persistence

Flux uses two persistent storage locations on the SD card:
- **Task Data**: SQLite database at `/mnt/sdcard/flux/data/data.sqlite`
- **User Home**: Home directory at `/mnt/sdcard/flux/home` for user-specific configurations

### Environment Variables

The following environment variables can be configured in `/mnt/sdcard/flux/.env`:

- `FLUX_DATA`: Path to SQLite database (default: `/app/packages/data/data.sqlite`)
- `PORT`: Web server port (default: `3000`)
- `FLUX_API_KEY`: Optional API key for authentication

## MCP Integration

Flux includes native MCP (Model Context Protocol) support for AI agents. Configure your AI assistant to connect to the Flux MCP server:

```bash
# For Claude Desktop/Code via Docker
docker exec -i flux-web bun packages/mcp/dist/index.js
```

See the [MCP documentation](https://github.com/sirsjg/flux/blob/main/docs/mcp.md) for full integration details.

## CLI Usage

Install the Flux CLI globally:

```bash
npm install -g flux-tasks
```

Initialize with your Docker server:

```bash
flux init --server http://localhost:3100
flux project list
flux ready
```

## Additional Documentation

- [Installation (Docker)](https://github.com/sirsjg/flux/blob/main/docs/installation-docker.md)
- [Installation (Source)](https://github.com/sirsjg/flux/blob/main/docs/installation-source.md)
- [CLI Reference](https://github.com/sirsjg/flux/blob/main/docs/cli.md)
- [API Documentation](https://github.com/sirsjg/flux/blob/main/docs/api.md)
- [MCP Integration](https://github.com/sirsjg/flux/blob/main/docs/mcp.md)
- [Webhooks](https://github.com/sirsjg/flux/blob/main/docs/webhooks.md)
- [Assistant Setup](https://github.com/sirsjg/flux/blob/main/docs/assistant-setup.md)
- [Architecture](https://github.com/sirsjg/flux/blob/main/docs/architecture.md)
- [Ideas & Use Cases](https://github.com/sirsjg/flux/blob/main/docs/ideas.md)

## GitHub Repository

https://github.com/sirsjg/flux

## License

MIT License - See the [GitHub repository](https://github.com/sirsjg/flux) for details.
