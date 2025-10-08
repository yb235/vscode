# Visual Studio Code - Open Source ("Code - OSS") Documentation

Welcome to the comprehensive documentation for the Visual Studio Code Open Source project. This documentation is designed to help first-time contributors and developers understand the codebase, architecture, and development workflow.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Directory Structure](#directory-structure)
4. [Key Entry Points](#key-entry-points)
5. [Build System](#build-system)
6. [Development Workflow](#development-workflow)
7. [Extension System](#extension-system)
8. [Platform Support](#platform-support)
9. [APIs and Services](#apis-and-services)
10. [Getting Started](#getting-started)

## Project Overview

Visual Studio Code is a free, open-source code editor developed by Microsoft. The project is built using TypeScript/JavaScript and Electron, providing a cross-platform desktop application that runs on Windows, macOS, and Linux.

### Key Facts
- **Name**: Code - OSS (Open Source)
- **Version**: 1.106.0
- **License**: MIT
- **Main Technologies**: TypeScript, Electron, Node.js
- **Package Manager**: npm
- **Build System**: Gulp + TypeScript

## Architecture Overview

VS Code follows a modular, layered architecture with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                    Workbench Layer                          │
│  (UI Components, Editors, Views, Contributions)            │
├─────────────────────────────────────────────────────────────┤
│                    Platform Layer                           │
│  (Services, Dependency Injection, Configuration)           │
├─────────────────────────────────────────────────────────────┤
│                      Base Layer                             │
│  (Common Utilities, Event System, Async Helpers)          │
├─────────────────────────────────────────────────────────────┤
│                   Runtime Layer                             │
│  (Electron Main, Renderer, CLI, Web)                      │
└─────────────────────────────────────────────────────────────┘
```

### Core Principles

1. **Dependency Injection**: Extensive use of DI for service management
2. **Event-Driven**: Asynchronous communication via events
3. **Modular Design**: Clear separation between layers and components
4. **Cross-Platform**: Single codebase supporting multiple platforms
5. **Extensible**: Rich extension API and contribution points

## Directory Structure

```
vscode/
├── build/                    # Build scripts and configuration
├── cli/                      # Rust-based CLI implementation
├── extensions/               # Built-in extensions
├── remote/                   # Remote development support
├── resources/                # Platform-specific resources
├── scripts/                  # Development and build scripts
├── src/                      # Main source code
│   ├── bootstrap-*.ts        # Bootstrap modules
│   ├── main.ts              # Electron main process entry
│   ├── cli.ts               # CLI entry point
│   └── vs/                   # Core VS Code modules
│       ├── base/             # Base utilities and common code
│       ├── platform/         # Platform services and abstractions
│       ├── editor/           # Monaco Editor integration
│       ├── workbench/        # Main UI and application logic
│       └── code/             # Application startup and lifecycle
├── test/                     # Test suites
└── package.json             # Main package configuration
```

### Key Directories Explained

- **`src/vs/base/`**: Foundation layer with utilities, events, async helpers
- **`src/vs/platform/`**: Service layer with dependency injection, configuration, logging
- **`src/vs/workbench/`**: UI layer with editors, views, and user interactions
- **`src/vs/code/`**: Application layer handling startup and process management
- **`extensions/`**: Built-in extensions (language support, themes, etc.)

## Key Entry Points

### 1. Main Process (`src/main.ts`)
The Electron main process entry point that:
- Configures the application environment
- Sets up crash reporting and logging
- Manages application lifecycle
- Handles command-line arguments
- Bootstraps the main application

### 2. CLI Entry (`src/cli.ts`)
Command-line interface that:
- Processes CLI commands
- Handles extension management
- Supports tunnel/remote operations
- Provides help and version information

### 3. Workbench Entry Points
- **Desktop**: `src/vs/workbench/workbench.desktop.main.ts`
- **Web**: `src/vs/workbench/workbench.web.main.ts`
- **Common**: `src/vs/workbench/workbench.common.main.ts`

### 4. Application Startup (`src/vs/code/electron-main/main.ts`)
The main application class that:
- Creates and initializes services
- Manages multiple instances
- Handles IPC communication
- Sets up the workbench

## Build System

VS Code uses a sophisticated build system based on:

### Primary Tools
- **Gulp**: Task runner for build orchestration
- **TypeScript**: Primary language with strict type checking
- **esbuild/SWC**: Fast transpilation for development
- **Webpack**: Module bundling for web deployment

### Key Build Tasks
```bash
npm run compile          # Full compilation
npm run watch           # Watch mode for development
npm run compile-web     # Web-specific build
npm run compile-cli     # CLI compilation
npm run test           # Run test suites
```

### Build Configuration
- **`gulpfile.js`**: Main build orchestration
- **`tsconfig.json`**: TypeScript configuration
- **`build/`**: Build scripts and utilities
- **Multiple tsconfig files**: Different configs for different targets

## Development Workflow

### 1. Setup
```bash
git clone https://github.com/microsoft/vscode.git
cd vscode
npm install
npm run compile
```

### 2. Development
```bash
npm run watch          # Start watch mode
./scripts/code.sh      # Launch development build
```

### 3. Testing
```bash
npm run test-browser   # Browser tests
npm run test-node      # Node.js tests
npm run smoketest      # Integration tests
```

### 4. Extensions
```bash
npm run compile-extensions    # Compile built-in extensions
npm run watch-extensions     # Watch extensions
```

## Extension System

VS Code has a rich extension ecosystem with multiple types:

### Built-in Extensions (`extensions/`)
- **Language Support**: TypeScript, JavaScript, Python, etc.
- **Themes**: Color themes and icon themes
- **Features**: Git, Debug, Search, etc.
- **Tools**: Emmet, Markdown support, etc.

### Extension Architecture
- **Extension Host**: Separate process for extension execution
- **API Surface**: Well-defined extension API
- **Contribution Points**: Declarative extension capabilities
- **Activation Events**: Lazy loading of extensions

### Key Extension Files
- **`package.json`**: Extension manifest
- **`extension.ts`**: Main extension code
- **Contribution points**: Commands, views, languages, etc.

## Platform Support

VS Code supports multiple deployment targets:

### Desktop (Electron)
- **Windows**: x64, arm64
- **macOS**: x64, arm64 (Universal)
- **Linux**: x64, arm64

### Web
- **Browser-based**: Full VS Code in the browser
- **Remote development**: Connect to remote environments
- **GitHub Codespaces**: Cloud development environments

### CLI
- **Rust-based**: High-performance CLI tool
- **Tunneling**: Secure remote connections
- **Extension management**: Install/manage extensions

## APIs and Services

### Core Services (Dependency Injection)
```typescript
// Example service registration
registerSingleton(IFileService, FileService);
registerSingleton(IConfigurationService, ConfigurationService);
registerSingleton(ILogService, LogService);
```

### Key Service Categories
- **File System**: File operations, watchers, providers
- **Configuration**: Settings, preferences, workspace config
- **Lifecycle**: Application state management
- **Telemetry**: Usage analytics and crash reporting
- **Extensions**: Extension management and hosting
- **Editor**: Text editing, language services
- **Workbench**: UI components, layout, themes

### Extension API
```typescript
// Extension API example
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
    // Register commands, providers, etc.
}
```

## Getting Started

### For Contributors
1. **Read the Code**: Start with `src/main.ts` and follow the bootstrap process
2. **Understand Services**: Explore `src/vs/platform/` for core services
3. **Study the Workbench**: Look at `src/vs/workbench/` for UI components
4. **Build and Run**: Use the development workflow above
5. **Write Tests**: Add tests for any new functionality

### For Extension Developers
1. **API Documentation**: Study the extension API
2. **Sample Extensions**: Look at built-in extensions for examples
3. **Contribution Points**: Understand how to extend VS Code
4. **Testing**: Use the extension testing framework

### Key Resources
- **Contributing Guide**: `CONTRIBUTING.md`
- **Build Instructions**: Repository wiki
- **Extension API**: Official documentation
- **Architecture Decisions**: GitHub discussions

## Development Environment

### Recommended Setup
- **Node.js**: Latest LTS version
- **Python**: For native module compilation
- **Git**: Version control
- **VS Code**: Self-hosting for development

### Environment Variables
- **`VSCODE_DEV`**: Development mode flag
- **`VSCODE_CLI`**: CLI mode flag
- **`ELECTRON_RUN_AS_NODE`**: Electron environment control

This documentation provides a comprehensive overview of the VS Code codebase. For specific implementation details, refer to the individual source files and their inline documentation.