# n8n-MCP Setup Guide

This guide explains how to set up and use the [n8n-MCP](https://github.com/czlonkowski/n8n-mcp) (Model Context Protocol) server with the AI Lead Generation workflow system.

## What is n8n-MCP?

n8n-MCP is a Model Context Protocol server that provides AI assistants (like Claude, GPT-4, etc.) with comprehensive access to n8n's workflow automation platform. It enables AI to:

- 📚 Access documentation for **1,084 n8n nodes** (537 core + 547 community)
- 🔧 Understand **node properties** with 99% coverage
- ⚡ Know **node operations** with 63.6% coverage
- 📄 Reference **official documentation** with 87% coverage
- 💡 Use **real-world examples** from 2,646+ templates
- 🎯 Browse **2,709 workflow templates** with full metadata

## Why Use n8n-MCP?

When building or modifying n8n workflows with AI assistance:

### Without n8n-MCP
- AI relies on general training data (may be outdated)
- Manual lookup required for node configurations
- Limited understanding of n8n-specific features
- Generic suggestions that may not work

### With n8n-MCP
- AI has real-time access to node documentation
- Accurate property schemas and operation details
- Validated configurations from working templates
- Context-aware recommendations specific to n8n

## Setup Options

### Option 1: Hosted Service (Recommended for Beginners)

The easiest way to get started - no installation required!

**Website**: [dashboard.n8n-mcp.com](https://dashboard.n8n-mcp.com)

**Features**:
- ✅ Free tier: 100 tool calls/day
- ✅ Instant access, no setup
- ✅ Always up-to-date
- ✅ No infrastructure to manage

**Steps**:
1. Go to [dashboard.n8n-mcp.com](https://dashboard.n8n-mcp.com)
2. Sign up for an account
3. Get your API key
4. Configure your MCP client

---

### Option 2: npx (Quick Local Setup)

Run n8n-MCP locally with a single command.

**Prerequisites**: 
- Node.js installed ([download](https://nodejs.org/))

**Steps**:

1. **Test that it works**:
   ```bash
   npx n8n-mcp
   ```

2. **Configure Claude Desktop**:
   
   Find your config file:
   - **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
   - **Linux**: `~/.config/Claude/claude_desktop_config.json`

3. **Add the configuration**:

   **Documentation tools only** (no n8n API access):
   ```json
   {
     "mcpServers": {
       "n8n-mcp": {
         "command": "npx",
         "args": ["n8n-mcp"],
         "env": {
           "MCP_MODE": "stdio",
           "LOG_LEVEL": "error",
           "DISABLE_CONSOLE_OUTPUT": "true"
         }
       }
     }
   }
   ```

   **Full configuration** (with n8n workflow management):
   ```json
   {
     "mcpServers": {
       "n8n-mcp": {
         "command": "npx",
         "args": ["n8n-mcp"],
         "env": {
           "MCP_MODE": "stdio",
           "LOG_LEVEL": "error",
           "DISABLE_CONSOLE_OUTPUT": "true",
           "N8N_API_URL": "http://localhost:5678",
           "N8N_API_KEY": "your-n8n-api-key"
         }
       }
     }
   }
   ```

4. **Restart Claude Desktop**

---

### Option 3: Docker (Isolated & Reproducible)

Run n8n-MCP in a container for better isolation.

**Prerequisites**:
- Docker installed ([download](https://www.docker.com/products/docker-desktop/))

**Steps**:

1. **Pull the image**:
   ```bash
   docker pull ghcr.io/czlonkowski/n8n-mcp:latest
   ```

2. **Test that it works**:
   ```bash
   docker run -it --rm ghcr.io/czlonkowski/n8n-mcp:latest
   ```

3. **Configure Claude Desktop**:

   **Documentation tools only**:
   ```json
   {
     "mcpServers": {
       "n8n-mcp": {
         "command": "docker",
         "args": [
           "run", "-i", "--rm", "--init",
           "-e", "MCP_MODE=stdio",
           "-e", "LOG_LEVEL=error",
           "-e", "DISABLE_CONSOLE_OUTPUT=true",
           "ghcr.io/czlonkowski/n8n-mcp:latest"
         ]
       }
     }
   }
   ```

   **Full configuration** (with local n8n):
   ```json
   {
     "mcpServers": {
       "n8n-mcp": {
         "command": "docker",
         "args": [
           "run", "-i", "--rm", "--init",
           "-e", "MCP_MODE=stdio",
           "-e", "LOG_LEVEL=error",
           "-e", "DISABLE_CONSOLE_OUTPUT=true",
           "-e", "N8N_API_URL=http://host.docker.internal:5678",
           "-e", "N8N_API_KEY=your-api-key",
           "-e", "WEBHOOK_SECURITY_MODE=moderate",
           "ghcr.io/czlonkowski/n8n-mcp:latest"
         ]
       }
     }
   }
   ```

   > **Note**: Use `host.docker.internal` instead of `localhost` when n8n runs on your host machine.

4. **Restart Claude Desktop**

---

## Getting Your n8n API Key

To enable workflow management features, you need an n8n API key:

1. Open your n8n instance
2. Go to **Settings** → **API**
3. Click **Create API Key**
4. Copy the key and add it to your MCP configuration

## Using n8n-MCP with the Lead Generation Workflows

Once configured, you can ask AI assistants to help with:

### Understanding the Workflows

```
Using n8n-MCP, explain how the search-workflow.json works and what each node does.
```

### Customizing Configuration

```
Help me modify the AI Lead Generation workflow to search for dental clinics 
in London, UK instead of US cities.
```

### Adding New Features

```
Using n8n-MCP, add a node to the enrichment workflow that checks if the 
business has a booking system on their website.
```

### Debugging Issues

```
The HTTP Request node in the search workflow is failing. Using n8n-MCP, 
help me understand the correct configuration for the Google Places API.
```

### Integrating New Services

```
Using n8n-MCP, replace the Slack notification with a Microsoft Teams 
notification in the main workflow.
```

## Available n8n-MCP Tools

When using n8n-MCP, AI assistants have access to these tools:

| Tool | Description |
|------|-------------|
| `list_nodes` | List all available n8n nodes |
| `get_node_info` | Get detailed info about a specific node |
| `search_nodes` | Search for nodes by keyword |
| `get_node_properties` | Get property schemas for a node |
| `get_node_operations` | Get available operations |
| `list_templates` | Browse workflow templates |
| `get_template` | Get a specific template |
| `validate_workflow` | Validate workflow JSON |

## Safety Best Practices

⚠️ **Important**: When using AI to modify workflows:

1. **Always backup first**
   ```bash
   # Export your workflow before modifications
   n8n export:workflow --id=YOUR_WORKFLOW_ID --output=backup.json
   ```

2. **Test in development**
   - Use a separate n8n instance for testing
   - Never modify production workflows directly

3. **Review AI suggestions**
   - AI can make mistakes
   - Always validate configurations manually
   - Test workflows after modifications

4. **Use version control**
   - Store workflow JSON in git
   - Track changes over time
   - Easy rollback if needed

## Troubleshooting

### "Unexpected token" errors in Claude

Ensure `MCP_MODE=stdio` is set in your configuration. This prevents debug output from interfering with the JSON-RPC protocol.

### Cannot connect to local n8n

- Check that n8n is running: `curl http://localhost:5678/healthz`
- Verify your API key is correct
- For Docker, use `host.docker.internal` instead of `localhost`

### n8n-MCP not responding

- Restart Claude Desktop after configuration changes
- Check that Node.js or Docker is properly installed
- Review logs for error messages

### Rate limiting issues

- The hosted service has a 100 calls/day free tier
- Self-hosted options have no limits
- Consider upgrading or self-hosting for heavy usage

## Additional Resources

- [n8n-MCP GitHub Repository](https://github.com/czlonkowski/n8n-mcp)
- [n8n-MCP Documentation](https://github.com/czlonkowski/n8n-mcp#readme)
- [n8n Official Documentation](https://docs.n8n.io/)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)

---

*For questions or issues with n8n-MCP, please open an issue on the [n8n-MCP repository](https://github.com/czlonkowski/n8n-mcp/issues).*
