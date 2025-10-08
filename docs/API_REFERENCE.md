# VS Code API Reference

This document provides a comprehensive reference for the VS Code internal APIs, services, and extension points.

## Table of Contents

1. [Core Services](#core-services)
2. [Platform Services](#platform-services)
3. [Workbench Services](#workbench-services)
4. [Extension API](#extension-api)
5. [Contribution Points](#contribution-points)
6. [Event System](#event-system)
7. [Dependency Injection](#dependency-injection)
8. [Common Patterns](#common-patterns)

## Core Services

### IInstantiationService

The dependency injection service that manages service creation and lifecycle.

```typescript
export interface IInstantiationService {
    readonly _serviceBrand: undefined;
    
    /**
     * Create an instance of a service with automatic dependency resolution
     */
    createInstance<T>(ctor: new (...args: any[]) => T, ...args: any[]): T;
    
    /**
     * Invoke a function with service resolution
     */
    invokeFunction<R>(fn: (accessor: ServicesAccessor, ...args: any[]) => R, ...args: any[]): R;
}

// Usage example
const myService = instantiationService.createInstance(MyService, additionalArg);

instantiationService.invokeFunction(accessor => {
    const fileService = accessor.get(IFileService);
    const logService = accessor.get(ILogService);
    // Use services...
});
```

### ILogService

Centralized logging service with multiple log levels.

```typescript
export interface ILogService {
    readonly _serviceBrand: undefined;
    
    trace(message: string, ...args: any[]): void;
    debug(message: string, ...args: any[]): void;
    info(message: string, ...args: any[]): void;
    warn(message: string, ...args: any[]): void;
    error(message: string | Error, ...args: any[]): void;
    
    getLevel(): LogLevel;
    setLevel(level: LogLevel): void;
    
    onDidChangeLogLevel: Event<LogLevel>;
}

// Usage
constructor(@ILogService private readonly logService: ILogService) {}

this.logService.info('Operation completed successfully');
this.logService.error('Failed to process file', error);
```

### IConfigurationService

Configuration and settings management.

```typescript
export interface IConfigurationService {
    readonly _serviceBrand: undefined;
    
    /**
     * Get configuration value
     */
    getValue<T>(section?: string, overrides?: IConfigurationOverrides): T;
    
    /**
     * Update configuration value
     */
    updateValue(key: string, value: any, overrides?: IConfigurationOverrides): Promise<void>;
    
    /**
     * Configuration change events
     */
    onDidChangeConfiguration: Event<IConfigurationChangeEvent>;
    
    /**
     * Get configuration keys
     */
    keys(): IConfigurationKeys;
}

// Usage
const fontSize = configurationService.getValue<number>('editor.fontSize');
const editorConfig = configurationService.getValue('editor');

// Listen for changes
configurationService.onDidChangeConfiguration(e => {
    if (e.affectsConfiguration('editor.fontSize')) {
        // Handle font size change
    }
});
```

## Platform Services

### IFileService

File system abstraction with provider support.

```typescript
export interface IFileService {
    readonly _serviceBrand: undefined;
    
    /**
     * File operations
     */
    resolve(resource: URI, options?: IResolveFileOptions): Promise<IFileStat>;
    readFile(resource: URI, options?: IReadFileOptions): Promise<IFileContent>;
    writeFile(resource: URI, content: VSBuffer, options?: IWriteFileOptions): Promise<IFileStatWithMetadata>;
    
    /**
     * Directory operations
     */
    createFolder(resource: URI): Promise<IFileStatWithMetadata>;
    del(resource: URI, options?: Partial<IFileDeleteOptions>): Promise<void>;
    
    /**
     * File watching
     */
    watch(resource: URI): IDisposable;
    onDidFilesChange: Event<FileChangesEvent>;
    
    /**
     * Provider registration
     */
    registerProvider(scheme: string, provider: IFileSystemProvider): IDisposable;
}

// Usage
const fileContent = await fileService.readFile(URI.file('/path/to/file.txt'));
const fileText = fileContent.value.toString();

// Watch for changes
const watcher = fileService.watch(URI.file('/path/to/watch'));
fileService.onDidFilesChange(e => {
    for (const change of e.changes) {
        console.log(`File ${change.resource.toString()} was ${change.type}`);
    }
});
```

### IEnvironmentService

Environment and path information.

```typescript
export interface IEnvironmentService {
    readonly _serviceBrand: undefined;
    
    // Paths
    readonly userHome: URI;
    readonly userDataPath: string;
    readonly appRoot: string;
    readonly logsHome: URI;
    readonly extensionsPath: string;
    
    // Flags
    readonly isBuilt: boolean;
    readonly verbose: boolean;
    readonly args: NativeParsedArgs;
    
    // Configuration
    readonly machineSettingsResource: URI;
    readonly userSettingsResource: URI;
    readonly workspaceSettingsResource?: URI;
}
```

### ILifecycleService

Application lifecycle management.

```typescript
export const enum LifecyclePhase {
    Starting = 1,
    Ready = 2,
    Restored = 3,
    Eventually = 4
}

export interface ILifecycleService {
    readonly _serviceBrand: undefined;
    
    readonly phase: LifecyclePhase;
    
    /**
     * Lifecycle events
     */
    onWillShutdown: Event<WillShutdownEvent>;
    onDidShutdown: Event<void>;
    
    /**
     * Phase transitions
     */
    when(phase: LifecyclePhase): Promise<void>;
}

// Usage
await lifecycleService.when(LifecyclePhase.Restored);
console.log('Workbench is fully restored');

lifecycleService.onWillShutdown(e => {
    // Perform cleanup before shutdown
    e.veto(this.performCleanup());
});
```

## Workbench Services

### IEditorService

Editor management and operations.

```typescript
export interface IEditorService {
    readonly _serviceBrand: undefined;
    
    /**
     * Active editor
     */
    readonly activeEditor: IEditorInput | undefined;
    readonly activeEditorPane: IVisibleEditorPane | undefined;
    
    /**
     * Editor operations
     */
    openEditor(editor: IEditorInput, options?: IEditorOptions, group?: PreferredGroup): Promise<IEditorPane | undefined>;
    openEditors(editors: IUntypedEditorInput[], group?: PreferredGroup): Promise<IEditorPane[]>;
    
    /**
     * Editor events
     */
    onDidActiveEditorChange: Event<void>;
    onDidVisibleEditorsChange: Event<void>;
    
    /**
     * Editor groups
     */
    readonly groups: IEditorGroupsService;
}

// Usage
const editor = await editorService.openEditor({
    resource: URI.file('/path/to/file.ts'),
    options: {
        selection: { startLineNumber: 10, startColumn: 1 }
    }
});

editorService.onDidActiveEditorChange(() => {
    const activeEditor = editorService.activeEditor;
    if (activeEditor) {
        console.log('Active editor changed to:', activeEditor.resource?.toString());
    }
});
```

### ICommandService

Command execution and registration.

```typescript
export interface ICommandService {
    readonly _serviceBrand: undefined;
    
    /**
     * Execute command
     */
    executeCommand<T = any>(commandId: string, ...args: any[]): Promise<T>;
    
    /**
     * Command events
     */
    onWillExecuteCommand: Event<ICommandEvent>;
    onDidExecuteCommand: Event<ICommandEvent>;
}

// Usage
await commandService.executeCommand('workbench.action.files.save');
await commandService.executeCommand('vscode.open', URI.file('/path/to/file'));

commandService.onWillExecuteCommand(e => {
    console.log('About to execute command:', e.commandId);
});
```

### INotificationService

User notifications and messages.

```typescript
export interface INotificationService {
    readonly _serviceBrand: undefined;
    
    /**
     * Show notifications
     */
    info(message: string | Error): INotificationHandle;
    warn(message: string | Error): INotificationHandle;
    error(message: string | Error): INotificationHandle;
    
    /**
     * Show notification with actions
     */
    notify(notification: INotification): INotificationHandle;
    
    /**
     * Progress notifications
     */
    withProgress<T>(options: IProgressOptions, task: (progress: IProgress<IProgressStep>) => Promise<T>): Promise<T>;
}

// Usage
notificationService.info('Operation completed successfully');

const handle = notificationService.notify({
    severity: Severity.Warning,
    message: 'Are you sure?',
    actions: {
        primary: [
            { id: 'yes', label: 'Yes' },
            { id: 'no', label: 'No' }
        ]
    }
});

handle.onDidClose(e => {
    if (e.reason === NotificationCloseReason.ACTION) {
        console.log('User clicked:', e.action?.id);
    }
});
```

## Extension API

### Extension Context

The context object passed to extension activation.

```typescript
export interface ExtensionContext {
    /**
     * Extension metadata
     */
    readonly extension: Extension<any>;
    readonly extensionPath: string;
    readonly extensionUri: Uri;
    
    /**
     * Storage
     */
    readonly globalState: Memento;
    readonly workspaceState: Memento;
    readonly secrets: SecretStorage;
    
    /**
     * Lifecycle
     */
    readonly subscriptions: { dispose(): any }[];
    
    /**
     * Utilities
     */
    asAbsolutePath(relativePath: string): string;
}
```

### Commands API

```typescript
export namespace commands {
    /**
     * Register command
     */
    export function registerCommand(command: string, callback: (...args: any[]) => any): Disposable;
    
    /**
     * Register text editor command
     */
    export function registerTextEditorCommand(command: string, callback: (textEditor: TextEditor, edit: TextEditorEdit, ...args: any[]) => void): Disposable;
    
    /**
     * Execute command
     */
    export function executeCommand<T = unknown>(command: string, ...rest: any[]): Thenable<T>;
    
    /**
     * Get all commands
     */
    export function getCommands(filterInternal?: boolean): Thenable<string[]>;
}

// Usage
const disposable = vscode.commands.registerCommand('myExtension.hello', () => {
    vscode.window.showInformationMessage('Hello World!');
});

context.subscriptions.push(disposable);
```

### Window API

```typescript
export namespace window {
    /**
     * Show messages
     */
    export function showInformationMessage<T extends string>(message: string, ...items: T[]): Thenable<T | undefined>;
    export function showWarningMessage<T extends string>(message: string, ...items: T[]): Thenable<T | undefined>;
    export function showErrorMessage<T extends string>(message: string, ...items: T[]): Thenable<T | undefined>;
    
    /**
     * Input boxes
     */
    export function showInputBox(options?: InputBoxOptions): Thenable<string | undefined>;
    export function showQuickPick<T extends QuickPickItem>(items: readonly T[], options?: QuickPickOptions): Thenable<T | undefined>;
    
    /**
     * Active editor
     */
    export let activeTextEditor: TextEditor | undefined;
    export const onDidChangeActiveTextEditor: Event<TextEditor | undefined>;
    
    /**
     * Visible editors
     */
    export let visibleTextEditors: readonly TextEditor[];
    export const onDidChangeVisibleTextEditors: Event<readonly TextEditor[]>;
}
```

### Workspace API

```typescript
export namespace workspace {
    /**
     * Workspace folders
     */
    export let workspaceFolders: readonly WorkspaceFolder[] | undefined;
    export const onDidChangeWorkspaceFolders: Event<WorkspaceFoldersChangeEvent>;
    
    /**
     * Configuration
     */
    export function getConfiguration(section?: string, scope?: ConfigurationScope): WorkspaceConfiguration;
    export const onDidChangeConfiguration: Event<ConfigurationChangeEvent>;
    
    /**
     * File operations
     */
    export function openTextDocument(uri: Uri): Thenable<TextDocument>;
    export function openTextDocument(fileName: string): Thenable<TextDocument>;
    export function saveAll(includeUntitled?: boolean): Thenable<boolean>;
    
    /**
     * File system watcher
     */
    export function createFileSystemWatcher(globPattern: GlobPattern): FileSystemWatcher;
}
```

## Contribution Points

### Commands

```json
{
    "contributes": {
        "commands": [{
            "command": "myExtension.hello",
            "title": "Hello World",
            "category": "My Extension",
            "icon": "$(heart)"
        }]
    }
}
```

### Menus

```json
{
    "contributes": {
        "menus": {
            "commandPalette": [{
                "command": "myExtension.hello",
                "when": "editorLangId == typescript"
            }],
            "editor/context": [{
                "command": "myExtension.hello",
                "group": "myGroup@1"
            }]
        }
    }
}
```

### Languages

```json
{
    "contributes": {
        "languages": [{
            "id": "mylang",
            "aliases": ["My Language", "mylang"],
            "extensions": [".mylang"],
            "configuration": "./language-configuration.json"
        }],
        "grammars": [{
            "language": "mylang",
            "scopeName": "source.mylang",
            "path": "./syntaxes/mylang.tmGrammar.json"
        }]
    }
}
```

### Configuration

```json
{
    "contributes": {
        "configuration": {
            "title": "My Extension",
            "properties": {
                "myExtension.enable": {
                    "type": "boolean",
                    "default": true,
                    "description": "Enable my extension"
                },
                "myExtension.timeout": {
                    "type": "number",
                    "default": 5000,
                    "description": "Timeout in milliseconds"
                }
            }
        }
    }
}
```

## Event System

### Event Interface

```typescript
export interface Event<T> {
    (listener: (e: T) => any, thisArgs?: any, disposables?: Disposable[]): Disposable;
}

// Event utilities
export namespace Event {
    export function once<T>(event: Event<T>): Event<T>;
    export function map<I, O>(event: Event<I>, map: (i: I) => O): Event<O>;
    export function filter<T>(event: Event<T>, filter: (e: T) => boolean): Event<T>;
    export function any<T>(...events: Event<T>[]): Event<T>;
}
```

### Emitter Class

```typescript
export class Emitter<T> {
    private readonly _event: Event<T>;
    
    constructor() {
        // Implementation
    }
    
    get event(): Event<T> {
        return this._event;
    }
    
    fire(event: T): void {
        // Fire event to all listeners
    }
    
    dispose(): void {
        // Clean up resources
    }
}

// Usage
class MyService {
    private readonly _onDidChange = new Emitter<string>();
    readonly onDidChange = this._onDidChange.event;
    
    private fireChange(value: string): void {
        this._onDidChange.fire(value);
    }
}

// Subscribe to events
const disposable = myService.onDidChange(value => {
    console.log('Value changed:', value);
});
```

## Dependency Injection

### Service Registration

```typescript
// Service interface
export interface IMyService {
    readonly _serviceBrand: undefined;
    doSomething(): void;
}

// Service identifier
export const IMyService = createDecorator<IMyService>('myService');

// Service implementation
export class MyService implements IMyService {
    readonly _serviceBrand: undefined;
    
    constructor(
        @ILogService private readonly logService: ILogService,
        @IFileService private readonly fileService: IFileService
    ) {}
    
    doSomething(): void {
        this.logService.info('Doing something...');
    }
}

// Service registration
registerSingleton(IMyService, MyService, InstantiationType.Delayed);
```

### Service Consumption

```typescript
// Constructor injection
export class MyComponent {
    constructor(
        @IMyService private readonly myService: IMyService,
        @ILogService private readonly logService: ILogService
    ) {}
}

// Function injection
instantiationService.invokeFunction(accessor => {
    const myService = accessor.get(IMyService);
    const logService = accessor.get(ILogService);
    
    // Use services
});
```

## Common Patterns

### Disposable Pattern

```typescript
export class MyClass implements IDisposable {
    private readonly _disposables = new DisposableStore();
    
    constructor() {
        // Register disposables
        this._disposables.add(someService.onDidChange(this.handleChange, this));
        this._disposables.add(someOtherDisposable);
    }
    
    dispose(): void {
        this._disposables.dispose();
    }
}
```

### Async Operations

```typescript
export class MyService {
    private readonly _barrier = new Barrier();
    
    async initialize(): Promise<void> {
        // Perform initialization
        this._barrier.open();
    }
    
    async doSomething(): Promise<void> {
        await this._barrier.wait();
        // Service is now initialized
    }
}
```

### Configuration Watching

```typescript
export class MyService {
    constructor(
        @IConfigurationService private readonly configurationService: IConfigurationService
    ) {
        this.configurationService.onDidChangeConfiguration(e => {
            if (e.affectsConfiguration('myExtension')) {
                this.updateConfiguration();
            }
        });
    }
    
    private updateConfiguration(): void {
        const config = this.configurationService.getValue('myExtension');
        // Update service based on configuration
    }
}
```

This API reference covers the most commonly used services and patterns in VS Code development. For more detailed information, refer to the source code and inline documentation.