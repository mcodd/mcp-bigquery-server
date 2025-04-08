# MCP BigQuery Server (Dockerized Fork)

This repository is a fork of [ergut/mcp-bigquery-server](https://github.com/ergut/mcp-bigquery-server).

**Key changes in this fork:**
*   Added a `Dockerfile` for easy containerization.
*   Added support for configuring the server via environment variables.

For detailed information about the server's capabilities, original setup instructions, and the Model Context Protocol (MCP), please refer to the upstream repository.

## Configuration

This fork allows configuration via environment variables as an alternative to command-line arguments. Command-line arguments take precedence if both are provided.

| Environment Variable        | Corresponding Argument | Required?             | Default (if not set) |
|-----------------------------|------------------------|-----------------------|----------------------|
| `MCP_BIGQUERY_PROJECT_ID`   | `--project-id`         | **Yes**               | -                    |
| `MCP_BIGQUERY_LOCATION`     | `--location`           | No                    | `US`                 |
| `MCP_BIGQUERY_KEY_FILE`     | `--key-file`           | No (but usually needed) | -                    |

## Running with Docker

**1. Prerequisites:**

*   Docker installed and running.
*   A Google Cloud Platform (GCP) project with the BigQuery API enabled.
*   A GCP Service Account key file (`.json`) with permissions to access your BigQuery datasets (e.g., BigQuery Data Viewer, BigQuery User roles).

**2. Pull the Image:**

Pull the latest image from the registry:

```bash
docker pull mcodd/mcp-bigquery-server:latest
```

**3. Run the Container:**

Run the container interactively, setting the required environment variables and mounting your service account key file.

```bash
docker run -it --rm \
  -e MCP_BIGQUERY_PROJECT_ID="your-gcp-project-id" \
  -e MCP_BIGQUERY_LOCATION="your-bigquery-location" \ # Optional, defaults to US
  -e MCP_BIGQUERY_KEY_FILE="/app/key.json" \
  -v /path/on/your/host/to/key.json:/app/key.json:ro \
  mcodd/mcp-bigquery-server:latest
```

**Notes:**
*   Ensure the path used in `-v ...` points to your actual key file on your host machine.
*   The path in `-e MCP_BIGQUERY_KEY_FILE` must match the target path inside the container specified in the `-v` flag (e.g., `/app/key.json`).

**4. Stopping the Container:**

Press `Ctrl+D` in the terminal where the container is running to gracefully shut down the server.

**5. Running the Container under Claude Desktop:**

To use this server with Claude Desktop, you need to configure it in your `claude_desktop_config.json` file. Add an entry under `mcpServers` like the following, ensuring you replace the placeholder values:

```json
{
  "mcpServers": {
    "gbif": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "MCP_BIGQUERY_PROJECT_ID",
        "-e",
        "MCP_BIGQUERY_KEY_FILE",
        "-v",
        // IMPORTANT: Use the correct host path syntax for your OS
        // Example for matt's Windows setup:
        "C:\\\\Users\\\\coddi\\\\Documents\\\\gbif_service_account_key.json:/app/key.json:ro",
        // Example for Linux/macOS/WSL:
        // "/path/on/your/host/to/key.json:/app/key.json:ro",
        "mcodd/mcp-bigquery-server:latest"
      ],
      "env": {
        // This MUST match the target path inside the container from the -v flag
        "MCP_BIGQUERY_KEY_FILE": "/app/key.json",
        // Replace with your actual GCP Project ID
        "MCP_BIGQUERY_PROJECT_ID": "gbif-data-if-i-can-get-it"
      }
    }
    // ... other servers if any
  }
}
```

*   **Key File Path (`-v` argument):** Make sure the host path (the part before the colon) points to your actual service account key file and uses the correct syntax for your operating system (e.g., `C:\\path\\to\\file` for Windows, `/path/to/file` for Linux/macOS). The example shows the Windows path from the previous troubleshooting step, remember to escape backslashes (`\\`) in JSON strings.
*   **Internal Key File Path (`env` variable):** The value for `MCP_BIGQUERY_KEY_FILE` in the `env` section must exactly match the target path inside the container (the part after the colon in the `-v` argument), which is `/app/key.json` in this example.
*   **Project ID (`env` variable):** Replace `"gbif-data-if-i-can-get-it"` with your actual GCP project ID.
*   **`-i` vs `-it`:** The `-t` (allocate pseudo-TTY) flag is generally not needed when Claude Desktop runs the container, so `-i` (interactive, keep STDIN open) is sufficient.

## Development

If you want to build the Docker image yourself or contribute to this fork:

**1. Clone the Repository:**

```bash
git clone https://github.com/mcodd/mcp-bigquery-server.git
cd mcp-bigquery-server
```

**2. Build the Docker Image:**

Build the image using the provided `Dockerfile`. You can tag it however you like, but `mcp-bigquery-server:latest` is a reasonable default for local builds.

```bash
docker build -t mcp-bigquery-server:latest .
```

**3. (Optional) Tag for a Registry:**

Before pushing to a registry like Docker Hub, you need to tag the image with the target repository name.

```bash
# Replace <docker-image-repository> with the target (e.g., yourusername/mcp-bigquery-server)
docker tag mcp-bigquery-server:latest mcodd/mcp-bigquery-server:latest
# Optionally, add a version tag
docker tag mcp-bigquery-server:latest mcodd/mcp-bigquery-server:0.1.0 
```

**4. (Optional) Push to Registry:**

Log in to your Docker registry (e.g., `docker login`) and then push the tagged image(s).

```bash
docker push mcodd/mcp-bigquery-server:latest
# If you created a version tag, push that too
docker push mcodd/mcp-bigquery-server:0.1.0
```
