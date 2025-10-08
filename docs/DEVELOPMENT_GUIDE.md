# VS Code Development Guide

This guide provides detailed instructions for setting up, developing, and contributing to the VS Code project.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Getting Started](#getting-started)
3. [Development Workflow](#development-workflow)
4. [Build System](#build-system)
5. [Testing](#testing)
6. [Debugging](#debugging)
7. [Extension Development](#extension-development)
8. [Contributing Guidelines](#contributing-guidelines)
9. [Troubleshooting](#troubleshooting)

## Prerequisites

### System Requirements

**Operating System:**
- Windows 10/11 (x64, arm64)
- macOS 10.15+ (x64, arm64)
- Linux (x64, arm64)

**Software Requirements:**
- **Node.js**: Version 18.x or later (LTS recommended)
- **npm**: Version 8.x or later (comes with Node.js)
- **Python**: Version 3.x (for native module compilation)
- **Git**: Latest version
- **C++ Build Tools**: Platform-specific compilers

### Platform-Specific Setup

#### Windows
```powershell
# Install Node.js from nodejs.org
# Install Visual Studio Build Tools
npm install -g windows-build-tools

# Or install Visual Studio Community with C++ workload
```

#### macOS
```bash
# Install Xcode Command Line Tools
xcode-select --install

# Install Node.js via Homebrew (recommended)
brew install node

# Or download from nodejs.org
```

#### Linux (Ubuntu/Debian)
```bash
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install build essentials
sudo apt-get install -y build-essential python3

# Install additional dependencies
sudo apt-get install -y libnss3-dev libatk-bridge2.0-dev libdrm2 libgtk-3-dev libgbm-dev
```

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/microsoft/vscode.git
cd vscode
```

### 2. Install Dependencies

```bash
# Install root dependencies
npm install

# This will also install dependencies for extensions
# and run postinstall scripts
```

### 3. Initial Build

```bash
# Full compilation (takes 5-10 minutes on first run)
npm run compile

# Or use watch mode for development
npm run watch
```

### 4. Launch VS Code

```bash
# Launch development version
./scripts/code.sh        # Linux/macOS
.\scripts\code.bat       # Windows

# Or use npm script
npm run electron
```

## Development Workflow

### Daily Development

1. **Start Watch Mode**
```bash
npm run watch
```
This will:
- Compile TypeScript in watch mode
- Watch for file changes
- Automatically recompile changed files

2. **Launch Development Build**
```bash
./scripts/code.sh --verbose
```

3. **Make Changes**
- Edit source files in `src/`
- Changes are automatically compiled
- Reload the window (Cmd/Ctrl+R) to see changes

### Development Scripts

```bash
# Core compilation
npm run compile              # Full compilation
npm run compile-client       # Client-side only
npm run compile-extensions   # Built-in extensions only

# Watch modes
npm run watch               # Watch all
npm run watch-client        # Watch client only
npm run watch-extensions    # Watch extensions only

# Web development
npm run compile-web         # Compile for web
npm run watch-web          # Watch web version
./scripts/code-web.sh      # Launch web version

# CLI development
npm run compile-cli         # Compile CLI
npm run watch-cli          # Watch CLI
```

### Code Organization

#### Source Structure
```
src/
├── bootstrap-*.ts          # Bootstrap modules
├── main.ts                # Electron main process
├── cli.ts                 # CLI entry point
└── vs/
    ├── base/              # Foundation utilities
    │   ├── common/        # Platform-agnostic code
    │   ├── browser/       # Browser-specific code
    │   └── node/          # Node.js-specific code
    ├── platform/          # Service layer
    │   ├── */common/      # Service interfaces
    │   ├── */browser/     # Browser implementations
    │   ├── */node/        # Node.js implementations
    │   └── */electron-*/  # Electron implementations
    ├── editor/            # Monaco Editor integration
    ├── workbench/         # Main UI application
    │   ├── api/           # Extension API
    │   ├── browser/       # Core workbench
    │   ├── contrib/       # Feature contributions
    │   └── services/      # Workbench services
    └── code/              # Application lifecycle
```

#### Coding Patterns

**Service Definition:**
```typescript
// 1. Define interface in common/
export interface IMyService {
    readonly _serviceBrand: undefined;
    doSomething(): Promise<void>;
}

// 2. Create service identifier
export const IMyService = createDecorator<IMyService>('myService');

// 3. Implement service
export class MyService implements IMyService {
    readonly _serviceBrand: undefined;
    
    constructor(
        @ILogService private readonly logService: ILogService
    ) {}
    
    async doSomething(): Promise<void> {
        this.logService.info('Doing something...');
    }
}

// 4. Register service
registerSingleton(IMyService, MyService, InstantiationType.Delayed);
```

**Contribution Pattern:**
```typescript
// 1. Define contribution interface
export interface IMyContribution extends IWorkbenchContribution {
    // Contribution-specific methods
}

// 2. Implement contribution
export class MyContribution implements IMyContribution {
    constructor(
        @IMyService private readonly myService: IMyService
    ) {
        // Initialize contribution
    }
}

// 3. Register contribution
Registry.as<IWorkbenchContributionsRegistry>(WorkbenchExtensions.Workbench)
    .registerWorkbenchContribution(MyContribution, LifecyclePhase.Restored);
```

## Build System

### Gulp-Based Build

The build system uses Gulp with TypeScript compilation:

```javascript
// gulpfile.js structure
const compileTask = task.define('compile-client', 
    task.series(
        util.rimraf('out'),           // Clean output
        compileApiProposalNamesTask,  // Generate API names
        compileTask('src', 'out', false) // Compile TypeScript
    )
);
```

### TypeScript Configuration

Multiple TypeScript configurations for different targets:

```json
// tsconfig.json - Main configuration
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "commonjs",
        "strict": true,
        "experimentalDecorators": true
    }
}

// tsconfig.monaco.json - Monaco Editor
// tsconfig.vscode-dts.json - API definitions
```

### Build Outputs

```
out/                    # Compiled JavaScript
├── main.js            # Main process
├── cli.js             # CLI
└── vs/                # Core modules

extensions/            # Built-in extensions
├── */out/            # Compiled extension code
└── */package.json    # Extension manifests
```

## Testing

### Test Structure

```
test/
├── automation/        # End-to-end tests
├── integration/       # Integration tests
├── smoke/            # Smoke tests
└── unit/             # Unit tests
    ├── browser/      # Browser environment tests
    └── node/         # Node.js environment tests
```

### Running Tests

```bash
# Unit tests
npm run test-node              # Node.js tests
npm run test-browser           # Browser tests

# Integration tests
npm run test-integration       # Full integration suite

# Smoke tests
npm run smoketest             # Basic functionality tests

# Specific test suites
npm test -- --grep "FileService"  # Run specific tests
```

### Writing Tests

```typescript
// Unit test example
import * as assert from 'assert';
import { MyService } from 'vs/platform/myService/common/myService';

suite('MyService', () => {
    test('should do something', async () => {
        const service = new MyService();
        const result = await service.doSomething();
        assert.strictEqual(result, 'expected');
    });
});

// Integration test with services
suite('MyService Integration', () => {
    let instantiationService: IInstantiationService;
    
    setup(() => {
        const serviceCollection = new ServiceCollection();
        // Set up required services
        instantiationService = new InstantiationService(serviceCollection);
    });
    
    test('should work with dependencies', async () => {
        const service = instantiationService.createInstance(MyService);
        // Test service with real dependencies
    });
});
```

## Debugging

### VS Code Development

1. **Launch Configuration** (`.vscode/launch.json`):
```json
{
    "name": "Launch VS Code",
    "type": "node",
    "request": "launch",
    "program": "${workspaceFolder}/out/main.js",
    "args": ["--no-sandbox"],
    "env": {
        "VSCODE_DEV": "1"
    }
}
```

2. **Debug Main Process:**
```bash
# Launch with debugger
./scripts/code.sh --inspect=9229

# Attach debugger to chrome://inspect
```

3. **Debug Renderer Process:**
- Open Developer Tools (Help > Toggle Developer Tools)
- Use browser debugging tools
- Set breakpoints in TypeScript source

4. **Debug Extension Host:**
```bash
# Launch with extension host debugging
./scripts/code.sh --inspect-extensions=9230
```

### Logging

```typescript
// Use logging service
constructor(
    @ILogService private readonly logService: ILogService
) {}

// Log levels
this.logService.trace('Detailed debug info');
this.logService.debug('Debug information');
this.logService.info('General information');
this.logService.warn('Warning message');
this.logService.error('Error occurred', error);
```

### Performance Profiling

```bash
# CPU profiling
./scripts/code.sh --prof

# Memory profiling
./scripts/code.sh --inspect --inspect-brk
```

## Extension Development

### Built-in Extension Structure

```
extensions/my-extension/
├── package.json           # Extension manifest
├── src/
│   ├── extension.ts      # Main extension file
│   └── ...
├── syntaxes/             # Language grammars
├── themes/               # Color themes
└── README.md
```

### Extension Manifest

```json
{
    "name": "my-extension",
    "displayName": "My Extension",
    "version": "1.0.0",
    "engines": {
        "vscode": "^1.106.0"
    },
    "activationEvents": [
        "onLanguage:typescript"
    ],
    "main": "./out/extension.js",
    "contributes": {
        "commands": [{
            "command": "myExtension.hello",
            "title": "Hello World"
        }],
        "languages": [{
            "id": "mylang",
            "extensions": [".mylang"]
        }]
    }
}
```

### Extension Development Workflow

1. **Create Extension:**
```bash
cd extensions
mkdir my-extension
cd my-extension
npm init
```

2. **Develop Extension:**
```typescript
// src/extension.ts
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
    const disposable = vscode.commands.registerCommand('myExtension.hello', () => {
        vscode.window.showInformationMessage('Hello World!');
    });
    
    context.subscriptions.push(disposable);
}

export function deactivate() {}
```

3. **Build Extension:**
```bash
npm run compile
```

4. **Test Extension:**
- Launch VS Code development build
- Extension is automatically loaded
- Test functionality

## Contributing Guidelines

### Code Style

1. **TypeScript Guidelines:**
   - Use strict TypeScript settings
   - Prefer interfaces over types
   - Use readonly for immutable properties
   - Explicit return types for public methods

2. **Naming Conventions:**
   - PascalCase for classes and interfaces
   - camelCase for methods and properties
   - UPPER_CASE for constants
   - Prefix interfaces with 'I'

3. **File Organization:**
   - One class per file
   - Group related functionality
   - Use barrel exports (index.ts)

### Pull Request Process

1. **Fork and Branch:**
```bash
git fork https://github.com/microsoft/vscode.git
git checkout -b feature/my-feature
```

2. **Make Changes:**
   - Follow coding guidelines
   - Add tests for new functionality
   - Update documentation

3. **Test Changes:**
```bash
npm run compile
npm run test
npm run smoketest
```

4. **Submit PR:**
   - Clear description of changes
   - Link to related issues
   - Include test results

### Commit Guidelines

```bash
# Format: type(scope): description
feat(editor): add new syntax highlighting
fix(files): resolve file watcher memory leak
docs(api): update extension API documentation
test(search): add unit tests for search service
```

## Troubleshooting

### Common Issues

1. **Build Failures:**
```bash
# Clean and rebuild
npm run clean
npm install
npm run compile
```

2. **Node.js Version Issues:**
```bash
# Check Node.js version
node --version  # Should be 18.x+

# Use nvm to manage versions
nvm install 18
nvm use 18
```

3. **Native Module Compilation:**
```bash
# Rebuild native modules
npm rebuild

# Clear npm cache
npm cache clean --force
```

4. **Extension Host Issues:**
```bash
# Launch with extension host debugging
./scripts/code.sh --disable-extensions
./scripts/code.sh --inspect-extensions
```

### Performance Issues

1. **Slow Compilation:**
   - Use incremental compilation (`npm run watch`)
   - Exclude unnecessary files in tsconfig.json
   - Use faster hardware (SSD, more RAM)

2. **Runtime Performance:**
   - Profile with Developer Tools
   - Check for memory leaks
   - Optimize hot code paths

### Getting Help

- **GitHub Issues**: Report bugs and feature requests
- **Discussions**: Ask questions and share ideas
- **Discord/Gitter**: Real-time community chat
- **Documentation**: Official VS Code documentation
- **Stack Overflow**: Technical questions with 'vscode' tag

This guide should get you started with VS Code development. Remember to check the official contributing guidelines and join the community for support!