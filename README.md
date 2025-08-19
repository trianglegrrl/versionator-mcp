# Versionator MCP Server

[![CI Pipeline](https://github.com/trianglegrrl/versionator-mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/trianglegrrl/versionator-mcp/actions/workflows/ci.yml)
[![PyPI version](https://badge.fury.io/py/versionator-mcp.svg)](https://badge.fury.io/py/versionator-mcp)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)

An MCP (Model Context Protocol) server that queries package registries (npm, RubyGems, PyPI, Hex.pm) to retrieve the latest release versions of packages. It follows a strict fail-hard policy and always returns current data.

## Features

- 🔍 Query latest versions from npm, RubyGems, PyPI, and Hex.pm
- 🏷️ Support for language/ecosystem aliases  
- ⚡ No caching - always returns current latest version
- 💥 Fail-hard error handling (no fallbacks)
- 📋 Optional package metadata (description, homepage, license)
- ⏱️ Configurable request timeout
- 🖥️ **Optimized for local Claude Desktop integration**

## Quick Start

### Option 1: Using uvx (Recommended)

The easiest way to use Versionator with Claude Desktop is via [uvx](https://github.com/astral-sh/uv):

```bash
# Install uvx if you haven't already
curl -LsSf https://astral.sh/uv/install.sh | sh

# Run Versionator directly with uvx
uvx versionator-mcp
```

### Option 2: Install Locally

```bash
# Install from PyPI
pip install versionator-mcp

# Or install with pipx for isolated environment
pipx install versionator-mcp
```

## Claude Desktop Integration

### Method 1: Using uvx (Recommended)

Add this configuration to your Claude Desktop MCP settings:

**On macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`  
**On Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "versionator": {
      "command": "uvx",
      "args": ["versionator-mcp"]
    }
  }
}
```

### Method 2: Using pipx

```json
{
  "mcpServers": {
    "versionator": {
      "command": "pipx",
      "args": ["run", "versionator-mcp"]
    }
  }
}
```

### Method 3: Direct Python execution

```json
{
  "mcpServers": {
    "versionator": {
      "command": "python",
      "args": ["-m", "versionator_mcp.main"]
    }
  }
}
```

### Method 4: Using virtual environment

If you have the package installed in a specific virtual environment:

```json
{
  "mcpServers": {
    "versionator": {
      "command": "/path/to/your/venv/bin/python",
      "args": ["-m", "versionator_mcp.main"]
    }
  }
}
```

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

## Configuration

Environment variables (optional):

- `VERSIONATOR_REQUEST_TIMEOUT`: API request timeout in seconds (default: 30)

## Troubleshooting

### Claude Desktop Issues

1. **Server not starting**: Check that the command path is correct in your configuration
2. **Permission errors**: Ensure the Python executable has proper permissions
3. **Package not found**: Verify the package is installed and accessible from the command line

### Testing Your Setup

You can test the server directly from the command line:

```bash
# Test with uvx
uvx versionator-mcp

# Test with pipx
pipx run versionator-mcp

# Test direct installation
python -m versionator_mcp.main
```

The server should start and show initialization messages. Press `Ctrl+C` to stop.

## Alternative: HTTP Server Mode

For advanced use cases, you can run Versionator as an HTTP server:

```bash
# Start HTTP server (default port 8083)
FASTMCP_PORT=8083 python -m versionator_mcp.main

# Custom port
FASTMCP_PORT=9000 python -m versionator_mcp.main
```

Then configure Claude Desktop with:

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

**Note**: HTTP mode requires manually starting the server before using Claude Desktop.

## Development

### Setup

```bash
# Clone the repository
git clone https://github.com/trianglegrrl/versionator-mcp.git
cd versionator-mcp

# Install in development mode with dev dependencies
pip install -e ".[dev]"
```

### Running Tests

```bash
# Run tests
pytest

# Run tests with coverage
pytest --cov=versionator_mcp

# Run linting
black --check .
isort --check-only .
mypy versionator_mcp/
```

### Testing the MCP Server

```bash
# Test stdio transport (default)
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | python -m versionator_mcp.main
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes and add tests
4. Run the test suite (`pytest`)
5. Commit your changes (`git commit -m 'Add amazing feature'`)
6. Push to the branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

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

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Changelog

### v1.0.0
- Initial release
- Support for npm, RubyGems, PyPI, and Hex.pm
- Optimized for Claude Desktop integration
- uvx and pipx support
- Comprehensive error handling
- MCP protocol compliance