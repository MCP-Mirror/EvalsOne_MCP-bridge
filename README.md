# MCP Bridge

    ███╗   ███╗ ██████╗██████╗     ██████╗ ██████╗ ██║██████╗  ██████╗ ███████╗
    ████╗ ████║██╔════╝██╔══██╗    ██╔══██╗██╔══██╗██║██╔══██╗██╔════╝ ██╔════╝
    ██╔████╔██║██║     ██████╔╝    ██████╔╝██████╔╝██║██║  ██║██║  ███╗█████╗  
    ██║╚██╔╝██║██║     ██╔═══╝     ██╔══██╗██╔══██╗██║██║  ██║██║   ██║██╔══╝  
    ██║ ╚═╝ ██║╚██████╗██║         ██████╔╝██║  ██║██║██████╔╝╚██████╔╝███████╗
    ╚═╝     ╚═╝ ╚═════╝╚═╝         ╚═════╝ ╚═╝  ╚═╝╚═╝╚═════╝  ╚═════╝ ╚══════╝

The Model Context Protocol (MCP) introduced by Anthropic is cool. However, most MCP servers are built on Stdio transport, which, while excellent for accessing local resources, limits their use in cloud-based applications.

MCP bridge is a tiny tool that is created to solve this problem:

- **Cloud Integration**: Enables cloud-based AI services to interact with local Stdio based MCP servers
- **Protocol Translation**: Converts HTTP/HTTPS requests to Stdio communication
- **Security**: Provides secure access to local resources while maintaining control
- **Flexibility**: Supports various MCP servers without modifying their implementation
- **Easy to use**: Just run the bridge and the MCP server, zero modification to the MCP server
- **Tunnel**: Built-in support for Ngrok tunnel

By bridging this gap, we can leverage the full potential of local MCP tools in cloud-based AI applications without compromising on security.

## How it works

```
+-----------------+     HTTPS/SSE      +------------------+      stdio      +------------------+
|                 |                    |                  |                 |                  |
|  Cloud AI tools | <--------------->  |  Node.js Bridge  | <------------>  |    MCP Server    |
|   (Remote)      |       Tunnels      |    (Local)       |                 |     (Local)      |
|                 |                    |                  |                 |                  |
+-----------------+                    +------------------+                 +------------------+
```

## Prerequisites

- Node.js

## Quick Start

1. Clone the repository
   ```bash
   git clone https://github.com/modelcontextprotocol/mcp-bridge.git
   ```
   and enter the directory
   ```bash
   cd mcp-bridge
   ```
2. Copy `.env.example` to `.env` and configure the port and auth_token:
   ```bash
   cp .env.example .env
   ```
3. Install dependencies:
   ```bash
   npm install
   ```
4. Run the bridge
   ```bash
   # build the bridge
   npm run build
   # run the bridge
   npm run start
   # or, run in dev mode (supports hot reloading by nodemon)
   npm run dev
   ```
Now MCP bridge should be running on `http://localhost:3000/bridge`.

Note:
- The bridge is designed to be run on a local machine, so you still need to build a tunnel to the local MCP server that is accessible from the cloud.
- Ngrok, Cloudflare Zero Trust, and LocalTunnel are recommended for building the tunnel.

## Running with Ngrok Tunnel

MCP bridge has built-in support for Ngrok tunnel. To run the bridge with a public URL using Ngrok:

1. Get your Ngrok auth token from https://dashboard.ngrok.com/authtokens
2. Add to your .env file:
   ```
   NGROK_AUTH_TOKEN=your_ngrok_auth_token
   ```
3. Run with tunnel:
   ```bash
   # Production mode with tunnel
   npm run start:tunnel
   
   # Development mode with tunnel
   npm run dev:tunnel
   ``` 
After the bridge is running, you can see the MCP Bridge URL in the console.

## API Endpoints

After the bridge is running, there are two endpoints exposed:

- `GET /health`: Health check endpoint
- `POST /bridge`: Main bridge endpoint for receiving requests from the cloud

For example, the following is a configuration of the official [Github MCP](https://github.com/modelcontextprotocol/servers/tree/main/src/github):

```json
{
  "command": "npx",
  "args": [
    "-y",
    "@modelcontextprotocol/server-github"
  ],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "<your_github_personal_access_token>"
  }
}
```

You can send a request to the bridge as the following to list the tools of the MCP server and call a specific tool.

**Listing tools:**

```bash
curl -X POST http://localhost:3000/bridge \
     -d '{
       "method": "tools/list",
       "serverPath": "npx",
       "args": [
         "-y",
         "@modelcontextprotocol/server-github"
       ],
       "params": {},
       "env": {
         "GITHUB_PERSONAL_ACCESS_TOKEN": "<your_github_personal_access_token>"
       }
     }'
```

**Calling a tool:**

Using the search_repositories tool to search for repositories related to modelcontextprotocol

```bash
curl -X POST http://localhost:3000/bridge \
     -d '{
       "method": "tools/call",
       "serverPath": "npx",
       "args": [
         "-y",
         "@modelcontextprotocol/server-github"
       ],
       "params": {
         "name": "search_repositories",
         "arguments": {
            "query": "modelcontextprotocol"
         },
       },
       "env": {
         "GITHUB_PERSONAL_ACCESS_TOKEN": "<your_github_personal_access_token>"
       }
     }'
```

## Authentication

The bridge uses a simple token-based authentication system. The token is stored in the `.env` file. If the token is set, the bridge will use it to authenticate the request.

Sample request with token:

```bash
curl -X POST http://localhost:3000/bridge \
     -H "Authorization: Bearer <your_auth_token>" \
     -d '{
       "method": "tools/list",
       "serverPath": "npx",
       "args": [
         "-y",
         "@modelcontextprotocol/server-github"
       ],
       "params": {},
       "env": {
         "GITHUB_PERSONAL_ACCESS_TOKEN": "<your_github_personal_access_token>"
       }
     }'
```

## Configuration

Required environment variables:

- `AUTH_TOKEN`: Authentication token for the bridge API (Optional)
- `PORT`: HTTP server port (default: 3000, required)
- `LOG_LEVEL`: Logging level (default: info, required)
- `NGROK_AUTH_TOKEN`: Ngrok auth token (Optional)

## License

MIT 
