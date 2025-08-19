# Versionator MCP Server

An MCP (Model Context Protocol) server that queries package registries (npm, RubyGems, PyPI, Hex.pm) to retrieve the latest release versions of packages. It follows a strict fail-hard policy and always returns current data.

## Features

- Query latest versions from npm, RubyGems, PyPI, and Hex.pm
- Support for language/ecosystem aliases  
- No caching - always returns current latest version
- Fail-hard error handling (no fallbacks)
- Optional package metadata (description, homepage, license)
- Configurable request timeout
- Supports both stdio and HTTP transports

## Installation

### From PyPI (recommended)

```bash
pip install versionator-mcp
```

### From Source

```bash
git clone https://github.com/trianglegrrl/versionator-mcp.git
cd versionator-mcp
pip install -e .
```

## Usage

### As a Standalone Server

```bash
# Start with HTTP transport (default port 8083)
python -m versionator_mcp.main

# Start with stdio transport
MCP_TRANSPORT=stdio python -m versionator_mcp.main

# Custom port
FASTMCP_PORT=9000 python -m versionator_mcp.main
```

### Integration with Claude Desktop

Add to your Claude Desktop MCP configuration:

```json
{
  "mcpServers": {
    "versionator": {
      "url": "http://localhost:8083/mcp",
      "transport": "http"
    }
  }
}
```

For remote deployments, replace `localhost` with your server's IP address.

## Configuration

Environment variables:

- `FASTMCP_HOST`: Server bind address (default: 0.0.0.0)
- `FASTMCP_PORT`: Server port (default: 8083)
- `EXTERNAL_IP`: External IP for access (default: localhost)
- `MCP_TRANSPORT`: Transport mode (default: streamable-http, can be: stdio)
- `VERSIONATOR_REQUEST_TIMEOUT`: API request timeout in seconds (default: 30)

## Available Functions

### 1. `get_package_version` - Universal Package Version Query

Query the latest version from any supported registry.

**Parameters:**
- `package_manager` (str): Registry name or alias
- `package_name` (str): Name of the package

**Supported Registries:**
- `npm` (aliases: `node`, `nodejs`)
- `rubygems` (aliases: `gem`, `ruby`)
- `pypi` (aliases: `pip`, `python`)
- `hex` (aliases: `elixir`, `hex.pm`)

**Examples:**
```python
# Query npm package
get_package_version("npm", "react")
# Returns: {"name": "react", "version": "19.1.1", ...}

# Query with alias
get_package_version("python", "django")
# Returns: {"name": "django", "version": "5.2.5", ...}
```

### 2. Registry-Specific Functions

- `get_npm_package(package_name)` - NPM packages
- `get_ruby_gem(gem_name)` - RubyGems packages  
- `get_python_package(package_name)` - PyPI packages
- `get_elixir_package(package_name)` - Hex.pm packages

## Response Format

All functions return a PackageVersion object:

```json
{
  "name": "react",
  "version": "19.1.1", 
  "registry": "npm",
  "registry_url": "https://registry.npmjs.org/react/latest",
  "query_time": "2025-08-13T10:30:00Z",
  "description": "React is a JavaScript library for building user interfaces.",
  "homepage": "https://react.dev/",
  "license": "MIT"
}
```

## Error Handling

The server follows a strict **FAIL HARD** policy:

- **No Fallbacks**: Never returns cached or default values
- **No Suppression**: All errors propagate to the caller
- **Clear Messages**: Errors include context and details
- **Input Validation**: Validates before making API calls

Common errors:
- `ValueError`: Invalid package name or unknown package manager
- `Exception`: Package not found or API failures

## Development

### Setup

```bash
# Clone the repository
git clone https://github.com/trianglegrrl/versionator-mcp.git
cd versionator-mcp

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install in development mode
pip install -e .
```

### Running Tests

```bash
# Install test dependencies
pip install pytest pytest-asyncio pytest-cov

# Run tests
pytest

# Run tests with coverage
pytest --cov=versionator_mcp
```

### Testing the Server

```bash
# Test HTTP transport
python -m versionator_mcp.main &
curl -X POST http://localhost:8083/mcp/ \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'

# Test stdio transport  
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | MCP_TRANSPORT=stdio python -m versionator_mcp.main
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Registry APIs

The server queries these endpoints:

- **npm**: `https://registry.npmjs.org/{package}/latest`
- **RubyGems**: `https://rubygems.org/api/v1/versions/{gem}/latest.json`
- **PyPI**: `https://pypi.org/pypi/{package}/json`
- **Hex.pm**: `https://hex.pm/api/packages/{package}`

## Performance Considerations

- **No Caching**: Each call makes a fresh API request
- **Timeout**: Configurable via `VERSIONATOR_REQUEST_TIMEOUT`
- **Concurrent Requests**: Async implementation allows parallel queries
- **Rate Limits**: Be mindful of registry rate limits

## Changelog

### v1.0.0
- Initial release
- Support for npm, RubyGems, PyPI, and Hex.pm
- HTTP and stdio transport modes
- Comprehensive error handling
- MCP protocol compliance