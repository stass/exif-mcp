# exif-mcp
[![Trust Score](https://archestra.ai/mcp-catalog/api/badge/quality/stass/exif-mcp)](https://archestra.ai/mcp-catalog/stass__exif-mcp)

An MCP server that allows LLMs (or humans) to read image metadata on-demand, entirely offline. Based on the excellent exifr library it's exremely fast and does not rely on any external tools.

Usecases:
* Analyze image metadata and visualize it
* Perform analysis of your image library: what are my most used cameras?  Lens distribution?  Which dates of the week I take most pictures on?  Most favorite locations?
* Debugging image manipulation code.

Ths tool is used extensively by the reverse geolocation service [PlaceSpotter](https://www.placespotter.com/) for development and testing.

## Overview

`exif-mcp` is a Model Context Protocol (MCP) server that provides tools for extracting various metadata segments from images. Built with TypeScript, it leverages the excellent [exifr](https://github.com/MikeKovarik/exifr) library to parse metadata from images in common formats like JPEG, PNG, TIFF, and HEIC.  This allows this service to parse image metadata without executing any external tools which allows it to be both highly efficient and secure.

### Features

- **Local operation**: Works completely offline with no remote network required
- **Multiple segments**: Extracts EXIF, GPS, XMP, ICC, IPTC, JFIF, and IHDR metadata
- **Various input formats**: Supports JPEG, TIFF, HEIC/AVIF, and PNG
- **Flexible image sources**: Read from file system, URLs, base64 data, or buffers
- **Specialized tools**: Get orientation, rotation info, GPS coordinates, and thumbnails

## Installation

```csh
# Clone the repository
git clone https://github.com/stass/exif-mcp.git
cd exif-mcp

# Install dependencies
npm install

# Build the project
npm run build
```

## Usage
### Claude Desktop

Put this into Claude config file (claude_desktop_config.json):
```json
"mcpServers": {
    "exif-mcp": {
      "command": "node",
      "args": [
        "/path/to/exif-mcp/dist/server.js"
      ]
    }
  },
```

Restart Claude.  Now you can ask Claude to inspect images for you or e.g. find files taken with specific camera.  This works best in combination with filesystem MCP tools so Claude can find files and list directories.

### Starting the server

```csh
# Start the server
npm start

# For development with auto-reload
npm run dev
```

The server uses the `StdioServerTransport` from the MCP SDK, making it compatible with any MCP client that supports STDIO transport.

You can use mcp-proxy to enable remote access.

### Available Tools

The following tools are provided by the server:

| Tool name | Description |
|-----------|-------------|
| `read-metadata` | Reads all or specified metadata segments |
| `read-exif` | Reads EXIF data specifically |
| `read-xmp` | Reads XMP data |
| `read-icc` | Reads ICC color profile data |
| `read-iptc` | Reads IPTC metadata |
| `read-jfif` | Reads JFIF segment data |
| `read-ihdr` | Reads IHDR segment data |
| `orientation` | Gets image orientation (1-8) |
| `rotation-info` | Gets rotation and flip information |
| `gps-coordinates` | Extracts GPS coordinates |
| `thumbnail` | Extracts embedded thumbnail |

### Debugging with MCP Inspector

1. Start the inspector: `npx @modelcontextprotocol/inspector node dist/server.js`
2. Connect to it with MCP Inspector using the STDIO transport
3. Call a tool, e.g., `read-metadata` with parameter:
   ```json
   {
     "image": {
       "kind": "path",
       "path": "/path/to/image.jpg"
     }
   }
   ```
4. You cal also use MCP inspector command line like this: `npx @modelcontextprotocol/inspector --cli node dist/server.js --method tools/call --tool-name read-exif --tool-arg image='{"kind": "path", "path": "/path/to/image.jpeg"}' --tool-arg pick="[]"`

### Image Source Types

The server supports multiple ways to provide image data:

```typescript
// From local file system
{
  "kind": "path",
  "path": "/path/to/image.jpg"
}

// From URL (http, https, or file://)
{
  "kind": "url",
  "url": "https://example.com/image.jpg"
}

// From base64 data (raw or data URI)
{
  "kind": "base64",
  "data": "data:image/jpeg;base64,/9j/4AAQSkZ..."
}

// From base64 buffer
{
  "kind": "buffer",
  "buffer": "/9j/4AAQSkZ..."
}
```

## Development

### Running Tests

```bash
# Run tests
npm test

# Run tests with watch mode
npm run test:watch
```

### Project Structure

```
exif-mcp/
├── src/
│   ├── server.ts         # Main entry point
│   ├── tools/
│   │   ├── index.ts      # Tool registration
│   │   ├── loaders.ts    # Image loading utilities
│   │   └── segments.ts   # exifr options builders
│   └── types/
│       └── image.ts      # Type definitions
├── tests/                # Test files
└── README.md
```

## Error Handling

The server provides standardized error handling for common issues:

- Unsupported formats or missing metadata
- Network fetch failures
- Oversized payloads
- Internal exifr errors

## License

BSD 2-clause

## Acknowledgements

- [exifr](https://github.com/MikeKovarik/exifr) - Extremely fast and robust EXIF parsing library
