# mcpServers Configuration Guide for MCP

The `mcpServers` configuration is a JSON format used in Model Context Protocol (MCP) to define and configure MCP servers in applications like Cursor, Claude Desktop, and VS Code. This guide provides detailed information about the configuration parameters and structure.

## Basic Structure

```json
{
  "mcpServers": {
    "server-name": {
      // Server-specific configuration
    },
    // Additional servers...
  }
}
```

## Server Configuration Parameters

MCP supports two primary transport types: stdio and SSE.

### For stdio Transport (Local Servers)

Stdio servers run locally on your machine and communicate through standard input/output:

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `command` | string | The command to execute to start the MCP server | Yes |
| `args` | string[] | Array of command-line arguments to pass to the server | No |
| `env` | object | Environment variables to set for the server process | No |
| `disabled` | boolean | Whether the server is disabled (won't be loaded) | No |
| `alwaysAllow` | string[] | List of tool names that should always be allowed without confirmation | No |

### For SSE Transport (Remote Servers)

SSE (Server-Sent Events) servers can run locally or remotely and communicate over HTTP:

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `url` | string | URL to the /sse endpoint of the MCP server | Yes |
| `headers` | object | HTTP headers to include in requests to the server | No |
| `disabled` | boolean | Whether the server is disabled (won't be loaded) | No |
| `alwaysAllow` | string[] | List of tool names that should always be allowed without confirmation | No |

## Configuration Examples

### Example 1: Stdio Transport (Node.js Server)
```json
{
  "mcpServers": {
    "nodejs-server": {
      "command": "npx",
      "args": ["-y", "mcp-server"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
```

### Example 2: Stdio Transport (Python Server)
```json
{
  "mcpServers": {
    "python-server": {
      "command": "python",
      "args": ["/path/to/server.py"],
      "env": {
        "PYTHONPATH": "/path/to/lib"
      }
    }
  }
}
```

### Example 3: SSE Transport
```json
{
  "mcpServers": {
    "remote-server": {
      "url": "http://example.com:8000/sse",
      "headers": {
        "Authorization": "Bearer token123"
      }
    }
  }
}
```

### Example 4: With Auto-Allow and Disabled Settings
```json
{
  "mcpServers": {
    "postgres": {
      "command": "node",
      "args": [
        "/path/to/server-postgres/dist/index.js",
        "postgresql://postgres:postgres@localhost/database"
      ],
      "alwaysAllow": [
        "query"
      ],
      "disabled": true
    }
  }
}
```

## Configuration Locations

The mcpServers configuration can be placed in different locations depending on the application:

### Cursor
- **Project-specific**: Create a `.cursor/mcp.json` file in your project directory
- **Global**: Create a `~/.cursor/mcp.json` file in your home directory

### Claude Desktop
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Similar paths for Windows and Linux

### VS Code
- **Project-specific**: Create a `.vscode/mcp.json` file in your project directory
- **Global**: Through VS Code user settings under `mcp.servers`

## Special Parameters

### alwaysAllow

The `alwaysAllow` parameter lets you specify tools that should run without requiring user confirmation:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "...",
      "args": ["..."],
      "alwaysAllow": [
        "tool1",
        "tool2"
      ]
    }
  }
}
```

### disabled

The `disabled` parameter allows you to temporarily disable an MCP server without removing its configuration:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "...",
      "args": ["..."],
      "disabled": true
    }
  }
}
```

## Security Considerations

- **API Keys**: Never commit API keys or tokens to version control. Use environment variables instead.
- **Server Trust**: Only add MCP servers from trusted sources, as they can execute code on your machine.
- **Tool Access**: Use the `alwaysAllow` parameter carefully, especially for tools that modify data. 