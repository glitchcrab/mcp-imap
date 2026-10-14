# Running mcp-imap in Docker

The image is based on `python:3.12-slim`, contains only `src/`, and runs as `nobody`. There are no dependencies to install.

`.env` is excluded from the image via `.dockerignore`, so credentials are never baked in — pass them at runtime instead.

## Build

```bash
docker build -t mcp-imap .
```

## Run

```bash
docker run -i --rm --env-file .env mcp-imap
```

The server speaks MCP over stdin/stdout, so `-i` is required. Do **not** use `-t`: a TTY corrupts the JSON-RPC stream.

Instead of `--env-file`, you can pass variables individually with `-e MAIL_USER=...` etc. See the README for the full list, including the multi-account `MAIL_<ACCOUNT>_*` pattern.

## MCP client configuration

Let the MCP client launch the container itself.

### Claude Desktop / Cursor

```json
{
  "mcpServers": {
    "imap": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "--env-file", "/path/to/mcp-imap/.env", "mcp-imap"]
    }
  }
}
```

### Hermes

```yaml
mcp_servers:
  - name: imap
    command: docker
    args:
      - run
      - -i
      - --rm
      - --env-file
      - /path/to/mcp-imap/.env
      - mcp-imap
```

## Podman

The same `Dockerfile` works with Podman — replace `docker` with `podman` in the commands above.

## Quick check

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | docker run -i --rm --env-file .env mcp-imap
```

This should print a JSON response listing the available tools.
