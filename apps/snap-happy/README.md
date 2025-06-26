# Snap Happy

https://github.com/user-attachments/assets/f2a96139-70ae-44db-b924-ac7b043a9f00

A Model Context Protocol (MCP) server that provides screenshot functionality for AI assistants. Take screenshots and retrieve recent captures programmatically across macOS, Linux, and Windows.

## Features

- **Cross-platform screenshot capture**: Works on macOS, Linux, and Windows
- **Three main tools**:
   - `GetLastScreenshot()`: Returns the most recent screenshot as base64 PNG
   - `TakeScreenshot()`: Takes a new screenshot and returns it as base64 PNG
      - **macOS**: Full screen or specific window capture (captures actual window content, ignoring overlapping windows)
      - **Linux**: Full screen or specific window capture using ImageMagick
      - **Windows**: Full screen capture only
   - `ListWindows()`: Lists all visible windows with IDs, titles, and application names
      - **macOS**: Uses Core Graphics APIs for precise window enumeration
      - **Linux**: Uses wmctrl and xprop for X11 window management
- **Window-specific screenshots**: Capture individual windows without interference from overlapping content (macOS and Linux)

## Installation

```bash
npm install -g @mariozechner/snap-happy
```

## Prerequisites

- **macOS**: Built-in `screencapture` command and Swift compiler (Screen Recording permission required for window capture)
- **Linux**: `gnome-screenshot` or `scrot`, `wmctrl`, `xdotool`, and `imagemagick` packages
- **Windows**: PowerShell with .NET Framework

### Build Requirements

The package includes pre-built universal binaries for macOS (Intel + Apple Silicon). End users don't need any additional tools.

**For development/publishing:**

- **macOS**: Swift compiler (Xcode or Swift toolchain) to build universal binaries
- **Linux/Windows**: No additional build requirements

## Configuration

Optionally set an environment variable for your screenshot directory:

```bash
export SNAP_HAPPY_SCREENSHOT_PATH="/path/to/screenshots"
```

The directory will be created automatically if it doesn't exist.

## Usage

### With Claude Code

```bash
# Add the MCP server
claude mcp add snap-happy npx @mariozechner/snap-happy

# Use in Claude
echo "Take a screenshot" | claude -p
echo "Show me the last screenshot" | claude -p
```

### As MCP Server

Add to your MCP client configuration:

```json
{
	"mcpServers": {
		"snap-happy": {
			"command": "npx",
			"args": ["@mariozechner/snap-happy"],
			"env": {
				"SNAP_HAPPY_SCREENSHOT_PATH": "/Users/username/Screenshots"
			}
		}
	}
}
```

## Development

```bash
# Start building in watch mode
npm run dev

# Build everything (TypeScript + universal binaries for distribution)
npm run build

# Build for development (current architecture only, faster)
npm run build:dev

# Build only native utilities (universal binaries)
npm run build:native

# Build only native utilities (development, current architecture)
npm run build:native:dev

# Add to Claude for testing (after building)
claude mcp add snap-happy node /path/to/git/clone/of/snap-happy/dist/index.js

# Test using Claude Code
echo "Take a screenshot" | claude -p

# Test with JSON-RPC directly
echo '{"jsonrpc": "2.0", "id": 1, "method": "tools/call", "params": {"name": "ListWindows", "arguments": {}}}' | node dist/index.js
echo '{"jsonrpc": "2.0", "id": 1, "method": "tools/call", "params": {"name": "TakeScreenshot", "arguments": {"windowId": 2}}}' | node dist/index.js

# Run all tests
npm test
```

## Troubleshooting

### macOS Permissions

On first use, macOS will prompt for permissions:

- **Screen Recording**: Required for taking screenshots and window capture
- Window listing uses Core Graphics APIs and doesn't require additional permissions. Might trigger some macOS security bullshit tho. Works on my machine.

Grant permissions in System Preferences → Security & Privacy → Privacy → Screen Recording.

### Linux Dependencies

```bash
# Ubuntu/Debian
sudo apt install gnome-screenshot wmctrl xdotool imagemagick

# Fedora/RHEL
sudo dnf install gnome-screenshot wmctrl xdotool ImageMagick
```

### Common Issues

- **"Environment variable not set"**: Set `SNAP_HAPPY_SCREENSHOT_PATH`
- **"Screenshot path is not writable"**: Check directory permissions
- **"No screenshots found"**: Verify directory contains PNG files
- **"Window-specific screenshots are only supported on macOS and Linux"**: Window capture with `windowId` parameter works on macOS and Linux only
- **Native utility build errors**: Only relevant for development - end users get pre-built binaries. For development, ensure Swift compiler is available (`xcode-select --install`)
- **"Failed to list windows on Linux"**: Ensure `wmctrl`, `xdotool`, and `imagemagick` are installed on your system

## License

MIT License - see LICENSE file for details.
