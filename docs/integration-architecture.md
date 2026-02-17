# Gas Town - Integration Architecture

**Generated:** 2026-02-17
**Repository Type:** Monolith with NPM Distribution Wrapper

## Overview

Gas Town consists of two parts that integrate to provide cross-platform distribution:

1. **gastown-cli** (Primary) - Go-based CLI application
2. **npm-wrapper** (Secondary) - Node.js distribution wrapper

## Integration Points

### 1. NPM Wrapper → Go Binary

**Type:** Binary Wrapper
**Protocol:** Process Execution
**Direction:** npm-wrapper invokes gastown-cli

```
┌──────────────────────────────────────────────────────────┐
│                     User Environment                      │
└──────────────────────────────────────────────────────────┘
                          │
                    npm install -g @gastown/gt
                          │
                          ▼
┌──────────────────────────────────────────────────────────┐
│                   npm-package (Wrapper)                   │
│                                                            │
│  ┌──────────────────────────────────────────────────┐   │
│  │  scripts/postinstall.js                          │   │
│  │  • Detect platform (darwin/linux/win32)         │   │
│  │  • Detect architecture (x64/arm64)              │   │
│  │  • Download/reference Go binary                 │   │
│  │  • Set up executable permissions                │   │
│  └──────────────────────────────────────────────────┘   │
│                                                            │
│  ┌──────────────────────────────────────────────────┐   │
│  │  bin/gt.js (Wrapper Script)                      │   │
│  │  #!/usr/bin/env node                            │   │
│  │  • Resolve binary path                          │   │
│  │  • Pass through arguments                       │   │
│  │  • Spawn Go binary process                      │   │
│  │  • Pipe stdin/stdout/stderr                     │   │
│  │  • Exit with binary's exit code                 │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
                          │
                    spawn() process
                          │
                          ▼
┌──────────────────────────────────────────────────────────┐
│              gastown-cli (Go Binary)                      │
│                                                            │
│  cmd/gt/main.go                                          │
│  • Process arguments                                     │
│  • Execute commands                                      │
│  • Return exit code                                      │
└──────────────────────────────────────────────────────────┘
```

### Integration Mechanism

**Wrapper Script Structure:**
```javascript
#!/usr/bin/env node

const { spawn } = require('child_process');
const path = require('path');

// Resolve binary path (platform-specific)
const binaryPath = resolveBinaryPath();

// Spawn Go binary with args
const child = spawn(binaryPath, process.argv.slice(2), {
  stdio: 'inherit'
});

// Exit with binary's exit code
child.on('exit', (code) => {
  process.exit(code);
});
```

**Platform Detection:**
```javascript
function resolveBinaryPath() {
  const platform = process.platform;  // darwin, linux, win32
  const arch = process.arch;          // x64, arm64

  // Map to binary location
  return path.join(__dirname, '..', 'bin', getBinaryName(platform, arch));
}
```

## Data Flow

### Installation Flow

```
1. User runs: npm install -g @gastown/gt
   └─> NPM downloads package

2. NPM executes postinstall script
   └─> scripts/postinstall.js runs
       ├─> Detects: platform = darwin, arch = arm64
       ├─> Downloads/references: gt-darwin-arm64
       └─> Sets permissions: chmod +x

3. NPM sets up bin link
   └─> Creates symlink: /usr/local/bin/gt -> gt.js
```

### Execution Flow

```
1. User runs: gt convoy create "My Work"
   └─> Shell resolves to: /usr/local/bin/gt (symlink)

2. Node.js executes: bin/gt.js
   └─> gt.js script runs
       ├─> Resolves binary path
       ├─> Spawns Go binary
       └─> Passes args: ["convoy", "create", "My Work"]

3. Go binary executes
   └─> cmd/gt/main.go
       └─> internal/cmd.Execute()
           └─> Cobra routing
               └─> convoy create command

4. Go binary exits with code
   └─> gt.js receives exit code
       └─> Node process exits with same code
```

## Communication Protocol

### Arguments
- **Passed Through:** All command-line arguments
- **No Transformation:** Arguments passed as-is
- **Format:** Standard command-line args

### Standard I/O
- **stdin:** Piped directly to Go binary
- **stdout:** Piped directly from Go binary
- **stderr:** Piped directly from Go binary

### Exit Codes
- **Transparent:** Wrapper exits with binary's exit code
- **No Interception:** No exit code transformation

### Environment Variables
- **Inherited:** Go binary inherits wrapper's environment
- **No Modification:** Wrapper doesn't set additional env vars

## Shared Dependencies

### None at Runtime

The two parts have **no shared runtime dependencies**:
- npm-wrapper depends on: Node.js runtime
- gastown-cli depends on: Go runtime (compiled in)

### Build-Time Relationship

**Independent Builds:**
- Go binary built separately (make build, GoReleaser)
- NPM package built separately (npm publish)

**Versioning:**
- Maintained separately
- Version numbers should match but managed independently

## Configuration

### No Shared Configuration

Each part maintains its own configuration:

**npm-wrapper:**
- `package.json` - NPM package metadata
- Node.js version requirements (14+)

**gastown-cli:**
- `go.mod` - Go module definition
- Configuration files in `internal/config/`
- User config in `~/.config/gt/`

### Configuration Flow

```
NPM Config (package.json)
    ↓
  [no interaction]
    ↓
Go CLI Config (config files, env vars)
```

## Authentication Flow

**N/A** - No authentication between parts

The wrapper is a thin execution layer with no authentication requirements.

## Data Formats

### Command-Line Arguments
**Format:** Standard POSIX/Windows command-line format
**Encoding:** UTF-8
**Example:** `gt convoy create "My Work"`

### Standard I/O Streams
**Format:** Text streams (UTF-8)
**Binary:** Not used between parts

## Error Handling

### Wrapper Errors

**Binary Not Found:**
```javascript
if (!fs.existsSync(binaryPath)) {
  console.error('Error: Gas Town binary not found');
  process.exit(1);
}
```

**Spawn Failure:**
```javascript
child.on('error', (error) => {
  console.error('Error spawning binary:', error);
  process.exit(1);
});
```

### Binary Errors

**Handled Internally:** Go binary handles all operational errors
**Exit Codes:** Returned to wrapper, then to user

## Deployment Coordination

### GitHub Releases
- **Go Binary:** Built by GoReleaser
- **NPM Package:** References GitHub release binaries

### Version Coordination

**Release Process:**
1. Tag Git repository: `v1.2.3`
2. GoReleaser builds binaries → GitHub Releases
3. Update NPM package version: `1.2.3`
4. NPM package references GitHub release `v1.2.3` binaries
5. Publish to NPM

## Testing Integration

### Unit Tests
- **npm-wrapper:** Test script in `scripts/test.js`
- **gastown-cli:** Go tests in `*_test.go` files
- **No Cross-Part Tests:** Parts tested independently

### Integration Tests
**E2E Tests** verify full flow:
1. Install via NPM (wrapper)
2. Execute commands (wrapper → binary)
3. Verify functionality (binary)

## Performance Characteristics

### Overhead
**Minimal:** Wrapper adds <10ms overhead
- Process spawn time
- Argument parsing
- I/O piping setup

### Memory
**Wrapper:** ~10MB (Node.js runtime)
**Binary:** ~20-50MB (Go binary + runtime)

### Scalability
**No Bottleneck:** Wrapper simply spawns binary
- All performance characteristics determined by Go binary
- No wrapper-induced limitations

## Security Considerations

### Trust Boundary

```
User
  ├─> Trusts NPM package
  │     └─> Downloads from npmjs.com
  │
  └─> npm-wrapper
        └─> References binary from GitHub Releases
              └─> Trusts GitHub + Code Signing
```

### Attack Surface
**Minimal:** Wrapper is <100 lines of JavaScript
- No network calls at runtime
- No data processing
- Simple process execution

### Supply Chain
1. **Go Binary:** Built by GitHub Actions, signed on macOS
2. **NPM Package:** Published to npmjs.com
3. **Integration:** Wrapper references specific binary version

## Monitoring

### No Inter-Process Communication
- No IPC channels
- No shared memory
- No network sockets

### Observability
**Transparent:** All logging from Go binary
- Wrapper doesn't intercept logs
- Wrapper doesn't add telemetry

## Future Enhancements

### Potential Improvements

1. **Binary Bundling**
   - Bundle Go binary in NPM package (larger package size)
   - Eliminates GitHub download dependency

2. **Version Verification**
   - Verify binary version matches NPM package version
   - Warn on mismatch

3. **Fallback Mechanism**
   - Download binary on first run if missing
   - Better error messages for binary not found

4. **Platform-Specific Packages**
   - Separate NPM packages per platform
   - Smaller download sizes

## Summary

**Integration Type:** Binary Wrapper
**Coupling:** Loose (execution-only)
**Data Flow:** Unidirectional (wrapper → binary)
**Protocol:** Process execution with I/O piping
**Configuration:** Independent
**Testing:** Independent with E2E verification
**Deployment:** Coordinated versioning, independent builds
**Performance Impact:** Minimal (<10ms overhead)
**Security:** Transparent pass-through, minimal attack surface
