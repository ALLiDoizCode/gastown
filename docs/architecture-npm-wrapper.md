# Architecture: NPM Wrapper

**Part:** npm-wrapper (Distribution Package)
**Type:** Binary Distribution Wrapper
**Language:** JavaScript (Node.js)
**Generated:** 2026-02-17

## Executive Summary

The NPM wrapper is a lightweight Node.js package that provides cross-platform distribution of the Gas Town CLI via the NPM ecosystem. It consists of ~100 lines of JavaScript that detect the user's platform, manage binary installation, and provide a transparent execution wrapper for the Go binary.

## Technology Stack

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| **Runtime** | Node.js | 14+ | Execution environment |
| **Package Manager** | NPM | - | Distribution channel |
| **Package Name** | @gastown/gt | 0.7.0 | Scoped package |

## Architecture Pattern

**Pattern:** Binary Distribution Wrapper

**Characteristics:**
- Minimal code (<100 lines)
- Platform detection
- Binary execution proxy
- Zero runtime dependencies

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    NPM Package                           │
│                   @gastown/gt                           │
│                                                          │
│  ┌───────────────────────────────────────────────────┐ │
│  │ package.json                                      │ │
│  │ • Metadata (name, version, author)               │ │
│  │ • Bin entry: "gt": "bin/gt.js"                  │ │
│  │ • Postinstall: "node scripts/postinstall.js"    │ │
│  │ • Platforms: darwin, linux, win32               │ │
│  │ • Architectures: x64, arm64                     │ │
│  └───────────────────────────────────────────────────┘ │
│                                                          │
│  ┌───────────────────────────────────────────────────┐ │
│  │ scripts/postinstall.js                           │ │
│  │ • Detect platform (process.platform)            │ │
│  │ • Detect architecture (process.arch)            │ │
│  │ • Download/reference Go binary                  │ │
│  │ • Set executable permissions                    │ │
│  └───────────────────────────────────────────────────┘ │
│                                                          │
│  ┌───────────────────────────────────────────────────┐ │
│  │ bin/gt.js                                        │ │
│  │ • Resolve binary path                           │ │
│  │ • Spawn child process                           │ │
│  │ • Pipe stdio (inherit)                          │ │
│  │ • Exit with binary's code                       │ │
│  └───────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
                         │
                   spawn() → Go Binary
```

## Component Overview

### package.json
**Purpose:** NPM package manifest

**Key Fields:**
```json
{
  "name": "@gastown/gt",
  "version": "0.7.0",
  "main": "bin/gt.js",
  "bin": {
    "gt": "bin/gt.js"
  },
  "scripts": {
    "postinstall": "node scripts/postinstall.js",
    "test": "node scripts/test.js"
  },
  "engines": {
    "node": ">=14.0.0"
  },
  "os": ["darwin", "linux", "win32"],
  "cpu": ["x64", "arm64"]
}
```

### scripts/postinstall.js
**Purpose:** Binary installation

**Pseudo-code:**
```javascript
const platform = process.platform;  // darwin, linux, win32
const arch = process.arch;          // x64, arm64

// Determine binary filename
const binaryName = `gt-${platform}-${arch}`;

// Download or reference binary
// (from GitHub Releases or bundled)

// Set executable permissions (Unix)
if (platform !== 'win32') {
  fs.chmodSync(binaryPath, 0o755);
}
```

### bin/gt.js
**Purpose:** Execution wrapper

**Pseudo-code:**
```javascript
#!/usr/bin/env node

const { spawn } = require('child_process');
const path = require('path');

// Resolve binary path
const binaryPath = path.join(__dirname, '..', 'bin', getBinaryName());

// Verify binary exists
if (!fs.existsSync(binaryPath)) {
  console.error('Gas Town binary not found');
  process.exit(1);
}

// Spawn Go binary with args
const child = spawn(binaryPath, process.argv.slice(2), {
  stdio: 'inherit'  // Pipe stdin/stdout/stderr
});

// Handle errors
child.on('error', (error) => {
  console.error('Error spawning binary:', error);
  process.exit(1);
});

// Exit with binary's code
child.on('exit', (code) => {
  process.exit(code || 0);
});
```

### scripts/test.js
**Purpose:** Basic functionality test

**Pseudo-code:**
```javascript
const { execSync } = require('child_process');

try {
  // Test binary execution
  execSync('gt --version', { stdio: 'inherit' });
  console.log('✓ Binary executes successfully');
} catch (error) {
  console.error('✗ Binary test failed:', error);
  process.exit(1);
}
```

## Data Flow

### Installation Flow
```
1. npm install -g @gastown/gt
   ↓
2. NPM downloads package from registry
   ↓
3. NPM runs postinstall script
   ↓
4. postinstall.js detects platform/arch
   ↓
5. Binary downloaded/referenced
   ↓
6. Permissions set (chmod +x on Unix)
   ↓
7. NPM creates bin symlink
   ↓
8. Installation complete
```

### Execution Flow
```
1. User runs: gt <command> <args>
   ↓
2. Shell resolves to bin symlink
   ↓
3. Node.js executes bin/gt.js
   ↓
4. gt.js resolves binary path
   ↓
5. spawn() creates child process
   ↓
6. Go binary executes command
   ↓
7. Output piped to user's terminal
   ↓
8. Binary exits with code N
   ↓
9. gt.js exits with code N
```

## Platform Support

### Supported Platforms
- **darwin** (macOS): x64, arm64
- **linux**: x64, arm64
- **win32** (Windows): x64

### Platform Detection
```javascript
function getBinaryName() {
  const platform = process.platform;
  const arch = process.arch;

  const ext = platform === 'win32' ? '.exe' : '';
  return `gt-${platform}-${arch}${ext}`;
}
```

## Error Handling

### Binary Not Found
```javascript
if (!fs.existsSync(binaryPath)) {
  console.error('Error: Gas Town binary not found');
  console.error('Platform:', platform);
  console.error('Architecture:', arch);
  console.error('Expected path:', binaryPath);
  process.exit(1);
}
```

### Spawn Failure
```javascript
child.on('error', (error) => {
  console.error('Error spawning Gas Town binary:', error.message);
  process.exit(1);
}```

### Unsupported Platform
```javascript
const supported = ['darwin', 'linux', 'win32'];
if (!supported.includes(platform)) {
  console.error(`Unsupported platform: ${platform}`);
  process.exit(1);
}
```

## Testing Strategy

### Functional Test
**Purpose:** Verify binary execution

```javascript
// scripts/test.js
execSync('gt --version');  // Should succeed
```

### Installation Test
**Purpose:** Verify postinstall works

```bash
# Manual test
npm pack
npm install -g ./gastown-gt-0.7.0.tgz
gt --version
```

## Deployment

### NPM Publishing
```bash
# From npm-package/ directory
npm version 0.7.0
npm publish
```

### Version Coordination
- NPM version should match Git tag
- Reference matching Go binary version
- Update package.json manually

## Performance Characteristics

### Overhead
**Minimal:** <10ms
- Process spawn time: ~5ms
- Argument parsing: <1ms
- I/O setup: ~2ms

### Memory
- Node.js runtime: ~10MB
- Wrapper script: <1MB
- Total overhead: ~10MB

### Binary Size
- NPM package: <100KB (without bundled binary)
- With bundled binary: ~20MB per platform

## Security Considerations

### Supply Chain
1. NPM package published to npmjs.com
2. Binary downloaded from GitHub Releases
3. Checksums verified (future enhancement)

### Attack Surface
- Minimal: <100 lines of code
- No network calls at runtime
- No data processing
- Simple process execution

### Recommendations
- Verify binary checksums (TODO)
- HTTPS for binary downloads
- SRI hashes in package.json (future)

## Limitations

### No Fallback
- If binary missing, hard failure
- No automatic download retry

### Platform-Specific
- Each platform needs correct binary
- No universal binary

### Version Lock
- NPM version must match binary version
- Manual coordination required

## Future Enhancements

### Potential Improvements
1. **Binary Bundling:** Include binary in package (larger size)
2. **Checksum Verification:** Verify downloaded binary
3. **Auto-Download:** Download on first run if missing
4. **Platform-Specific Packages:** Separate packages per platform
5. **Update Notifications:** Check for newer versions

## File Structure

```
npm-package/
├── package.json         # NPM manifest
├── bin/
│   └── gt.js           # Wrapper script (~50 lines)
├── scripts/
│   ├── postinstall.js  # Installation script (~30 lines)
│   └── test.js         # Test script (~20 lines)
├── README.md           # Package documentation
└── LICENSE             # MIT License
```

**Total Code:** ~100 lines JavaScript

## Dependencies

### Runtime Dependencies
**None** - Zero npm dependencies at runtime

### Dev Dependencies
**None** - Pure Node.js built-ins

### External Dependencies
- **Node.js:** 14+ (runtime)
- **Go Binary:** From GitHub Releases

## Integration with Go Binary

**See:** `integration-architecture.md` for detailed integration documentation

**Summary:**
- Transparent pass-through
- No data transformation
- Exit code preservation
- stdio inheritance

## Monitoring & Observability

### No Telemetry
- Wrapper doesn't log
- Wrapper doesn't track usage
- All logging from Go binary

### Debugging
```bash
# Enable verbose output (if supported by binary)
DEBUG=* gt <command>

# Or use Node.js debugging
node --inspect bin/gt.js <command>
```

## References

- **Integration:** `integration-architecture.md`
- **Main Architecture:** `architecture-gastown-cli.md`
- **Deployment:** `deployment-guide.md`
