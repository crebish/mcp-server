# MCP Server

A remote Model Context Protocol (MCP) server implementation that provides tools for AI assistants to interact with external services and perform various operations.

## Overview

This MCP server implements the [Model Context Protocol](https://modelcontextprotocol.io/) specification, allowing AI assistants to access custom tools and capabilities through a standardized JSON-RPC interface. The server runs on port 8000 and provides several built-in tools for basic operations and conversation archiving.

## Features

- **JSON-RPC 2.0 Protocol**: Full implementation of the MCP specification
- **Built-in Tools**: 
  - `add`: Perform basic arithmetic operations
  - `reverse`: Reverse text strings
  - `save_conversation`: Archive conversations to aiarchives.duckdns.org
- **Health Monitoring**: Built-in health check endpoint
- **CORS Support**: Cross-origin resource sharing enabled
- **Error Handling**: Comprehensive error handling with proper JSON-RPC error codes

## Prerequisites

- Node.js (version 16 or higher)
- npm or yarn package manager

## Installation

1. Clone the repository:
```bash
git clone <your-repo-url>
cd mcp-server
```

2. Install dependencies:
```bash
npm install
```

3. Create a `.env` file (optional, for environment-specific configuration):
```bash
cp .env.example .env
```

## Usage

### Configuring Claude Desktop

To use this MCP server with Claude Desktop, you need to add it to your Claude Desktop configuration file.

#### Finding the Configuration File

1. Open Claude Desktop
2. Go to **Settings** (gear icon in the bottom left)
3. Click on **Advanced** in the left sidebar
4. Click **Open Config Folder** - this will open the folder containing `claude_desktop_config.json`
5. Open the `claude_desktop_config.json` file in your preferred text editor

#### Adding the MCP Server Configuration

Add the following configuration to your `claude_desktop_config.json` file:

```json
{
  "mcpServers": {
    "aiarchives": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://aiarchives.duckdns.org/mcp",
        "--allow-http",
        "--transport",
        "http-only"
      ]
    }
  }
}
```

**Note:** Make sure to restart Claude Desktop after making changes to the configuration file.

### Server Access

This MCP server is hosted remotely and accessible at `https://aiarchives.duckdns.org/mcp`. Once you've configured Claude Desktop as described above, you can immediately start using the available tools without needing to run anything locally.

**Note:** The server is always running and ready to accept requests through the MCP protocol.

### Local Deployment (Optional)

If you want to run the server locally for development or testing purposes:

1. **Start the server:**
   ```bash
   npm start
   ```
   
   Or run directly with Node.js:
   ```bash
   node server-mcp.js
   ```

2. **Update the Claude Desktop configuration** to point to your local server:
   ```json
   {
     "mcpServers": {
       "aiarchives": {
         "command": "npx",
         "args": [
           "-y",
           "mcp-remote",
           "http://localhost:8000/mcp",
           "--allow-http",
           "--transport",
           "http-only"
         ]
       }
     }
   }
   ```

**Important:** When running locally, the `save_conversation` tool will not work because it depends on the remote aiarchives API that's only available on the hosted server. The `add` and `reverse` tools will work normally.

### Available Endpoints

#### POST `/mcp`
Main MCP protocol endpoint that handles JSON-RPC requests.

**Example request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}
```

#### GET `/health`
Health check endpoint that returns server status.

**Response:**
```json
{
  "status": "healthy",
  "server": "Remote MCP Server",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

#### GET `/mcp`
Debug endpoint that confirms the server is running.

## Available Tools

### 1. `add`
Performs addition of two numbers.

**Parameters:**
- `a` (number): First number
- `b` (number): Second number

**Example:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "add",
    "arguments": {
      "a": 5,
      "b": 3
    }
  }
}
```

### 2. `reverse`
Reverses a text string.

**Parameters:**
- `text` (string): Text to reverse

**Example:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "reverse",
    "arguments": {
      "text": "Hello World"
    }
  }
}
```

### 3. `save_conversation`
Saves conversation content to aiarchives.duckdns.org and returns a shareable URL.

**Parameters:**
- `conversation` (string): Full conversation content as HTML or plain text

**Example:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "save_conversation",
    "arguments": {
      "conversation": "<html><body>Conversation content...</body></html>"
    }
  }
}
```

## MCP Protocol Methods

The server implements the following MCP protocol methods:

- `initialize`: Initialize the MCP connection
- `tools/list`: List available tools
- `tools/call`: Execute a specific tool
- `notifications/initialized`: Handle client initialization notification
- `notifications/cancelled`: Handle request cancellation notification

## Error Handling

The server returns proper JSON-RPC 2.0 error responses with standard error codes:

- `-32600`: Invalid Request
- `-32601`: Method not found
- `-32602`: Invalid params
- `-32603`: Internal error

## Development

### Project Structure
```
mcp-server/
├── server-mcp.js      # Main server implementation
├── package.json       # Dependencies and scripts
├── .gitignore        # Git ignore rules
└── README.md         # This file
```

### Adding New Tools

To add a new tool, modify the `tools/list` response and add a corresponding case in the `tools/call` switch statement in `server-mcp.js`.

### Testing

Currently, there are no automated tests. You can test the server manually using curl or any HTTP client:

```bash
# Health check
curl http://localhost:8000/health

# List tools
curl -X POST http://localhost:8000/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Dependencies

- **express**: Web framework for Node.js
- **body-parser**: Middleware for parsing request bodies
- **cors**: Cross-origin resource sharing middleware