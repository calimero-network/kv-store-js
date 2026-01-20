# KV Store - Starter Template & Reference Implementation

A complete starter template for building Calimero applications with a TypeScript/JavaScript logic and React frontend. This repository demonstrates how to build a key-value store application on the Calimero network.
The architecture is designed to be easily extensible—modify the  logic in `logic/src/index.ts` and update the frontend in `app/src/` to match your needs.

## Repository Structure

This repository contains two independent projects that work together:

```
kv-store-js/
├── logic/                    # TypeScript smart  (compiled to WASM)
│   ├── src/
│   │   └── index.ts         #  implementation
│   ├── res/                 # Build output (WASM files, gitignored)
│   ├── package.json         #  dependencies
│   └── build.sh             # Build script
│
├── app/                      # React frontend (Vite)
│   ├── src/
│   │   ├── App.tsx          # Main app with CalimeroProvider
│   │   ├── api/             # Generated TypeScript client (from ABI)
│   │   ├── pages/           # React pages (home, login)
│   │   └── features/        # Feature modules
│   └── package.json         # Frontend dependencies
│
├── scripts/                  # Development automation scripts
│   ├── on-res-change.mjs    # File watcher handler
│   ├── sync-wasm.sh         # Sync WASM to Merobox nodes
│   └── registry-sync.sh     # Sync WASM to local registry
│
├── workflows/                # Merobox workflow definitions
│   └── workflow-example.yml  # Example local network setup
│
└── package.json             # Root scripts and workspace config
```

### Architecture Overview

1. **Logic Layer** (`logic/`): TypeScript smart  using `@calimero-network/calimero-sdk-js`
   - Written in TypeScript with decorators (`@State`, `@Logic`, `@View`, `@Event`)
   - Compiled to WebAssembly (WASM) using the Calimero SDK
   - Outputs: `logic/res/service.wasm`

2. **Frontend Layer** (`app/`): React application using `@calimero-network/calimero-client`
   - Uses generated TypeScript client from ABI
   - Provides UI for interacting with the 
   - Handles authentication and connection to Calimero nodes

3. **Development Tools**: Automated watchers and sync scripts for seamless development

## Prerequisites

Before you begin, ensure you have:

- **Node.js 18+** with WASI support
- **pnpm** (or npm) - Package manager
- **Access to a Calimero node** - Either:
  - Local network via Merobox (recommended for development)
  - Remote Calimero node URL


## Quick Start

### 1. Install Dependencies

```bash
# Install root dependencies
pnpm install

# Install app dependencies
pnpm run app:install
```

### 2. Build the Smart 

```bash
pnpm run logic:build
```

This compiles the TypeScript  to `logic/res/service.wasm`.

### 3. Start Development Environment

**Option A: With Local Network (Recommended)**

```bash
# Terminal 1: Start local Calimero network
pnpm run network:bootstrap

# Terminal 2: Start dev server with file watchers
pnpm run app:dev
```

**Option B: Connect to Existing Node**

```bash
# Just start the dev server
pnpm run app:dev
```

Then open `http://localhost:5173` in your browser and connect to your Calimero node.

## Development Workflow

### Typical Development Loop

1. **Start the development environment:**
   ```bash
   pnpm run app:dev
   ```
   This starts:
   - Vite dev server (frontend) on `http://localhost:5173`
   - File watcher for `logic/res/` directory

2. **Edit your :**
   - Modify `logic/src/index.ts`
   - Rebuild: `pnpm run logic:build`
   - The watcher automatically detects changes and syncs WASM files

3. **Update the frontend:**
   - Edit files in `app/src/`
   - Changes hot-reload automatically

### Available Scripts

**Root-level scripts** (run from repository root):

| Script | Description |
|--------|-------------|
| `pnpm run logic:build` | Build the TypeScript  to WASM |
| `pnpm run logic:clean` | Clean build artifacts |
| `pnpm run logic:watch` | Watch `logic/res/` for changes |
| `pnpm run logic:sync` | Sync WASM to Merobox nodes |
| `pnpm run app:dev` | Start dev server + file watcher |
| `pnpm run app:build` | Build production frontend |
| `pnpm run app:generate-client` | Generate TypeScript client from ABI |
| `pnpm run network:bootstrap` | Start local network via Merobox |

**Logic scripts** (run from `logic/` directory):

| Script | Description |
|--------|-------------|
| `pnpm build` | Build  (called by `logic:build`) |
| `pnpm clean` | Clean build output |

**App scripts** (run from `app/` directory):

| Script | Description |
|--------|-------------|
| `pnpm dev` | Start Vite dev server |
| `pnpm build` | Build for production |
| `pnpm preview` | Preview production build |

### File Watcher Behavior

The `logic:watch` script monitors `logic/res/` for changes:

- **On `*.wasm` change**: Automatically syncs to Merobox nodes (if `SYNC_MEROBOX !== 'false'`)
- **On `abi.json` change**: Logs the change (ABI codegen can be triggered manually if needed)

> **Note**: The ABI is typically generated during the build process. If you need to regenerate the TypeScript client, run `pnpm run app:generate-client`.

##  API

The KV Store  provides the following operations:

### Mutations (State Changes)

- `set(key: string, value: string): void` - Set a key-value pair
  - Emits `Inserted` event if key is new
  - Emits `Updated` event if key exists
- `remove(key: string): string` - Remove a key, returns the removed value
  - Emits `Removed` event
- `clear(): void` - Clear all entries
  - Emits `Cleared` event

### Views (Read-Only)

- `get(key: string): string | null` - Get value by key (returns null if not found)
- `get_unchecked(key: string): string` - Get value by key (throws if not found)
- `get_result(key: string): string | {error: {kind: "NotFound", data: string}}` - Get value with error result
- `entries(): Record<string, string>` - Get all key-value pairs as JSON
- `len(): number` - Get the number of entries

### Events

The  emits the following events:

- `Inserted(key: string, value: string)` - New key-value pair added
- `Updated(key: string, value: string)` - Existing key-value pair updated
- `Removed(key: string)` - Key-value pair removed
- `Cleared()` - All entries cleared

## Configuration

### Frontend Configuration

The app connects to Calimero in `app/src/App.tsx`:

```tsx
<CalimeroProvider
  packageName="com.calimero.kvstore"
  registryUrl="https://apps.calimero.network"
  mode={AppMode.SingleContext}
>
  {/* Your app */}
</CalimeroProvider>
```

### Local Network Setup

The `workflows/workflow-example.yml` file defines the Merobox workflow for local development. It:

1. Creates 2 local Calimero nodes
2. Installs the  on node 1
3. Creates a context
4. Sets up node 2 as a member
5. Demonstrates  calls and state synchronization

## Troubleshooting

### Build Issues

**"WASM file not found"**
```bash
# Ensure you've built the  first
pnpm run logic:build
# Check that logic/res/service.wasm exists
```

**"Missing dependencies"**
```bash
# Install root dependencies
pnpm install
# Install app dependencies
pnpm run app:install
```

### Development Issues

**"Watcher not working"**
```bash
# Ensure dev dependencies are installed
pnpm add -D concurrently chokidar-cli
# Restart the dev server
pnpm run app:dev
```

**"ABI codegen fails"**
```bash
# Ensure ABI file exists (generated during build)
# Manually regenerate client if needed
pnpm run app:generate-client
```

**"Cannot connect to node"**
- Verify your Calimero node is running
- Check the node URL in `app/src/App.tsx` or connection settings
- For local development, ensure Merobox network is running: `pnpm run network:bootstrap`

### Network Issues

**"Merobox network fails to start"**
- Ensure Docker is running (Merobox uses Docker)
- Check that ports are not in use
- Review `workflows/workflow-example.yml` for configuration

## Additional Resources

- [Calimero Documentation](https://docs.calimero.network/)
- [Calimero Network](https://calimero.network)
- [Calimero GitHub](https://github.com/calimero-network)

## Contributing

This is a starter template. Feel free to:

- Fork and customize for your own projects
- Submit improvements via pull requests
- Report issues or suggest enhancements
---