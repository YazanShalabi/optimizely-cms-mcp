# Optimizely MCP Server

A Model Context Protocol (MCP) server for Optimizely CMS, providing AI assistants with comprehensive access to Optimizely's GraphQL API and Content Management API.

## Version

**Current Version**: 2.0.0-beta
**Status**: Beta / Pre-Release

This is an active development version and is **not yet a release candidate**. Features are subject to change, and additional testing is required before production use.

## Features

### Core Capabilities
- **Discovery-First Architecture**: Zero hardcoded assumptions about content types or fields
- **Dynamic Schema Introspection**: Discovers available content types and fields at runtime
- **Unified Content Retrieval**: Get any content by URL, key, GUID, or search term in one call
- **Visual Builder Support**: Full support for Optimizely Visual Builder pages with composition structure
- **Content Management**: Create and manage content via interactive wizard
- **Intelligent Field Mapping**: Pattern-based field matching with confidence scoring
- **GraphQL & CMA Integration**: Direct access to both Graph API (read) and Content Management API (write)
- **Smart Caching**: Built-in caching for improved performance
- **Type Safety**: Full TypeScript support with runtime validation

### API Support
- **Graph API**: Fast content retrieval, search, and discovery
- **Content Management API**: Content creation, updates, and draft access
- **Dual Authentication**: Supports both Graph (single key, HMAC) and CMA (OAuth2) authentication

## Installation

```bash
# Clone the repository
git clone https://github.com/your-org/optimizely-mcp-server.git
cd optimizely-mcp-server

# Install dependencies
npm install

# Build the project
npm run build
```

## Configuration

Create a `.env` file in the project root:

```env
# Server Configuration
SERVER_NAME=optimizely-mcp-server
SERVER_VERSION=1.0.0
TRANSPORT=stdio

# Optimizely Graph Configuration
GRAPH_ENDPOINT=https://cg.optimizely.com/content/v2
GRAPH_AUTH_METHOD=single_key # Options: single_key, hmac, basic, bearer, oidc
GRAPH_SINGLE_KEY=your-single-key
# For HMAC auth:
# GRAPH_APP_KEY=your-app-key
# GRAPH_SECRET_KEY=your-secret-key

# Content Management API Configuration
CMA_BASE_URL=https://api.cms.optimizely.com/preview3
CMA_CLIENT_ID=your-client-id  # Get from Settings > API Keys in CMS
CMA_CLIENT_SECRET=your-client-secret
CMA_GRANT_TYPE=client_credentials
CMA_TOKEN_ENDPOINT=https://api.cms.optimizely.com/oauth/token
CMA_IMPERSONATE_USER=  # Optional: User email to impersonate (see Impersonation section)

# Optional Configuration
CACHE_TTL=300000 # Cache TTL in milliseconds (default: 5 minutes)
LOG_LEVEL=info # Options: debug, info, warn, error
MAX_RETRIES=3
TIMEOUT=30000
```

## Running the Server

### Development Mode

```bash
# Run with hot reloading
npm run dev

# Run with debug logging
LOG_LEVEL=debug npm run dev
```

### Production Mode

```bash
# Build and run
npm run build
npm start

# Or run directly
node dist/index.js
```

### Testing the Server

```bash
# Run all unit tests
npm test

# Run tests with coverage
npm run test:coverage

# Type checking
npm run typecheck

# Linting
npm run lint
```

## How MCP Servers Work

MCP servers communicate via **stdio** (standard input/output), not HTTP ports:

- **No port required** - The server doesn't listen on any network port
- **Process-based** - Claude Desktop spawns your server as a child process
- **JSON-RPC messages** - Communication happens through stdin/stdout pipes
- **Secure** - No network exposure, runs only when Claude needs it

## MCP Client Configuration

### Claude Desktop Setup

#### Step 1: Find your config file

Open the configuration file in a text editor:
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Linux**: `~/.config/Claude/claude_desktop_config.json`

For Windows, you can open it quickly with:
```cmd
notepad %APPDATA%\Claude\claude_desktop_config.json
```

#### Step 2: Add the server configuration

```json
{
  "mcpServers": {
    "optimizely": {
      "command": "node",
      "args": ["%USERPROFILE%\\path\\to\\optimizely-mcp-server\\dist\\index.js"],
      "env": {
        "LOG_LEVEL": "error",
        "GRAPH_ENDPOINT": "https://cg.optimizely.com/content/v2",
        "GRAPH_AUTH_METHOD": "single_key",
        "GRAPH_SINGLE_KEY": "your-key",
        "CMA_BASE_URL": "https://api.cms.optimizely.com/preview3/experimental",
        "CMA_CLIENT_ID": "your-client-id",
        "CMA_CLIENT_SECRET": "your-client-secret",
        "CMA_GRANT_TYPE": "client_credentials",
        "CMA_TOKEN_ENDPOINT": "https://api.cms.optimizely.com/oauth/token",
        "CMA_IMPERSONATE_USER": ""
      }
    }
  }
}
```
In JSON on Windows, you must use double backslashes (\\). If your folder path has spaces, this still works because each argument is a separate JSON string.

- Windows: %USERPROFILE% expands to your home directory (e.g., C:\Users\Alice). If Claude doesn’t expand it automatically, replace it with your actual path (e.g., C:\\Users\\Alice\\path\\to\\optimizely-mcp-server\\dist\\index.js). In PowerShell, the equivalent is $env:USERPROFILE, but inside this JSON config you should keep %USERPROFILE% or use the full path.
- macOS/Linux: the equivalent shortcut is ~ or $HOME (e.g., /Users/alice or /home/alice). If ~/$HOME isn’t expanded correctly, replace it with the full path.

#### Step 3: Restart Claude Desktop

After saving the config file:
1. Completely quit Claude Desktop (not just close the window)
2. Start Claude Desktop again
3. The Optimizely tools should now be available

#### Step 4: Verify it's working

In a new Claude conversation, try:
- "Can you list the available Optimizely tools?"
- "Use the health-check tool to test the connection"

### Troubleshooting

If the server doesn't load:
1. Check the file path is correct and uses proper escaping (`\\` for Windows)
2. Ensure you've built the project (`npm run build`)
3. Verify the `dist/index.js` file exists
4. Check Claude's logs for errors

### Other MCP Clients

For other MCP-compatible clients, use the stdio transport configuration:

```json
{
  "name": "optimizely",
  "transport": {
    "type": "stdio",
    "command": "node",
    "args": ["/path/to/optimizely-mcp-server/dist/index.js"]
  },
  "env": {
    // Environment variables as above
  }
}
```

## Available Tools (14 Total)

### 🌟 Core Discovery & Retrieval Tools

These tools use the Graph API to dynamically discover your CMS structure and retrieve content without hardcoded assumptions.

1. **`help`** - 🚀 START HERE! Get context-aware help and learn the discovery-first workflow
   - Examples: `help({})`, `help({"topic": "workflow"})`

2. **`get`** - 🎯 UNIFIED TOOL - Get content by ANY identifier in ONE call
   - Replaces the old `search` → `locate` → `retrieve` workflow
   - Auto-discovers fields and returns complete content
   - **✅ Supports Visual Builder pages with full composition structure**
   - Examples: `get({"identifier": "/"})`, `get({"identifier": "Article 4"})`

3. **`discover`** - Find content types and fields dynamically
   - No hardcoded assumptions about your CMS structure
   - Examples: `discover({"target": "types"})`, `discover({"target": "fields", "contentType": "ArticlePage"})`

4. **`analyze`** - Deep analysis of content type requirements
   - Understand fields, constraints, and defaults
   - Example: `analyze({"contentType": "ArticlePage"})`

5. **`search`** - Intelligent content search with auto-discovery
   - ⚠️ Note: `get` is usually better for most use cases
   - Example: `search({"query": "mcp", "contentTypes": ["ArticlePage"]})`

6. **`locate`** - Find specific content by ID, key, or path
   - ⚠️ Note: `get` is usually better for most use cases
   - Example: `locate({"identifier": "/news/article-1"})`

7. **`retrieve`** - Get full content from Content Management API
   - ⚠️ Note: `get` is usually better (uses faster Graph API)
   - Use only when `get` suggests it or you need CMA-specific data
   - Example: `retrieve({"identifier": "12345"})`

### 🔧 Utility Tools (3)
- **`health-check`** - Check API connectivity and server health
- **`get-config`** - Get current server configuration (sanitized)
- **`get-documentation`** - Get documentation for available tools by category

### 🔧 Content Management Tools (CMA API)

These tools use the Content Management API for write operations and detailed content access:

1. **`content_creation_wizard`** - Interactive content creation with discovery
   - Essential for creating new content
   - Example: `content_creation_wizard({"step": "start"})`

2. **`content-test-api`** - Test CMA connectivity and endpoints
   - Validates authentication and permissions
   - Example: `content-test-api({})`

Note: The `retrieve` tool (listed in Core Tools above) also uses CMA for accessing draft content and version history.

### ⚠️ Deprecated Tools (Being Removed)

These Graph API discovery tools are duplicates of the new `discover` tool and will be removed in a future version:

- `graph-introspection` - Use `discover` instead
- `type-discover` - Use `discover({"target": "types"})` instead
- `type-match` - Use `discover` instead
- `content_type_analyzer` - Use `analyze` instead
- `graph_discover_types` - Use `discover({"target": "types"})` instead
- `graph_discover_fields` - Use `discover({"target": "fields"})` instead
- `graph-query` - Use `get` or `search` instead

## Key Architecture Principles

### Discovery-First Design
Unlike traditional integrations that hardcode content types and field names, this MCP server:
- **Never hardcodes content types** - No assumptions about "ArticlePage", "StandardPage", etc.
- **Never hardcodes field mappings** - No predefined paths like "SeoSettings.MetaTitle"
- **Discovers everything dynamically** - Uses introspection to understand your CMS
- **Adapts to any CMS configuration** - Works with custom content types and fields

### Intelligent Field Mapping
The server uses pattern matching and similarity scoring to:
- Map user-friendly field names to actual CMS fields
- Handle nested properties automatically
- Generate appropriate default values based on field types
- Provide confidence scores for mappings

### Recommended Workflows

#### Simple Content Retrieval (Most Common)
```
1. get({"identifier": "homepage"})  # That's it! One call gets everything.
```

The `get` tool automatically:
- Detects identifier type (search term, URL, key, or GUID)
- Finds the content
- Discovers all available fields
- Returns complete content including Visual Builder composition

#### Advanced Discovery Workflow
```
1. help({})                                              # Learn the workflow
2. discover({"target": "types"})                         # Find content types
3. discover({"target": "fields", "contentType": "..."}) # Get fields
4. get({"identifier": "..."})                           # Retrieve content
```

#### Content Creation Workflow
```
1. discover({"target": "types"})              # Find available types
2. analyze({"contentType": "ArticlePage"})    # Understand requirements
3. content_creation_wizard({...})             # Create with guidance
```

## Visual Builder Support

The `get` tool fully supports Optimizely Visual Builder (formerly known as Visual Experience Composer) pages:

### Features
- ✅ **Automatic Detection** - Recognizes Visual Builder pages by interface (`_IExperience`)
- ✅ **Complete Composition Retrieval** - Returns full structure in a single call
- ✅ **Nested Structure** - Handles grids, rows, columns, and components
- ✅ **Component Content** - Includes inline component data directly in composition
- ✅ **Recursive Depth** - Supports any level of nesting

### Understanding Component Types

Visual Builder components come in two types:

#### 1. Inline Components (Embedded Content)
- **Key**: `null` or not present
- **Content Location**: Stored directly in the composition structure
- **Access**: Content is already included in the `get` response
- **Example**: Text components with content like "Welcome to our site"

```json
{
  "component": {
    "_metadata": {
      "types": ["Text", "_Component"],
      "key": null  // ← NULL = inline
    },
    "Content": "Welcome Text"  // ← Content is here
  }
}
```

**Important**: Do NOT try to retrieve inline components separately - the content is already provided!

#### 2. Referenced Components (Separate Content Items)
- **Key**: Valid GUID (e.g., "f7e7f5c9-1e77-4884-a8fc-a9c9ae56560c")
- **Content Location**: Stored as separate content items in CMS
- **Access**: Use `get({"identifier": "component-key"})` to retrieve full details
- **Example**: Shared components like Site Settings, reusable blocks

```json
{
  "component": {
    "_metadata": {
      "types": ["ArticleList", "_Component"],
      "key": "f7e7f5c91e774884a8fca9c9ae56560c"  // ← Has key
    }
    // May include basic fields, use get() for full content
  }
}
```

### Best Practices

**When working with Visual Builder pages:**

1. **First**, retrieve the page with `get({"identifier": "/"})`
2. **Inspect** the composition structure for components
3. **For inline components** (null key): Content is already in the response ✅
4. **For referenced components** (has key): Use `get({"identifier": "key"})` to fetch full details

### Example Usage
```javascript
// Get a Visual Builder homepage
get({"identifier": "/"})

// Returns complete structure with inline content:
{
  "content": {
    "_metadata": { ... },
    "composition": {
      "nodes": [
        {
          "key": "grid-id",
          "displayName": "Welcome Section",
          "nodes": [
            {
              "component": {
                "_metadata": {
                  "types": ["Text"],
                  "key": null  // Inline - content included
                },
                "Content": "Welcome to our site"
              }
            },
            {
              "component": {
                "_metadata": {
                  "types": ["ArticleList"],
                  "key": "f7e7f5c9..."  // Referenced - fetch separately
                }
              }
            }
          ]
        }
      ]
    }
  }
}
```

### Known Limitations
- **Performance** - Large compositions may take longer to retrieve due to nested structure
- **Referenced Component Details** - Basic metadata only; full content requires separate `get()` call
- **Display Settings** - Not included in current implementation (can be added if needed)

## Important Notes

### Content Indexing Delay
After creating new content using the `content_creation_wizard` or other creation tools:
- **Immediate availability in CMA**: Content is immediately available via `retrieve` tool
- **Graph API indexing delay**: Content may take 1-5 minutes to appear in Graph API results
- **Tool behavior**: The `get` and `search` tools use Graph API and will return "not found" for newly created content until indexing completes

**Best practice**: After creating content, wait a few minutes before attempting to retrieve it with `get` or `search`. Alternatively, use the `retrieve` tool which queries the CMA directly and has no indexing delay.

### Draft vs Published Content
- **Graph API**: Only returns published content
- **CMA API**: Returns both draft and published content
- **New content**: Created in draft status by default
- To make content searchable via `get`/`search`, it must be published first

## Development

### Project Structure

```
optimizely-mcp-server/
├── src/
│   ├── index.ts          # Server entry point
│   ├── register.ts       # Tool registration
│   ├── config.ts         # Configuration management
│   ├── clients/          # API clients
│   │   ├── graph-client.ts
│   │   └── cma-client.ts
│   ├── logic/            # Tool implementations
│   │   ├── utility/
│   │   ├── graph/
│   │   └── content/
│   ├── types/            # TypeScript types
│   └── utils/            # Utilities
├── tests/                # Test files
├── dist/                 # Built output
└── package.json
```

### Adding New Tools

1. Create tool implementation in `src/logic/`
2. Add tool registration in appropriate section
3. Add TypeScript types if needed
4. Write tests in `tests/`
5. Update documentation

### Testing Guidelines

- Unit tests for all tool implementations
- Integration tests for API clients
- Mock external API calls
- Test error scenarios
- Maintain >80% coverage

## Testing & Debugging

### Unit Tests

Run automated tests with Vitest:

```bash
# Run unit tests
npm test

# Run with coverage report
npm run test:coverage
```

Unit tests are located in `/tests/` and cover:
- GraphQL client functionality
- CMA client operations
- Health check features

### Integration Testing & Debugging

Test your setup with these npm scripts:

```bash
# Check credentials are valid
npm run check:credentials

# Test MCP tools
npm run test:tools

# Test with debug output
npm run test:tools:debug

# Test GraphQL connection
npm run debug:graph

# Validate API key format
npm run validate:key
```

## Troubleshooting

### Common Issues

1. **Authentication Errors**
   - Verify your API credentials in `.env`
   - For CMA: Create API keys in Settings > API Keys in your Optimizely CMS instance
   - Check token expiration for CMA (tokens expire after 5 minutes)
   - Ensure correct auth method for Graph

2. **Connection Issues**
   - Verify network connectivity
   - Check firewall settings
   - Confirm API endpoints are accessible

3. **Build Errors**
   - Run `npm install` to ensure dependencies
   - Check Node.js version (>=18 required)
   - Clear `dist/` and rebuild

4. **403 Forbidden Errors (Content Creation)**
   - This typically means insufficient permissions
   - See the Impersonation section below for a solution
   - Verify the user has content creation rights
   - Check the target container allows the content type

### Debug Mode

Enable debug logging for troubleshooting:

```bash
LOG_LEVEL=debug npm start
```

### Health Check

Test server connectivity:

```bash
# Using the built tool
echo '{"method": "tools/call", "params": {"name": "health_check"}}' | node dist/index.js
```

## User Impersonation

If you encounter 403 Forbidden errors when creating content, you can use **user impersonation** to execute API calls as a specific user who has the necessary permissions.

### When to Use Impersonation

Use impersonation when:
- The API client lacks content creation permissions
- You need to test with different user permission levels
- You want actions attributed to a specific user

### Setup Instructions

1. **Enable Impersonation in Optimizely CMS**:
   - Log into Optimizely CMS as an administrator
   - Navigate to **Settings > API Clients**
   - Find your API client
   - Enable the **"Allow impersonation"** option
   - Save the changes

2. **Configure the MCP Server**:
   ```env
   # In your .env file
   CMA_IMPERSONATE_USER=user@example.com
   ```

3. **Update Claude Desktop Config** (if using environment variables):
   ```json
   {
     "mcpServers": {
       "optimizely": {
         "env": {
           "CMA_IMPERSONATE_USER": "user@example.com",
           // ... other settings
         }
       }
     }
   }
   ```

### How It Works

When impersonation is configured:
- Authentication requests use JSON format with `act_as` field
- All content operations execute as the impersonated user
- Created content shows the impersonated user as the author

### Testing Impersonation

Test that impersonation is working:

```bash
# Run the impersonation test script
node scripts/test-impersonation-final.js
```

This will create test content and show which user created it.

### Security Best Practices

- Only enable impersonation when necessary
- Use accounts with minimal required permissions
- Regularly review API client permissions
- Monitor API usage logs for unusual activity

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Run `npm test` and `npm run typecheck`
6. Submit a pull request

## License

MIT
