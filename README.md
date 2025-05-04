# Onyx MCP Server

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![npm version](https://img.shields.io/npm/v/onyx-mcp-server.svg)](https://www.npmjs.com/package/onyx-mcp-server)
[![npm downloads](https://img.shields.io/npm/dm/onyx-mcp-server.svg)](https://www.npmjs.com/package/onyx-mcp-server)
[![smithery badge](https://smithery.ai/badge/@lupuletic/onyx-mcp-server)](https://smithery.ai/server/@lupuletic/onyx-mcp-server)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/lupuletic/onyx-mcp-server/blob/main/CONTRIBUTING.md)

> A Model Context Protocol (MCP) server for seamless integration with Onyx AI knowledge bases.

This MCP server connects any MCP-compatible client to your Onyx knowledge base, allowing you to search and retrieve relevant context from your documents. It provides a bridge between MCP clients and the Onyx API, enabling powerful semantic search and chat capabilities.

<img width="1166" alt="image" src="https://github.com/user-attachments/assets/6396b010-cb1b-489c-98ec-dbb53e3996d2" />


## Features

- **Enhanced Search**: Semantic search across your Onyx document sets with LLM relevance filtering
- **Context Window Retrieval**: Retrieve chunks above and below the matching chunk for better context
- **Full Document Retrieval**: Option to retrieve entire documents instead of just chunks
- **Chat Integration**: Use Onyx's powerful chat API with LLM + RAG for comprehensive answers
- **Configurable Document Set Filtering**: Target specific document sets for more relevant results
- **Multiple Transport Methods**: Supports both stdio (for CLI) and HTTP transport (for API clients)
- **Environment Configuration**: Easy configuration via .env file

## Installation

### Installing via Smithery

To install Onyx MCP Server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@lupuletic/onyx-mcp-server):

```bash
npx -y @smithery/cli install @lupuletic/onyx-mcp-server --client claude
```

### Prerequisites

- Node.js (v16 or higher)
- An Onyx instance with API access
- An Onyx API token

### Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/lupuletic/onyx-mcp-server.git
   cd onyx-mcp-server
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Build the server:
   ```bash
   npm run build
   ```

4. Configure your environment:
   ```bash
   # Copy the example .env file
   cp .env.example .env
   
   # Edit the .env file with your API token and settings
   nano .env
   ```

5. Start the server:
   ```bash
   npm start
   ```

## Environment Configuration

Onyx MCP Server can be configured using environment variables or a `.env` file. The following variables are available:

| Variable | Description | Default |
|----------|-------------|---------|
| `ONYX_API_URL` | URL of the Onyx API | `http://localhost:8080/api` |
| `ONYX_API_TOKEN` | Your Onyx API token | *(required)* |
| `ONYX_DEFAULT_PERSONA_ID` | Default persona ID to use for chat if not specified by the client | `15` |
| `HTTP_PORT` | Port for the HTTP server to listen on | `3000` |
| `DEBUG` | Enable debug logging (true/false) | `false` |

## Transport Methods

The server supports two transport methods:

1. **stdio** - Used for CLI applications and integration with desktop applications
2. **HTTP** - Used for API clients and web applications

When started, the server will listen on both transports simultaneously, so you can use whichever is most convenient for your application.

### HTTP Endpoint

The HTTP endpoint is available at:

```
http://localhost:3000/mcp
```

See the test client documentation for examples of how to interact with this endpoint.

## Configuring MCP Clients

### For Claude Desktop App

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "onyx-search": {
      "command": "node",
      "args": ["/path/to/onyx-mcp-server/build/index.js"],
      "env": {
        "ONYX_API_TOKEN": "your-api-token-here",
        "ONYX_API_URL": "http://localhost:8080/api"
      },
      "disabled": false,
      "alwaysAllow": []
    }
  }
}
```

### For Claude in VSCode (Cline)

Add to your Cline MCP settings file:

```json
{
  "mcpServers": {
    "onyx-search": {
      "command": "node",
      "args": ["/path/to/onyx-mcp-server/build/index.js"],
      "env": {
        "ONYX_API_TOKEN": "your-api-token-here",
        "ONYX_API_URL": "http://localhost:8080/api"
      },
      "disabled": false, 
      "alwaysAllow": []
    }
  }
}
```

### For HTTP Clients

For HTTP clients, simply point your client to the HTTP endpoint:

```
http://localhost:3000/mcp
```

See the [test client documentation](test-client/README.md) for examples of how to interact with the HTTP API.

## Available Tools

Once configured, your MCP client will have access to two powerful tools:

### 1. Search Tool

The `search_onyx` tool provides direct access to Onyx's search capabilities with enhanced context retrieval:

```
<use_mcp_tool>
<server_name>onyx-search</server_name>
<tool_name>search_onyx</tool_name>
<arguments>
{
  "query": "customer onboarding process",
  "documentSets": ["Company Policies", "Training Materials"],
  "maxResults": 3,
  "chunksAbove": 1,
  "chunksBelow": 1,
  "retrieveFullDocuments": true
}
</arguments>
</use_mcp_tool>
```

Parameters:
- `query` (required): The topic to search for
- `documentSets` (optional): List of document set names to search within (empty for all)
- `maxResults` (optional): Maximum number of results to return (default: 5, max: 10)
- `chunksAbove` (optional): Number of chunks to include above the matching chunk (default: 1)
- `chunksBelow` (optional): Number of chunks to include below the matching chunk (default: 1)
- `retrieveFullDocuments` (optional): Whether to retrieve full documents instead of just chunks (default: false)

### 2. Chat Tool

The `chat_with_onyx` tool leverages Onyx's powerful chat API with LLM + RAG for comprehensive answers:

```
<use_mcp_tool>
<server_name>onyx-search</server_name>
<tool_name>chat_with_onyx</tool_name>
<arguments>
{
  "query": "What is our company's policy on remote work?",
  "personaId": 15,
  "documentSets": ["Company Policies", "HR Documents"],
  "chatSessionId": "optional-existing-session-id"
}
</arguments>
</use_mcp_tool>
```

Parameters:
- `query` (required): The question to ask Onyx
- `personaId` (optional): The ID of the persona to use (default: 15)
- `documentSets` (optional): List of document set names to search within (empty for all)
- `chatSessionId` (optional): Existing chat session ID to continue a conversation

#### Chat Sessions

The chat tool supports maintaining conversation context across multiple interactions. After the first call, the response will include a `chat_session_id` in the metadata. You can pass this ID in subsequent calls to maintain context.

## Choosing Between Search and Chat

- **Use Search When**: You need specific, targeted information from documents and want to control exactly how much context is retrieved.
- **Use Chat When**: You need comprehensive answers that combine information from multiple sources, or when you want the LLM to synthesize information for you.

For the best results, you can use both tools in combination - search for specific details and chat for comprehensive understanding.

## Use Cases

- **Knowledge Management**: Access your organization's knowledge base through any MCP-compatible interface
- **Customer Support**: Help support agents quickly find relevant information
- **Research**: Conduct deep research across your organization's documents
- **Training**: Provide access to training materials and documentation
- **Policy Compliance**: Ensure teams have access to the latest policies and procedures

## Test Client

A Python test client is included in the `test-client` directory to help you test and debug the server. The client supports both interactive and command-line modes.

### Prerequisites

- Python 3.6+
- `requests` library (`pip install requests`)

### Running the Test Client

```bash
cd test-client
./mcp_client.py
```

For more information, see the [test client documentation](test-client/README.md).

## Development

### Running in Development Mode

```bash
npm run dev
```

### Committing Changes

This project enforces the [Conventional Commits](https://www.conventionalcommits.org/) specification for all commit messages. To make this easier, we provide an interactive commit tool:

```bash
npm run commit
```

This will guide you through creating a properly formatted commit message. Alternatively, you can write your own commit messages following the conventional format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Where `type` is one of: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert

### Building for Production

```bash
npm run build
```

### Testing

Run the test suite:

```bash
npm test
```

Run tests with coverage:

```bash
npm run test:coverage
```

### Linting

```bash
npm run lint
```

Fix linting issues:

```bash
npm run lint:fix
```

## Continuous Integration

This project uses GitHub Actions for continuous integration and deployment. The CI pipeline runs on every push to the main branch and on pull requests. It performs the following checks:

- Linting
- Building
- Testing
- Code coverage reporting

### Automated Version Bumping and Publishing

When a PR is merged to the main branch, the project automatically determines the appropriate version bump type and publishes to npm. The system analyzes both PR titles and commit messages to determine the version bump type.

1. **PR Title Validation**: All PR titles are validated against the [Conventional Commits](https://www.conventionalcommits.org/) specification:
   - PR titles must start with a type (e.g., `feat:`, `fix:`, `docs:`)
   - This validation happens automatically when a PR is created or updated
   - PRs with invalid titles will fail the validation check

2. **Commit Message Validation**: All commit messages are also validated against the conventional commits format:
   - Commit messages must start with a type (e.g., `feat:`, `fix:`, `docs:`)
   - This is enforced by git hooks that run when you commit
   - Commits with invalid messages will be rejected
   - Use `npm run commit` for an interactive commit message creation tool

3. **Version Bump Determination**: The system analyzes both the PR title and commit messages to determine the appropriate version bump:
   - PR titles starting with `feat` or containing new features → minor version bump
   - PR titles starting with `fix` or containing bug fixes → patch version bump
   - PR titles containing `BREAKING CHANGE` or with an exclamation mark → major version bump
   - If the PR title doesn't indicate a specific bump type, the system analyzes commit messages
   - The highest priority bump type found in any commit message is used (major > minor > patch)
   - If no conventional commit prefixes are found, the system automatically defaults to a patch version bump without failing

4. **Version Update and Publishing**:
   - Bumps the version in package.json according to semantic versioning
   - Commits and pushes the version change
   - Publishes the new version to npm

This automated process ensures consistent versioning based on the nature of the changes, following semantic versioning principles, and eliminates manual version management.

## Contributing

Contributions are welcome! Please see our [Contributing Guide](CONTRIBUTING.md) for more details.

## Security

If you discover a security vulnerability, please follow our [Security Policy](SECURITY.md).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.


<a href="https://glama.ai/mcp/servers/@lupuletic/onyx-mcp-server">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/@lupuletic/onyx-mcp-server/badge" />
</a>